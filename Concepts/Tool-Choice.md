---
type: "concept"
domain: "[[Domain-2-Tool-Design-and-MCP]]"
tasks:
  - "2.3"
  - "4.3"
tags:
  - "api"
aliases:
  - "Tool Choice"
  - "tool_choice"
confident: false
---
# Tool Choice: Must Claude Call a Tool? (Tasks 2.3, 4.3)

> The distinction between tool_choice: "auto" (model may return text instead of calling a tool), "any" (model must call a tool but can choose which), and forced tool selection (model must call a specific named tool)

`tool_choice` is a request field that says whether Claude must call one of the listed tools. The examples use an extraction request whose tools include `extract_metadata`, `extract_invoice` and `extract_receipt`.

| `tool_choice` | Meaning | What can come back |
|---|---|---|
| `{"type": "auto"}` (default) | Claude decides | Text only, or tool call(s) |
| `{"type": "any"}` | Must call **a** tool; Claude picks which | A tool call |
| `{"type": "tool", "name": "extract_metadata"}` | Must call **this** tool | Exactly that tool call |
| `{"type": "none"}` | Must not call a tool | Text only |


## `auto`: Claude decides
Right for a chat agent: "What are your opening hours?" gets text, and "Where's order #123?" gets a `lookup_order` call. Risky for extraction: Claude may reply in text ("This looks like an invoice from ACME…") and you get no JSON.


## `any`: a tool call is guaranteed, Claude picks which
Use it when the document type is unknown ([[Scenario-6-Structured-Extraction|Scenario 6]]):
```json
"tool_choice": { "type": "any" }
```
```json
{
  "content": [
    {
      "type": "tool_use",
      "name": "extract_receipt",
      "input": { "store": "ACME", "total": 42.50, "date": "2026-11-02" }
    }
  ],
  "stop_reason": "tool_use"
}
```
Claude chose `extract_receipt` because the document was a receipt.


## Forced: you pick the tool and the order
Use it when one step must always run first:
```json
"tool_choice": { "type": "tool", "name": "extract_metadata" }
```
```json
{
  "content": [
    {
      "type": "tool_use",
      "name": "extract_metadata",
      "input": { "document_type": "invoice", "language": "en", "pages": 3 }
    }
  ],
  "stop_reason": "tool_use"
}
```
The next request switches back to `auto` or `any` for the enrichment steps.


## `none`: text only
The tools stay listed, so the prompt cache still works, but Claude can't call them. Use it for a final turn like "Summarize what you found for the customer."

**Extras:**
- `disable_parallel_tool_use: true` can be added to any `tool_choice` to allow at most one tool call per response.
- With `any` or a forced tool, the response starts straight with the `tool_use` block; Claude writes no text before it.

> [!tip] Exam tip
> "Claude sometimes answers in text instead of calling the extraction tool" → `any`. "This step must always run first" → forced tool. "Claude keeps picking the wrong tool" → better descriptions and fewer tools per agent (Tasks [[Domain-2-Tool-Design-and-MCP#2.1 Design effective tool interfaces with clear descriptions and boundaries|2.1]], [[Domain-2-Tool-Design-and-MCP#2.3 Distribute tools appropriately across agents and configure tool choice|2.3]]), not `tool_choice`.

^exam-tip

**Exam vs real code:** learn all four, as the guide lists them. The newest models (Opus 5.5, Sonnet 5.5, Fable 5.1) reject `any` and forced selection with an error. There you use `auto`, tell Claude in the prompt which tool to call, and set `strict: true` on the tool so its input always matches the schema.

## Related
- [[JSON-Schema-for-Structured-Extraction|JSON Schema for Structured Extraction]]
- [[Message-Batches-API|Message Batches API]]
- [[Claude-Technologies-Reference#Request fields|Request fields (reference)]]
