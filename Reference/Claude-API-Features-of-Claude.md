---
type: "reference"
tags:
  - "api"
  - "client-sdk"
  - "thinking"
  - "vision"
  - "pdf"
  - "citations"
  - "caching"
  - "files"
  - "code-execution"
  - "python"
  - "claude-api-course"
aliases:
  - "Features of Claude"
  - "Extended thinking"
  - "Thinking blocks"
  - "Redacted thinking"
  - "Image support"
  - "PDF support"
  - "Citations"
  - "Prompt caching"
  - "Cache breakpoints"
  - "Files API"
  - "Code execution"
---
# Features of Claude

← [[Claude-API-RAG|RAG and Agentic Search]] · [[Claude-API-Anthropic-Client-SDK|All chapters]]

<br>

## Extended thinking
---

Claude reasons in a `thinking` block before it writes the answer in a `text` block.

![With thinking: the user asks for a short guide on recursion, and the assistant message holds a thinking block, I should cover what recursion is, how it works, then a text block with the guide](../assets/Extended-Thinking-Message-Flow.png)

### Signature and redacted thinking
![The signature: a cryptographic token tied to the thinking text, which proves Claude generated that exact text and nobody changed it](../assets/Extended-Thinking-Signature.png)

![Redacted thinking: reasoning flagged by Claude's safety systems comes back encrypted in a redacted_thinking block's data, only so you can hand it back to Claude on follow-up requests](../assets/Extended-Thinking-Redacted.png)

Send both kinds of block back unchanged in later requests, for example in a tool-use loop: the API rejects a thinking block whose text was changed.

### Turning it on
By now the course's `chat` from [[Claude-API-Accessing-Claude#Multi-Turn Conversations|Multi-Turn Conversations]] takes parameters for the earlier features and two for thinking. Its `temperature=1.0` is left out, since `claude-opus-5-5` rejects [[Claude-API-Accessing-Claude#Temperature|temperature]]:

```python
def chat(
    messages,
    system=None,
    stop_sequences=[],
    tools=None,
    thinking=False,
    thinking_budget=1024
): ...
```

`thinking=True` sends `thinking={"type": "enabled", "budget_tokens": thinking_budget}`, a budget of at least 1024 tokens and below `max_tokens`, which older models still take. `claude-opus-5-5` always thinks and rejects a budget with a 400: `output_config.effort` sets how hard it thinks instead. Thinking counts toward `max_tokens`, and its text comes back empty unless you ask for a summary:

```python
message = client.messages.create(
    model=model,
    max_tokens=16000,
    thinking={"type": "adaptive", "display": "summarized"},
    messages=messages,
)
```

Read the reply by block type, since thinking blocks come first ([[Claude-API-Accessing-Claude#Extracting the Response|Extracting the Response]]). On the exam, extended thinking is no substitute for an independent reviewer: a second instance without the first one's reasoning catches more ([[Domain-4-Prompt-Engineering#4.6 Design multi-instance and multi-pass review architectures|Task 4.6]]).

<br>

## Image support
---

An image goes in an `image` block in the user message, next to a `text` block that asks about it.

![Our server sends Claude a user message with an image block, holding the raw image data, and a text block, What do you see in this image?, and Claude answers with a text block, I see a tall tree…](../assets/Image-Support-Message-Flow.png)

![Image handling: up to 100 images in a request, 5MB each, 8000px a side for one image or 2000px for several, base64 or a URL, and width × height / 750 tokens per image, beside code that sends image.png as a base64 image block with a text block](../assets/Image-Support-Handling.png)

`add_user_message` is the helper from [[Claude-API-Accessing-Claude#Multi-Turn Conversations|Multi-Turn Conversations]], here passed a list of blocks instead of a string:

```python
with open("image.png", "rb") as image_file:
    image_bytes = base64.standard_b64encode(image_file.read()).decode("utf-8")

add_user_message(messages, [
    # Image Block
    {
        "type": "image",
        "source": {
            "type": "base64",
            "media_type": "image/png",
            "data": image_bytes,
        }
    },
    # Text Block
    {
        "type": "text",
        "text": "What do you see in this image?"
    }
])
```

For an image on the web, the `source` is `{"type": "url", "url": "https://…"}` instead.

Some limits have moved since the slide: on the Claude API, `claude-opus-5-5` takes up to 600 images of 10 MB each (base64), the smaller size cap applies only above 20 images, and an image costs one token per 28×28-pixel patch, with anything over 2,576 px on the long edge or 4,784 tokens scaled down first. Vision is out of scope for the exam.

<br>

## PDF support
---

A PDF goes in a `document` block, with the same base64 or URL `source` as an image. Claude gets each page's text and an image of the page, so it can read charts and tables too.

```python
with open("earth.pdf", "rb") as pdf_file:
    file_bytes = base64.standard_b64encode(pdf_file.read()).decode("utf-8")

messages = []

add_user_message(
    messages,
    [
        {
            "type": "document",
            "source": {
                "type": "base64",
                "media_type": "application/pdf",
                "data": file_bytes,
            },
        },
        {"type": "text", "text": "Summarize the document in one sentence"},
    ],
)

chat(messages)
```

[[Scenario-6-Structured-Extraction|Scenario 6]] extracts structured data from documents like this one.

### Citations
Give the document block a `title` and turn on `citations`:

```python
{
    "type": "document",
    "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": file_bytes,
    },
    "title": "earth.pdf",
    "citations": { "enabled": True }
}
```

![Citation structure: cited_text, the text Claude cites, such as Earth's atmosphere and oceans were formed by volcanic activity and outgassing; document_index 0, which document when several are sent; document_title earth.pdf; start_page_number 4 and end_page_number 5, the pages of the cited text](../assets/Citations-Structure.png)

Claude's answer then comes back as several `text` blocks, and each cited claim carries a `citations` list with these fields. `end_page_number` is exclusive, so 4 to 5 means page 4 only.

Plain text goes straight into the `source`, and its citations give `start_char_index` and `end_char_index` instead of pages:

```python
{
    "type": "document",
    "source": {
        "type": "text",
        "media_type": "text/plain",
        "data": article_text,
    },
    "title": "earth_article",
    "citations": { "enabled": True }
}
```

Either way, each claim stays tied to its source, the same pairing [[Preserving-Attribution|Preserving Attribution]] keeps when agents hand findings on.

<br>

## Prompt caching
---

Before it writes a word, Claude tokenizes the prompt, embeds each token and adds context from the text around it ([[Claude-API-Accessing-Claude#Accessing the API|Accessing the API]]). That work is normally thrown away after the request. Prompt caching keeps it, so a later request that starts with the same content skips it and runs faster and cheaper.

![Prompt caching: a user message asks Claude to summarize a long text and Claude answers with the summary, while the work of tokenizing the prompt, creating embeddings for each token and adding context from the surrounding text goes into a cache instead of the trash](../assets/Prompt-Caching-Cache-vs-Trash.png)

`"cache_control": {"type": "ephemeral"}` on a block sets a cache breakpoint: everything up to and including that block is cached. A text block needs its long form, a dict with `type` and `text`, to take the field.

![Breakpoint location: a breakpoint can also go on the last tool definition, caching the tools, or on the system prompt's text block, caching the system prompt; both rarely change between requests, so they are the most common caching opportunities](../assets/Prompt-Caching-Breakpoint-Location.png)

![Minimum content length: what gets cached must add up to at least 1024 tokens across all its blocks, so a lone Hi there! block isn't cached, while Hi there! repeated 500 times before it is](../assets/Prompt-Caching-Minimum-Content-Length.png)

The minimum has moved since the slide and depends on the model: 512 tokens on `claude-opus-5-5`, 1,024 to 4,096 on older ones. Anything shorter just isn't cached, with no error.

![Cache ordering: request 1 sends tools 1 to 3 with a breakpoint on tool 3, then the system prompt, a user message, an assistant message with a second breakpoint, and a user message; request 2 starts with the same three tools and breakpoint](../assets/Prompt-Caching-Cache-Ordering.png)

A request is joined in the order tools, system prompt, messages, and a cache hit needs everything up to the breakpoint to match exactly: change one block and the cache misses from that block on. Each breakpoint writes its own cache entry, so request 2 still reuses the cached tools when its system prompt or messages change.

A cache lasts 5 minutes and each use resets the clock; `"ttl": "1h"` keeps it an hour at a higher write price. A request takes up to 4 breakpoints. The response's `usage` counts `cache_creation_input_tokens`, written at 1.25× the input price, and `cache_read_input_tokens`, read at a tenth of it or less.

For the exam, just know caching exists ([[Claude-Technologies-Reference#Prompt caching|Prompt caching]]).

<br>

## Files API
---

Upload a file once, then refer to it by its id instead of sending its bytes in every request.

![Files API: our server sends a file upload request and gets back FileMetadata with an id; a user message then holds an image block whose source is the file by that file_id, plus a text block, What do you see in this image?, and Claude answers with a text block, I see a tall tree…](../assets/Files-API-Upload-Flow.png)

```python
file_metadata = upload('streaming.csv')
```

`upload` is the course's helper, and `file_metadata.id` then goes in an `image`, `document` or `container_upload` block. The Files API has moved since the course: it's out of beta, so the SDK call is `client.files.upload(...)` with no beta header. The exam guide doesn't name the Files API.

<br>

## Code execution
---

A server tool: Claude writes code and runs it in a sandbox on Anthropic's servers, and the code, its output and the answer all come back in one response.

![Code execution: our server sends the code execution tool schema with a user message, Calculate PI to the 50th decimal place, use your code execution tool; Claude executes code in a Docker container and answers with a server tool use block holding the code, a code execution tool result block with 3.1415926535…, and a text block giving the answer](../assets/Code-Execution-Message-Flow.png)

A `container_upload` block puts a file from the [[#Files API|Files API]] into that sandbox for the code to read:

![Code execution with a file: our server uploads my_data.csv and gets back a file id, then sends a container upload block with that file_id and a text block, Analyze the data in the file; Claude runs code on it in a Docker container and answers with a server tool use block, a code execution tool result block with the analysis and a text block](../assets/Code-Execution-Container-Upload.png)

```python
messages = []
add_user_message(
    messages,
    [
        {
            "type": "text",
            "text": """Run a detailed analysis to determine major drivers of churn.
            Your final output should include at least one detailed plot summarizing your findings."""
        },
        {"type": "container_upload", "file_id": file_metadata.id},
    ],
)

chat(
    messages,
    tools=[{"type": "code_execution_20250522", "name": "code_execution"}]
)

download_file("file_id_from_response")
```

![The response: a text block, I'll help you analyze the major drivers of churn in your streaming dataset…, then a server_tool_use block named code_execution whose input is the Python code, importing pandas, numpy and matplotlib, then a code_execution_tool_result block with the same tool_use_id](../assets/Code-Execution-Response-Blocks.png)

A file the code saves, such as the plot, comes back as a `file_id` in the tool result, and `download_file` fetches it through the Files API. Only files Claude creates can be downloaded, not ones you upload.

The tool has moved since the course: the current type is `code_execution_20260521`, which runs Bash as well as Python and needs no beta header, and a command's result comes back as a `bash_code_execution_tool_result` block.

On the exam, it's one of the server tools: you only list it in `tools`, Anthropic runs it, and your code sends no `tool_result` ([[Claude-Technologies-Reference#Server tools|Server tools]]).
