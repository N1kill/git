---
tags: [theory, ai, cache, inference]
---

# Caching in LLM Systems

Related: [[Transformers]] · [[LLM's]] · [[Agentic AI]]

## Intuition
Autoregressive generation (see [[LLM's]]) produces one token at a time, and each new token requires attending over every previous token in the sequence (see self-attention in [[Transformers]]). Naively, this means regenerating the Key and Value vectors for the *entire* sequence-so-far at every single new token — massively redundant, since the Keys/Values for earlier tokens never change once computed. The **KV cache** is the fix: compute Keys and Values for each token once, store them, and reuse them for every subsequent token's attention computation. This turns generation from roughly O(n²) recomputation into something closer to O(n) incremental work, and it's the single most important inference-level optimization that makes LLM serving practical at all.

At a higher level than a single model call, there's a second, distinct kind of caching that matters for LLM *applications*: **prompt caching** — many real workloads send the same large chunk of context (a system prompt, a long document, tool definitions) repeatedly across calls, with only a small part of the input actually changing each time. Providers let you cache the processed representation of that stable prefix, so subsequent calls skip reprocessing it entirely, cutting both cost and latency.

Both kinds of caching exploit the same underlying fact: a lot of what an LLM computes is redundant across time (KV cache) or across requests (prompt caching), and computation, unlike correctness, is something you're allowed to skip if you've already done it once.

## Reference

**KV Cache (inference-level)**
- During generation, each attention layer stores the Key and Value vectors it has already computed for all prior tokens
- At each new decoding step, only the new token's Q/K/V need to be computed; K/V for prior tokens are read from cache, not recomputed
- Memory cost scales with `sequence_length × num_layers × num_heads × head_dim × 2 (K and V)` — this, not raw parameter count, is often what limits how many concurrent requests a server can batch, and how long a context you can serve
- **Multi-Query Attention (MQA)** and **Grouped-Query Attention (GQA)**: reduce KV cache size by sharing Key/Value projections across multiple attention heads (instead of every head having its own) — a deliberate quality/memory tradeoff, GQA is the common middle ground used in most modern LLMs
- **PagedAttention** (used in vLLM and similar serving engines): manages KV cache memory in non-contiguous, page-like blocks (borrowing the idea from OS virtual memory) instead of requiring one contiguous memory allocation per sequence — dramatically reduces memory fragmentation/waste when serving many concurrent requests of different lengths

**Prompt Caching (application/API level)**
- Providers (Anthropic, OpenAI, etc.) let you mark a prefix of your prompt (system instructions, long reference documents, few-shot examples, tool definitions) as cacheable
- On a cache hit, the provider skips reprocessing that prefix's tokens through the model, serving from a stored intermediate state — much cheaper and faster than a full cache miss
- Cache entries typically have a short TTL (minutes) and are invalidated by any change earlier in the cached prefix — content must match exactly up to the cache breakpoint
- Design implication: put stable content (system prompt, long static context) *before* variable content (the actual user query) in your prompt structure, so the stable part can actually be cached and reused across calls

**Semantic caching (application-level, different from both above)**
- Cache full LLM *responses* keyed not by exact prompt match but by semantic similarity of the query (via embeddings, see [[Embeddings & Vector DBs]])
- If a new query is semantically close enough to a previously answered one, return the cached answer instead of calling the model again
- Riskier than the above two — two "similar" questions can have meaningfully different correct answers, so this needs a well-tuned similarity threshold and is generally used for high-volume, low-variance query types (FAQs, support bots) rather than general-purpose chat

**Why this matters for Agentic AI**
Agent loops (see [[Agentic AI]]) make many sequential LLM calls that typically share a large, unchanging prefix (system prompt, tool definitions, task description) with only the growing tool-call history changing. This makes agent workloads especially good candidates for prompt caching — without it, cost and latency scale badly as the agent's context grows turn over turn.
