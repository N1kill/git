---
tags: [flare, bugs, python, rag]
component: rag-ingestion
---

# Bug Log — RAG Ingestion Pipeline

Part of [[FLARE - Project Hub|FLARE]]. 11 bugs identified and fixed in `ingestion.py`, part of the [[RAG Ingestion Pipeline]].

## Fix Categories

### Adaptive Sizing
Hardcoded chunk/embed batch sizes failed on edge-case file sizes. Replaced with adaptive functions keyed to row count, making the pipeline robust across small and very large inputs alike. See [[Types of RAG]] for the theory behind chunk sizing trade-offs.

### Async Task Handling
Fire-and-forget `asyncio` tasks were silently dying on failure with no error surfaced. Fixed by ensuring all tasks are properly awaited via `asyncio.gather()`, so exceptions propagate instead of vanishing.

### Semaphore Enforcement
Concurrency wasn't properly bounded, risking resource exhaustion (e.g. too many simultaneous embedding calls). Fixed with proper semaphore-gated concurrency control.

### Retry Logic
Transient failures (e.g. network hiccups on embedding/API calls) previously had no retry path. Added retry logic to handle these gracefully.

### Proportional Progress Tracking
Progress reporting wasn't accurately reflecting actual work completed. Fixed to track progress proportionally to real throughput.

*(Note: this captures the 5 fix categories at a summary level — backfill individual bug-by-bug detail here as they're revisited, per the project roadmap.)*

## Related

- [[RAG Ingestion Pipeline]]
- [[Bug Log - Go Agent]]
- [[RAG]]
