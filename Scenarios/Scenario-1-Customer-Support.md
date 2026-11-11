---
type: "scenario"
number: 1
domains:
  - "[[Domain-1-Agentic-Architecture]]"
  - "[[Domain-2-Tool-Design-and-MCP]]"
  - "[[Domain-5-Context-Management]]"
aliases:
  - "Scenario 1"
  - "Customer Support Resolution Agent"
---
# Scenario 1: Customer Support Resolution Agent

**Easy to confuse with:** [[Scenario-3-Multi-Agent-Research|Scenario 3: Multi-Agent Research System]]. See [[Exam-Scenarios#Pairs that are easy to confuse|how to tell them apart]].

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 320}}}%%
flowchart LR
    C(["👤 Customer<br/>'Refund order #123,<br/>and why was I charged twice?'"])

    subgraph APP["Support agent — Claude Agent SDK"]
        direction TB
        SPR["📋 System prompt<br/>escalation rules + examples"]
        L{{"🔁 Agentic loop<br/>tool_use → run tool · end_turn → reply"}}
        CF["📌 Case facts<br/>order #, amounts, dates"]
        SPR -.-> L
        CF -.-> L
    end

    subgraph HK["Hooks — enforced in code"]
        direction TB
        H1["⛔ Before tool call<br/>refund only after verified customer<br/>refund > $500 → escalate"]
        H2["🔄 After tool call<br/>normalize formats"]
    end

    subgraph MCP["MCP tools — clear descriptions"]
        direction TB
        T1["get_customer"]
        T2["lookup_order"]
        T3["process_refund"]
        T4["escalate_to_human"]
    end

    BE[("🏢 Backend<br/>CRM · orders · payments")]

    OK(["✅ Resolved on first contact<br/>target 80%+"])
    HU(["🙋 Human agent<br/>+ handoff summary"])

    C --> L
    L -- tool call --> H1
    H1 --> MCP
    T1 & T2 & T3 <--> BE
    MCP -. "result or structured error" .-> H2
    H2 -.-> L
    T4 --> HU
    L -- end_turn --> OK
```

## Concepts in play
- [[Agentic-Loop-Message-Cycle|Agentic Loop: The Message Cycle]]: the loop in the middle of the diagram
- [[Deterministic-Compliance|Deterministic Compliance]]: refund only after verification, enforced by a hook
- [[Sycophancy]]: "I already verified earlier!"
- [[Tool-Choice|Tool Choice]]: `auto` lets the agent reply in text or call a tool
- [[User-vs-Assistant-Messages|User vs Assistant Messages in Tool Calls]]: a `lookup_order` result goes back as a user message
- [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]]: built with the Agent SDK
- [[Claude-Technologies-Reference#Hooks|Hooks (reference)]]
- [[Claude-Technologies-Reference#MCP (Model Context Protocol)|MCP tools and structured errors (reference)]]
