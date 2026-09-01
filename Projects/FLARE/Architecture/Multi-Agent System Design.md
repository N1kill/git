---
tags: [flare, architecture, agents, multi-agent]
component: orchestration
status: designed-in-paper
---

# Multi-Agent System Design

Part of [[FLARE - Project Hub|FLARE]]. Full breakdown of the **16 specialized agents across 4 pipelines**, per the paper's Section IV. Each agent follows a deterministic/probabilistic hybrid: `[CODE]` for computationally intensive/deterministic steps, `[LLM]` for semantic reasoning. Every agent communicates via a standardized JSON envelope.

General theory backing this design: [[Agentic AI]]. FLARE-specific rationale: [[Decisions/Why Multi-Agent Design]].

---

## Pipeline 1 — RAG Query Engine (4 agents)

The primary analyst-facing interface.

**1.1 Orchestrator Agent** — routing brain. `[LLM]` validates query relevance/safety (<50 tokens, <200ms), classifies as "semantic" vs "statistical", `[CODE]` dispatches accordingly (hybrid queries run both via `asyncio.gather()`). Defaults to semantic on ambiguous routing — safer graceful failure than a code-exec crash. Every query logged to `activity_log`.

**1.2 Semantic Agent** — narrative answers to contextual queries (attack patterns, lateral movement). `[CODE]` embeds query locally, runs **three-path parallel retrieval**: (A) ANN on L3 raw chunks, (B) ANN on L1/L2 summaries, (C) regex extraction of IPs/usernames → exact SQL `ILIKE`. `[MODEL]` CrossEncoder re-ranks to top-20 (Precision@5: 0.74→0.89). `[CODE]` assembles hierarchical context window (L3 doc summary → L2 → L1 → L0 raw chunks — big picture first). `[LLM]` generates answer citing `[Chunk N]`, instructed to never fabricate.

**1.3 Statistical Agent** — exact counts/averages/aggregations via sandboxed Pandas codegen, zero estimation. `[CODE]` loads evidence CSV, auto-detects timestamp columns. `[LLM]` generates Pandas code from query + schema + 3 sample rows, assigns to `FINAL_RESULT`. `[CODE]` executes in a restricted namespace (only `df`, `pd`, `np`; no `open`/`os`/`pathlib`), 30s timeout. On `KeyError` (wrong column name), single retry with actual column list — 85% first-pass → 95% with retry.

**1.4 Validator Agent** — quality gate after every response. `[CODE]` citation check (every `[Chunk N]` must exist in retrieved set). `[LLM]` grounding check (any claim unsupported by retrieved chunks?). `[CODE]` structural check (non-empty, valid UTF-8, no stack traces, reasonable length). Assigns confidence 0–1; failed checks flagged "uncertain" with reasons shown to analyst. **Measured: caught 8/100 hallucinated chunk citations (8%) in testing.**

---

## Pipeline 2 — Visual Report Pipeline / PIP (5 agents)

Four agents run **simultaneously** via `asyncio.gather()`, pushing JSON to the frontend over WebSocket as each completes. **Sequential: 34s → Parallel: 11s (68% reduction).** Dashboard hydrates progressively.

**2.1 Overview Agent** — first-pass stats: total events, severity breakdown, anti-forensics activity count, 2-sentence plain-English summary. `[CODE]` builds bar/pie chart data via `value_counts()`. `[LLM]` writes the summary.

**2.2 Timeline Builder Agent** — hourly event frequency + cumulative progression charts, with anomaly highlighting. `[CODE]` parses timestamps (multi-format fallback: ISO8601, Unix epoch, custom), groups by hour, flags any hour exceeding mean+2σ as anomalous. `[LLM]` writes 1–2 sentence narrative on peak timing pattern (burst/sustained/escalating).

**2.3 Network and Protocol Analyzer Agent** — protocol/source-IP/connection analysis. `[CODE]` identifies network columns by name pattern, classifies IPs (private/Tor-exit-lookup/external — external or Tor as source auto-flagged suspicious), builds protocol/IP charts + ranked connection table. `[LLM]` writes insight on what the distribution/concentration reveals.

**2.4 Threat Classification Agent** — maps evidence to **MITRE ATT&CK** framework. `[CODE]` loads L2 chunk summaries. `[LLM]` classifies each against 11 MITRE tactics with confidence 0–1. `[CODE]` builds radar chart data. `[LLM]` writes threat assessment paragraph on attacker sophistication.

**2.5 Validator Agent (PIP)** — merges all 4 parallel outputs via `asyncio.gather(return_exceptions=True)`, validates schema per section, fills safe defaults for failed sections (`{"data": [], "summary": "Data unavailable"}` — prevents frontend crash), assembles final payload + top-50 highest-severity event table.

---

## Pipeline 3 — Report Generation (4 agents)

Technical + Legal run in parallel; Executive reads both; Formatting/Validator assembles the final PDF.

**3.1 Technical Agent** — core forensic findings: timestamped key findings with chunk citations, evidence inventory (E-001, E-002...), chronological attack timeline. `[CODE]` reads only L1/L2/L3 summaries (not raw chunks — forces structured reasoning). `[LLM]` generates findings/evidence/timeline, instructed never to fabricate timestamps/IPs/usernames.

**3.2 Legal Analyzer Agent** — statute-referencing legal analysis for prosecutors/defence/judges. `[LLM]` identifies applicable statutes by jurisdiction (UK Computer Misuse Act 1990, US CFAA 18 U.S.C. §1030, EU Directive 2013/40/EU), maps each finding to the specific legal element it satisfies (cited `[Chunk N]`), assesses evidentiary strength (strong/circumstantial/insufficient), writes exactly a 6-sentence formal paragraph using "evidence establishes" / "is consistent with" — never "proves guilt."

**3.3 Executive Agent** — C-suite summary, jargon-free, <5 min read. `[CODE]` reads doc_summary + Legal Analyzer's first paragraph + Technical Agent's findings for consistency. `[LLM]` generates 3-paragraph narrative, 5-sentence remediation plan, severity rating (CRITICAL/HIGH/MEDIUM/LOW), case title. Consistency check cross-verifies dates/names/scope against Technical output — discrepancies trigger regeneration.

**3.4 Formatting and Validator Agent** — final assembly. `[CODE]` validates every required schema field across all 3 prior agents' outputs, fills missing from global case stats (no empty PDF sections). Assembles PDF: cover page → executive summary → technical findings → legal analysis → evidence inventory → attack timeline → chain-of-custody certificate. Inserts `reports` table row with `model_used`, incrementing `version`, `report_status` lifecycle ("draft" → "finalized", finalized reports locked from modification).

---

## Pipeline 4 — Disk Image Extractor (5 agents, sequential)

Runs inside a Docker container pinned to a specific TSK version — guarantees byte-identical output across host OSes (legal admissibility requirement). Each agent writes structured JSON to `disk_artifacts` before the next begins.

**4.1 Disk Overview Agent** — integrity + format ID. `[CODE]` computes SHA-256, compares against `expected_hash` (mismatch → halt pipeline, `status = "hash_failed"`, alert analyst). Detects format via magic bytes (EVF/E01, raw DD, KDMV/VMDK, conectix/VHD). `[CODE+TSK]` runs `mmls`/`fsstat` for partition table + volume stats. `[LLM]` 2-sentence summary of system type + forensic state.

**4.2 Partition Analyst Agent** — deep partition mapping incl. hidden/deleted/gap partitions, anti-forensic detection. `[CODE+TSK]` `mmls -aB -t dos` for full byte-offset partition list. `[CODE]` classifies each slot (allocated/unallocated/deleted/gap), detects partition overlap, HPA, DCO, VBR/MBR/GPT mismatches as `suspicious_finding`. `[CODE+TSK]` `fsstat` per partition for internal consistency (`fs_healthy: false` → Data Carving becomes essential).

**4.3 Data Carving Agent** — recovers deleted files from unallocated sectors by magic-byte signature scan, independent of file system. `[CODE]` loads signature library (JPG/PDF/DOCX/EXE/ZIP/SQLite/PNG/MP4 supported), memory-maps the disk image (232GB image scanned without full RAM load), scans each unallocated range. Type-specific validation (PIL for JPG, `%%EOF`/`/Page` for PDF, PE header for EXE, magic+page-size for SQLite). SHA-256 per recovered file + embedded metadata extraction (EXIF GPS/device, PDF Author/CreationDate, EXE compile timestamp, SQLite schema).

**4.4 Metadata Extractor Agent** — per-file inode metadata incl. **timestomping detection**. `[CODE+TSK]` `ils` (incl. deleted inodes) + `fls` for full paths. Extracts 4 MAC timestamps (mtime/atime/ctime/crtime). Timestomping check: `$STANDARD_INFORMATION` mtime predating `$FILE_NAME` mtime by >1 year → `timestomp_suspected: true`. Records permissions/ownership/link count/NTFS attributes.

**4.5 Timeline Agent** — unified chronological event sequence + narrative reconstruction. `[CODE+TSK]` `mactime` sorts all MAC events into flat CSV. `[CODE]` enriches each row: IOC list match → `known_malicious`, sensitive path match (System32, /etc/, startup, email stores) → `sensitive_path`, timestomping flags carried over. **Burst window detection**: 60s windows exceeding mean+2σ event density flagged. **Significance scoring** (max 10): base 2, +3 known malicious, +2 sensitive path, +2 deleted, +1 burst window, +1 timestomped. `[LLM]` sends top-20 highest-scored events for a 200–400 word attack narrative (phases: Recon → Initial Access → Execution → Persistence → Exfiltration → Anti-Forensics). Top-500 events ingested into `chunk_summaries` (L3) for RAG Query Engine access.

---

## Representative Agent Summary (paper Table II)

| Pipeline | Agent | Output | Logic |
|---|---|---|---|
| Query Engine | Orchestrator | Routing signal | LLM |
| Query Engine | Semantic | Narrative answer | LLM (RAG) |
| Query Engine | Statistical | Numerical result | CODE |
| Query Engine | Validator | Grounding status | CODE+LLM |
| Visual PIP | Timeline | Temporal JSON | CODE+LLM |
| Visual PIP | Threat | Radar chart | LLM |
| Report Gen | Technical | Findings list | CODE |
| Report Gen | Legal | Statute mapping | LLM |
| Disk Extr. | Partition | Partition map | CODE |
| Disk Extr. | Timeline | Unified timeline | CODE+LLM |

## Multi-Model Resilience Layer (cross-cutting, not a 5th pipeline)

Not one of the 16 pipeline agents, but a system-wide layer motivated by real Groq 403/429 errors hit during development:
- **Adaptive routing**: on non-retryable error from primary provider (Groq), falls back to secondary (Google Gemini) transparently to agent logic.
- **Tiered reasoning**: high-criticality tasks (report synthesis) → Pro-tier models (Gemini-1.5-Pro); low-complexity tasks (L3 summarization) → fast/cheap models (Llama-3-8B, Gemini Flash).
- **Health-check heartbeats**: background periodic checks cached in-memory, so routing decisions use recent availability data rather than waiting for a live failure — reduces "first-failure latency."
- Reported target: 99.9% orchestration-layer uptime.

## Related

- [[Go Disk Acquisition Agent]] — imaging engine feeding Pipeline 4
- [[RAG Ingestion Pipeline]] — L0–L3 hierarchy queried by Semantic Agent
- [[Decisions/Why Multi-Agent Design]]
- [[Agentic AI]] — general multi-agent theory
