---
tags: [theory, ai, neural-networks, foundations]
---

# Neural Networks — Overview

Related: [[DL]] · [[ML]] · [[Transformers]] · [[CNNs & Computer Vision]] · [[RNNs, LSTM & GRU]] · [[Autoencoders & GNNs]] · [[Generative Models]]

## The core idea
A neural network is a function built from layers of simple units. Each unit takes a weighted sum of its inputs, adds a bias, and passes the result through a nonlinear function. One unit alone is basically logistic regression — nothing special. But stack many of them into layers, and layers into a deep network, and the composition of simple nonlinear pieces can approximate extremely complex functions.

The network learns by adjusting its weights so predictions get closer to correct, across many examples. **Backpropagation** is the mechanism that figures out which weights to adjust and by how much: it applies the chain rule from calculus, working backward from the final error through every layer.

**Why nonlinearity is the whole point**: composing linear functions is still linear — a 100-layer network with no nonlinear activations collapses mathematically to one linear layer. The nonlinearity (ReLU, GELU, etc.) is what actually gives depth its expressive power.

**Why layers matter, conceptually**: each layer transforms the representation handed to it by the previous one. Early layers tend to learn simple, general patterns; later layers combine those into increasingly abstract, task-specific features. This is the core practical difference from classical ML — a human doesn't hand-engineer these features, the network learns them.

## Why different architectures exist at all
A plain fully-connected network (every neuron connected to every neuron in the next layer) makes *no assumption* about the structure of your input — which means it also can't *exploit* any structure your input has. Three architecture families exist because three kinds of input structure are extremely common, and encoding the right assumption directly into the architecture makes learning dramatically more data- and parameter-efficient:

- Input is grid-like/spatial (images) → **[[CNNs & Computer Vision]]**
- Input is sequential/ordered (time series, text) → **[[RNNs, LSTM & GRU]]**
- Input needs compression without labels, or is graph-structured (relationships between entities) → **[[Autoencoders & GNNs]]**
- Input is sequential but you have the data/compute for something stronger than RNNs → **[[Transformers]]** (now the default for most sequence modeling, including language)

Each linked file covers its own architectures conceptually, with where/how to use each. What's below is the shared machinery underneath all of them.

- Input needs pure generation of new samples (images, audio) rather than understanding existing ones → **[[Generative Models]]** (GANs, Diffusion)

## Weight initialization — why it's not arbitrary
Starting all weights at zero would make every neuron compute an identical gradient, so the network can never break that symmetry and differentiate — it has to start random. Beyond "just random," the *scale* of that randomness matters: **Xavier/Glorot init** is tuned for sigmoid/tanh activations, **He init** is tuned for ReLU (accounting for the fact that ReLU zeroes out roughly half its inputs) — using the mismatched one tends to slow training or destabilize it early on.

## Common failure modes and what causes them
- **Vanishing gradients**: in deep networks, especially with saturating activations (sigmoid/tanh), gradients shrink as they propagate backward through many layers — early layers barely update, effectively stop learning. This is *the* motivating problem behind LSTM/GRU design (see [[RNNs, LSTM & GRU]]) and residual connections in Transformers.
- **Exploding gradients**: the opposite — gradients grow unboundedly backward through the network, causing NaN losses and divergence. Fixed with gradient clipping, careful initialization, and normalization layers.
- **Dead neurons**: a ReLU unit that ends up always outputting zero (stuck with a large negative bias) stops contributing anything — Leaky ReLU is a common fix, since it never fully zeroes out the gradient.

## Parameters vs Hyperparameters
- **Parameters** are learned from data — weights and biases, adjusted by gradient descent.
- **Hyperparameters** are chosen before training and never learned by gradient descent — learning rate, number of layers, batch size, which architecture to use. Tuned by trying different values and checking validation performance, not learned automatically.

## Universal Approximation Theorem
A network with just one sufficiently wide hidden layer can, in theory, approximate any continuous function to arbitrary precision. This sounds like it makes "deep" unnecessary — but the theorem says nothing about *how easy* that function actually is to learn, or how many neurons it would practically take. In practice, depth turns out to be far more parameter-efficient than width for representing the kinds of functions we actually care about — which is the real justification for building networks deep rather than just wide.
