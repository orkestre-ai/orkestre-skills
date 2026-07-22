```

                         ██████╗ ██████╗ ██╗  ██╗███████╗███████╗████████╗██████╗ ███████╗
                        ██╔═══██╗██╔══██╗██║ ██╔╝██╔════╝██╔════╝╚══██╔══╝██╔══██╗██╔════╝
                        ██║   ██║██████╔╝█████╔╝ █████╗  ███████╗   ██║   ██████╔╝█████╗
                        ██║   ██║██╔══██╗██╔═██╗ ██╔══╝  ╚════██║   ██║   ██╔══██╗██╔══╝
                        ╚██████╔╝██║  ██║██║  ██╗███████╗███████║   ██║   ██║  ██║███████╗
                         ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚══════╝   ╚═╝   ╚═╝  ╚═╝╚══════╝

                                ┌────────────────────────────────────────────────┐
                                │  ░▒▓█   CLAUDE CODE PLUGIN MARKETPLACE   █▓▒░  │
                                │  ────────────────────────────────────────────  │
                                │  [*] SKILLS [*] AGENTS [*] COMMANDS [*] HOOKS  │
                                └────────────────────────────────────────────────┘

```

# Orkestre Skills

Public Claude Code skills, commands, and agents from Orkestre.

> **Generated file.** This repository is a curated public mirror produced by
> `scripts/publish-public.py` in the private source repo, from
> `public-allowlist.json`. Do not hand-edit — changes here are overwritten on
> the next publish.

---

## What's in `orkestre`

### Skills

| Skill | Category | Description |
|-------|----------|-------------|
| **session-retrospective** | development | Post-session retrospective: diagnose where Claude's model of reality diverged (7 divergence categories, severity-scored) and propose minimum-viable context patches (FACT/GUARDRAIL/WORKFLOW). Environment-adaptive — full repo verification in Claude Code, attachment-based in Claude Desktop/claude.ai (upload as a custom skill). |

### Slash Commands

| Command | Model | Description |
|---------|-------|-------------|
| `/orkestre:ask` | Haiku | Ask a read-only question about the codebase. Delegates to `codebase-explorer` for multi-file search and `deep-reasoner` for complex synthesis. |
| `/orkestre:ask-deep` | Opus | Same workflow as `/orkestre:ask` but escalates to Opus for deeper reasoning — architectural tradeoffs, subtle bugs, design critique. |
| `/orkestre:ask-simple` | default | Minimal read-only Q&A. No subagent delegation, no escalation — just read, search, and answer directly. |

### Subagents

| Agent | Model | Description |
|-------|-------|-------------|
| **codebase-explorer** | Haiku | Gathers relevant code and context across multiple files. Read-only, no edits. Returns findings with file paths, line numbers, and flagged patterns. |
| **deep-reasoner** | Opus | Analyzes complex questions requiring careful reasoning — architecture tradeoffs, subtle bug hypotheses, design critique, root cause analysis. Expects pre-gathered context in the prompt. |

The `/orkestre:ask` and `/orkestre:ask-deep` commands chain these two agents: explorer collects the code, then deep-reasoner synthesizes the answer.

---

## Installation

### Add the Marketplace

```
/plugin marketplace add grassriots/orkestre-skills
```

### Install the Plugin

```
/plugin install orkestre@orkestre-skills
```

Commands are then available as `/orkestre:<command>` (e.g. `/orkestre:ask`).

### Update to Latest Version

```
/plugin marketplace update
```

```
        ╔══════════════════════════════════════════════════════╗
        ║  g r a s s r i o t s  //  o r k e s t r e  2 0 2 5   ║
        ╚══════════════════════════════════════════════════════╝
```
