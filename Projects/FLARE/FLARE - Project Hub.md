---
tags: [flare, project-hub, digital-forensics, active]
status: paper-drafted
completion: 65%
started: 
last-updated: 2026-08-20
---

# FLARE — Multi-Agent AI Platform for Automated Digital Forensic Investigation

A multi-agent AI platform that automates the full pipeline of digital forensic investigation — from raw evidence ingestion (disk images + log files) to court-admissible report generation. Built around **16 specialized agents across 4 pipelines**, a **Go-based disk acquisition engine**, and a **hierarchical (L0–L3) RAG pipeline** backed by pgvector.

**Overall completion: ~65%** · **Full IEEE draft exists** — see [[Research Paper - Overview]] and [[Paper Asset Index|the actual paper]].

## System Architecture (per the draft paper)

Four core pipelines, coordinated through a unified PostgreSQL + pgvector evidence store:

1. **RAG Query Engine** — Orchestrator → Semantic Agent (RAG) or Statistical Agent (Pandas codegen) → Validator Agent. 4 agents.
2. **Visual Report Pipeline (PIP)** — 4 parallel agents (Overview, Timeline Builder, Network/Protocol Analyzer, Threat Classification) + a Validator that merges outputs. 5 agents. Runs via `asyncio.gather()`: 34s sequential → 11s parallel (68% reduction).
3. **Report Generation Pipeline** — Technical Agent + Legal Analyzer (parallel) → Executive Agent → Formatting/Validator Agent. 4 agents. Produces versioned, court-admissible PDF.
4. **[[Go Disk Acquisition Agent|Disk Image Extractor]]** — Go imaging engine + 5 sequential agents inside a pinned Docker/TSK container (Overview, Partition Analyst, Data Carving, Metadata Extractor, Timeline). 5 agents (+ imaging engine).

Full agent-by-agent breakdown: [[Multi-Agent System Design]].

## Quick Links

- Architecture: [[Go Disk Acquisition Agent]] · [[RAG Ingestion Pipeline]] · [[Multi-Agent System Design]]
- Fixes so far: [[Bug Log - Go Agent]] · [[Bug Log - RAG Pipeline]]
- Decisions: [[Decisions/Why Go]] · [[Decisions/Why Hierarchical RAG]] · [[Decisions/Why Adaptive Chunking]] · [[Decisions/Why Multi-Agent Design]]
- Theory backing the design: [[RAG]] · [[Types of RAG]] · [[Embeddings & Vector DBs]] · [[Agentic AI]]
- Paper: [[Research Paper - Overview]] · [[Paper Review Notes]]
- Assets: [[Paper Asset Index]] · [[System Architecture Diagram]] · [[Hierarchical RAG Pipeline Diagram]]

## What's Been Accomplished

### Go Disk Acquisition Engine — functional, benchmarked
All five critical stubs debugged and fixed (see [[Bug Log - Go Agent]]). Confirmed on real Windows NVMe hardware with correct partition output.

Paper adds real performance numbers: **dual-stream goroutine architecture** (one goroutine writes the image, another hashes SHA-256 simultaneously off the same channel) achieves **0.4 GB/s** throughput on a 10GB image in 25.1s — vs 52.4s/0.19 GB/s for sequential Python and 41.2s/0.24 GB/s for multiprocessed Python. Cross-compiles to a single static binary (`ldflags="-s -w"`, <100MB constant memory via circular buffer) for Windows/`CreateFile()`+`FILE_FLAG_NO_BUFFERING`, macOS/`/dev/diskN`, Linux/`/dev/sda`+`O_DIRECT`.

### Hierarchical RAG Ingestion Pipeline — corrected, evaluated
`ingestion.py` fixed across 11 bugs (see [[Bug Log - RAG Pipeline]]). The paper formalizes the design as a **four-level (L0–L3) hierarchy**, not three:
- **L3** — raw chunks (50 rows each, finest granularity)
- **L2** — subsection summaries (1 per 250 raw rows / 5 L3 chunks)
- **L1** — section summaries (aggregated L2)
- **L0** — single root/document summary

Embedded with `BAAI/bge-small-en` (384-dim, fully local — no evidence leaves the machine, satisfying data sovereignty for forensic chain-of-custody). Stored in `pgvector` with a `parent_index` field forming a navigable tree. Retrieval is **hybrid**: one SQL query does vector distance + case_id/level filtering in a single round-trip (no separate vector DB needed) — the paper calls this solving the "Two-Query Problem." Top-50 candidates from ANN search are re-ranked by a CrossEncoder (`ms-marco-MiniLM-L-6-v2`) down to top-10 for LLM context.

**Measured results (paper Table V, 50 forensic queries, 2 independent analysts):**

| Metric | Flat RAG | FLARE Hierarchical | Change |
|---|---|---|---|
| Accuracy | 72% | 91% | +26.4% |
| Precision@5 | 0.68 | 0.89 | +30.8% |
| Recall@20 | 0.74 | 0.94 | +27.0% |
| Hallucination rate | 12% | 2% | −83.3% |

### Validator Agent — new since last notes
A dedicated hallucination-mitigation agent that runs after every LLM inference: checks that every cited `[Chunk N]` exists in the retrieved evidence set, runs an LLM grounding check for unsupported claims, and does a structural sanity check. Caught **8 hallucinated citations in 100 tested answers (8%)** in evaluation.

### Multi-Model Resilience Layer — new since last notes
Adaptive routing across Groq → Google Gemini → OpenAI on 403/429 errors, plus a tiered strategy (cheap/fast models for L3 summarization, Pro-tier models for final report synthesis) and periodic health-check heartbeats. Motivated by real Groq free-tier rate-limiting encountered during development.

### Knowledge Management
This Obsidian vault — tracking architecture, bugs, decisions, and the paper draft.

## Roadmap / What's On the Horizon

- [ ] Resolve paper issues before submission — see [[Paper Review Notes]] (duplication between Section III-C and Section IV, table numbering, code listing syntax, citation hedging, scope trim on Sections XVI–XVIII)
- [ ] Backfill any remaining bug-log detail for the RAG pipeline (currently summarized at category level, not bug-by-bug)
- [ ] Real-time/streaming ingestion (currently requires full ingestion before analysis — stated limitation in the paper)
- [ ] Multi-modal forensics: memory dumps (Volatility 3), packet captures (PCAP) — stated future work
- [ ] Complete remaining ~35% of FLARE development

## Key Learnings & Principles

- **Go on Windows PowerShell**: env vars use `$env:GOOS="windows"` syntax, not Linux-style `KEY=VALUE command`. When already on Windows, `go build -o flare-agent.exe .` is sufficient.
- **Adaptive sizing matters**: hardcoded chunk/embed batch sizes fail on edge-case file sizes; adaptive functions keyed to row count are more robust.
- **Async correctness**: fire-and-forget `asyncio` tasks silently die; all tasks must be awaited via `asyncio.gather()`. Paper confirms this pattern at scale: `asyncio.Semaphore(2)` bounds concurrent embedding batches to prevent resource exhaustion on large uploads.
- **Chunk size is a real tuning problem, not a guess**: paper reports 500-token chunks were too coarse for precise citation, 50-token chunks lost local context — 50-row chunking was the empirically chosen balance.
- **Never trust a single LLM provider in production**: real 403/429 rate-limit failures from Groq during dev drove the multi-model fallback design — treat the LLM provider as a replaceable commodity, not a fixed dependency.
- **Obsidian vault generation via bash tool**: brace expansion (e.g. `mkdir -p {a,b,c}`) is unreliable — use explicit separate `mkdir` calls per path, and verify with `ls`.

## Tools & Stack (per paper Table I)

| Component | Technology |
|---|---|
| Evidence store | Supabase, PostgreSQL, pgvector |
| Disk acquisition | Go (cross-compiled), TSK, Docker |
| Embedding model | BAAI/bge-small-en (384-dim, local) |
| Re-ranking | CrossEncoder (ms-marco-MiniLM-L-6-v2) |
| LLM backbone | Llama-4-Scout (+ Gemini/OpenAI fallback tier) |
| Frontend | React, Recharts |
| Report output | PDF (court-ready, versioned) |

Plus: Napkin.ai / draw.io / Figma for diagramming, IEEE as target venue.
