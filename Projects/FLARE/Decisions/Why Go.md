---
tags: [flare, decision]
status: confirmed-by-eval
---

# Decision: Why Go for the Disk Acquisition Agent

Part of [[FLARE - Project Hub|FLARE]]. Rationale for choosing Go to build the [[Go Disk Acquisition Agent]].

## Context
Two operations must run truly simultaneously during forensic disk acquisition: reading raw sectors from the drive and computing a SHA-256 hash for chain-of-custody integrity. This is a CPU-bound-plus-I/O-bound parallelism problem.

## Reasoning
- **Python's GIL is the specific, named blocker** (not just "Python is slow" — the paper is precise about this): the Global Interpreter Lock prevents true parallel execution of CPU-bound threads, so a Python imaging script must either run sectors-then-hash sequentially (doubling time) or pay significant inter-process communication overhead to fake parallelism.
- Go's goroutines run on separate OS threads with no GIL-equivalent lock — genuine parallel acquisition across CPU cores.
- **Dual-stream architecture**: single `ReadAt()` fetches a sector block → passed via channel to two goroutines → one writes to the image file, one feeds SHA-256 simultaneously. Hashing happens at disk I/O speed, not as a second pass.
- Single static binary, cross-compiled from one source to Windows/macOS/Linux — critical for forensic triage tooling that needs to run from a write-protected USB without touching the target system's libraries/registry.
- Constant, predictable memory footprint (<100MB via custom circular buffer) regardless of disk size — matters because the tool itself must not overwrite volatile evidence in the target machine's RAM.

## Confirmed by evaluation (paper Table IV — this decision is no longer speculative)

| Tool/Language | Data Size | Total Time | Throughput |
|---|---|---|---|
| Python (sequential) | 10 GB | 52.4s | 0.19 GB/s |
| Python (multiprocessing) | 10 GB | 41.2s | 0.24 GB/s |
| **FLARE (Go)** | 10 GB | **25.1s** | **0.40 GB/s** |

~50% reduction in total acquisition time vs. sequential Python — directly attributable to eliminating the "second-pass penalty" the GIL forces on Python.

## Trade-offs Accepted
- Smaller low-level forensics library ecosystem than C/C++ (no direct equivalent to some mature C forensic libraries) — mitigated by leaning on Docker-contained TSK (C++) for the deeper extraction work, and keeping Go scoped to acquisition + hashing only
- OS-specific implementation branches required per platform (Windows `CreateFile()`+`FILE_FLAG_NO_BUFFERING`, macOS `/dev/diskN`+SIP handling, Linux `/dev/sda`+`O_DIRECT`) — more platform-aware code than a higher-level language would need, but necessary for the direct sector-level access forensic acquisition requires
- Current throughput (0.4 GB/s) is well-suited to SATA drives but under-saturates high-speed NVMe/SAN storage (5–10 GB/s) — stated as future work requiring FPGA-based hashing acceleration

## Related
- [[Go Disk Acquisition Agent]]
- [[Decisions/Why Multi-Agent Design]]
- [[Paper Review Notes]]
