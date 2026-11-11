---
type: "scenario"
number: 4
domains:
  - "[[Domain-1-Agentic-Architecture]]"
  - "[[Domain-2-Tool-Design-and-MCP]]"
  - "[[Domain-3-Claude-Code-Configuration]]"
aliases:
  - "Scenario 4"
  - "Developer Productivity with Claude"
---
# Scenario 4: Developer Productivity with Claude

**Easy to confuse with:** [[Scenario-2-Code-Generation|Scenario 2: Code Generation with Claude Code]]. See [[Exam-Scenarios#Pairs that are easy to confuse|how to tell them apart]].

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 320}}}%%
flowchart LR
    E(["👩‍💻 Engineer<br/>'How does the refund flow work?'"])

    subgraph SDK["Developer productivity agent — Claude Agent SDK"]
        direction TB
        A["🤖 Agent<br/>Grep → Read → trace the flow"]
        BT["🧰 Built-in tools<br/>Grep · Glob · Read · Write · Edit · Bash"]
        MC["🔌 MCP servers<br/>Jira · GitHub · DB"]
        SA["🔍 Explore subagent<br/>returns a summary"]
        SC["📝 Scratchpad file<br/>key findings"]
        A <--> BT
        A <--> MC
        A <--> SA
        A <--> SC
    end

    CB[("📁 Legacy codebase")]
    EXT[("🏢 Team systems")]
    RESP(["📬 Back to engineer<br/>explanation · diff · test output"])

    E --> A
    BT <--> CB
    MC <--> EXT
    A -- final answer --> RESP
```

## Concepts in play
- [[Grep]]: find the entry point before reading files
- [[Glob]]: find files by name or extension
- [[Task-Decomposition|Decomposition]]: dynamic decomposition for a legacy codebase
- [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]]: built with the Agent SDK
- [[Claude-Technologies-Reference#Claude Code configuration|Explore subagent (reference)]]
