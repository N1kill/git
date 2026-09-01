---
tags: [theory, ai, embeddings, vector-db]
---

# Embeddings & Vector Databases

Related: [[RAG]] · [[NLP Basics]] · [[Transformers]]

## Intuition
An embedding is a way of representing something — a word, sentence, document, image — as a dense vector of numbers, such that "similar meaning" translates to "close together in vector space." This is the bridge that lets you do math on meaning: instead of asking "does this text contain the same words as that text," you can ask "does this vector point in roughly the same direction as that vector," which captures semantic similarity even when the actual words are completely different (paraphrases, synonyms, related concepts).

Where do embeddings come from? A model (often a Transformer encoder, see [[Transformers]]) is trained so that inputs with similar meaning end up with similar vectors — usually via a contrastive objective: pull embeddings of known-similar pairs closer together, push embeddings of dissimilar pairs apart.

Once everything is a vector, "search" becomes "find the nearest vectors to my query vector" — a geometric problem. A **vector database** exists because doing this nearest-neighbor search efficiently over millions or billions of vectors, in real time, is a genuinely hard indexing problem — brute-force comparing your query against every stored vector doesn't scale, so vector DBs use approximate nearest neighbor (ANN) algorithms that trade a small amount of accuracy for massive speed gains.

## Reference

**Embedding models**
- Sentence/document embedding models (e.g. OpenAI text-embedding, Cohere embed, BGE, E5): map text → fixed-length dense vector (commonly 384–3072 dimensions)
- Trained via contrastive learning: minimize distance between embeddings of semantically similar pairs, maximize distance between dissimilar pairs
- **Must use the same embedding model for indexing and querying** — vectors from different models aren't comparable, they don't share a coordinate space

**Similarity metrics**
- **Cosine similarity**: `(A·B)/(||A||·||B||)` — measures angle between vectors, ignores magnitude. Most common for text embeddings.
- **Dot product**: `A·B` — factors in magnitude too; equivalent to cosine similarity if vectors are normalized
- **Euclidean (L2) distance**: straight-line distance — more common in non-text embedding spaces (e.g. some image applications)

**Approximate Nearest Neighbor (ANN) algorithms**
Exact nearest-neighbor search is O(n) per query against n stored vectors — too slow at scale. ANN trades a small accuracy loss for large speed gains:
- **HNSW (Hierarchical Navigable Small World)**: builds a multi-layer graph structure, navigate from sparse top layer down to dense bottom layer — most widely used, strong accuracy/speed tradeoff, used by most modern vector DBs by default
- **IVF (Inverted File Index)**: cluster vectors into buckets (via k-means-like partitioning), search only the most relevant buckets at query time
- **Product Quantization (PQ)**: compress vectors into compact codes to reduce memory footprint, often combined with IVF (IVF-PQ)
- **LSH (Locality-Sensitive Hashing)**: hash similar vectors into the same buckets with high probability — older approach, largely superseded by HNSW for most use cases

**Vector database options**
- Dedicated: Pinecone, Weaviate, Milvus, Qdrant, Chroma
- Bolt-on to existing DBs: pgvector (Postgres), Redis (vector search module), Elasticsearch/OpenSearch (kNN plugin)
- Tradeoffs: managed vs self-hosted, metadata filtering support, hybrid (sparse+dense) search support, scale ceiling

**Metadata filtering**
Real systems combine vector similarity with structured filters (e.g. "similar to this AND date > X AND category = Y") — vector DBs support pre-filtering (filter first, then search — can miss globally-similar results outside the filter) or post-filtering (search first, then filter — can return too few results if the filter is restrictive).

**Dimensionality tradeoffs**
- Higher dimensions → capture more nuance, but more storage, slower search, and can suffer from the curse of dimensionality (see [[ML]]) where distances become less discriminative
- Some modern embedding models support **Matryoshka embeddings** — trained so that truncating the vector to fewer dimensions still gives a usable (if less precise) embedding, letting you trade accuracy for speed/storage on demand

**Chunking is upstream of embeddings, not a vector-DB concern itself** — see [[RAG]] for chunking strategy, since what you embed matters as much as how you search it.
