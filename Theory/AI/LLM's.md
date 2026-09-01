---
tags: [theory, ai, llm, foundations]
---

# LLMs — Large Language Models

Related: [[Transformers]] · [[NLP Basics]] · [[Fine-tuning & Alignment]] · [[Prompt Engineering]] · [[RAG]] · [[Agentic AI]]

## Intuition
An LLM is, at its core, a decoder-only Transformer (see [[Transformers]]) trained on an enormous amount of text to do one deceptively simple thing: **predict the next token**, given all previous tokens. That's it. Everything an LLM appears to "know" or "reason about" is a byproduct of getting extremely good at that one objective across trillions of tokens of human-written text — language, facts, code, reasoning patterns, argument structure all get compressed into the model's weights because predicting the next token well *requires* implicitly modeling all of that.

Training happens in stages because "predict the next token on random internet text" and "be a helpful, safe assistant that follows instructions" are different objectives:
1. **Pretraining** — next-token prediction on a massive, broad corpus. This is where the model gets its raw capability and knowledge. Extremely compute-expensive, done once by a handful of labs.
2. **Supervised Fine-Tuning (SFT)** — train on curated (instruction, ideal response) pairs so the base model, which just completes text, learns to behave like an assistant that follows instructions.
3. **Alignment (RLHF / DPO)** — further tune the model's behavior using human preference data, so it's not just capable but also helpful, honest, and safe by some standard. See [[Fine-tuning & Alignment]].

A crucial mental model: the LLM has no persistent memory or "self" between calls. Every response is generated fresh from the current context window — this is why context management, RAG, and prompt engineering matter so much operationally: the model only "knows" what's currently in its input.

**Scaling laws** are the empirical finding that loss decreases smoothly and predictably as you scale up model size, data, and compute together — this predictability (not any single architectural trick) is largely *why* the field bet on "just make it bigger" and it kept working, up to the limits currently being explored.

## Reference

**Tokenization**
- Text is split into subword tokens (not full words, not characters) — handles rare words, typos, and multiple languages without an unbounded vocabulary
- **BPE (Byte-Pair Encoding)**: iteratively merges the most frequent adjacent character/subword pairs to build a vocabulary
- Vocabulary size is typically 30k–200k tokens; roughly 1 token ≈ 0.75 English words

**Training objective**
- Next-token prediction: `P(xₜ | x₁, x₂, ..., xₜ₋₁)` — Cross-Entropy loss between predicted distribution and actual next token
- **Autoregressive generation**: at inference, sample the next token from this distribution, append it, repeat

**Sampling / decoding strategies**
- **Greedy**: always pick highest-probability token — deterministic, often repetitive/boring
- **Temperature**: scales logits before softmax (`logits/T`) — higher T = more random/creative, lower T = more focused/deterministic
- **Top-k sampling**: sample only from the k highest-probability tokens
- **Top-p (nucleus) sampling**: sample from the smallest set of tokens whose cumulative probability ≥ p — adapts set size to the model's confidence
- **Beam search**: track multiple candidate sequences in parallel, keep the best-scoring ones — more common in translation than open-ended chat

**Scaling laws**
- Loss decreases as a power law with model size, dataset size, and compute (Kaplan et al., then refined by Chinchilla/Hoffmann et al.)
- **Chinchilla-optimal**: for a fixed compute budget, there's an optimal ratio of model size to training tokens — earlier models (GPT-3) were undertrained relative to their size; modern training uses far more tokens per parameter
- Emergent capabilities: some abilities (multi-step arithmetic, certain reasoning) appear to show up somewhat abruptly at scale rather than improving smoothly — an active area of debate on whether this is a real phenomenon or a measurement artifact

**Context window**
- The maximum number of tokens (input + output) the model can attend to at once
- Bounded by the O(n²) cost of self-attention (see [[Transformers]]) and by what the model was actually trained/extended to handle well
- "Lost in the middle": empirically, models tend to attend better to the start and end of a long context than the middle — relevant when stuffing lots of retrieved content into a prompt (see [[RAG]])

**Post-training / alignment (see [[Fine-tuning & Alignment]] for detail)**
- SFT: supervised learning on instruction-response pairs
- Reward Modeling: train a separate model to score responses by human preference
- RLHF: use the reward model as a signal to fine-tune the LLM via RL (originally PPO)
- DPO (Direct Preference Optimization): skips the separate reward model + RL loop, directly optimizes the policy on preference pairs — simpler and more stable, increasingly the default over classic RLHF

**Hallucination**
The model generates fluent, confident-sounding text that is factually wrong or unsupported. Root cause: the model is optimized to produce plausible continuations, not verified truth — it has no built-in mechanism to "know that it doesn't know." Mitigations: RAG (ground responses in retrieved facts), better alignment training, uncertainty calibration, tool use for verification.

**Key model families to know by name**: GPT (OpenAI), Claude (Anthropic), Llama (Meta), Gemini (Google), Mistral — differ in architecture details, training data, and alignment approach, but all are fundamentally decoder-only Transformers at massive scale.

**Multimodal LLMs**: extend the same architecture to accept image/audio/video tokens alongside text tokens (typically via a separate vision/audio encoder that projects into the same embedding space the Transformer operates on).
