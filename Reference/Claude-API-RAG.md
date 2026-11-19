---
type: "reference"
tags:
  - "api"
  - "rag"
  - "embeddings"
  - "python"
  - "claude-api-course"
aliases:
  - "RAG"
  - "Retrieval Augmented Generation"
  - "RAG and Agentic Search"
  - "Text embeddings"
  - "Vector search"
---
# RAG and Agentic Search

← [[Claude-API-Tool-Use|Tool Use with Claude]] · [[Claude-API-Anthropic-Client-SDK|All chapters]]

Retrieval Augmented Generation (RAG): split a long document into chunks, find the chunks closest to the user's question, and put only those in the prompt.

> [!tip] Exam tip
> Embedding models and vector database details are out of scope for the exam. Chunking does come up: a document too big for the context window is split before it's resubmitted, for example a failed batch item ([[Message-Batches-API|Message Batches API]], [[Domain-4-Prompt-Engineering#4.5 Design efficient batch processing strategies|Task 4.5]]).

^exam-tip

<br>

## The idea
---

![Option 2: break the document up into many chunks, such as Strategy Outlook, Balance Sheet and Risk Factors, and put the ones relevant to the question in the prompt](../assets/RAG-Document-to-Chunks.png)

![The user's question, What risks does this company face?, and the relevant chunks go into the prompt inside user_question and financial_document tags](../assets/RAG-Relevant-Chunks-in-Prompt.png)

Option 1, the whole document in every prompt, runs into the context limit, costs more, and [[Inference-Next-Token-Prediction#^attention-dilution|spreads Claude's attention thinner]].

<br>

## Chunking
---

![Chunk source text: the source document, an annual financial report, is split into chunks of text](../assets/RAG-Chunk-Source-Text.png)

![Chunking strategies: size based cuts equal-length strings with overlap, structure based cuts at headers, paragraphs and sections, semantic based groups related sentences at a higher compute cost](../assets/RAG-Chunking-Strategies.png)

![Overlap: each chunk also takes some text from its neighbors, so the chunk for Section 1 runs into the start of Section 2](../assets/RAG-Chunk-Overlap.png)

![Structure based: the intro, Section 1: Medical Research and Section 2: Software Engineering each become one chunk](../assets/RAG-Chunk-by-Section.png)

`chunk_by_char` cuts every 150 characters and `chunk_by_sentence` every 5 sentences, both with some overlap. `chunk_by_section` cuts at each `##` header.

```python
def chunk_by_char(text, chunk_size=150, chunk_overlap=20): ...

def chunk_by_section(document_text): ...

def chunk_by_sentence(text, max_sentences_per_chunk=5, overlap_sentences=1): ...
```

<br>

## Embeddings
---

![Text embeddings: an embedding model turns each text, such as I'm very happy today!, into a list of numbers that represents its meaning](../assets/RAG-Text-Embeddings.png)

![Each number scores some quality of the text, such as how happy or how formal it is, but nobody knows exactly what each number measures](../assets/RAG-Embedding-Scores.png)

Anthropic has no embedding model of its own, so `client` here is a Voyage AI client: `voyageai.Client()`, with the key in `VOYAGE_API_KEY`. Pass `input_type="document"` for chunks and `"query"` for questions. `voyage-4-large` is the current generation of `voyage-3-large`.

```python
def generate_embedding(text, model="voyage-3-large", input_type="query"):
    result = client.embed([text], model=model, input_type=input_type)
    return result.embeddings[0]
```

<br>

## The RAG flow
---

Chunk the text, embed the chunks, store them, embed the question to find the closest chunks, then add those to the prompt.

### Embed and store the chunks
![Generate embeddings: an imaginary two-number model scores Medical Research (.97, 0.34) and Software Engineering (.30, .97) on medicine and software](../assets/RAG-Generate-Embeddings.png)

![On a unit circle, Software Engineering (0.295, 0.955) points toward software and Medical Research (0.944, 0.331) toward medicine](../assets/RAG-Chunks-on-Unit-Circle.png)

![Store the embeddings in a vector database, built to store and compare long lists of numbers](../assets/RAG-Vector-Database.png)

`VectorIndex` is the small in-memory store written in the course; a real app uses a vector database.

```python
embeddings = [generate_embedding(chunk, input_type="document") for chunk in chunks]

store = VectorIndex()

for embedding, chunk in zip(embeddings, chunks):
    store.add_vector(embedding, {"content": chunk})
```

### Find the closest chunks
![Embed the user's question with the same model: what did the software engineering department do this year? becomes (0.1, 0.89)](../assets/RAG-Embed-User-Query.png)

![Normalization scales the vector to length 1: (0.1, 0.89) becomes (0.112, 0.993)](../assets/RAG-Normalization.png)

![The user query (0.112, 0.993) lands next to Software Engineering on the unit circle, far from Medical Research](../assets/RAG-Query-on-Unit-Circle.png)

![Cosine similarity: the query scores 0.983 against Software Engineering and 0.398 against Medical Research; 1 means the same direction, 0 perpendicular, −1 opposite](../assets/RAG-Cosine-Similarity.png)

Voyage's vectors are already length 1, so cosine similarity is just their dot product ([[Inference-Next-Token-Prediction#Vectors: direction is meaning|interactive diagram]]). `search` returns the 2 nearest chunks with their distance:

```python
user_embedding = generate_embedding("What did the software engineering dept do last year?")

results = store.search(user_embedding, 2)

for document, distance in results:
    print(distance, "\n", document["content"][0:200], "\n")
```

### Add them to the prompt
![Add related chunks to the prompt: for How many bugs did engineers fix this year?, the Software Engineering chunk goes inside the report tags](../assets/RAG-Add-Chunks-to-Prompt.png)

<br>

## Lexical search (BM25)
---

Embeddings can miss an exact term such as the incident ID `INC-2023-Q4-011`. BM25 finds it by giving rare terms more weight, and its results are merged with the semantic ones.

![Hybrid search: for What happened with INC-2023-Q4-011?, semantic search with embeddings and lexical search with classic text search run side by side, and their results are merged](../assets/RAG-Hybrid-Search.png)

![BM25: tokenize the query, count how often each term appears across all documents, give rarer terms more importance, then pick the chunk that uses the higher-weighted terms most](../assets/RAG-BM25-Steps.png)
