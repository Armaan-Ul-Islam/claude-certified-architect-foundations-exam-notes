---
type: "concept"
domain: "[[Domain-1-Agentic-Architecture]]"
tasks:
  - "1.3"
tags:
  - "agent-sdk"
aliases:
  - "Synthesis Subagent & Passing Context to Subagents"
  - "Synthesis subagent"
confident: false
---
# Synthesis Subagent & Passing Context to Subagents (Task 1.3)

> Including complete findings from prior agents directly in the subagent's prompt (e.g., passing web search results and document analysis outputs to the synthesis subagent)

Here, **synthesis** means *combining several sources into one coherent result*. The **synthesis subagent** is the agent that takes everyone else's findings and writes the final answer or report.

The quote comes from the guide's multi-agent research system example:

```
Coordinator
 ├── Web search subagent        → finds articles, facts
 ├── Document analysis subagent → extracts info from PDFs/docs
 └── Synthesis subagent         → combines both into the final report
```

The synthesis subagent didn't do the searching or the document reading itself.


## What the guide's point is
Subagents **don't share memory** with the coordinator or with each other. Each one starts with a fresh context. So the synthesis subagent only knows what's written in its own prompt.

- ❌ **Wrong:** The prompt just says "Write a report on topic X based on the research done." The synthesis subagent never saw that research, so it guesses or hallucinates.
- ✅ **Right:** The coordinator puts the **complete findings** into the prompt:
  ```
  Write a report on topic X using these findings:

  ## Web search results
  - Source A (url): claim 1, claim 2 ...
  ## Document analysis
  - Report.pdf p.4: figure Y ...
  ```

**In short:** "synthesis" is just the merge-everything step. The exam point is that context must be **passed explicitly** to subagents, because they don't inherit it.

## Related
- [[Preserving-Attribution|Passing Information Between Agents & Preserving Attribution]]
- [[Task-Decomposition|Decomposition]]
