---
tags: [flare, architecture, go]
component: disk-acquisition
status: functional-benchmarked
---

# Go Disk Acquisition Agent / Imaging Engine

Part of [[FLARE - Project Hub|FLARE]]. Forensic disk imaging front-end (`flare-agent`), feeding into the 5-agent Disk Image Extractor pipeline.

## Two Layers

1. **Imaging engine** — raw acquisition + hashing (this note's focus)
2. **5-agent extraction pipeline** — Overview, Partition Analyst, Data Carving, Metadata Extractor, Timeline — runs inside a pinned Docker/TSK container against the acquired image. Detailed in [[Multi-Agent System Design]] (Pipeline 4).

## Why Go: The Python GIL Problem

Two operations must happen simultaneously during acquisition: reading raw sectors and computing a SHA-256 hash for integrity. Python's GIL prevents true parallel execution of CPU-bound threads, forcing either sequential imaging-then-hashing (doubling time) or costly inter-process communication. Go's goroutines run on separate OS threads with no GIL-equivalent lock.

## Dual-Stream Architecture

1. A single `ReadAt()` call fetches a block of sectors.
2. Bytes pass through a Go channel to two goroutines.
3. Goroutine A writes bytes to the target image file.
4. Goroutine B simultaneously feeds the same bytes into a SHA-256 accumulator.

Hashing happens at the speed of disk I/O rather than as a second pass.

## Measured Performance (paper Table IV)

| Tool/Language | Data Size | Total Time | Throughput |
|---|---|---|---|
| Python (sequential) | 10 GB | 52.4s | 0.19 GB/s |
| Python (multiprocessing) | 10 GB | 41.2s | 0.24 GB/s |
| **FLARE (Go)** | 10 GB | **25.1s** | **0.40 GB/s** |

~50% reduction in total acquisition time vs. sequential Python, by eliminating the "second-pass penalty."

## Cross-Platform Sector Access
Single source, cross-compiled to platform-native binaries:
- **Windows**: `CreateFile()` with `FILE_FLAG_NO_BUFFERING` + `FILE_FLAG_SEQUENTIAL_SCAN`, bypassing system cache for direct sector-aligned reads from `\\.\PhysicalDriveN`.
- **Linux**: direct block device access (`/dev/sda`) with `O_DIRECT` where supported.
- **macOS**: `/dev/diskN`, handling APFS + System Integrity Protection privilege requirements.

## Integrity & Footprint
- SHA-256 computed in a single pass, recorded to `activity_log` alongside case metadata immediately (chain-of-custody baseline — before any analysis begins).
- Statically compiled (`ldflags="-s -w"`), zero external dependencies — runs from a write-protected USB without touching the target system.
- Custom circular buffer keeps memory usage constant (<100MB) regardless of disk size — important so the tool itself doesn't overwrite volatile evidence in target RAM.
- Output format: `.E01` (preferred for court — embeds hash/metadata in-container) or `.DD`.

## Status: Functional & Benchmarked
All five critical stubs debugged and fixed — see [[Bug Log - Go Agent]]. Confirmed on real Windows NVMe hardware with correct partition enumeration, and now has a real throughput benchmark against Python baselines.

## Open Items

- [ ] FPGA-based line-rate hashing for NVMe/SAN storage requiring 5–10 GB/s (stated future work — current 0.4 GB/s is SATA-appropriate, not NVMe-saturating)
- [ ] Quantum-resistant hashing (SPHINCS+) for long-term evidentiary integrity (stated future work, low near-term priority)
- [ ] "Edge Forensic Workers" — deployable local acquisition units uploading only L1/L2 summaries to cut bandwidth by >95% (stated future work)

## Related

- [[RAG Ingestion Pipeline]] — consumes output from this agent
- [[Decisions/Why Go]] — rationale for choosing Go for this component
- [[Multi-Agent System Design]] — Pipeline 4, the 5-agent extraction layer this engine feeds
