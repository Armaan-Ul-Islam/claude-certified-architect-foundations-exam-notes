---
type: "reference"
tags:
  - "api"
  - "agent-sdk"
  - "mcp"
  - "claude-code"
aliases:
  - "Technologies Reference"
---
# Claude Technologies Reference

Names, fields and calls from the Claude API, Client SDK, Agent SDK, MCP and Claude Code that the exam can touch.

★ = named in the exam guide. Unmarked = not named, but connected to an exam topic.

**Out of scope, so skip:** streaming / server-sent events, vision, computer use, token counting, pricing and rate limits, authentication, and prompt caching details (just know caching exists).

<br>

## How the layers fit
---

Diagrams of how the Claude API, Client SDK, Agent SDK, Claude Code and MCP servers connect, and of one tool call end to end: [[Agentic-Loop-Message-Cycle|Agentic Loop: The Message Cycle]].

<br>

## Claude API: Messages API
---

### Request fields
| Field | Meaning |
|---|---|
| `model` | Which Claude model |
| ★ `max_tokens` | Cap on the reply's length. Hitting it gives `stop_reason: "max_tokens"` (reply cut off). Not a length hint to Claude (see [[Agentic-Loop-Message-Cycle#^max-tokens-cutoff\|why]]) |
| ★ `system` | The system prompt. A top-level field, not a message |
| `messages` | The full history, alternating `user` / `assistant`, resent on every call |
| ★ `tools` | Every tool Claude may call: `name`, `description`, `input_schema`, optional `strict: true` |
| ★ `tool_choice` | `auto` / `any` / `tool` (forced) / `none`, optional `disable_parallel_tool_use` (see [[Tool-Choice\|Tool Choice]]) |
| `output_config.format` | Newer way to get schema-matching JSON without a tool. The exam's answer is still tool use + JSON schema |

### Response fields
| Field | Meaning |
|---|---|
| `role` | Always `assistant` |
| `content` | The new turn's blocks (see below) |
| ★ `stop_reason` | `tool_use` → run the tools and call again. `end_turn` → done. `max_tokens` → cut off. (Also `pause_turn` for long server-tool runs, `refusal`) |
| `usage` | Input and output token counts |

### Content blocks
| Block | Sent by | Meaning |
|---|---|---|
| `text` | either side | Plain text |
| ★ `tool_use` | Claude | A tool request: `id`, `name`, `input` |
| `tool_result` | you, in a `user` message | `tool_use_id`, `content`, optional `is_error: true` |
| `document` | you | A PDF or text file to read, e.g. for extraction |
| `server_tool_use` + `web_search_tool_result` | Anthropic's servers | A server tool call and its result, both inside one response |

### Server tools
`web_search`, `web_fetch`, `code_execution`: you only list them in `tools`. Anthropic's servers run them and put the result in the same response, so your code sends no `tool_result`.

### Prompt caching
`cache_control` marks a prompt prefix (tools → system → messages) for reuse, so resending the same history costs less. For the exam, just know it exists.

<br>

## Message Batches API ★
---

| Fact | Detail |
|---|---|
| Cost | 50% cheaper than normal calls |
| Timing | Up to 24 hours, no guaranteed latency |
| ★ `custom_id` | Your ID on each request. Results come back in any order, so match them by `custom_id` |
| Polling | Check the batch's status until it's `ended` |
| Result types | `succeeded`, `errored`, `canceled`, `expired`. Resubmit only the failed ones, e.g. split an oversized document |
| Limit | No multi-turn tool calling: each item is one request and one response |

**Use for:** overnight reports, weekly audits, nightly test generation. **Not for:** blocking checks someone waits on, like pre-merge review.

See also: [[Message-Batches-API|Message Batches API]] for how a batch runs and the exam tips.

<br>

## Client SDK
---

Packages: `anthropic` (Python), `@anthropic-ai/sdk` (TypeScript). The TypeScript calls use the same paths.

**Client SDK vs Claude API:** the API is the service; the SDK is a library that calls it for you. It builds the JSON, adds your key, retries on errors and gives you typed objects. It adds no capabilities, so the same request also works with plain HTTPS:

```
POST https://api.anthropic.com/v1/messages
{ "model": "claude-opus-5-5", "max_tokens": 1024, "messages": [ … ] }
```
```python
client.messages.create(model="claude-opus-5-5", max_tokens=1024, messages=[…])
```

| Call | What it does |
|---|---|
| `client.messages.create(model=…, max_tokens=…, system=…, tools=[…], tool_choice=…, messages=[…])` | One request → one assistant message. Your code runs the loop |
| `client.messages.batches.create(requests=[{"custom_id": "doc-1", "params": {…}}])` | Submits a batch |
| `client.messages.batches.retrieve(batch_id)` | Polls it: `.processing_status == "ended"` when done |
| `client.messages.batches.results(batch_id)` | Reads the results. Match each one by `custom_id` |

See also: [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]], and [[Claude-API-Anthropic-Client-SDK|Claude API: Anthropic Client SDK]] for a first request and reading the reply.

<br>

## Agent SDK
---

Packages: `claude-agent-sdk` (Python), `@anthropic-ai/claude-agent-sdk` (TypeScript). Python spells options in snake_case (`allowed_tools`), TypeScript in camelCase (`allowedTools`). The guide mixes both, so treat them as the same thing.

### Starting the agent
| Python | TypeScript | What it is |
|---|---|---|
| `query()` | `query()` | Runs the agent on a prompt and streams back each message as the loop runs |
| `ClaudeAgentOptions` | `options` | Holds all the settings below |

### Settings
| Python | TypeScript | What it does |
|---|---|---|
| `allowed_tools` | ★ `allowedTools` | Tools this agent may use. A coordinator needs `"Task"` here |
| `disallowed_tools` | `disallowedTools` | Tools to hide or block |
| `system_prompt` | `systemPrompt` | The system prompt |
| `agents` | `agents` | The subagents: each name maps to an `AgentDefinition` |
| `mcp_servers` | `mcpServers` | MCP servers to connect to |
| `hooks` | `hooks` | Hook event name → matchers and your functions |
| `permission_mode` | `permissionMode` | `default`, `acceptEdits`, `plan`, `dontAsk`, `bypassPermissions` |
| `max_turns` | `maxTurns` | Turn cap. A safety net, not the normal way to stop |
| `resume` + ★ `fork_session` | `resume` + `forkSession` | Continue an earlier session, or branch it into a new one |
| `setting_sources` | `settingSources` | Whether CLAUDE.md and `.claude/settings.json` are loaded |

**Precisely:** `allowedTools` pre-approves tools so they run without asking. `disallowedTools` is what hides a tool from Claude completely.

### ★ `AgentDefinition`: one subagent
| Field | Meaning |
|---|---|
| `description` | When the coordinator should use it. Claude reads it like a tool description |
| `prompt` | The subagent's system prompt |
| `tools` | Its own subset of tools. Leave it out to inherit all |
| `model` | Optional different model, e.g. a cheaper one for simple work |

### Tool names
- **Built-in:** `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, ★ `Task` (starts a subagent; called `Agent` in current versions).
- **MCP tools:** `mcp__<server>__<tool>`, e.g. `mcp__jira__get_issue`.

See also: [[Grep]] and [[Glob]].

### Hooks
| Name | When it runs |
|---|---|
| `PreToolUse` | Before a tool runs: let it run, block it, or change its inputs |
| ★ `PostToolUse` | After it runs, before Claude sees the result: add to or replace the result |
| `Stop` / `SubagentStop` | When the agent or a subagent finishes |
| `HookMatcher` | Chooses which tools a hook applies to |

What a hook returns, inside `hookSpecificOutput`:
- `permissionDecision`: `"allow"`, `"deny"` or `"ask"`.
- `permissionDecisionReason`: shown to Claude, so it knows why it was blocked.
- `updatedInput`: replacement tool inputs.
- `updatedToolOutput` (`PostToolUse`): a replacement result, e.g. timestamps converted to one format.
- `additionalContext` (`PostToolUse`): extra text added to the result.

See also: [[Deterministic-Compliance|Deterministic Compliance]] for why a hook beats a prompt rule.

### Messages you get back
| Python | TypeScript `type` | Contains |
|---|---|---|
| `AssistantMessage` | `"assistant"` | Claude's turn: `TextBlock`, `ToolUseBlock` |
| `UserMessage` | `"user"` | Tool results: `ToolResultBlock` |
| `SystemMessage` | `"system"` | Session information |
| `ResultMessage` | `"result"` | The last message: final answer, cost, `is_error` |

### All together: Scenario 3 as configuration
```python
options = ClaudeAgentOptions(
    system_prompt="You coordinate research and delegate to subagents.",
    allowed_tools=["Task", "Read"],
    agents={
        "web-searcher": AgentDefinition(
            description="Use to find sources on the web",
            prompt="Search the web and return findings with their URLs.",
            tools=["WebSearch", "WebFetch"],
        ),
    },
    hooks={"PreToolUse": [HookMatcher(matcher="Bash", hooks=[check_command])]},
)

async for message in query(prompt="Research the impact of AI on software development", options=options):
    print(message)
```

See also: [[Scenario-3-Multi-Agent-Research|Scenario 3: Multi-Agent Research System]].

<br>

## MCP (Model Context Protocol)
---

| Term | Meaning |
|---|---|
| MCP server | An ordinary program that offers tools to an agent and runs them. No AI inside |
| ★ MCP tools | Actions the agent can call, e.g. `get_customer`, `lookup_order`. Tools from all configured servers are available at the same time |
| ★ MCP resources | Read-only content the agent can browse, e.g. issue summaries, doc hierarchies, database schemas, so it needs fewer exploratory tool calls |
| ★ `isError` | Flag on a tool result saying the call failed |
| ★ `errorCategory`, `isRetryable` | Example fields of a structured error (transient / validation / permission), plus a readable message. You design these |
| ★ `.mcp.json` | Project-level server config, shared with the team through git |
| ★ `~/.claude.json` | User-level config for personal or experimental servers |
| ★ `${GITHUB_TOKEN}` | Environment variable expansion, so secrets stay out of the file |

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

<br>

## Claude Code CLI
---

| Flag | What it does |
|---|---|
| ★ `claude -p "…"` / `--print` | Non-interactive: runs the task, prints the result and exits. Used in CI |
| ★ `--output-format json` | Machine-readable output |
| ★ `--json-schema '{…}'` | The output must match this JSON schema |
| ★ `--resume <session-name>` | Continue a named earlier session |
| `--allowedTools "Read,Grep"` | Same idea as the SDK setting, for the CLI |

Example: `claude -p "Review this PR for security issues" --output-format json --json-schema '{…}'`

See also: [[Claude-Code-Headless-Mode|Claude Code Headless Mode]].

<br>

## Claude Code configuration
---

| Item | Meaning |
|---|---|
| ★ `~/.claude/CLAUDE.md` | User-level instructions. Only for you, not shared through git |
| ★ `CLAUDE.md` at the repo root | Project-level instructions, shared with the team |
| ★ `CLAUDE.md` in a subfolder | Directory-level instructions for that part of the codebase |
| ★ `@path/to/file.md` | Imports another file into CLAUDE.md, to keep it modular |
| ★ `.claude/rules/*.md` + `paths:` | Rules that load only when editing files matching the glob |
| ★ `.claude/commands/` vs `~/.claude/commands/` | Project slash commands (shared) vs personal ones |
| ★ `.claude/skills/<name>/SKILL.md` | A skill. Frontmatter: `context: fork` (runs in an isolated subagent), `allowed-tools` (limits tools while it runs), `argument-hint` (prompts for missing arguments) |
| ★ `/memory` | Shows which memory files are loaded |
| ★ `/compact` | Summarizes the history to free up context |
| ★ Plan mode | Explore and plan before changing anything. For large or architectural changes |
| ★ Explore subagent | Does verbose discovery in its own context and returns only a summary |

See also: [[Glob]] for the `paths:` syntax, and [[Scenario-2-Code-Generation|Scenario 2: Code Generation with Claude Code]].
