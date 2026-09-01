---
tags: [flare, decision]
status: confirmed-by-eval
---

# Decision: Why Adaptive Chunking

Part of [[FLARE - Project Hub|FLARE]]. Rationale for adaptive (vs hardcoded) chunk/embed batch sizing in the [[RAG Ingestion Pipeline]].

## Context
Hardcoded chunk/embed batch sizes broke on edge-case file sizes — very small forensic artifacts and very large disk images both stressed a fixed-size scheme differently.

## Reasoning
- Batch/chunk size computed adaptively, keyed to row count, rather than a fixed constant
- Keeps memory/throughput reasonable regardless of input scale, without manual tuning per dataset
- `asyncio.Semaphore(2)` bounds concurrent embedding batches — backpressures the upload stream without crashing the event loop, even at millions of log rows
- Directly ties into chunking theory — see [[Types of RAG]] for general tradeoffs between fixed-size, semantic/recursive, and document-type-aware chunking

## Confirmed by empirical tuning (paper Section VIII-C)
This was genuinely tested, not assumed: **500-token chunks were too coarse** for precise citation (evidence spread too thin per chunk), **50-token chunks lacked sufficient local context** (too fragmented for coherent narrative). **50-row chunking** (mapped to the L3 level of the [[Decisions/Why Hierarchical RAG|hierarchical RAG design]]) was the empirically-found balance between record-level precision and semantic meaningfulness.

## Trade-offs Accepted
- More complex sizing logic than a single constant
- Chunk-size tuning is dataset/domain-specific — the 50-row figure is right for this forensic-log use case, not necessarily transferable to other RAG domains without re-tuning

## Related
- [[RAG Ingestion Pipeline]]
- [[Bug Log - RAG Pipeline]]
- [[Decisions/Why Hierarchical RAG]]
