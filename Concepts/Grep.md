---
type: "concept"
domain: "[[Domain-2-Tool-Design-and-MCP]]"
tasks:
  - "2.5"
  - "5.4"
tags:
  - "claude-code"
  - "agent-sdk"
aliases:
  - "Global Regular Expression Print"
confident: false
---
# Grep (Task 2.5)

> Grep for content search (searching file contents for patterns like function names, error messages, or import statements). Full form of grep is Global Regular Expression Print.

**Grep** searches the text inside files and finds the lines that match a pattern. The pattern is a **regex** (regular expression). The name comes from the Unix `grep` command. By default it returns the list of matching files. It can also show the matching lines, or count them.

| Pattern | Finds |
|---|---|
| `processRefund` | every mention of the name |
| `processRefund\(` | only calls to it (`(` is special in regex, so it's escaped with `\`) |
| `function \w+Refund` | any function whose name ends in `Refund` |
| `import .*payments` | files that import from payments (`.*` = anything in between) |
| `^export` | lines that start with `export` (`^` = start of line) |
| `Card declined` | where an error message comes from |

> [!tip] Exam tip
> To understand unfamiliar code, start with Grep to find the entry point, then Read the files along the imports. Don't read everything up front. To trace a function through wrapper modules, first find all the names it's exported under, then Grep for each one.

^exam-tip

## Related
- [[Glob]]
