---
type: "concept"
domain: "[[Domain-1-Agentic-Architecture]]"
tasks:
  - "1.6"
  - "1.2"
tags:
  - "agent-sdk"
aliases:
  - "Decomposition"
  - "Prompt chaining"
  - "Dynamic decomposition"
confident: false
---
# Decomposition (Task 1.6)

> When to use fixed sequential pipelines (prompt chaining) versus dynamic adaptive decomposition based on intermediate findings

To **decompose** a task is to break it into smaller subtasks, each handled by its own prompt or subagent. There are two ways to do it:
- **Prompt chaining (fixed):** the steps are known up front and run in order. It fits predictable work, e.g. review each file, then do one cross-file pass.
- **Dynamic (adaptive):** each next subtask depends on what was just found. It fits open-ended work, e.g. "add tests to a legacy codebase": map the structure, find the high-impact areas, then plan and adjust as you go.

> [!tip] Exam tip
> If every subagent did its job but the result still misses whole areas, the coordinator split the topic **too narrowly** ([[Domain-1-Agentic-Architecture#1.2 Orchestrate multi-agent systems with coordinator-subagent patterns|Task 1.2]]). Example: "AI in software development" split into only code completion, code review and test generation, so requirements, deployment and maintenance never get covered. Fix the decomposition, not the subagents.

^exam-tip

## Related
- [[Passing-Context-to-Subagents|Synthesis Subagent & Passing Context to Subagents]]
- [[Preserving-Attribution|Passing Information Between Agents & Preserving Attribution]]
