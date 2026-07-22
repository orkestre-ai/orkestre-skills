---
description: Ask a read-only question with deeper reasoning (Opus)
allowed-tools: Read, Grep, Glob, LS, WebFetch, WebSearch, Task
argument-hint: [your question]
model: opus
---

Answer the following question using read-only investigation.

For anything requiring codebase search or multi-file understanding, delegate
to the `codebase-explorer` subagent via the Task tool. Do this BEFORE any
deeper reasoning step, so the exploration results are available to pass
forward.

If the question requires deep reasoning — architectural tradeoffs, subtle
bug hypotheses, design critique, anything where a quick answer would be
thin — delegate the synthesis to the `deep-reasoner` subagent via Task.
When you do, pass it ALL relevant findings from the explorer in the prompt
string: file paths, code snippets, and the original question. Deep-reasoner
has a fresh context and sees only what you send it.

For simpler questions, answer directly without escalating.

You may propose changes as part of your answer, but do not apply them.

Question: $ARGUMENTS
