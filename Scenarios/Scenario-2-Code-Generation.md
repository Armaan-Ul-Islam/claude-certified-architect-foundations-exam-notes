---
type: "scenario"
number: 2
domains:
  - "[[Domain-3-Claude-Code-Configuration]]"
  - "[[Domain-5-Context-Management]]"
aliases:
  - "Scenario 2"
  - "Code Generation with Claude Code"
---
# Scenario 2: Code Generation with Claude Code

**Easy to confuse with:** [[Scenario-4-Developer-Productivity|Scenario 4: Developer Productivity with Claude]], [[Scenario-5-Claude-Code-in-CI|Scenario 5: Claude Code for Continuous Integration]]. See [[Exam-Scenarios#Pairs that are easy to confuse|how to tell them apart]].

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 320}}}%%
flowchart LR
    D(["👩‍💻 Developer<br/>generate · refactor · debug · document"])

    subgraph CC["Claude Code"]
        direction TB
        subgraph CFG["Context"]
            direction TB
            M1["📄 Always loaded<br/>CLAUDE.md: user · project · folder"]
            M2["🎯 Matching files only<br/>.claude/rules with paths glob"]
            M3["⚡ On demand<br/>/commands · skills"]
            M1 ~~~ M2 ~~~ M3
        end
        P{"Task?"}
        PM["🗺️ Plan mode<br/>explore → plan → approve"]
        DX["⚡ Direct execution"]
        CFG -.-> P
        P -- "large · architectural" --> PM
        P -- "small · clear" --> DX
        PM -- approved --> DX
    end

    R[("📁 Codebase")]
    V(["👀 Developer reviews<br/>refines with examples · failing tests"])

    D -- "prompt or /command" --> P
    DX --> R
    R -- diff --> V
```

## Concepts in play
- [[Glob]]: `paths:` globs in `.claude/rules`
- [[Claude-Technologies-Reference#Claude Code configuration|CLAUDE.md, rules, commands, skills and plan mode (reference)]]
