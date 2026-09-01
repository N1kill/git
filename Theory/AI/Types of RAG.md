---
tags: [theory, ai, rag]
---

# Types of RAG

Related: [[RAG]] · [[Embeddings & Vector DBs]] · [[Agentic AI]]

## Why "type" is really "which constraint are you optimizing for"
These aren't competing ideas where one is simply "better" — each answers a different constraint: what shape your data is in, whether questions require relational reasoning or plain lookup, how much latency/infra cost you can afford, and how much autonomy you want the retrieval step itself to have. Most production systems end up combining more than one of these, not picking exactly one.

## Vector RAG — the default
Retrieval is pure embedding similarity search: embed the query, find the closest chunk vectors in a vector database (see [[Embeddings & Vector DBs]]). This is what "RAG" means by default when no qualifier is given.
- **Use when**: your knowledge is mostly unstructured prose, and semantic/conceptual matching matters more than exact keyword matching — someone can phrase a question completely differently from how the source document phrases the answer, and this still finds it.
- **Where it falls short**: dense embeddings can blur exact matches — a specific product code, name, or ID might not be the closest embedding match even when it's the exact right answer.

## Vectorless RAG — sometimes the vector DB isn't worth it
Retrieval without any embedding step — pure keyword/BM25 search, structured queries over metadata, or even an LLM directly navigating a file structure or API.
- **Use when**: the corpus is small enough that building and maintaining a vector index is unnecessary overhead, exact-match/structured lookup matters more than semantic similarity (codes, dates, IDs), or you want zero extra infrastructure for a lightweight use case.
- **This is the case for simplifying**, not upgrading — reach for it when vector search would be solving a problem you don't actually have.

## Hybrid RAG — combining lexical and semantic search
Runs dense (embedding) retrieval and sparse (BM25/keyword) retrieval in parallel, then merges the results (commonly via reciprocal rank fusion).
- **Why this beats either alone**: dense and sparse retrieval fail on *different* query types — dense retrieval misses exact terms and proper nouns, sparse retrieval misses paraphrases and synonyms. Combining them covers both failure modes at once.
- **Use when**: your real query mix includes both exact-term lookups (names, codes) and conceptual/semantic questions — which is most production systems once you look at actual user queries. This is the common real-world default once plain vector RAG starts missing things.

## Graph RAG — for questions about relationships, not just content
Builds a knowledge graph from the source documents — entities as nodes, relationships as edges — often via an LLM extraction pass over the documents first. Retrieval then walks the graph (traversal, subgraph extraction) instead of, or alongside, similarity search.
- **Why plain vector retrieval isn't enough here**: some questions depend on *relationships between* entities, not the similarity of any single chunk to the query — "who reports to the person who approved X's budget" isn't something any one chunk answers; it requires connecting facts across the graph.
- **Use when**: your questions genuinely require multi-hop relational reasoning across entities, not just finding the single most relevant passage.
- **The real tradeoff**: building and maintaining a knowledge graph (entity extraction, entity resolution, keeping it in sync with source updates) is substantially more expensive than maintaining a vector index — worth it only when the relational reasoning is actually the point.

## Agentic RAG — retrieval as a decision, not a fixed step
Instead of one fixed retrieve-then-generate pass, the LLM decides *when*, *how many times*, and *what* to retrieve — as one tool among possibly several, mid-reasoning. It can re-query if the first retrieval came back insufficient, decompose a complex question into sub-questions, or decide retrieval isn't even needed for a given query. This overlaps heavily with [[Agentic AI]] — RAG becomes just one tool the agent chooses to use.
- **Use when**: questions are complex or multi-part, may genuinely require several rounds of lookup to fully answer, or your system has other tools too and needs to *decide* whether retrieval is the right move at all for a given input.
- **The cost**: more LLM calls, more latency, more places for something to go wrong (an agent that decides not to retrieve when it should have) — not worth it for simple, single-lookup question types.

## Hierarchical / document-structure-aware RAG — for long, structured sources
Respects document structure instead of flattening everything into same-sized chunks. A common pattern: retrieve at a coarse level first (which section is this even about), then drill into fine-grained chunks within that section — or keep parent-child relationships so a small matched chunk can pull in its surrounding section for full context.
- **Use when**: source documents are long and genuinely structured — legal contracts, technical manuals, textbooks — where meaning depends heavily on which section/heading a passage sits under, and flat chunking would sever that structural context.

## How to actually choose, given all of the above
1. Start with **vector RAG** — it validates whether RAG helps your use case at all, fastest to build.
2. Exact-term queries (names, codes, IDs) failing? → add **hybrid search**.
3. Answers shallow/wrong on multi-part questions? → try **re-ranking** first (cheap, see [[RAG]]), then **agentic RAG** if that's still not enough.
4. Questions genuinely about relationships between entities, not just content similarity? → **Graph RAG**.
5. Source documents are long/structured and flat chunks lose context? → **hierarchical/document-aware chunking**.
6. Corpus is small or infra constraints are real? → consider whether **vectorless** is actually sufficient before building a vector index you don't need.
