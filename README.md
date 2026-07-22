# orkestre-skills

Public Claude Code skills, commands, and agents from Orkestre.

> **Generated file.** This repository is produced by `scripts/publish-public.py`
> in the private source repo from `public-allowlist.json`. Do not hand-edit —
> changes here will be overwritten on the next publish.

## Install

```
/plugin marketplace add grassriots/orkestre-skills
/plugin install orkestre@orkestre-skills
```

Commands are then available as `/orkestre:<command>` (e.g. `/orkestre:ask`).

## Skills

| Name | Description |
| --- | --- |
| `session-retrospective` | Post-session retrospective that diagnoses where Claude's model of reality diverged from actual reality — hallucinated APIs, stale knowledge, unfounded assumptions, tool misuse, plan drift — and proposes minimum-viable patches to the context that governs the next session (project instructions, plans, skills, commands, memory). Use when the user asks for a "session retrospective", "retro this session", "post-mortem", "why did this session go off the rails", "audit this handoff/transcript", or wants to turn a session's failures into fixes. Works in any Claude surface — Claude Code, Claude Desktop, claude.ai — on the live conversation, an attached/linked handoff doc, or an exported transcript. |

## Commands

| Name | Description |
| --- | --- |
| `ask` | Ask a read-only question about the codebase. Runs on your session model; delegates search to the Haiku codebase-explorer subagent. |
| `ask-deep` | Ask a read-only question with deeper reasoning (Opus) |
| `ask-simple` | Ask questions without making any changes (read-only mode) |

## Agents

| Name | Description |
| --- | --- |
| `codebase-explorer` | Explores the codebase to answer questions. Use when a question requires searching multiple files, finding symbol definitions, understanding how pieces connect, or mapping unfamiliar code. Read-only. |
| `deep-reasoner` | Analyzes complex questions requiring careful reasoning — architecture tradeoffs, subtle bug hypotheses, design critique, root cause analysis. Expects all necessary code context in the prompt; does minimal exploration. |

