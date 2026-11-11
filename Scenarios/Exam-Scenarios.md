---
type: "hub"
aliases:
  - "Telling the Exam Scenarios Apart"
  - "Exam scenarios"
---
# Telling the Exam Scenarios Apart (Section 5: Exam Scenarios)

> During the exam, 4 scenarios are presented and picked at random from the full set of the 6 scenarios below.

The exam states each scenario up front, and it frames a set of questions, so you don't have to guess which one you're in. The useful skill is the reverse: recognizing what each scenario is likely to test.

| # | Scenario | Giveaway words | Who acts |
|---|---|---|---|
| [[Scenario-1-Customer-Support\|1]] | Customer support | customer, refund, order, billing, escalate, `get_customer` / `process_refund`, 80% first-contact resolution | **One agent** talking to a customer through MCP tools |
| [[Scenario-2-Code-Generation\|2]] | Code generation with Claude Code | CLAUDE.md, `/command`, `.claude/rules`, skills, plan mode, "your team" | **A developer** using Claude Code interactively |
| [[Scenario-3-Multi-Agent-Research\|3]] | Multi-agent research | coordinator, subagents, web search, synthesis, citations, report, Task tool | **A coordinator** directing several subagents |
| [[Scenario-4-Developer-Productivity\|4]] | Developer productivity | legacy or unfamiliar codebase, Grep / Glob / Read, explore, boilerplate, MCP servers, long sessions | **An agent you build** with the Agent SDK for engineers |
| [[Scenario-5-Claude-Code-in-CI\|5]] | Claude Code in CI | pipeline, PR, `claude -p`, automated review, false positives, inline comments | **No human at the keyboard**: Claude Code runs in a pipeline |
| [[Scenario-6-Structured-Extraction\|6]] | Structured extraction | documents, JSON schema, `tool_choice`, null fields, validation, retry, confidence, human review | **A pipeline** turning documents into validated JSON |


## Pairs that are easy to confuse
- **[[Scenario-2-Code-Generation|2]] vs [[Scenario-4-Developer-Productivity|4]]:** both involve code. In 2 a person uses Claude Code, so the answers are about config files and plan mode. In 4 you build an agent with the SDK, so the answers are about tool choice, MCP, and context during long exploration.
- **[[Scenario-2-Code-Generation|2]] vs [[Scenario-5-Claude-Code-in-CI|5]]:** both are Claude Code. In 5 nobody is at the keyboard, so look for `-p`, pipelines and PRs.
- **[[Scenario-1-Customer-Support|1]] vs [[Scenario-3-Multi-Agent-Research|3]]:** in 1 a single agent serves a customer. In 3 a coordinator directs subagents, and everything goes through the coordinator.
- **[[Scenario-5-Claude-Code-in-CI|5]] vs [[Scenario-6-Structured-Extraction|6]]:** both produce structured JSON. In 5 it's findings from reviewing code. In 6 it's fields pulled out of documents.

## Primary domains of each scenario
From the exam guide. Each scenario note lists the concepts it leans on.

![[Study-Tracker.base#Scenarios]]
