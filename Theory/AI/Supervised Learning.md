---
tags: [theory, ai, ml, supervised-learning]
---

# Supervised Learning

Related: [[ML]] · [[Evaluation & Guardrails]]

## The core idea
You have (input, correct answer) pairs, and the goal is to learn the mapping well enough to predict the answer for inputs you haven't seen. There are only two shapes this prediction takes — a number (**regression**) or a category (**classification**) — and almost every algorithm below can be adapted to do either.

The real decision isn't "which task type" (that's usually obvious from your problem), it's "which algorithm's assumptions match my data." Below, each algorithm is really a different bet about the *shape* of the true relationship: a straight line, a set of if/else splits, a margin between groups, or "trust whatever's nearby." Picking wrong doesn't break anything — it just leaves accuracy on the table that a better-matched algorithm would have captured.

## Regression vs Classification
- **Regression**: predict a continuous value (price, temperature, demand). Linear regression fits `y = Xw + b`, minimizing squared error between predictions and actuals.
- **Classification**: predict a discrete category (spam/not spam, which of 5 classes). Logistic regression uses a sigmoid to squash its output into a 0–1 probability; multi-class versions use softmax across categories.
- **Where to use which**: if the honest answer to "what am I predicting" is a number, it's regression; if it's "which bucket," it's classification. Some problems can be framed either way (e.g. predicting a rating 1–5 as regression or as 5-class classification) — pick based on whether the *order and distance* between outcomes matters (regression) or only the category does (classification).

## Linear / Logistic Regression — the baseline
Fits the simplest possible relationship: a straight line (or hyperplane) through the data.
- **Why start here**: fast to train, easy to interpret (each feature's weight tells you its effect directly), and a good sanity check — if a linear model already does well, you may not need anything fancier.
- **Where it falls short**: real relationships are often not linear, so it underfits complex patterns.
- **Use when**: you need interpretability, you're establishing a baseline before trying anything heavier, or you have reason to believe the relationship really is close to linear.

## Decision Trees
Recursively splits the data on feature thresholds ("is age > 30? then is income > 50k? ...") until it reaches a prediction. You can literally trace the exact path that led to any prediction.
- **Why it's useful**: fully interpretable, handles nonlinear relationships and feature interactions naturally, no need to scale features first.
- **Where it falls short**: a single tree overfits easily — it'll happily grow deep enough to memorize noise in the training data.
- **Use when**: you need to explain *individual* predictions to a non-technical audience (e.g. "why was this loan denied") and can tolerate somewhat lower accuracy than an ensemble for that transparency.

## Random Forest — fixing the Decision Tree's overfitting
Trains many decision trees, each on a random subset of data and features, and averages/votes their predictions.
- **Why it works**: individual trees overfit in different, semi-random ways; averaging many of them cancels out most of that noise while keeping the ability to model nonlinear patterns.
- **Where it falls short**: loses the single-tree's easy interpretability (you can't trace "the" decision path anymore), and typically underperforms well-tuned gradient boosting.
- **Use when**: you want a strong, low-effort result on tabular data fast, without much hyperparameter tuning — this is usually the right *first real model* to try, before reaching for gradient boosting.

## Gradient Boosting (XGBoost / LightGBM / CatBoost)
Instead of training trees independently like Random Forest, gradient boosting builds trees **sequentially** — each new tree is trained specifically to correct the errors the ensemble has made so far.
- **Why it usually wins on tabular data**: directly targeting the current errors, rather than averaging independent guesses, tends to squeeze out more accuracy — this family dominates tabular ML competitions for a reason.
- **Where it falls short**: more hyperparameters to tune, more prone to overfitting if not tuned carefully, slower to train than Random Forest.
- **The three implementations differ in one practical way each**:
  - **XGBoost**: the original, well-regularized, reliable default — reach for this if you're not sure which of the three to pick.
  - **LightGBM**: grows trees leaf-wise instead of level-wise and bins data into histograms — meaningfully faster on large datasets, similar accuracy.
  - **CatBoost**: handles categorical features natively (no manual encoding needed) and uses ordered boosting to avoid a subtle overfitting bias the other two are prone to — reach for this specifically when your dataset has many categorical columns (e.g. country, product category, user ID).
- **Use when**: you've already tried Random Forest, need more accuracy, and can afford to tune hyperparameters (or use each library's sane defaults + basic tuning).

## SVM (Support Vector Machine)
Finds the boundary between classes that maximizes the margin (distance) to the nearest points of each class. The **kernel trick** lets it draw nonlinear boundaries by implicitly projecting data into a higher-dimensional space where a straight boundary *would* separate the classes.
- **Why it's useful**: strong theoretical guarantees, works well even with relatively few training examples if the classes are cleanly separable.
- **Where it falls short**: doesn't scale well to large datasets, kernel choice/tuning is fiddly, gradient boosting usually wins on typical tabular problems now.
- **Use when**: smaller-to-medium datasets with a reasonably clear margin between classes — less commonly the first choice today, but still solid for that niche.

## k-NN (k-Nearest Neighbors)
No real training phase — to predict, it just looks at the k closest points in the training data (by some distance metric) and averages/votes their labels.
- **Why it's useful**: dead simple to understand, no assumptions about the data's shape.
- **Where it falls short**: gets slow at prediction time as data grows (has to search the whole training set each time), and sensitive to feature scaling and the curse of dimensionality (see [[ML]]).
- **Use when**: small datasets, quick prototyping, or as a simple baseline — rarely the production choice for large-scale problems.

## Semi-supervised learning
A variation on supervised learning for when you have a small labeled set and a much larger unlabeled one.
- **Self-training**: train on the labeled data, use that model to label the unlabeled data (pseudo-labels), fold the confident pseudo-labels back into training.
- **Label propagation**: spread known labels across a similarity graph of all the data, so unlabeled points near labeled ones inherit that label.
- **Use when**: labeling is expensive or slow (e.g. requires expert review, like medical images) but raw unlabeled data is cheap and abundant — squeezes more value out of the labels you already paid for.
