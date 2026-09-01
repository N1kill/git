
---
### 2026-08-26 — Post-leakage-fix training results logged (pre-v3 rebuild)

**Run context:** After the 2-day cutoff leakage fix was applied (delta_risk/risk_slope no longer computed from terminal CDM), before the v3 notebook rebuild (feature-parity fix for LSTM/Transformer not yet applied in this run).

**Reported metrics (validation split):**

| Model | RMSE | MAE | R² | Bias | P90 tail MAE | P90 tail RMSE | High-risk count | Accuracy (±5/±10/±20%) |
|---|---|---|---|---|---|---|---|---|
| CatBoost | 5.330 | 3.159 | 0.539 | 0.112 | 8.614 | 10.174 | 142 | 56.6 / 65.3 / 73.7 |
| XGBoost | 5.366 | 3.125 | 0.532 | 0.189 | 8.246 | 9.780 | 142 | 56.8 / 64.2 / 73.2 |
| LSTM | 7.428 | 5.532 | 0.104 | 3.387 | 6.705 | 8.471 | 142 | 20.9 / 36.2 / 58.8 |
| Transformer | 7.087 | 4.916 | 0.184 | 2.642 | 6.942 | 8.567 | 142 | 30.9 / 45.7 / 62.9 |

**Interpretation at the time:**
- CatBoost/XGBoost ~5.3 RMSE is the honest, leakage-free number (previous ~2.88 RMSE was inflated by the delta_risk/risk_slope leak).
- LSTM/Transformer R² near-zero (0.10, 0.18) with large positive Bias — consistent with NaN propagation through hidden states, NOT an architecture-vs-data-volume problem per se.
- First NaN/outlier fix to CDMSequenceDataset (clip + impute) applied between two runs of this same experiment — R² moved only 0.08→0.10 (LSTM) and stayed flat at 0.18 (Transformer), confirming NaN handling alone was not the dominant bug.
- Deeper notebook review (`new-try-after-leakage__1_.ipynb`) then found the real remaining issues — see separate entry "Notebook rebuild v3" for the three bugs (duplicate means/stds sources, split misalignment in metrics calls, and CatBoost/XGBoost given 15 engineered trend features that LSTM/Transformer never received).

**Status:** These numbers are superseded by whatever `prahari_training_v3.ipynb` produces once run. Keeping this entry as the "before" baseline for comparison — do not delete when v3 results come in, append the new results as a separate dated entry instead (append-never-overwrite convention).





---
### 2026-08-26 — v3 notebook results: feature-parity fix confirmed real, new finding on tail performance

**Run context:** `prahari_training_v3.ipynb` executed top-to-bottom. LSTM/Transformer now receive raw+engineered features (broadcast per timestep), single means/stds source, correct split alignment throughout.

**Results (validation split):**

| Model | RMSE | MAE | R² | Bias | P90 tail MAE | P90 tail RMSE | High-risk count | Accuracy (±5/±10/±20%) |
|---|---|---|---|---|---|---|---|---|
| CatBoost | 5.330 | 3.159 | 0.539 | 0.112 | 8.614 | 10.174 | 142 | 56.6 / 65.3 / 73.7 |
| XGBoost | 5.366 | 3.125 | 0.532 | 0.189 | 8.246 | 9.780 | 142 | 56.8 / 64.2 / 73.2 |
| LSTM | 6.881 | 4.491 | 0.231 | 2.749 | **5.213** | 7.143 | 142 | 36.6 / 54.1 / 67.4 |
| Transformer | 6.827 | 4.246 | 0.243 | 1.587 | **5.218** | 7.062 | 142 | 44.2 / 59.1 / 67.4 |

**Comparison to pre-v3 (raw-only sequence input) baseline logged previously:**
- LSTM: RMSE 7.428→6.881 (↓0.55), R² 0.104→0.231 (~2.2x)
- Transformer: RMSE 7.087→6.827 (↓0.26), R² 0.184→0.243 (+32%)
- Confirms the feature-parity fix (giving LSTM/Transformer the same 15 engineered trend features as CatBoost/XGBoost) was a real, meaningful contributor — not noise.

**New finding, not yet investigated — flagging for follow-up:**
LSTM/Transformer P90_tail_MAE (5.21, 5.22) is notably BETTER than CatBoost/XGBoost tail MAE (8.61, 8.25), even though overall RMSE/R² still favor CatBoost/XGBoost. Possible read: sequence models are more accurate specifically on high-risk tail events (the operationally important case), despite worse average-case performance. Alternative read: this could be noise given only 142 high-risk events in the tail bucket.

**Open question, not yet resolved:** is LSTM/Transformer's remaining R² gap vs CatBoost (0.23-0.24 vs 0.53-0.54) a sign the architecture still isn't learning well (undersized hidden dims, early stopping too aggressive, attention pooling collapsing to near-last-timestep), or is this close to an honest ceiling for this architecture on this data, with the tail-accuracy advantage being the actual reportable contribution instead of "sequence beats tabular overall"? Per-risk-bucket (low/watch/high) breakdown across all four models proposed as the next diagnostic, not yet run.

**Status:** Feature-parity fix validated as real and worth keeping. Tail-performance finding is promising but unconfirmed — do not cite in paper until verified against bucket breakdown and checked it's not an artifact of the small (n=142) high-risk sample.

---
### 2026-08-26 — Cutoff sweep (0.5/1/2/3 days), classification threshold analysis started, model persistence set up

**Context:** Nikhil ran a GPT-recommended extension of `prahari_training_v3.ipynb` (same v3 root, new cells appended — not a separate pipeline). Added: full cutoff sweep for regression (Section 12-13 style) and a dangerous-event classification layer (threshold=-7.0 in log-risk space, ~3.7% of events labeled dangerous).

**Regression cutoff sweep results (validation split, RMSE by model x cutoff):**

| Cutoff | CatBoost | XGBoost | LSTM | Transformer |
|---|---|---|---|---|
| 0.5d | 4.327 | 4.333 | 4.836 | 4.913 |
| 1.0d | 4.969 | 4.995 | 5.973 | 5.938 |
| 2.0d | 5.330 | 5.366 | 7.077 | 6.512 |
| 3.0d | 5.241 | 5.322 | 6.742 | 7.035 |

**Key finding: the CatBoost-vs-Transformer RMSE gap WIDENS as cutoff moves earlier (further from TCA).** Gap = 0.59 at 0.5d, 0.97 at 1.0d, 1.18 at 2.0d, 1.79 at 3.0d. Sequence models degrade faster than CatBoost as prediction horizon lengthens — plausible mechanism: shorter/fewer available CDM sequences further from TCA starve the sequence architecture of the timesteps it needs, while CatBoost's engineered summary features compress available info more gracefully regardless of sequence length.

**However — tail-MAE advantage for sequence models HOLDS at every cutoff** (Transformer's P90_tail_MAE stays below CatBoost's at all four cutoffs: 1.52 vs 3.49 at 0.5d, up to 6.44 vs 9.95 at 3.0d) — this is a stronger, more robust version of today's earlier tail-accuracy finding (previously only confirmed at 2.0d). Real, cutoff-robust research contribution.

**Implication for orchestrator:** today's v1-v4 orchestrator attempts were all built/evaluated ONLY at the 2.0-day cutoff. Given the gap changes SHAPE across cutoffs, a gate trained only at 2.0d will likely behave wrong at 0.5d/3.0d. Decision: orchestrator should be made cutoff-aware by adding `cutoff_days` as an explicit gating feature (pooled training across all 4 cutoffs) rather than building 4 separate orchestrators. Not yet implemented — v4 soft-blend script needs this update before being trusted.

**Classification analysis (dangerous-event detection, threshold=-7.0, 2.0-day cutoff only so far):**
- CatBoost: precision=1.000, recall=0.075 (only flagged 4/53 real dangerous events — too conservative to function as a triage tool despite "perfect" precision)
- XGBoost: precision=1.000, recall=0.038 (flagged 2/53, even more conservative)
- LSTM: precision=0.375, recall=0.375 (flagged 16, caught 6)
- Transformer: precision=0.245, recall=0.245, PR-AUC=0.329 (highest PR-AUC of all four — best ranking ability even though this specific threshold isn't ideal for it)

**Same tail-tradeoff pattern as the regression bootstrap-CI finding, independently confirmed via a completely different methodology (classification, not tail-bucket MAE)** — strengthens rather than duplicates today's earlier evidence.

**Concern flagged:** the -7.0 threshold is SHARED across all 4 models and was only evaluated at 2.0d — this likely advantages whichever model's predicted-risk scale happens to sit closest to -7.0 by coincidence, not genuine ranking skill. PR-AUC (threshold-independent) shows all four models are more similar in ranking ability (0.32-0.35) than the shared-threshold precision/recall numbers suggest.

**In progress, not yet run:** `classification_sweep_all_cutoffs.py` — full PR-AUC/F1-optimal threshold sweep across all 4 cutoffs x 4 models, retraining fresh at each cutoff, comparing shared-threshold vs per-model-optimal-threshold results side by side. This will settle whether -7.0 was quietly unfair to some models, and reveal whether classification performance shows the same cutoff-shape-change pattern the regression sweep did.

**Model persistence:** `save_all_cutoff_models.py` built — saves all 16 models (4 models x 4 cutoffs) from the same training pass used for the classification sweep, to `models_store/cutoff_sweep/` with cutoff-tagged filenames (e.g. `catboost_2d.cbm`, `lstm_0p5d.pt`) plus a `manifest.csv`. Deliberately does NOT overwrite the existing `models_store/v1/` production path automatically — a commented-out promotion step exists for later, deliberate use if the 2.0-day sweep models are chosen to become the production v1 models.

**Status:** Awaiting the classification sweep run to complete before finalizing the cutoff-aware orchestrator design or deciding on model promotion to v1.
