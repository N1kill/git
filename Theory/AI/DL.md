---
tags: [theory, ai, dl, foundations]
---

# Deep Learning — Training & Optimization

Related: [[Neural Networks]] · [[ML]] · [[CNNs & Computer Vision]] · [[RNNs, LSTM & GRU]] · [[Transformers]]

## The core idea
DL is ML where the model is a deep stack of neural network layers (see [[Neural Networks]] for what a layer actually is, and the architecture files linked above for *which* structure to stack). What makes DL its own topic, separate from "neural networks" as a concept, is everything involved in actually getting a deep network to train well — because depth doesn't come for free. A deeper network has more expressive power, but also more ways for training to go wrong: gradients dying or exploding on their way backward through many layers, training becoming unstable, or the model simply needing far more data than a shallow one to avoid overfitting.

This file is about that training machinery: what function each neuron uses to introduce nonlinearity, how the optimizer actually moves the weights toward better values, and what keeps a deep network's training numerically stable. The architectural question — CNN vs RNN vs Transformer vs something else — is covered in each architecture's own file, since which structure to use is a different decision from how to train it once chosen.

## Activation functions — why the choice isn't arbitrary
Every activation function is solving the same problem (introduce nonlinearity) with different tradeoffs in *how* it does that:
- **Sigmoid / Tanh**: squash outputs into a bounded range, but saturate — for large positive or negative inputs, the gradient becomes nearly zero, which is exactly what causes vanishing gradients in deep networks. Rarely used in hidden layers today for this reason.
- **ReLU** (`max(0,x)`): the modern default — cheap to compute, no saturation for positive inputs, so gradients flow much better through deep networks. Its own failure mode is the "dying ReLU" problem: a unit that ends up always receiving negative input gets permanently stuck outputting zero, contributing nothing further to training.
- **Leaky ReLU**: fixes dying ReLU by allowing a small nonzero slope for negative inputs, so a unit can't get permanently stuck at exactly zero gradient.
- **GELU**: a smoother variant that's become standard in Transformers (BERT, GPT) — smoother gradients than ReLU's hard cutoff, which tends to help in very deep, large-scale architectures.
- **Softmax**: not really used as a hidden-layer activation — it's the standard *output* layer choice for multi-class classification, converting raw scores into a valid probability distribution across classes.

## Optimization — how weights actually get updated
**Gradient Descent** is the base idea: nudge each weight in the direction that reduces the loss, scaled by a learning rate η. In practice, nobody uses plain gradient descent as originally stated — a few refinements matter enough to be the actual defaults:
- **SGD (mini-batch)**: compute the gradient on a small batch of examples instead of the whole dataset each step — noisier, but far faster per step, and the noise itself sometimes helps escape shallow local minima rather than hurting.
- **Momentum**: keeps a running average of past gradients, smoothing out the update direction rather than reacting to each noisy mini-batch gradient in isolation — speeds convergence, especially in directions where the loss surface is consistently sloped.
- **Adam**: adapts the learning rate *per parameter*, using running estimates of both the gradient and its variance — this is why Adam is the default optimizer for most deep learning today: it needs far less manual learning-rate tuning than plain SGD to get reasonable results.
- **Learning rate schedules**: start small and ramp up (**warmup**), then gradually reduce (**decay**, e.g. cosine or linear) over training — warmup avoids destabilizing the model with large updates before it's learned anything useful yet; decay lets it settle into a more precise minimum later in training. This pattern is now standard in training Transformers at scale.

## Keeping a deep network numerically stable
The deeper a network gets, the more its training becomes a fight against numbers becoming too small or too large as they propagate through many layers:
- **Vanishing gradients**: gradients shrink as they multiply back through many layers (worse with saturating activations like sigmoid/tanh) — early layers stop receiving a meaningful learning signal.
- **Exploding gradients**: the opposite — gradients grow unboundedly, causing NaN losses and outright training divergence. **Gradient clipping** (capping the gradient's magnitude before applying it) is the direct fix.
- **Batch Normalization**: normalizes each layer's inputs across the current batch — stabilizes training and allows using a higher learning rate than would otherwise be safe.
- **Layer Normalization**: normalizes across features instead of across the batch — batch-independent, which is why it's the version used in Transformers (where batches can contain variable-length sequences that don't normalize cleanly together).

## Regularization specific to deep networks
(See [[ML]] for L1/L2, which apply here too.)
- **Dropout**: randomly zeroes out a fraction of neurons on each forward pass *during training only* — forces the network to not over-rely on any single neuron or co-adapted group of neurons, which improves generalization. Turned off at inference time.
- **Weight decay**: an L2 penalty applied directly through the optimizer rather than added to the loss function explicitly — same underlying idea as L2 regularization, different implementation path.

## Training practicalities worth knowing
- **Epoch**: one full pass over the training data. **Batch size**: how many examples are used per gradient update — a hyperparameter with real tradeoffs (larger batches = more stable gradient estimates but more memory and sometimes worse generalization).
- **Transfer learning**: rather than training a new network from scratch, take one already pretrained on a large dataset and fine-tune it on your smaller, task-specific data. This is standard practice — training a large model from zero requires far more data and compute than most projects have access to.
- **Data augmentation**: artificially expand training data by creating modified versions of existing examples (crops/flips/rotations for images; back-translation/paraphrasing for text) — helps generalization especially when labeled data is limited.
- **Why DL is GPU/TPU-bound**: the dominant computation in a deep network is matrix multiplication, which parallelizes extremely well on GPU/TPU hardware — this is a large part of *why* deep learning became practical at scale only once that hardware became accessible, independent of the underlying ideas being decades old in some cases.
