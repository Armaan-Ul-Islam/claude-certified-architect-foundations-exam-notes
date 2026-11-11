---
type: "scenario"
number: 3
domains:
  - "[[Domain-1-Agentic-Architecture]]"
  - "[[Domain-2-Tool-Design-and-MCP]]"
  - "[[Domain-5-Context-Management]]"
aliases:
  - "Scenario 3"
  - "Multi-Agent Research System"
---
# Scenario 3: Multi-Agent Research System

**Easy to confuse with:** [[Scenario-1-Customer-Support|Scenario 1: Customer Support Resolution Agent]]. See [[Exam-Scenarios#Pairs that are easy to confuse|how to tell them apart]].

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 320}}}%%
flowchart LR
    U(["🧑 User<br/>'Impact of AI on software development'"])

    subgraph SDK["Claude Agent SDK — hub and spoke"]
        direction LR
        CO["🧭 Coordinator<br/>splits topic · delegates in parallel"]

        subgraph SUB["Subagents · each has its own context"]
            direction TB
            W["🔎 Web search agent"]
            DA["📑 Document analysis agent"]
            SY["🧩 Synthesis agent<br/>keeps sources · flags conflicts"]
            RG["📝 Report agent"]
        end

        CO <-->|"task ⇄ findings + sources"| W
        CO <-->|"task ⇄ findings + sources"| DA
        CO <-->|"findings ⇄ gaps to fill"| SY
        CO -- synthesis --> RG
    end

    WEB[("🌐 Web")]
    DOCS[("📂 Documents")]
    OUT(["📄 Cited report"])

    U --> CO
    W <--> WEB
    DA <--> DOCS
    RG --> OUT
```

## Concepts in play
- [[Passing-Context-to-Subagents|Synthesis Subagent & Passing Context to Subagents]]: the synthesis subagent only knows what is in its prompt
- [[Preserving-Attribution|Passing Information Between Agents & Preserving Attribution]]: every claim keeps its source between agents
- [[Task-Decomposition|Decomposition]]: splitting "AI in software development" too narrowly
- [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]]: built with the Agent SDK
- [[Claude-Technologies-Reference#All together Scenario 3 as configuration|This scenario as Agent SDK configuration (reference)]]
