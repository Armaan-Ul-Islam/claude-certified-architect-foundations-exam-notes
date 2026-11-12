---
type: "concept"
domain: "[[Domain-4-Prompt-Engineering]]"
tasks:
  - "4.5"
tags:
  - "api"
aliases:
  - "Message Batches API"
  - "Message batches"
  - "Batch processing"
confident: false
---
# Message Batches API (Task 4.5)

> The Message Batches API: 50% cost savings, up to 24-hour processing window, no guaranteed latency SLA

A **message batch** is a set of normal Messages API requests, sent in one call. The answers come back later, at half the price. There's no special "batch message".


## One batch item
```json
{
  "custom_id": "invoice-042",
  "params": { "model": "…", "max_tokens": 1024, "tools": […], "tool_choice": {…}, "messages": […] }
}
```
`params` is exactly what you'd send to `messages.create` ([[Claude-API-Anthropic-Client-SDK#Making Your First Request|example]]). `custom_id` is your own label.


## How it runs
1. **Submit:** send all the items in one call (`POST /v1/messages/batches`). You get back a batch ID.
2. **Poll:** check the status until `processing_status` is `"ended"`.
3. **Read the results:** one per item, tagged with its `custom_id` and a type:
   - `succeeded`: holds the normal assistant message.
   - `errored`, `canceled` or `expired`: no answer for that item.


## Key points
- **When to use it:** work nobody waits on, like overnight reports, weekly audits and nightly test generation. Not a blocking check like a pre-merge review.
- **`custom_id`:** results come back in any order. Match them to requests by `custom_id`.
- **No multi-turn tool calling:** each item is one request and one response.
  - If Claude calls a tool, the item stops at `stop_reason: "tool_use"`, and nothing runs the tool.
  - Fine for extraction with a forced `tool_choice`, because the tool call is the answer.
  - Useless for agent work like "grep, run the tests, then fix".
- **Failures:** resubmit only the failed items, found by `custom_id`, after fixing the cause (e.g. splitting a document that was too big).
- **Test first:** refine the prompt on a small sample before sending a large batch.
- **SLA math:** if you submit every 4 hours, a document waits at most 4 hours and processes for at most 24. That's 28 hours, inside a 30-hour SLA.

> [!tip] Exam tip
> "Switch both workflows to batches to save 50%" → batch only the one nobody waits on (Sample Question 11). "Results come back out of order" is not a reason to avoid batches, because `custom_id` handles it.

^exam-tip

## Related
- [[Tool-Choice|Tool Choice]]
- [[JSON-Schema-for-Structured-Extraction|JSON Schema for Structured Extraction]]
- [[Claude-Technologies-Reference#Message Batches API ★|Message Batches API (reference)]]
