---
type: "concept"
domain: "[[Domain-2-Tool-Design-and-MCP]]"
tasks:
  - "2.5"
  - "3.3"
tags:
  - "claude-code"
  - "agent-sdk"
aliases:
  - "Glob patterns"
confident: false
---
# Glob (Task 2.5)

> Glob for file path pattern matching (finding files by name or extension patterns)

**Glob** finds files by their **path and name**. It returns a list of paths and doesn't open the files.

**Is it regex?** No. It's the simpler wildcard syntax you know from the shell (`ls *.txt`). A few symbols look alike but mean different things:

| Symbol | In a glob | In a regex |
|---|---|---|
| `*` | any characters | "repeat the previous character" |
| `.` | a literal dot | any one character |
| `?` | exactly one character | "the previous character is optional" |

**Wildcards:**

| Wildcard | Meaning | Example | Matches |
|---|---|---|---|
| `*` | any characters within one folder | `*.ts` | `index.ts`, `refund.ts` (current folder only) |
| `**` | any number of folders deep | `**/*.ts` | every `.ts` file, at any depth |
| `?` | exactly one character | `log?.txt` | `log1.txt`, `logA.txt`, but not `log10.txt` |
| `[0-9]` | one character from a set | `v[0-9].md` | `v1.md` … `v9.md` |
| `{a,b}` | either alternative | `*.{ts,tsx}` | `.ts` and `.tsx` files |

**Common patterns:**
- `**/*.test.tsx`: every React test file (the guide's example).
- `src/**/*`: everything under `src`.
- `**/CLAUDE.md`: every CLAUDE.md, at any level.
- `**/*.{yml,yaml}`: config files, whichever extension was used.

**Exam link:** the same syntax goes in the `paths:` of `.claude/rules` files, e.g. `paths: ["terraform/**/*"]` or `["**/*.test.*"]` ([[Domain-3-Claude-Code-Configuration#3.3 Apply path-specific rules for conditional convention loading|Task 3.3]]).

## Related
- [[Grep]]
- [[Claude-Technologies-Reference#Claude Code configuration|Claude Code configuration (reference)]]
