# Treewise

**Vectorless, reasoning-based retrieval for long documents.**

Treewise is the Python SDK and CLI for the Treewise API. It does for reasoning-based retrieval what a vector database does for similarity search. You index a document once, query it as many times as you need, and delete it when you're done.

Treewise doesn't use embeddings or chunks. It builds a hierarchical tree of the document (sections, subsections, pages), and an LLM walks that tree to find the parts that actually answer your question. The results come back with page-level citations.

---

## Why Treewise

| | Vector retrieval | Treewise |
|---|---|---|
| Unit of retrieval | Fixed-size chunks | Document sections, as the author structured them |
| How matches are found | Embedding similarity | LLM reasoning over a document tree |
| Chunk size tuning | Required | Not needed |
| Citations | Chunk offsets | Page and section references |
| Good at | Broad semantic recall | Long, structured documents such as contracts, filings, manuals and reports |

## How it works

1. **Index (server side).** You upload a document. Treewise extracts it with [LLMWhisperer](https://unstract.com/llmwhisperer/) V2 (layout preserved), builds a tree of its structure, and stores it under an index ID.
2. **Query (client side).** The SDK fetches the tree and runs the tree search, so you control the reasoning step.
3. **Delete.** You remove the index when you no longer need it.

```
document ──► LLMWhisperer V2 extraction ──► tree index (stored)
                                                │
question ──► SDK tree search ◄────────────────────┘
                   │
                   ▼
          relevant sections + page citations
```

## Installation

```bash
pip install treewise
```

## Authentication

Every request is authenticated with a Treewise API key. The SDK and CLI read it from the `TREEWISE_API_KEY` environment variable:

```bash
export TREEWISE_API_KEY="tw_..."
```

You can also pass it explicitly with `TreewiseClient(api_key="tw_...")`.

You don't need an LLMWhisperer account. Extraction runs on the service side, and usage is metered and billed per Treewise API key.

## Quick start

> The API is in early development, so method names and parameters may change before v1.

```python
from treewise import TreewiseClient

client = TreewiseClient()  # reads TREEWISE_API_KEY

# 1. Index once
index = client.index("annual-report-2025.pdf")
print(index.id, index.status)

# 2. Query many times
result = client.query(index.id, "What were the main risk factors cited for FY2025?")
for node in result.nodes:
    print(node.title, node.pages)
print(result.answer)

# 3. Delete when done
client.delete(index.id)
```

## CLI

```bash
treewise index annual-report-2025.pdf
treewise query <index-id> "What were the main risk factors?"
treewise tree <index-id>          # print the document tree
treewise delete <index-id>
```

## Bring your own LLM (optional)

By default, Treewise builds the index with a managed LLM. If you'd rather use your own provider and key for indexing, pass them in:

```python
index = client.index(
    "contract.pdf",
    llm={"provider": "openai", "model": "<model>", "api_key": "<your-key>"},
)
```

The tree search runs inside the SDK, so you decide which LLM it uses at query time.

## Status

Treewise is an early-stage personal project. The SDK and CLI are open source in this repository. The indexing API lives in a separate service repository.

## Contributing

Issues and pull requests are welcome. Please open an issue first for larger changes.

## License

<!-- TODO: choose a license -->
