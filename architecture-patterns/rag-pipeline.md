# RAG Pipeline Architecture

Retrieval-Augmented Generation: ground an LLM's output in a specific corpus instead of relying on what it memorized during training. The pattern shows up any time an answer needs to be traceable to a source document — legal, financial, internal knowledge bases, support — anywhere "the model made it up" is not an acceptable failure mode.

## The problem it solves

An LLM's context window is finite and its training data is frozen at a cutoff. RAG works around both: instead of putting the whole corpus in the prompt, or fine-tuning the model on it, you index the corpus separately and pull in only the pieces relevant to the current query at request time. The model never needs to "know" the corpus — it needs the right five paragraphs handed to it.

## Two pipelines, not one

The architecture splits cleanly into an **offline (indexing) pipeline**, which runs whenever the corpus changes, and an **online (query) pipeline**, which runs on every request. Conflating them — e.g., re-chunking and re-embedding documents on the request path — is the single most common design mistake; it turns a cache-able, batchable job into per-request latency.

```
Offline / indexing pipeline (runs on ingest, batched)
  Documents → Extract → Chunk → Embed → Index (vector DB + metadata store)

Online / query pipeline (runs per request, latency-sensitive)
  Query → Embed query → Retrieve (+ rerank) → Assemble context → Generate → (Cite)
```

Everything in the offline pipeline can be slow and thorough. Everything in the online pipeline is on the user's clock — that asymmetry should drive most of the engineering effort.

## Offline pipeline: components

- **Extraction** — pull text (and, for multimodal corpora, images/tables) out of source formats. This is where OCR, layout-aware parsing, and structure preservation (headings, tables, footnotes) happen. Garbage in here means ungrounded answers later, no matter how good retrieval is.
- **Chunking** — split documents into retrievable units. The chunk is the unit of retrieval, so its size is a direct tradeoff: too large and irrelevant text dilutes the embedding and wastes context; too small and you lose the surrounding meaning a chunk needs to be useful on its own. A commonly cited working range is **512–1,024 tokens per chunk with 10–20% overlap** between adjacent chunks, to avoid severing a sentence or idea at a boundary — treat that as a starting point to tune against your own corpus, not a fixed constant.

  | Strategy | What it does | Best for |
  |---|---|---|
  | Fixed-size | Split at a token/character count | Uniform technical docs |
  | Semantic | Split at natural boundaries (paragraph, section) | Mixed content types |
  | Hierarchical | Multiple chunk granularities (doc → section → paragraph) | Large, deeply structured documents |
  | Agentic | An LLM decides chunk boundaries | Heterogeneous, messy source material |

- **Embedding** — convert each chunk into a dense vector via a transformer embedding model, so semantic similarity becomes vector proximity. The embedding model is a dependency you're committing to: re-embedding the whole corpus is the cost of switching later.
- **Indexing** — store vectors in a vector database (FAISS, Chroma, Pinecone, Weaviate) alongside the chunk text and metadata (source doc, section, entities, timestamps). Plain vector similarity misses exact-match cases (IDs, codes, proper nouns) — pairing it with a keyword index (BM25, Elasticsearch) for **hybrid search** covers both.

## Online pipeline: components

- **Query embedding** — embed the incoming query with the *same* embedding model used at index time. A mismatch here silently degrades every retrieval — it won't error, it'll just return mediocre results.
- **Retrieval** — nearest-neighbor search against the vector index, typically combined with metadata filters (date range, document type, access permissions) and the keyword index if hybrid search is in play.
- **Reranking (optional but common)** — retrieval optimizes for recall over a large index cheaply; a smaller, more expensive cross-encoder reranks the top-k results for precision before they reach the prompt. Worth adding once retrieval quality, not generation quality, is the bottleneck.
- **Context assembly** — decide how many chunks make the final prompt, in what order, and how they're formatted (with or without source citations inline). This is where token-budget tradeoffs between "more context" and "room for the model to reason" get made explicit.
- **Generation** — the LLM call itself, prompted to answer *from the provided context* and (ideally) to say when the context doesn't contain an answer, rather than falling back to parametric knowledge.
- **Citation/attribution** — linking generated claims back to source chunks. Straightforward when the model is instructed to cite inline; harder to retrofit after the fact, so design for it from the start if traceability matters.

## Variants

- **Naive RAG** — the pipeline above, one retrieval pass, no reranking. Fine for well-structured corpora and simple factual queries.
- **Hybrid RAG** — vector + keyword retrieval combined, as above. Default choice once the corpus includes exact-match-sensitive content (codes, names, numbers).
- **Multi-hop RAG** — the query is decomposed into sub-questions, each retrieved separately, results synthesized. Needed when an answer requires connecting facts that don't live in the same chunk.
- **Self-RAG / adaptive retrieval** — the model decides *whether* to retrieve at all, and can issue follow-up retrievals if the first pass looks insufficient. Trades a fixed, predictable pipeline for one that costs more on hard queries and less on easy ones.
- **Agentic RAG** — retrieval becomes one tool among several a reasoning loop can call, alongside things like calculators, other APIs, or a second retrieval index. See [agentic-ai/](../agentic-ai/) for the loop side of this; the "R" in RAG stays the same pattern described above.

## When not to reach for it

- The corpus fits comfortably in the model's context window and doesn't change often — just put it in the prompt (or cache it) and skip the retrieval infrastructure entirely.
- The task needs the model's *behavior* to change (tone, format, domain reasoning style), not its factual grounding — that's a fine-tuning or prompt-engineering problem, not a retrieval problem. RAG adds facts; it doesn't change how the model uses them.
- Freshness doesn't matter and the corpus is small enough to fully re-embed on every deploy — a lighter-weight "stuff it in the prompt" approach can outperform a RAG stack in both accuracy and engineering cost at that scale.

## Failure modes

- **Chunk/embedding mismatch at query time** — re-embedding the corpus with a new model without re-indexing, or embedding queries with a different model than was used for chunks.
- **Silent retrieval failure** — the top-k chunks are irrelevant, but the model answers fluently anyway instead of admitting the context didn't contain the answer. Prompt for explicit "not found in context" behavior and evaluate for it directly.
- **Chunking that severs meaning** — a table, a legal clause, or a code block split across a chunk boundary produces confidently wrong retrieval. Structure-aware chunking (respecting tables/headings) is worth the extra preprocessing effort for corpora that have a lot of it.
- **No evaluation loop** — shipping retrieval without measuring precision/recall on a held-out query set means quality regressions from a re-embedding or reranker swap go unnoticed until a user hits one.

## Related

- [research/document-processing/](../research/document-processing/) — the source material this pattern was distilled from: chunking strategy detail, indexing techniques, large-document parsing, and industry case studies.
- [agentic-ai/](../agentic-ai/) — where retrieval becomes one tool in a larger reasoning loop rather than a fixed pipeline stage.
