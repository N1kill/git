---
tags: [theory, ai, ml, foundations]
---

# Machine Learning — Overview

Related: [[Supervised Learning]] · [[Unsupervised Learning]] · [[Reinforcement Learning]] · [[DL]] · [[Neural Networks]]

## The core idea
ML is the shift from **writing rules** to **learning rules from data**. Instead of hand-coding `if X then Y`, you show a system many examples and let it find the pattern connecting input to output itself, then trust it to generalize to new inputs.

The whole field is really one running battle: a model that just *memorizes* its training examples (overfitting) is useless the moment it meets something new, but a model too simple to capture the real pattern (underfitting) is useless from the start. Nearly every technique you'll meet — regularization, cross-validation, ensembling — is a tool for finding the middle ground between these two failures. This is called the **bias-variance tradeoff**, and it's worth understanding once, deeply, because it explains *why* almost every other ML decision gets made the way it does.

## Why four different learning setups exist
The type of problem you have determines what kind of data you can even use:

- You have correct answers to learn from → **[[Supervised Learning]]** (predict a label/value from labeled examples)
- You only have raw data, no answers, and want to find structure in it → **[[Unsupervised Learning]]** (clustering, compression)
- You have a little labeled data and a lot of unlabeled data → semi-supervised (covered inside [[Supervised Learning]], since it's a variation on the same goal)
- You don't have a fixed dataset at all — an agent has to act and learn from consequences → **[[Reinforcement Learning]]**

Each has its own file — the "how do I pick an algorithm" reasoning genuinely differs enough between them that mixing them together just makes the choice harder to see.

## Bias-Variance Tradeoff (the idea underneath everything else)
- Total error = Bias² + Variance + Irreducible noise
- **High bias** = model too simple → underfitting → poor on *both* training and new data
- **High variance** = model too complex → overfitting → great on training data, poor on new data
- Fixing high bias: bigger/more flexible model, more features, less regularization
- Fixing high variance: more data, regularization, simpler model, early stopping

This single idea is *why* regularization (below), cross-validation, and "try a simpler model first" all exist as practices.

## Regularization (fights overfitting, applies across supervised methods)
- **L1 (Lasso)**: penalizes `Σ|weight|` — pushes some weights to exactly zero, effectively doing feature selection for you
- **L2 (Ridge)**: penalizes `Σweight²` — shrinks all weights smoothly, keeps every feature but dampens its influence
- **Early stopping**: stop training once performance on held-out data starts getting worse, even if training performance keeps improving
- **Use when**: your model is overfitting (training performance much better than validation performance) — that mismatch is the signal to reach for these, not a fixed rule to always apply

## Evaluating any ML model
- **Train/Val/Test split**: fit parameters on train, tune choices (which algorithm, which hyperparameters) on val, get one honest final number on test — touching test data during development quietly invalidates it
- **k-fold cross-validation**: rotate which slice of data is held out for validation, average the results — gives a more reliable performance estimate than one fixed split, especially on smaller datasets
- **Classification metrics**: Accuracy (misleading when classes are imbalanced), Precision (of what I flagged positive, how much was right), Recall (of what was actually positive, how much did I catch), F1 (balance of the two)
- **Regression metrics**: MSE/RMSE (penalizes large errors heavily), MAE (treats all errors linearly), R² (how much variance is explained)
- Use precision-heavy metrics when false positives are costly (e.g. flagging legitimate transactions as fraud); use recall-heavy metrics when false negatives are costly (e.g. missing an actual fraud case)

## Curse of dimensionality
As you add more features, data gets sparse in that high-dimensional space and distance-based reasoning (which a lot of ML quietly relies on) gets less meaningful — points all start looking roughly equidistant from each other. This is *why* dimensionality reduction and careful feature selection matter as data gets wider, not just as a nice-to-have (see [[Unsupervised Learning]] for the techniques).
