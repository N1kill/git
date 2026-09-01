---
tags: [theory, ai, transformers, foundations]
---

# Transformers

Related: [[DL]] · [[Neural Networks]] · [[LLM's]] · [[Embeddings & Vector DBs]]

## Intuition
Before Transformers, sequence models (RNN/LSTM) processed tokens one at a time, in order — token 50 couldn't be computed until tokens 1-49 were done. This was slow (no parallelization across sequence position) and struggled with long-range dependencies (information from token 1 has to survive being squeezed through 49 hidden-state updates to influence token 50).

The Transformer's core idea (the 2017 "Attention Is All You Need" paper): instead of passing information step by step through a hidden state, let **every token look directly at every other token** in one shot, and learn how much attention to pay to each. This is **self-attention**. It's fully parallelizable (huge speed win on GPUs) and gives direct paths between any two positions regardless of distance (no long-range decay problem).

The mechanism: for every token, generate a **Query** (what am I looking for), a **Key** (what do I contain, for others to find), and a **Value** (what I actually offer if selected). A token's new representation is a weighted sum of all tokens' Values, where the weights come from how well its Query matches every token's Key. This is literally a soft, differentiable lookup table.

Because attention has no inherent sense of order (it's a set operation over tokens), **positional encoding** is added so the model knows token 3 came before token 7. And because one attention pattern can't capture every kind of relationship (syntax vs. coreference vs. topic), **multi-head attention** runs several attention operations in parallel, each free to specialize.

## Reference

**Self-attention, the math**
- `Q = XWq`, `K = XWk`, `V = XWv`
- `Attention(Q,K,V) = softmax(QKᵀ / √dₖ) V`
- `√dₖ` scaling prevents dot products from growing too large and pushing softmax into near-zero-gradient regions
- Output is a weighted combination of Value vectors, weights determined by Query-Key similarity

**Multi-head attention**
- Run h independent attention computations in parallel, each with its own Wq, Wk, Wv (smaller dimension each)
- Concatenate outputs, project back to model dimension
- Lets different heads specialize (one might track syntax, another long-range coreference, etc.)

**Positional encoding**
- Sinusoidal (original paper): fixed functions of position, added to token embeddings
- Learned positional embeddings: trained like any other parameter
- **RoPE** (Rotary Position Embedding): rotates Q/K vectors by an angle proportional to position — current standard in most modern LLMs (Llama, GPT-NeoX-style), generalizes better to longer sequences than absolute encodings

**Full block structure**
Each Transformer layer = Multi-Head Attention → Add & LayerNorm (residual connection) → Feedforward (two linear layers with nonlinearity in between) → Add & LayerNorm
- Residual connections are critical — without them, gradients can't flow cleanly through many stacked layers (same vanishing-gradient issue as any deep net)
- **Pre-LN vs Post-LN**: whether LayerNorm happens before or after the sub-layer — Pre-LN (norm first) trains more stably at scale, is the modern default

**Encoder vs Decoder vs Encoder-Decoder**
| Type | Attention | Use case | Examples |
|---|---|---|---|
| Encoder-only | Bidirectional (sees full context both directions) | Understanding tasks — classification, embeddings | BERT |
| Decoder-only | Causal/masked (each token only sees previous tokens) | Generation — this is what modern LLMs use | GPT, Llama, Claude |
| Encoder-Decoder | Encoder bidirectional, decoder causal + cross-attends to encoder | Seq2seq tasks — translation, summarization | T5, original Transformer, BART |

**Causal masking**
Decoder-only models mask future tokens in the attention computation (set their scores to -∞ before softmax) so token i can only attend to tokens ≤ i — necessary for autoregressive generation (can't cheat by looking ahead during training).

**Complexity**
- Self-attention is O(n²) in sequence length n — every token attends to every other token
- This is why long-context is expensive/hard, and why techniques like sparse attention, sliding-window attention, and linear-attention variants exist to reduce the quadratic cost
- KV cache exists specifically to avoid recomputing this at every generation step — see [[Cache]]

**Why Transformers scale so well**
Fully parallel training (no sequential dependency like RNNs) + the architecture's performance improves smoothly and predictably with more data/parameters/compute (see scaling laws in [[LLM's]]) — this combination is why Transformers became the default architecture for essentially all large-scale sequence modeling, not just language.
