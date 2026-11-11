---
type: "concept"
domain: "[[Domain-1-Agentic-Architecture]]"
tasks:
  - "1.1"
tags:
  - "api"
  - "agent-sdk"
aliases:
  - "User vs Assistant Messages in Tool Calls"
  - "Tool results are user messages"
confident: false
---
# User vs Assistant Messages in Tool Calls (Task 1.1)

> How tool results are appended to conversation history so the model can reason about the next action

A conversation has only two roles. **Assistant** is whatever Claude says. **User** is everything sent back to Claude, whether a person typed it or your code produced it.
1. Claude asks for a tool: an **assistant** message with a `tool_use` block.
2. Your code (or the Agent SDK) runs the tool. The result goes back in a **user** message as a `tool_result` block.

```json
[
  {
    "role": "assistant",
    "content": [
      {
        "type": "tool_use",
        "id": "toolu_01",
        "name": "lookup_order",
        "input": {
          "order_id": "123"
        }
      }
    ]
  },
  {
    "role": "user",
    "content": [
      {
        "type": "tool_result",
        "tool_use_id": "toolu_01",
        "content": "status: shipped"
      }
    ]
  }
]
```
- `tool_use_id` matches the `id`, so Claude knows which call the result answers.
- In the Agent SDK you don't build these yourself. The message stream still shows the same pair: an `AssistantMessage` with a `ToolUseBlock`, then a `UserMessage` with a `ToolResultBlock`.

**In short:** a tool result is a user message, even though no person wrote it.

## Related
- [[Agentic-Loop-Message-Cycle|Agentic Loop: The Message Cycle]]
- [[Agentic-Loop-Anti-Patterns|Agentic Loop Anti-Patterns]]
- [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]]
