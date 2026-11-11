---
type: "concept"
domain: "[[Domain-1-Agentic-Architecture]]"
tasks:
  - "1.1"
tags:
  - "api"
aliases:
  - "Agentic Loop Anti-Patterns"
  - "When to stop the agentic loop"
confident: false
---
# Agentic Loop Anti-Patterns (Task 1.1)

> Avoiding anti-patterns such as parsing natural language signals to determine loop termination, setting arbitrary iteration caps as the primary stopping mechanism, or checking for assistant text content as a completion indicator

All three mistakes come from the same rule: **the agent loop should keep going while `stop_reason == "tool_use"` and stop when it is `"end_turn"`.** Claude sets `stop_reason` on every response, so it is the reliable signal that the work is finished. Each anti-pattern below swaps it for a guess.


## 1. Parsing natural language to decide when to stop
```python
if "task complete" in response_text.lower() or "I'm done" in response_text:
    break
```
**Why it's wrong:** The wording changes from response to response. Claude might say "All set!" and the loop keeps going. Or it might say "Once the task is complete, I'll…" partway through and the loop stops early.


## 2. Using an arbitrary iteration cap as the main way to stop
```python
for _ in range(5):
    response = call_claude(messages)
    ...
```
**Why it's wrong:** A complex task needing 8 tool calls gets cut off at 5 and leaves partial results. A simple task gets no benefit from the cap. A cap is fine as a **safety net** against runaway loops. It shouldn't be how the loop normally ends.


## 3. Treating "the response has text" as "the task is done"
```python
if any(block.type == "text" for block in response.content):
    break
```
**Why it's wrong:** One response can hold text and a tool call together, such as "Let me check the order status…" followed by a `tool_use` block. Stopping on the text means that tool call never runs.


## The correct pattern
```python
while True:
    response = client.messages.create(...)
    messages.append({"role": "assistant", "content": response.content})

    if response.stop_reason == "end_turn":
        break
    if response.stop_reason == "tool_use":
        tool_results = run_tools(response.content)
        messages.append({"role": "user", "content": tool_results})
```

> [!tip] Exam tip
> If an answer choice decides when to stop by reading the text or counting turns, it's probably the distractor. The right answer checks `stop_reason`.

^exam-tip

## Related
- [[Agentic-Loop-Message-Cycle|Agentic Loop: The Message Cycle]]
- [[User-vs-Assistant-Messages|User vs Assistant Messages in Tool Calls]]
