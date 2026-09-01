---
tags: [flare, architecture, python, rag]
component: rag-ingestion
status: corrected-evaluated
---

# RAG Ingestion Pipeline

Part of [[FLARE - Project Hub|FLARE]]. Python/FastAPI pipeline (`ingestion.py`) that takes log files (from disk analysis or direct upload) and turns them into a semantically searchable, hierarchically chunked index. Consumed by the Semantic Agent in the [[Multi-Agent System Design|RAG Query Engine]].

## Architecture: Four-Level Hierarchy (L0–L3)

**Correction from earlier notes**: the hierarchy is **four levels (L0–L3)**, not three (L1–L3).

- **L3** — raw chunks, finest granularity. CSV rows read individually, JSON parsed at object level, plain text segmented by line/window. 50 rows per chunk.
- **L2** — subsection summaries. LLM-generated, one per 250 raw rows (5 L3 chunks batched).
- **L1** — section summaries. Aggregated from L2.
- **L0** — single root/document summary for the entire file. Generated first, by sampling the head and tail of the document, to give the LLM guiding context for subsequent L3 summarization.

Each node stores a `parent_index` linking it to its parent, forming a navigable evidence tree. Built via **recursive task spawning**: as each L3 chunk completes, a "Section Monitor" waits for 5 chunks (250 rows), then spawns the parent L2 task; this bubbles up to L1 and finally L0.

Rationale for going hierarchical instead of flat: see [[Types of RAG]] for the general theory, and [[Decisions/Why Hierarchical RAG]] for FLARE's specific reasoning — flat RAG scored 72% accuracy / 12% hallucination vs. 91% / 2% for hierarchical (paper Table V).

## Embedding & Storage

- **Model**: `BAAI/bge-small-en`, 384-dim, runs **entirely locally** — no evidence transmitted to an external API. Hard requirement for chain-of-custody / data sovereignty.
- **Storage**: `pgvector` `vector` column in the `chunk_summaries` table, alongside raw content and a metadata `JSONB` column.
- **Indexing**: IVFFlat index on embedding columns for sub-second Approximate Nearest Neighbour search at scale.

## Hybrid Retrieval — the "Two-Query Problem"

Standard architecture: one SQL query to filter (case_id, severity, level), one vector query to rank by similarity — two round-trips, often needing a separate vector DB (Pinecone/Weaviate).

FLARE's fix: pgvector lets both resolve in **one query**:
```sql
SELECT *, embedding <=> $1 AS distance
FROM chunk_summaries
WHERE case_id = $2 AND level = 3
ORDER BY distance LIMIT 50;
```
This is architecturally the paper's strongest single insight (Section XIV-E) — no separate vector DB infra, no data sovereignty concern from a second external system, single atomic transaction.

## Cross-Encoder Re-Ranking

Top-50 candidates from ANN search are re-ranked by a CrossEncoder (`ms-marco-MiniLM-L-6-v2`), which reads query+chunk jointly (vs. the bi-encoder's independent encoding at initial retrieval) — finer relevance judgment at higher latency. Top-10 survive to LLM context. Paper reports Precision@5 improving 0.74 → 0.89 from this stage alone.

**Retrieval latency breakdown (paper Table VI):**

| Stage | Avg. Latency |
|---|---|
| Hybrid pgvector query | 120ms |
| Semantic similarity scoring | 45ms |
| Keyword match (regex) | 15ms |
| Cross-Encoder re-ranking | 340ms |
| Context window population | 10ms |
| **Total** | **530ms** |

Re-ranking is the dominant cost — expected, given it's the highest-precision but highest-latency stage.

## Async Correctness at Scale

`asyncio.Semaphore(2)` bounds concurrent embedding batches — backpressures the upload stream without crashing the event loop under massive uploads (millions of log rows). This is the production-scale version of the fire-and-forget fix from the original 11-bug pass.

## Status: Corrected (11 bugs) + Evaluated

Bug fix detail: [[Bug Log - RAG Pipeline]]. Evaluation detail: see Project Hub's results table, sourced from paper Section XI-C/Table V.

## Chunk Size Tuning (empirical finding, not assumption)

500-token chunks were too coarse for precise citation; 50-token chunks lost local context. **50-row chunking** was the empirically-found balance between record-level precision and semantic meaningfulness — this replaces the earlier "confirm chunk boundaries are tuned" open item; it's now a settled, documented decision.

## Open Items

- [ ] Real-time/streaming ingestion — currently requires full ingestion before analysis (stated paper limitation)
- [ ] Graph-based cross-case correlation (stated future work, beyond current Pandas-sandboxed Statistical Agent scope)

## Related

- [[RAG]] — why RAG at all for this use case
- [[Types of RAG]]
- [[Embeddings & Vector DBs]]
- [[Decisions/Why Hierarchical RAG]]
- [[Decisions/Why Adaptive Chunking]]
- [[Multi-Agent System Design]] — how the Semantic Agent queries this pipeline's output
- [[Go Disk Acquisition Agent]] — the other evidence ingestion path
