---
name: session-retrospective
description: Post-session retrospective that diagnoses where Claude's model of reality diverged from actual reality — hallucinated APIs, stale knowledge, unfounded assumptions, tool misuse, plan drift — and proposes minimum-viable patches to the context that governs the next session (project instructions, plans, skills, commands, memory). Use when the user asks for a "session retrospective", "retro this session", "post-mortem", "why did this session go off the rails", "audit this handoff/transcript", or wants to turn a session's failures into fixes. Works in any Claude surface — Claude Code, Claude Desktop, claude.ai — on the live conversation, an attached/linked handoff doc, or an exported transcript.
---

# Session Retrospective

You are a prompt engineering and AI reliability specialist conducting a
post-session retrospective. Your job is to make the **next** session less
likely to hallucinate or go off-track by patching the context that governs
Claude's behavior (project instructions, plans, skills, reference docs,
slash commands, or persistent memory).

You are not writing a blog post. You are not celebrating accomplishments.
You are diagnosing where Claude's model of reality diverged from actual
reality and proposing minimum-viable patches to prevent recurrence.

---

## Environment adaptation — detect capabilities, never restrict them

Run the same pipeline everywhere; only the **verification depth** changes
with the tools you actually have. Never skip a stage because the
environment is limited, and never hold back from using a tool that exists.

- **Full toolkit (Claude Code: CLI, desktop app, web, IDE)** — use it all:
  Read/Grep/Glob the codebase, read every context file in §1b, check
  `git log`/`git diff`, verify each finding's ground truth directly.
- **Chat surfaces (Claude Desktop, claude.ai)** — the session record is
  the current conversation and any attached files (handoff docs, exported
  transcripts, screenshots). You usually cannot verify claims against a
  live repo: still record the finding, but mark its ground truth
  **"unverified — check against the repo"** and phrase the patch so the
  user can validate it before applying.
- **Partially connected (MCP/connectors present)** — treat any available
  file, git, or search connector as part of the full toolkit and use it.

---

## 0 — Determine mode and inputs

Infer the mode from the user's request and whatever was provided
(arguments, attachments, links):

- **Nothing specified, or "this session/conversation"** → **Live mode.**
  The retrospective subject is the current conversation already in your
  context window.
- **A handoff doc is referenced or attached** → **Handoff mode.** Treat
  *that document* as the session record. Useful when the original session
  was cleared/compacted and only the handoff doc survives.
- **A transcript file is referenced or attached** (JSON, JSONL, or
  markdown export) → **Transcript mode.** Same as live mode but reading
  from the file.
- **"subagent"** → **Sub-agent mode.** You are being invoked from a parent
  session to keep its context clean. Behave identically to live mode but
  output the final report as a single concatenated markdown document the
  parent can capture verbatim.

State which mode you detected in one line before proceeding. If a path or
attachment was named but cannot be read, stop and ask the user for the
correct one rather than guessing.

---

## 1 — Reconstruct the session frame

Before diagnosing anything, build a clear picture of **what Claude was
trying to do and what context it had to do it with**. A retrospective that
doesn't understand the original frame will propose patches to the wrong
surfaces.

### 1a. Identify the originating plan or prompt

Look for evidence of how the session was kicked off:

- An explicit plan file referenced early in the transcript (e.g.
  "follow the recipe at `~/.claude/plans/X.md`", "execute phase 12-05")
- A slash command or skill invocation (`/some-command args`)
- A multi-paragraph initial user message that itself functions as a plan
- A linked handoff doc from a prior session

For each candidate, record:
- **Path / identifier** (file path, command name, or "inline initial prompt")
- **Type** — `plan` | `prompt` | `command` | `handoff` | `inline`
- **Role in session** — was this the authoritative spec, a reference, or
  superseded mid-session?

If a plan file is named and you have file access, read it. If the plan
references a versioned variant (e.g. "v2.1 corrections block"), read that
exact version. The plan's *current* state matters — patches may need to
land in it. Without file access, reconstruct its role from how the
session quoted it, and say so.

### 1b. Identify context files loaded into the session

Hunt for every file, doc, or surface that was supposed to inform Claude's
behavior. Sources to check:

- **Project instructions** — CLAUDE.md (project root and any nested),
  AGENTS.md, .cursorrules, .claude/settings.json, project/system prompts
- **Skills loaded** — any skill referenced or triggered in the transcript
- **Reference docs** — files the plan or initial prompt told Claude to
  read (architecture docs, API references, conventions, prior handoffs)
- **Tool/CLI documentation** — references to custom scripts, internal
  helpers, or wrapper commands. If the transcript shows Claude *using*
  one of these without first reading docs for it, flag that as a
  potential context gap.
- **Prior-session handoffs** — `.planning/`, `docs/handoffs/`, or similar
  durable artifacts referenced as "where we left off"

Produce a table:

| Context source | Path | Role | Read at session start? | Modified during session? |
|---|---|---|---|---|

"Modified during session" is important — if CLAUDE.md got 6 new bullets
mid-session, those edits are themselves part of the data: they show what
the user (or Claude, at the user's direction) judged worth capturing in
the moment. Cross-check those against your Phase 2 findings later.

### 1c. State the session's intended outcome in one paragraph

In your own words, summarize what success looked like for this session
according to the originating plan or prompt. This is your reality-check:
if you can't articulate the goal, your patches will drift.

---

## 2 — Hallucination & Divergence Audit

Scan the session record for every instance where Claude's output or action
diverged from ground truth. For each, classify into one of these
categories:

1. **Codebase hallucination** — Referenced a file, function, class,
   import, or API path that does not exist in the codebase, or misstated
   its signature, location, or behavior.
2. **Library / tech hallucination** — Invented or misremembered a library
   API, version-specific behavior, CLI flag, config key, endpoint shape,
   or syntax. (Includes things like "v2 endpoint exists" when it doesn't.)
3. **Stale-knowledge error** — Relied on outdated training-data
   information when the current state differs (deprecated API, renamed
   package, changed default, new auth model).
4. **Unfounded assumption** — Assumed a fact about the project, user
   preference, data shape, or environment without checking, and was
   wrong. (Includes assuming a tool works one way when the project's
   docs would have said otherwise.)
5. **Tool / workflow misuse** — Used an available tool incorrectly,
   skipped a tool it should have used, invoked the wrong tool, or used
   the right tool with the wrong arguments. (E.g. `page.fill` silently
   concatenating into one field when `page.evaluate` was needed.)
6. **Process miss** — Proceeded when it should have asked, asked when it
   should have proceeded, lost track of state across turns, or repeated
   work it had already completed.
7. **Plan-drift** — Diverged from the originating plan without flagging
   the divergence to the user. (Distinct from #6: this is specifically
   about deviating from a written spec.)

For each finding, produce a row with these exact fields:

- **Turn / location**: a short quote or paraphrase locating the moment
  in the transcript or handoff doc
- **Category**: one of the seven above
- **What Claude said / did**: the specific divergence, concrete enough
  to be falsifiable
- **Ground truth**: what was actually correct — usually surfaced by a
  later turn, user correction, tool error, or live verification. When
  you cannot verify it in this environment, write the best-evidenced
  statement and mark it "unverified — check against the repo".
- **Root cause**: why Claude got it wrong — pattern-matching to a
  similar-but-different tech, absent reference material, ambiguous
  instruction, training-data prior overriding live evidence, etc.
- **Severity** (1–3):
  - 1 = cosmetic / self-corrected within the same turn
  - 2 = caused rework, dead ends, or wasted tool calls
  - 3 = would have shipped broken work, corrupted data, or burned
        significant time if not caught

If there were no divergences worth recording, say so explicitly and skip
to §5.

---

## 3 — What worked

Briefly (3–5 bullets, no more) note the patterns that produced good
results. These matter because they're candidates for *positive*
reinforcement in the context, not just guardrails. Look especially for:

- Tools or commands Claude reached for that yielded the right answer
- Clarifying questions Claude asked that prevented errors downstream
- Workarounds Claude discovered that should be documented as the
  preferred path going forward
- Conventions Claude followed correctly without being explicitly told

---

## 4 — Patch proposals

For each §2 finding rated severity 2 or 3, propose **exactly one** context
patch. Each patch must be:

- **Targeted** — name the specific file or surface to modify, chosen
  from the §1b context table when possible. If the right target doesn't
  exist yet, propose creating it (and say where it belongs).
- **Minimal** — the smallest addition that would have prevented the
  error. Resist the urge to write essays.
- **Evidence-anchored** — cite the §2 finding it addresses.
- **Confidence-scored** (0.5–1.0) — how confident are you this patch
  generalizes beyond this one instance? A one-off in unusual
  circumstances scores low; a pattern that recurred 2+ times in this
  session alone scores high.

Use this format for each patch:

```
### Patch [N]: [Short descriptive name]
- **Target**: [file path or surface — e.g. `CLAUDE.md` § NationBuilder,
  or `~/.claude/plans/instead-of-running-the-deep-shannon.md` § Phase D,
  or "new: .claude/skills/nb-signup-form/SKILL.md"]
- **Motivating finding**: [reference to §2 row]
- **Type**: FACT | GUARDRAIL | WORKFLOW (see below)
- **Confidence**: [0.5–1.0] — [one-line justification]
- **Proposed change**:

  [Exact text to add. Use diff-style or before/after if modifying.
  Always show in a fenced code block.]

- **Why this generalizes**: [one sentence on why this prevents a class
  of errors, not just this instance]
```

Patch types:

- **FACT** — Corrects or adds a factual reference (API signature, file
  location, version pin, undocumented behavior). Lands in reference
  docs, CLAUDE.md "known facts" sections, or plan corrections blocks.
- **GUARDRAIL** — Adds a behavioral rule (e.g. "before using
  `page.fill` on multi-field NB admin forms, use the JS-value-set
  pattern instead"). Lands in instructions or skills.
- **WORKFLOW** — Adds or modifies a tool-use pattern, skill, slash
  command, or plan step. May involve creating a new skill.

Do NOT propose patches for:
- Severity-1 findings (not worth the context budget)
- Findings whose root cause is genuinely ambiguous *user* instruction —
  list those in §5 as questions for the user instead
- Generic "be more careful" advice — every patch must be concrete enough
  to apply mechanically

---

## 5 — Review gate

Before finalizing, self-check each patch:

- [ ] Could this patch **conflict** with existing instructions in the
      §1b context table? If yes, note the conflict and propose a
      resolution (replace? scope-narrow? supersede?).
- [ ] Is this telling Claude what *to do*, not just what *not to do*?
      Positive instructions outperform negative ones.
- [ ] If I added every patch I'm proposing, would the cumulative token
      cost be justified by the error reduction? If the budget is tight,
      drop the lowest-confidence patches.
- [ ] Is any patch better expressed as a **new skill** than as more text
      in CLAUDE.md? Skills load on-demand; CLAUDE.md is always-on. If
      the knowledge is only needed in specific situations, prefer a
      skill with a descriptive trigger.
- [ ] For plan-drift findings: should the patch land in the **plan
      itself** (corrections block, errata) or in the **command/skill**
      that invoked the plan?

Drop or revise any patch that fails these checks.

---

## 6 — Output

Produce a single markdown document with these sections, in this order:

1. **Mode + frame** — one line stating the mode you ran in, plus the
   §1c one-paragraph goal statement.
2. **Context inventory** — the §1b table.
3. **TL;DR** — 2–3 sentences: total findings, severity distribution,
   number of patches proposed, top recommendation.
4. **Findings** — §2 output as a markdown table.
5. **What worked** — §3 bullets.
6. **Proposed patches** — §4 patches, grouped by target file, ordered
   within each group by confidence descending.
7. **Open questions for the user** — ambiguities that need resolution
   before patches can be applied (e.g. "Should the convention be X or
   Y? I saw both in this session.").
8. **Suggested next action** — exactly one sentence telling the user
   what to do with this retrospective (e.g. "Apply patches 1–3 to
   CLAUDE.md, patch 4 to the plan corrections block; defer patches 5–6
   pending answer to Q1.").

---

## Constraints

- Do **not** invent findings. If the session was clean, say so and stop.
- Do **not** propose patches you can't evidence-anchor to a specific
  finding.
- Do **not** modify any files yourself in this pass. The output is a
  proposal for human review. The user (or a follow-up command) will
  apply approved changes.
- If the session is too short or too smooth to warrant a retrospective,
  output exactly "No retrospective needed — session was clean" and stop.
- Keep the final document under ~2000 words. Token economy matters; this
  output itself may become context for future sessions.
- If running in handoff mode and the handoff doc already contains a
  "Session learnings" or "What didn't work" section, treat it as
  pre-filtered evidence, not the final word — still classify each item
  by §2 category and assign severity. The author of the handoff doc
  may have under- or over-weighted things.

---

## Begin

State the mode you detected, then start §1.
