---
tags: [theory, ai, rnn, lstm, gru]
---

# RNNs, LSTM & GRU

Related: [[Neural Networks]] · [[Transformers]] · [[DL]]

## The core idea
Sequential data — text, time series, audio — has a property images and tabular data don't: order matters, and earlier elements can influence how later ones should be interpreted. An RNN handles this by processing a sequence one step at a time, maintaining a **hidden state** that gets updated at each step and carries information forward. The same weights are reused at every timestep, which is what lets an RNN handle sequences of any length with a fixed number of parameters.

The problem: information from early in a long sequence has to survive being squeezed through many rounds of hidden-state updates to still influence something far later. In practice, gradients flowing backward through many timesteps tend to vanish (see [[Neural Networks]]) — so plain RNNs are structurally bad at long-range dependencies, forgetting things that happened even moderately far back.

**LSTM and GRU exist specifically to fix this.** Both add explicit "gates" — small learned mechanisms that decide what to keep, what to discard, and what to update — giving the network an explicit way to preserve important information over long distances instead of relying on it surviving by accident through repeated multiplication.

## LSTM (Long Short-Term Memory)
Adds a separate **cell state** — think of it as a conveyor belt of long-term memory that runs alongside the hidden state — plus three gates that control it:
- **Forget gate**: decides what to discard from the cell state
- **Input gate**: decides what new information to add to the cell state
- **Output gate**: decides what part of the cell state to expose as this step's hidden state

Because the cell state can be preserved largely unchanged across many steps (the gates can learn to just pass it through), gradients have a much easier path backward — this is what actually solves the vanishing gradient problem, not just mitigates it.
- **Use when**: sequential data with meaningful long-range dependencies, at a scale where a full Transformer would be overkill — time-series forecasting, moderate-length sequence tasks, resource-constrained settings where Transformer-level compute isn't available or justified.

## GRU (Gated Recurrent Unit)
A simplified LSTM: merges the forget and input gates into a single **update gate**, and adds a **reset gate**, with no separate cell state.
- **Why it exists**: fewer parameters than LSTM, faster to train, and in practice performs comparably on many tasks.
- **Use when**: same situations as LSTM, but you want a lighter, faster model and are willing to trade a small amount of representational capacity for that — a common default when you're not sure LSTM's extra complexity is actually buying you anything.

## Where these stand today
Since Transformers (see [[Transformers]]) can look at every position in a sequence directly and in parallel — rather than being forced through a single hidden state step by step — they've become the default for most large-scale sequence modeling, especially language, where RNNs/LSTMs are rarely chosen from scratch anymore. LSTM/GRU still show up in smaller-scale or resource-constrained sequential tasks (embedded/on-device time-series models, smaller forecasting problems) where a Transformer's quadratic attention cost and larger parameter count aren't justified by the task's difficulty.
