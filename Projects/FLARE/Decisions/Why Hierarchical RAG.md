---
tags: [flare, decision]
status: confirmed-by-eval
---

# Decision: Why Hierarchical RAG

Part of [[FLARE - Project Hub|FLARE]]. Rationale for the **L0–L3** hierarchical RAG design (corrected from an earlier draft of these notes, which used L1–L3) in the [[RAG Ingestion Pipeline]], instead of flat/single-level retrieval.

## Context
Forensic logs vary hugely in structure and query granularity: an investigator might ask "what happened at 03:14:22" (record-level) or "summarize all authentication failures" (document-level). A flat chunking scheme forces one granularity for both.

## Reasoning
- **L3** — raw chunks (50 rows) for precise, timestamp-level evidence lookup
- **L2** — subsection summaries (1 per 250 rows) for mid-level contextual grouping
- **L1** — section summaries, aggregated from L2
- **L0** — single document-level summary for high-level investigative questions, generated first (head/tail sampling) to seed context for the rest of the tree
- Broad queries resolve at L0/L1; specific evidential queries drill to L3 with precise source citations
- Theory backing: [[Types of RAG]], [[RAG]]

## Confirmed by evaluation (paper Table V — this decision is no longer speculative)

| Metric | Flat RAG | Hierarchical | Change |
|---|---|---|---|
| Accuracy | 72% | 91% | +26.4% |
| Precision@5 | 0.68 | 0.89 | +30.8% |
| Recall@20 | 0.74 | 0.94 | +27.0% |
| Hallucination | 12% | 2% | −83.3% |

Flat RAG performed fine on simple entity lookups (e.g. "find all instances of IP X") but fragmented on "big picture" questions where evidence spans multiple non-contiguous chunks — attack-progression queries ("how did the intruder move from web server to database") showed a 40% higher success rate under the hierarchical design. The hallucination drop is attributed to summary nodes acting as a stable narrative anchor, preventing the model from "filling gaps" between isolated chunks.

## Trade-offs Accepted
- Higher indexing complexity: recursive task spawning (L3 → Section Monitor → L2 → L1 → L0) vs. a single flat embedding pass
- Higher storage overhead (summary nodes at 3 extra levels, each separately embedded)
- Chunk-size tuning required real iteration: 500-token chunks were too coarse for citation, 50-token chunks lost context — 50-row chunking was the empirically-found balance (see Section VIII-C of the paper)

## Related
- [[RAG Ingestion Pipeline]]
- [[Decisions/Why Adaptive Chunking]]
- [[Paper Review Notes]]
