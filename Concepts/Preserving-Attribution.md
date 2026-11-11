---
type: "concept"
domain: "[[Domain-1-Agentic-Architecture]]"
tasks:
  - "1.3"
  - "5.6"
tags:
  - "agent-sdk"
aliases:
  - "Passing Information Between Agents & Preserving Attribution"
  - "Attribution"
confident: false
---
# Passing Information Between Agents & Preserving Attribution (Task 1.3)

> Using structured data formats to separate content from metadata (source URLs, document names, page numbers) when passing context between agents to preserve attribution

**Attribution** means keeping track of where each claim came from, so the final output can cite it.

**How to pass information:** The coordinator collects each agent's findings and puts them, in full, into the next agent's prompt, such as the synthesis agent's. The findings go in as **one structured payload** (e.g., JSON), where each record keeps the **content** (the claim) and its **metadata** (source URL, document name, page) as separate fields:

```json
[
  {"claim": "Revenue grew 12% in Q3", "source": "Q3-Report.pdf", "page": 4},
  {"claim": "Market expected to double by 2030", "source": "https://example.com/market-study", "page": null}
]
```

**Key points:**
- Send it **once, as structured data**. Don't send plain text plus a separate list of sources.
- Every claim **stays attached to its source** at every handoff, so the final report can cite exactly where each fact came from.
- If findings are merged into one block of plain text, the link between claim and source is lost and can't be rebuilt.

## Related
- [[Passing-Context-to-Subagents|Synthesis Subagent & Passing Context to Subagents]]
- [[Task-Decomposition|Decomposition]]
