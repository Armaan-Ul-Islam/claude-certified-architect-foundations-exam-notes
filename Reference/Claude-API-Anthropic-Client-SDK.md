---
type: "reference"
tags:
  - "api"
  - "client-sdk"
  - "python"
  - "claude-api-course"
aliases:
  - "Anthropic Client SDK"
  - "Python Client SDK"
  - "Anthropic Python SDK"
  - "Claude API in Python"
  - "Building with the Claude API"
---
# Claude API: Anthropic Client SDK

Calling the Claude API from Python with the `anthropic` package, the Client SDK, following Anthropic's course [Building with the Claude API](https://anthropic-partners.skilljar.com/claude-with-the-anthropic-api). For how it differs from the Agent SDK, see [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]].

<br>

## Chapters
---

One note per core chapter, in course order. Code that calls Claude uses the `client` and `model` from [[Claude-API-Accessing-Claude#Setup|Setup]].

| Chapter | What's in it |
|---|---|
| [[Claude-API-Accessing-Claude\|Accessing Claude with the API]] | How a request works, setup, multi-turn chat, system prompts, temperature, streaming, stop sequences |
| [[Claude-API-Prompt-Evaluation\|Prompt Evaluation]] | The eval workflow, code and model graders |
| [[Claude-API-Prompt-Engineering\|Prompt Engineering Techniques]] | Clear and direct prompts, specific guidelines, XML tags |
| [[Claude-API-Tool-Use\|Tool Use with Claude]] | Tool functions and schemas, tool results, multiple tools, stop reason, text editor and web search |
| [[Claude-API-RAG\|RAG and Agentic Search]] | Chunking, embeddings, vector search, BM25 |

Not in the notes yet: Features of Claude, Model Context Protocol, Anthropic apps, Agents and workflows.

<br>

## Related
---

- [[Agent-SDK-vs-Client-SDK|Agent SDK vs Client SDK]]
- [[Claude-Technologies-Reference#Client SDK|Client SDK (reference)]]
- [[Agentic-Loop-Message-Cycle|Agentic Loop: The Message Cycle]]
- [[Inference-Next-Token-Prediction|Inference: How Claude Predicts the Next Token]]: what happens inside the model
- [[Courses-by-Anthropic|Courses by Anthropic]]: course 2, Building with the Claude API
