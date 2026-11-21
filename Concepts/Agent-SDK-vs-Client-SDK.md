---
type: "concept"
tags:
  - "api"
  - "agent-sdk"
aliases:
  - "Agent SDK vs Client SDK"
  - "Client SDK vs Agent SDK"
confident: false
---
# Agent SDK vs Client SDK (Appendix: Technologies)

> Claude Agent SDK — agent definitions, agentic loops, stop_reason handling, hooks (PostToolUse, tool call interception), subagent spawning via Task tool, allowedTools configuration

The **Client SDK** is the library for calling the Claude API, which is what the guide calls it. The **Agent SDK** is the engine behind Claude Code, packaged as a library.

| | Client SDK | Agent SDK |
|---|---|---|
| Package | `anthropic` (Python), `@anthropic-ai/sdk` (TS) | `claude-agent-sdk`, `@anthropic-ai/claude-agent-sdk` |
| Agentic loop | You write it: check `stop_reason`, run tools, send results | Runs it for you |
| Tools | You define and run every tool | Built in (Read, Write, Edit, Bash, Grep, Glob) + MCP |
| Typical features | `tool_choice`, JSON schemas, Message Batches API | Hooks, subagents via the Task tool, `allowedTools`, sessions |

**In short:** the Client SDK sends one request to Claude at a time; the Agent SDK is a ready-made agent you configure.

**Under the hood:** the Agent SDK makes the same Messages API calls (`system` + `tools` + full history in, one assistant message + `stop_reason` out); it just runs the loop for you. So [Agentic Loop: The Message Cycle](Agentic-Loop-Message-Cycle.md) applies to both.

**What the exam tests:** the API concepts (`stop_reason`, `tool_use` / `tool_result`, `tool_choice`, JSON schemas, Message Batches), not SDK code. Language and framework details are out of scope.

> [!tip] Exam tip
> Scenarios [[Scenario-1-Customer-Support|1]], [[Scenario-3-Multi-Agent-Research|3]] and [[Scenario-4-Developer-Productivity|4]] are built with the Agent SDK. [[Scenario-6-Structured-Extraction|Scenario 6]] (extraction) works at the API level, with `tool_choice` and schemas.

^exam-tip

## Related
- [[Agentic-Loop-Message-Cycle|Agentic Loop: The Message Cycle]]
- [[User-vs-Assistant-Messages|User vs Assistant Messages in Tool Calls]]
- [[Claude-Technologies-Reference#Client SDK|Client SDK (reference)]]
- [[Claude-API-Anthropic-Client-SDK|Claude API: Anthropic Client SDK]]: the Client SDK in Python code
- [[Claude-Technologies-Reference#Agent SDK|Agent SDK (reference)]]
