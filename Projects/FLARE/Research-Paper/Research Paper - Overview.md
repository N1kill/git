---
tags: [flare, research-paper, ieee]
status: full-draft-exists
venue: IEEE
---

# Research Paper — Overview

Part of [[FLARE - Project Hub|FLARE]]. Full IEEE-format draft exists: **"Multi-Agent AI Platform for Automated Digital Forensic Investigation"** — Aluru Unnathi, Ande Tanmaya, Bhonagiri Sanjana, Afrah Sultana, Koyya Nikhil Sai Reddy — Dept. of CSE, Keshav Memorial Institute of Technology, Hyderabad. Reviewed 2026-08-20, see [[Paper Review Notes]] for fixes needed before submission.

## Section Status

- [x] **Abstract** — done. Leads with the four specialized pipelines, hybrid RAG, Go retrieval agent, hierarchical chunking, Validator Agent, and headline results.
- [x] **Introduction** — done. Frames the gap: existing tools (Autopsy/FTK/EnCase) are passive/analyst-driven; prior AI-in-forensics work is siloed (malware classifiers don't query logs, LLM summarizers don't validate outputs); no prior work unifies ingestion + semantic query + visualization + report generation with hallucination grounding.
- [x] **Related Work** — done. Covers traditional tools, ML for threat detection (Raff et al. malware-from-bytes, Ring et al. intrusion detection datasets), LLMs in cybersecurity (Ferrag et al.), RAG (Lewis et al.), and states the research gap explicitly.
- [x] **Methodology** — done, and extensive. System overview, dual-path ingestion (disk vs. log), disk acquisition/extraction pipeline, hierarchical RAG pipeline, full 16-agent breakdown (see [[Multi-Agent System Design]]), Go imaging engine deep-dive, database design, implementation details (async state machine, Docker/TSK integration, sandboxed statistical execution, frontend hydration).
- [x] **Results** — done. Disk imaging benchmark (Table IV), RAG accuracy comparison flat-vs-hierarchical (Table V), latency breakdown (Table VI), parallel agent latency reduction, statistical query reliability, Validator hallucination-catch rate. See needs-work items in [[Paper Review Notes]] #5 (methodology under-specification).
- [x] **Discussion / Limitations** — done. Covers legal admissibility framing (PACE 1984, FRE 901), stated limitations (no streaming ingestion yet, Pandas-only statistical sandbox, external LLM API restrictions in sensitive environments), human-in-the-loop philosophy.
- [x] **Conclusion** — done.
- [~] **Future Work / Global Impact / Ethics (Sections XVI–XVIII)** — drafted but flagged for scope trim, see [[Paper Review Notes]] #4.

## Source Material Backing Each Section

- Architecture detail: [[Go Disk Acquisition Agent]], [[RAG Ingestion Pipeline]], [[Multi-Agent System Design]]
- Bug/fix history (useful for an implementation-lessons subsection — paper already has this in Section VIII "Implementation Challenges and Lessons Learned"): [[Bug Log - Go Agent]], [[Bug Log - RAG Pipeline]]
- Design rationale (maps to Methodology justification): [[Decisions/Why Go]], [[Decisions/Why Hierarchical RAG]], [[Decisions/Why Adaptive Chunking]], [[Decisions/Why Multi-Agent Design]]

## Pre-Submission Checklist

See [[Paper Review Notes]] for full detail. Summary:
- [ ] Deduplicate Section III-C(2) vs Section IV-D (same 5 disk agents described twice)
- [ ] Fix table numbering (Table VI referenced for what is actually Table VII in Section XII-A)
- [ ] Fix Code Listing 2 syntax (missing parens — verify against source)
- [ ] Trim/reframe Sections XVI–XVIII (especially the "AIDE" standard proposal)
- [ ] Add trial-count/variance reporting to key evaluation numbers
- [ ] Hedge citation [1]'s $10.5T figure as an industry estimate, not established fact

## Related

- [[FLARE - Project Hub]]
- [[Paper Review Notes]]
