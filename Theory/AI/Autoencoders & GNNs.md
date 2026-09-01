---
tags: [theory, ai, autoencoders, gnn]
---
z
# Autoencoders & GNNs

Related: [[Neural Networks]] · [[RAG]] · [[Embeddings & Vector DBs]] · [[Generative Models]] · [[CNNs & Computer Vision]]

## Autoencoders — learning structure without labels
The core idea: an **encoder** compresses the input down into a much smaller representation (the "bottleneck"), and a **decoder** tries to reconstruct the original input from that compressed form. The network is trained purely to minimize reconstruction error — no labels required, since the input itself is the target.

Why this is useful at all: if the network can compress an image down to a fraction of its original size and still reconstruct it well, that compressed bottleneck must be capturing the input's essential structure, not noise. That compressed representation is often more useful than the raw input for downstream tasks.

**Where/how to use autoencoders in general**:
- **Dimensionality reduction**: similar goal to PCA, but can capture nonlinear structure PCA can't (see [[Unsupervised Learning]] for the PCA comparison)
- **Anomaly detection**: train on "normal" data only — anomalous inputs, being unlike anything the model learned to compress well, will reconstruct poorly, and that reconstruction error becomes your anomaly score
- **Pretraining a feature extractor** when you don't have labels yet, but want a useful starting representation before fine-tuning on a smaller labeled set

There are two separate axes of variation among autoencoders, and it's worth keeping them apart: **architecture** (what kind of layers the encoder/decoder are built from) and **training objective** (what constraint or corruption is added to shape what the bottleneck learns). The three below are architectural variants; denoising and VAE further down are objective variants — you can combine either architecture with either objective (e.g. a convolutional autoencoder trained with a denoising objective is a completely normal, common combination).

### Architectural variants

**Fully Connected (Vanilla) Autoencoder**
Encoder and decoder are just stacks of fully-connected (dense) layers, same as a plain MLP (see [[Neural Networks]]) — no assumption about input structure at all.
- **Use when**: input is tabular/flat data with no strong spatial or sequential structure to exploit — the simplest starting point, and a reasonable baseline before reaching for a more structured variant.
- **Where it falls short**: on image data specifically, it ignores spatial locality entirely (treats every pixel as an independent feature, the same weakness a plain MLP has on images) — wastes parameters and generalizes worse than a convolutional version would.

**Convolutional Autoencoder (CAE)**
Same encoder-bottleneck-decoder structure, but built from convolutional layers instead of fully-connected ones — the encoder uses convolutions (with pooling/stride to downsample), the decoder uses transposed convolutions (or upsampling + convolution) to reconstruct back to full size.
- **Why it exists**: for image data, convolution's weight-sharing and spatial-locality assumptions (see [[CNNs & Computer Vision]]) apply just as much to compressing/reconstructing images as they do to classifying them — far more parameter-efficient than a fully-connected autoencoder on the same image, and produces sharper reconstructions since spatial relationships between nearby pixels are preserved through the encoding.
- **Use when**: your input is image (or other grid-like/spatial) data — this is the default choice over a vanilla autoencoder for anything visual.

**Sparse Autoencoder**
Adds a **sparsity penalty** to the training objective, pushing most of the bottleneck's neurons to stay near-zero (inactive) for any given input, so only a small subset actively "fire" per example — even when the bottleneck layer itself isn't made small.
- **Why this matters**: a plain autoencoder forces compression by making the bottleneck *narrower* than the input — the small size itself is the constraint. A sparse autoencoder instead allows a wide (even overcomplete — larger than the input) bottleneck, but constrains *how many* of those units can be active at once. This encourages each neuron to specialize in detecting a specific, distinct feature, rather than the network spreading information densely across all of them — often yielding more interpretable, disentangled features than a plain narrow bottleneck does.
- **Use when**: you specifically want interpretable or disentangled features (each neuron corresponding to something meaningful), or you want a bottleneck that isn't forced to be smaller than the input to still achieve useful compression/regularization.

### Objective variants (combine with any architecture above)

**Denoising autoencoders**: trained to reconstruct a *clean* input from a deliberately *corrupted* version of it. This forces the network to learn genuinely robust features rather than just memorizing an identity mapping — use when you specifically want a model resilient to noisy/imperfect input (e.g. cleaning up scanned documents, noisy sensor data).

**VAEs (Variational Autoencoders)**: instead of mapping input to a single fixed point in the bottleneck, a VAE maps it to a *probability distribution* over the latent space. This means you can sample new points from that distribution and decode them into new, plausible synthetic data — turning the autoencoder from a pure compressor into a **generative model**. Use when you want to *generate* new samples (synthetic data augmentation, generative creative tools), not just compress/reconstruct existing ones. See [[Generative Models]] for how this compares to GANs and Diffusion models on the generation-quality/speed tradeoff.

## GNNs (Graph Neural Networks) — learning on relationships, not grids or sequences
CNNs assume grid structure; RNNs assume sequential order. Neither assumption fits data that's naturally **relational** — a social network, a molecule's atomic structure, a knowledge graph — where what matters is which entities connect to which, not their position in a line or grid.

A GNN operates directly on nodes and edges: each node updates its own representation by aggregating information from its neighbors, and this happens over several rounds so information can propagate further across the graph with each round (a node's representation after 2 rounds has effectively "seen" its neighbors' neighbors).

**Where/how to use GNNs**:
- Data is naturally graph-shaped and the relationships *are* the signal, not just metadata — social network analysis, molecule/drug property prediction, recommendation systems (users and items as a graph), fraud detection (transaction networks)
- Directly relevant alongside **Graph RAG** (see [[RAG]]) — once you've built a knowledge graph from documents, a GNN is one way to reason over that graph's structure, versus treating it as something to merely traverse
- **Don't reach for a GNN** if your data doesn't have genuine relational structure worth exploiting — forcing tabular or sequential data into a graph representation just to use a GNN usually isn't worth the added complexity over a simpler architecture that already fits the data's actual shape.
