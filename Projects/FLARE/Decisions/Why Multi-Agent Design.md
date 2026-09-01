---
tags: [flare, decision]
status: confirmed-by-eval
---

# Decision: Why Multi-Agent Design

Part of [[FLARE - Project Hub|FLARE]]. Rationale for a multi-agent orchestration layer — **16 agents across 4 pipelines** in the finished design (see [[Multi-Agent System Design]]) — rather than a single monolithic agent/prompt.

## Context
Forensic investigation involves distinct cognitive steps — routing, evidence retrieval, numerical analysis, hallucination checking, visualization, legal mapping, executive summarization — each with different failure modes and different verification needs. A single agent doing all of this has no natural checkpoint for catching errors before they reach a court-admissible report.

## Reasoning (as actually implemented)
- **Specialization by task type**: Orchestrator (routing) is separate from Semantic (RAG narrative) is separate from Statistical (sandboxed Pandas codegen) is separate from Validator (grounding check) — each independently promptable, testable, and constrained. Statistical queries get zero-hallucination exact numbers via code execution rather than LLM arithmetic.
- **Parallelism where independent**: the 4 Visual PIP agents (Overview, Timeline, Network, Threat) run simultaneously via `asyncio.gather()` because they don't depend on each other — measured 68% latency reduction (34s → 11s) is a direct, quantified payoff of this design choice, not just an architectural nicety.
- **Sequential where dependent**: the 5 Disk Extractor agents run strictly sequentially (Overview → Partition → Carving → Metadata → Timeline) because each needs the prior agent's output (e.g. Carving needs Partition's unallocated-sector map).
- **Separating reasoning from report generation** (Technical/Legal agents → Executive → Formatting/Validator) creates an explicit checkpoint: technical findings and legal mapping can be cross-verified for consistency before becoming a formal document — matters directly for court-admissibility.
- **A dedicated Validator Agent per pipeline** (Query Engine Validator, PIP Validator, Report Formatting/Validator) rather than one global validator — each catches structurally different failure modes (hallucinated citations vs. malformed dashboard JSON vs. incomplete report schema).
- Matches the RAG granularity from [[Decisions/Why Hierarchical RAG]]: different agents query different levels (Technical Agent reads L1/L2/L3 summaries only, not raw chunks, forcing structured reasoning from pre-synthesized evidence).

## Confirmed by evaluation
- Visual PIP parallelism: 68% latency reduction, measured not assumed.
- Validator Agent: caught 8/100 hallucinated citations in testing — a concrete failure-catching rate, not a theoretical safeguard.
- Statistical Agent's code-execution approach: 85% first-pass success, 95% with single retry-on-KeyError — validates the "code for numbers, LLM for narrative" split as more reliable than open-ended LLM arithmetic.

## Trade-offs Accepted
- Real orchestration complexity: 16 agents each need a standardized JSON envelope, error handling, and (for LLM-backed agents) provider fallback
- Latency cost: even with parallelism, the Semantic Agent path alone has 530ms end-to-end retrieval latency (Table VI) before LLM generation even starts
- Cost: multiple LLM calls per investigation — mitigated by the tiered-model resilience layer (cheap/fast models for low-stakes steps like L3 summarization, Pro-tier only for report synthesis)
- Coordination failure modes: addressed via `asyncio.gather(return_exceptions=True)` + safe-default fallback in the PIP Validator, rather than assumed away

## Related
- [[Multi-Agent System Design]]
- [[Decisions/Why Go]]
- [[Decisions/Why Hierarchical RAG]]
- [[Paper Review Notes]]
