---
type: "concept"
domain: "[[Domain-4-Prompt-Engineering]]"
tasks:
  - "4.3"
  - "4.4"
  - "5.5"
tags:
  - "api"
aliases:
  - "JSON Schema for Structured Extraction"
  - "Structured extraction"
  - "Schema vs validation"
confident: false
---
# JSON Schema for Structured Extraction (Task 4.3)

> That strict JSON schemas via tool use eliminate syntax errors but do not prevent semantic errors (e.g., line items that don't sum to total, values in wrong fields)

There's **one JSON schema**. It defines the *shape* of the data and is passed to Claude as an extraction tool's `input_schema`. The validation step afterwards is **not a second schema**. It's your own code checking whether the values make sense.


## 1. The schema: written once, as a tool definition
```json
{
  "name": "extract_invoice",
  "input_schema": {
    "type": "object",
    "properties": {
      "invoice_number": {
        "type": "string"
      },
      "due_date": {
        "type": [
          "string",
          "null"
        ]
      },
      "category": {
        "type": "string",
        "enum": [
          "software",
          "hardware",
          "other"
        ]
      },
      "line_items": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "description": {
              "type": "string"
            },
            "amount": {
              "type": "number"
            }
          }
        }
      },
      "stated_total": {
        "type": "number"
      }
    },
    "required": [
      "invoice_number",
      "line_items",
      "stated_total"
    ]
  }
}
```
Set `tool_choice: {"type": "tool", "name": "extract_invoice"}` so Claude must answer by calling this tool.
- `due_date` can be `null`: if the document has no due date, Claude returns `null` instead of inventing one.
- `category` has `"other"` for cases that don't fit the list.


## 2. Claude's output: JSON that fits the shape
```json
{
  "invoice_number": "INV-1042",
  "due_date": null,
  "category": "software",
  "line_items": [
    {
      "description": "License",
      "amount": 900
    },
    {
      "description": "Support",
      "amount": 200
    }
  ],
  "stated_total": 1000
}
```
The structure is guaranteed: no broken JSON, no wrong types.


## 3. Your validation code: does it make sense?
900 + 200 = 1100, but `stated_total` is 1000. The schema can't catch that. Your code spots it and retries, sending Claude the document, its previous answer and the specific error: "line items sum to 1100 but stated_total is 1000".

**In short:** the schema guarantees the **shape**; your validation code catches **wrong values**.

> [!tip] Exam tip
> Retrying fixes format and structure mistakes, but not information that simply isn't in the document. That field should be `null`, or the extraction goes to human review.

^exam-tip

## Related
- [[Tool-Choice|Tool Choice]]
- [[Message-Batches-API|Message Batches API]]
