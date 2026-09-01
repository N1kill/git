---
tags: [theory, ai, ml, unsupervised-learning]
---

# Unsupervised Learning

Related: [[ML]] · [[Embeddings & Vector DBs]]

## The core idea
There's no "correct answer" to check against here — you only have raw data, and the goal is to find structure in it that wasn't explicitly labeled. This splits into two genuinely different goals: **grouping similar things together** (clustering) and **compressing data down to its essential structure** (dimensionality reduction). They solve different problems and usually aren't substitutes for each other, though they're sometimes combined (reduce dimensions first, then cluster).

## Clustering — grouping similar points
The question clustering answers is "which things naturally belong together," with no predefined categories to sort into.

**k-Means**
Picks k cluster centers, assigns each point to its nearest center, recomputes centers as the average of their assigned points, repeats until stable.
- **Why it's the default**: fast, simple, scales well to large data.
- **Where it falls short**: you have to choose k upfront (often not obvious), and it assumes clusters are roughly round/similar-sized — it'll do a poor job on oddly-shaped or very differently-sized clusters.
- **Use when**: you have a rough idea of how many groups you expect, and the data doesn't have obviously weird cluster shapes.

**DBSCAN**
Groups points that are densely packed together, and marks sparse, isolated points as noise/outliers rather than forcing them into a cluster.
- **Why it's useful over k-Means**: finds arbitrarily-shaped clusters, doesn't need k chosen upfront, and naturally identifies outliers instead of distorting a cluster to include them.
- **Where it falls short**: sensitive to its density parameters, struggles when clusters have very different densities from each other.
- **Use when**: you don't know how many clusters to expect, your data likely has noise/outliers, or clusters aren't roughly circular.

**Hierarchical clustering**
Builds a tree of nested clusters (a dendrogram) — you can cut the tree at different heights to get coarser or finer groupings.
- **Use when**: you want to explore structure at multiple levels of granularity rather than commit to one fixed number of clusters, or the natural groupings in your domain really are hierarchical (e.g. taxonomies).

## Dimensionality reduction — compressing without losing the essence
The question this answers is "can I represent this data with far fewer numbers and still keep what matters."

**PCA (Principal Component Analysis)**
Finds the directions in the data that capture the most variance, and projects the data onto those directions.
- **Why it's the default**: fast, linear, mathematically well-understood, a solid general-purpose first step.
- **Where it falls short**: it's linear — if the real structure in your data is curved/nonlinear, PCA will miss it.
- **Use when**: you need to reduce feature count before feeding data into another model (speeds training, can reduce overfitting via the curse of dimensionality — see [[ML]]), or want a quick linear summary of your data's main axes of variation.

**t-SNE / UMAP**
Nonlinear techniques that try to preserve *local* neighborhood structure (which points are close to which) when squashing data down to 2-3 dimensions.
- **Why they're useful**: reveal cluster structure and relationships that PCA's linear projection can't see, and produce genuinely useful visualizations of high-dimensional data.
- **Where they fall short**: distances *between* clusters in the resulting plot aren't reliable (only local neighborhoods are preserved, not global distances) — don't over-interpret how far apart two clusters look. Also not deterministic in the same way PCA is, and not meant as general preprocessing for another model.
- **Use when**: you want to *visualize* and explore high-dimensional data (e.g. "do my embeddings actually cluster the way I expect") — this is their primary real use case, not feature reduction for a downstream model.

## How the two connect
Sometimes you reduce dimensions first (PCA) and *then* cluster the reduced representation — this can make clustering algorithms like k-Means work better, since it sidesteps some of the curse-of-dimensionality problems that make distance-based methods struggle in very high-dimensional spaces.
