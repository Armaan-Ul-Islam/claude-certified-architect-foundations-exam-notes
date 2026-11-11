---
type: "scenario"
number: 6
domains:
  - "[[Domain-4-Prompt-Engineering]]"
  - "[[Domain-5-Context-Management]]"
aliases:
  - "Scenario 6"
  - "Structured Data Extraction"
---
# Scenario 6: Structured Data Extraction

**Easy to confuse with:** [[Scenario-5-Claude-Code-in-CI|Scenario 5: Claude Code for Continuous Integration]]. See [[Exam-Scenarios#Pairs that are easy to confuse|how to tell them apart]].

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 320, "rankSpacing": 70}}}%%
flowchart LR
    DOC(["📄 Unstructured documents<br/>invoices · contracts · PDFs"])

    subgraph SYS["Extraction system"]
        direction LR
        SC["📐 JSON schema = tool input_schema<br/>────────────<br/>invoice_number: string<br/>due_date: string | null<br/>stated_total · calculated_total: number"]
        EX["🤖 Claude extracts<br/>forced tool_choice<br/>missing field → null"]
        VA["✅ Validate in code<br/>calculated 1100 ≠ stated 1000"]
        SC -. defines shape .-> EX
        EX -- JSON --> VA
        VA -. "❌ retry with error" .-> EX
    end

    DS[("🏢 Downstream systems<br/>DB · ERP · API")]
    HR["🙋 Human review"]

    DOC --> EX
    VA -- valid --> DS
    VA -- "low confidence" --> HR
```

## Concepts in play
- [[JSON-Schema-for-Structured-Extraction|JSON Schema for Structured Extraction]]: the schema fixes the shape, your code checks the values
- [[Tool-Choice|Tool Choice]]: `any` for unknown document types, forced for a fixed first step
- [[Message-Batches-API|Message Batches API]]: a forced `tool_choice` works inside a batch
- [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]]: works at the API level
