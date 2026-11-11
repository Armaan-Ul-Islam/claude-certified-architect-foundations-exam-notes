---
type: "scenario"
number: 5
domains:
  - "[[Domain-3-Claude-Code-Configuration]]"
  - "[[Domain-4-Prompt-Engineering]]"
aliases:
  - "Scenario 5"
  - "Claude Code for Continuous Integration"
---
# Scenario 5: Claude Code for Continuous Integration

**Easy to confuse with:** [[Scenario-2-Code-Generation|Scenario 2: Code Generation with Claude Code]], [[Scenario-6-Structured-Extraction|Scenario 6: Structured Data Extraction]]. See [[Exam-Scenarios#Pairs that are easy to confuse|how to tell them apart]].

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 320}}}%%
flowchart LR
    DEV(["👩‍💻 Developer<br/>opens / updates a PR"])

    CTX["📄 Context<br/>CLAUDE.md · explicit criteria · examples"]

    subgraph CI["CI pipeline — blocking pre-merge check"]
        CC["🤖 claude -p<br/>JSON output · independent reviewer"]
    end

    subgraph JOBS["Multi-pass review + tests"]
        direction TB
        PF["🔍 Per-file passes"]
        IN["🔗 Cross-file pass"]
        TG["🧪 Generate tests"]
        PF --> IN
    end

    OUT(["📬 Back on the PR<br/>inline comments · new tests"])

    DEV -- "PR event" --> CC
    CTX -.-> CC
    CC --> PF
    CC --> TG
    IN -- findings --> OUT
    TG --> OUT
```

## Concepts in play
- [[Claude-Code-Headless-Mode|Claude Code Headless Mode]]: `claude -p` runs in the pipeline
- [[Task-Decomposition|Decomposition]]: per-file passes, then one cross-file pass
- [[Message-Batches-API|Message Batches API]]: batch the nightly work, not the blocking pre-merge check
- [[Claude-Technologies-Reference#Claude Code CLI|Claude Code CLI flags (reference)]]
