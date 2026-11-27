---
type: "reference"
tags:
  - "api"
  - "mcp"
  - "python"
  - "claude-api-course"
aliases:
  - "Model Context Protocol"
  - "MCP servers"
  - "MCP clients"
  - "MCP resources"
  - "MCP prompts"
  - "MCP Inspector"
  - "FastMCP"
---
# Model Context Protocol

← [[Claude-API-Features-of-Claude|Features of Claude]] · [[Claude-API-Anthropic-Client-SDK|All chapters]]

MCP is a communication layer that gives Claude tools and context without you writing all the tool code yourself. Each primitive below shows the server side, then the client side (prompts: server side only so far).

<br>

## Why MCP
---

![Model Context Protocol shifts the burden of tool definitions and execution onto MCP servers: today our server holds the get_repos tool, its schema and its function, next to an empty MCP server and Claude](../assets/MCP-Shifts-Tool-Burden.png)

Without MCP, our server holds every tool's schema and function: for Claude to work with GitHub, we'd write and maintain a tool for each action (repos, pull requests, issues, …).

![MCP servers provide access to data or functionality implemented by some outside service: the get_repos tool, schema and function, now lives in an MCP server for GitHub, which calls GitHub, while our server talks to it and to Claude](../assets/MCP-Server-for-GitHub.png)

An MCP server is an ordinary program, no AI inside. Anyone can write one, and services often publish an official one. MCP doesn't replace tool use: Claude still asks for a tool and our server still sends back the result. MCP only moves who writes and runs the tool.

![Extending Claude with MCP: sentry-mcp discovers and fixes bugs logged in Sentry, playwright-mcp gives Claude browser automation to inspect the visual components it builds or troubleshoots, figma-context-mcp exposes Figma designs, mcp-atlassian gives access to Confluence and Jira, firecrawl-mcp-server adds web scraping, slack-mcp posts messages or replies to threads](../assets/MCP-Extending-Claude-Servers.png)

![Our server holds an MCP client connected to two MCP servers, each offering tools, prompts and resources and calling its own outside service](../assets/MCP-Client-and-Servers.png)

![MCP server primitives: tools are model-controlled, Claude decides when to call them and uses the results, to give Claude extra functionality; resources are app-controlled, our app decides when to read them and mainly uses the results, to get data into our app and add context to messages; prompts are user-controlled, the user decides when to use them, for workflows run from user input like a slash command, button click or menu option](../assets/MCP-Server-Primitives.png)

On the exam: where to configure servers (`.mcp.json` or `~/.claude.json`) and when to build your own ([[Domain-2-Tool-Design-and-MCP#2.4 Integrate MCP servers into Claude Code and agent workflows|Task 2.4]], [[Claude-Technologies-Reference#MCP (Model Context Protocol)|MCP reference]]).

> [!tip] Exam tip
> For a standard integration like Jira or GitHub, use the existing community MCP server. Build a custom one only for a workflow specific to your team.

^exam-tip

<br>

## Client and server
---

![Important note: normally a project implements either an MCP client or an MCP server; the course's project implements both, just to show how they work. Our server holds the MCP client, which talks to the MCP server](../assets/MCP-Client-or-Server.png)

![mcp_client.py: the MCP Client, a custom class we author to make the session easier to use, wraps the Client Session, the actual connection to the MCP server](../assets/MCP-Client-Session.png)

With the Python SDK (`mcp`), `mcp_server.py` serves documents kept in a dict, `docs` (document id → text), and `MCPClient` starts that server as a subprocess and talks to it over stdio.

Has moved since the course: SDK v2 adds `Client` (`from mcp import Client`), which connects to a server's URL or starts a local one over stdio with no wrapper of your own. Result fields are snake_case in Python (`is_error`, `input_schema`, `mime_type`).

On the exam: no SDK code. Claude Code and the Agent SDK have the client built in, so you only configure servers (`.mcp.json`, `mcp_servers`) ([[Domain-2-Tool-Design-and-MCP#2.4 Integrate MCP servers into Claude Code and agent workflows|Task 2.4]]).

<br>

## Tools
---

![One query through MCP: our server gets the tool list from the MCP server through the MCP client (ListToolsRequest, ListToolsResult), sends the query and tools to Claude, gets a ToolUse back, has the MCP server run it against GitHub (CallToolRequest, CallToolResult), sends the toolResult to Claude and returns its answer, Your repositories are…](../assets/MCP-Tool-Call-Sequence.png)

Claude never talks to the MCP server: our server stays in the loop, as in [[Claude-API-Tool-Use|plain tool use]]. On the wire, ListToolsRequest is `tools/list` and CallToolRequest is `tools/call`.

On the exam: tools from every configured server are found at connection time and are all available at once, and the description decides whether the agent picks a tool over a built-in like Grep ([[Domain-2-Tool-Design-and-MCP#2.4 Integrate MCP servers into Claude Code and agent workflows|Task 2.4]]). A failed call needs `errorCategory`, `isRetryable` and a readable message, not just an error string ([[Domain-2-Tool-Design-and-MCP#2.2 Implement structured error responses for MCP tools|Task 2.2]]).

### Defining tools
Two tools, `read_doc_contents` and `edit_document`, each a decorated function:

```python
from mcp.server.fastmcp import FastMCP
from pydantic import Field

mcp = FastMCP("DocumentMCP", log_level="ERROR")

@mcp.tool(
    name="read_doc_contents",
    description="Read the contents of a document and return it as a string."
)
def read_document(
    doc_id: str = Field(description="Id of the document to read")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")

    return docs[doc_id]
```

No hand-written JSON schema as in [[Claude-API-Tool-Use#Tool schemas|Tool Use]]: the SDK builds it from the decorator's `name` and `description`, the type hints and each argument's `Field(description=…)`. A raised error reaches the client as a tool result with `isError` set.

Has moved since the course: SDK v2 renamed `FastMCP` to `MCPServer` (`from mcp.server import MCPServer`), and the `mcp.server.fastmcp` import fails there. The decorators and `log_level` are unchanged.

### Listing and calling tools
The client's two tool methods just forward to the `ClientSession`:

```python
async def list_tools(self) -> list[types.Tool]:
    result = await self.session().list_tools()
    return result.tools

async def call_tool(
    self, tool_name: str, tool_input: dict
) -> types.CallToolResult | None:
    return await self.session().call_tool(tool_name, tool_input)
```

`list_tools` fetches the tools our server sends to Claude, and `call_tool` runs the one Claude asked for. To test, run `mcp_client.py` directly; its harness starts the server and prints its tools:

```python
async with MCPClient(
    command="uv", args=["run", "mcp_server.py"]
) as client:
    result = await client.list_tools()
    print(result)
```

<br>

## Resources
---

![When a user types @, the MCP server must give us a list of documents, each a Resource; when a message mentions a document, the MCP server must give us that one document's contents, which go into the prompt in a document tag](../assets/MCP-Resources-Document-Mentions.png)

Claude never asks for a resource the way it asks for a tool: our app reads it by URI and puts the content in the prompt, as in the slide's `<document>` tag.

On the exam: resources expose a content catalog (issue summaries, a documentation tree, database schemas) so the agent needs fewer exploratory tool calls ([[Domain-2-Tool-Design-and-MCP#2.4 Integrate MCP servers into Claude Code and agent workflows|Task 2.4]]).

### Defining resources
![Resources let the MCP server expose data to the client, like GET request handlers in an HTTP server; they return any type of data, with mime_type as a hint to the client; two types, direct and templated](../assets/MCP-Resources-Direct-and-Templated.png)

A direct resource has a fixed URI and lists the documents:

```python
@mcp.resource(
    "docs://documents",
    mime_type="application/json"
)
def list_docs() -> list[str]:
    return list(docs.keys())
```

A templated resource has a parameter in its URI, which the SDK passes to the function, and fetches one document:

```python
@mcp.resource(
    "docs://documents/{doc_id}",
    mime_type="text/plain"
)
def fetch_doc(doc_id: str) -> str:
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    return docs[doc_id]
```

### Reading resources
![Reading a resource: the user types What's in the @…, our code needs a list of document names for the autocomplete, the MCP client sends a ReadResourceRequest for docs://documents, and the MCP server answers with a ReadResourceResult holding the list of doc names](../assets/MCP-Read-Resource-Sequence.png)

`docs://documents` browses the document names for the `@` autocomplete, `docs://documents/report.pdf` fetches one document for the prompt. `read_resource` sends the ReadResourceRequest (`resources/read` on the wire) and uses the server's `mime_type` to decide how to parse what comes back:

```python
async def read_resource(self, uri: str) -> Any:
    result = await self.session().read_resource(AnyUrl(uri))
    resource = result.contents[0]

    if isinstance(resource, types.TextResourceContents):
        if resource.mimeType == "application/json":
            return json.loads(resource.text)

    return resource.text
```

Has moved since the course: in SDK v2, `read_resource()` takes the URI as a plain `str`, no `AnyUrl`.

<br>

## Prompts
---

The user runs a prompt as a slash command:

![The course's app: the user types /format plan.md, picking plan.md from an autocomplete of deposition.md, report.pdf, financials.docx, outlook.pdf, plan.md and spec.txt](../assets/MCP-Prompts-Format-Command.png)

Claude Code lists an MCP server's prompts as slash commands too, `/mcp__<server>__<prompt>`.

On the exam: MCP prompts aren't in the exam guide, so they're out of scope.

### Defining prompts
![Prompts define a set of user and assistant messages the client can use, and should be high quality, well tested and relevant to the overall purpose of the MCP server; the format prompt, format_document(doc_id), returns a list of messages](../assets/MCP-Prompts.png)

`format_document` returns one `base.UserMessage`: an instruction to rewrite the document as Markdown with `edit_document`, with `doc_id` filled in.

Has moved since the course: in SDK v2, `base` (`from mcp.server.fastmcp.prompts import base`) imports from `mcp.server.mcpserver.prompts`.

<br>

## The server inspector
---

```bash
mcp dev mcp_server.py
```

Opens the MCP Inspector in the browser, to try the server without Claude or a client: Connect, List Tools, pick one, fill in its arguments and run it.

![MCP Inspector connected over STDIO with uv run --with mcp mcp run mcp_server, its Tools tab listing read_doc_contents and edit_document, and History showing initialize then tools/list](../assets/MCP-Server-Inspector.png)

History logs each request the inspector sent, `initialize` first.

Has moved since the slide: the 2026-07-28 MCP spec deprecates Sampling and Roots (two of the tabs), and logging. They still work with servers on older spec versions.

The inspector is out of scope for the exam.
