---
tags:
  - prahari
  - models
  - source-of-truth
status: living-doc
last-updated: 2026-08-20
---

# Model Status

> Single source of truth for model state. `OCRA.html` / `prahari_final.html` are demo-only artifacts and do NOT reflect real model decisions — do not cross-reference them for this.


# Version 1.0
## Primary Model

**CatBoost** — confirmed best RMSE, primary model for the pipeline.

## Comparison Models

- XGBoost
- LSTM
- Transformer

## Metrics

> TBD — no verified numbers recorded yet. Fill in below, one row per model, once available.

| Model       | RMSE     | MAE      | Notes                          |
| ----------- | -------- | -------- | ------------------------------ |
| CatBoost    | 1.890772 | 2.878172 | primary                        |
| XGBoost     | 2.061414 | 3.282000 | comparison                     |
| LSTM        | 5.844999 | 7.891017 | comparison — affected by DQ-01 |
| Transformer | 6.072497 | 8.151043 | comparison — affected by DQ-01 |

Dataset split / event counts: 
train_data.csv : 162,634 rows, 103 columns, 13,154 unique events
test_data.csv  : 24,484 rows, 103 columns, 2,167 unique events

## Known Confound — DQ-01

The RMSE gap between CatBoost and LSTM/Transformer is a **data-contamination artifact**, not an architectural finding.

- **Root cause**: CatBoost handles NaNs natively; LSTM/Transformer do not.
- **Diagnostic findings**:
  - 46 features with real NaNs (covariance matrix terms, space-weather indices — genuinely absent in ESA data)
  - 21 features with extreme outliers (legitimate heavy-tailed physical values)
  - Minor sentinel values in relative position/velocity columns
- **Status**: unfixed, deferred. Nothing has changed on this since it was first flagged.
- **Why deferred**: backend/frontend build prioritized over diagnostic remediation.
- **What "clean" requires**: re-run LSTM/Transformer after the NaN/outlier fix to get a defensible architectural comparison. Without this, "CatBoost is architecturally better" is not a claim the project can make — only "CatBoost wins under current data conditions" is supported.

## Open TODOs
## Post-Fix Results — 2026-08-19

Four independent fixes applied and verified (see `Decisions.md` 2026-08-19 entry for full detail: missing-value/floor handling, floor-aware loss, standardization stats bug, LSTM padding bug). None closed the gap.

| Model       | val_rmse | test_rmse | Notes |
| ----------- | -------- | --------- | ----- |
| CatBoost    | 1.890772 | 2.878172  | primary — unchanged across all four fix attempts (expected, fixes only touched LSTM/Transformer code) |
| XGBoost     | 2.061414 | 3.282000  | comparison — unchanged, same reason |
| Transformer | 5.844999 | 7.891017  | comparison — flat across fixes 1–3, masking unaffected (Transformer's `src_key_padding_mask` was already correct, confirmed by code inspection, not the source of the LSTM's bug) |
| LSTM        | 6.072497 | 8.151043  | comparison — flat-to-slightly-worse across all four fixes, including the padding fix |

**Revised interpretation:** DQ-01's original single-cause diagnosis (NaN handling) is superseded. The gap is now suspected structural — dataset size (~13,150 events) likely insufficient for LSTM/Transformer to beat gradient-boosted trees on this task, consistent with prior literature on this exact dataset. Not confirmed; see Decisions.md for the two remaining open forks (diagnostic scan vs. accept-and-report).

**Still true from original DQ-01, not superseded:** the underlying NaN/missing-value and outlier issues were real and worth fixing regardless of whether they explain the RMSE gap — data quality work was not wasted effort, it just wasn't sufficient on its own.



- [x] Get real RMSE/MAE numbers into the table above
- [ ] Fix NaN/outlier issues (46 NaN features, 21 outlier features, sentinel values)
- [ ] Re-run LSTM/Transformer post-fix
- [ ] Update this doc once re-run is done — do not overwrite the DQ-01 section, append a new "Post-fix results" section instead so the history is preserved

## Saved Artifact Verification — 2026-08-20

The local saved-artifact directory `.../3rdproject/OCRA/prahari_trained_models/` was inspected directly, together with `catboost_info/`.

### CatBoost artifact

- File: `catboost_model.cbm` (2.3 MB), successfully loaded with CatBoost 1.2.10.
- Training completed: `2026-08-20T13:29:04Z`.
- Configuration: RMSE loss/evaluation, depth 6, learning rate 0.05, up to 2,000 iterations, `use_best_model=True`, early-stopping wait 50.
- Stored model: 1,997 trees; best validation RMSE **1.890772451** at iteration 1,996. The final logged validation RMSE was 1.890936862.
- Artifact feature contract: **189 named features** (full CDM measurements, their missingness flags, six engineered summary features, and `c_object_type`). This is more complete than the narrow feature list currently present in `features.py`.

### Saved comparison file

| Model | val_rmse | test_rmse |
| --- | ---: | ---: |
| CatBoost | 1.890772 | 2.878172 |
| XGBoost | 2.061414 | 3.282000 |
| Transformer | 5.581409 | 7.637846 |
| LSTM | 5.928822 | 8.107969 |

These values are the contents of the saved local `model_comparison_results.csv`. The CatBoost and XGBoost values agree with the table above; the sequential-model values differ from older logged values and require run/provenance reconciliation before external reporting.

### Serving status — not yet promotable

The artifact is technically loadable but must **not** yet be treated as a production predictor. Its two largest global feature importances are `risk_slope` (29.205) and `delta_risk` (16.306). The current builder calculates both with the final `risk` field, which is also the target being predicted. That terminal value is not available at live inference. Rebuild these as history-only cutoff features, retrain and version CatBoost, then fit and verify the finalized conformal uncertainty layer before deploying the endpoint.


## Leakage-Safe Retrain — 2026-08-20 (v2 pipeline)

**Resolves:** the target-leakage finding logged above under "Serving Status — not yet promotable." Confirmed root cause was correct: `delta_risk`/`risk_slope` were computed from the terminal `risk` value, the same value being predicted.

**New pipeline** (`preprocessing.py`, `feature_extraction.py`, `train_catboost.py`) rebuilds the task honestly:
- **Input:** only CDMs at least `horizon_days` (default 2.0) before an event's terminal CDM.
- **Target:** terminal event risk, held completely separate from feature construction.
- **Historical risk features are allowed** (`current_observed_risk`, `observed_risk_mean/std/min/max`, `risk_floor_fraction`, etc.) since they're computed only from CDMs inside the allowed context — this is the correct distinction, not a removal of risk-trend features entirely.

**Also corrected — a false claim in the old `features.py` docstring:** six covariance columns (`t_sigma_ndot`, `t_sigma_tdot`, `c_sigma_ndot`, `c_sigma_tdot`, `t_position_covariance_det`, `c_position_covariance_det`) were previously asserted as exact 1.000-correlation duplicates and dropped. Direct inspection of the real `train_data.csv` shows this is **not true** — they are not exact duplicates. The new pipeline keeps all 97 valid numeric CDM inputs and does not drop them. The earlier "confirmed duplicate" language should not be trusted; it was based on a partial correlation-table read, not full verification against real data.

**New honest validation numbers:**

| Model | RMSE | MAE | Notes |
|---|---:|---:|---|
| Persistence baseline (current risk stays the same) | 10.247 | 5.873 | the real floor to beat |
| CatBoost (leakage-safe, cutoff-safe) | 5.391 | 3.222 | best iteration 821 of 2,500 |

**This is not comparable to the old 1.89/2.88 numbers above — different task entirely.** The old number answered an easier, invalid question (predict the terminal risk using features partly derived from the terminal risk). This number answers the real question (predict terminal risk from data available ≥2 days before it happens). 5.391 vs the 10.247 persistence baseline is roughly a 47% RMSE reduction — that is the correct, honest headline result, not a regression from 1.89.

**Test set status:** untouched. `--evaluate-test` was not used for this run. `test_used: false` recorded in the model manifest, with a SHA256 dataset fingerprint for provenance.

**Still open — this is the important carry-over:** LSTM and Transformer were never re-trained on this corrected, cutoff-safe pipeline. Every LSTM/Transformer number in the "Post-Fix Results" section above (and the DQ-01 discussion) was computed against the *old*, leakage-affected data construction — `build_sequences()` in the old `features.py` also pulled `g[TARGET_COL].iloc[-1]` with no cutoff boundary applied. That means the entire CatBoost-vs-LSTM/Transformer comparison done throughout this project needs to be redone from scratch on the new pipeline before any conclusion (including the "likely structural, dataset-size-limited" hypothesis) can be trusted. Nothing about the sequence models' real performance is currently known.



## 2026-08-21 — Leakage-Safe Model Comparison Completed

### Status

The first complete leakage-safe model comparison for Prahari has been completed at a **2-day forecasting horizon**.

The corrected pipeline predicts the terminal event risk using only CDMs available at least 2 days before the terminal CDM:

```text
context = CDMs where time_to_tca >= terminal_time_to_tca + 2 days
```

## Version 1.1:
## 2026-08-21 — Leakage-Safe Model Comparison

The corrected leakage-safe pipeline has now been evaluated across four
model families at a 2-day forecasting horizon.

All models use the same event-level train/validation split and the same
cutoff rule:

- Terminal CDM risk = prediction target only.
- Features are restricted to CDMs at least 2 days before the terminal CDM.
- Validation is performed on events unseen during training.
- Metrics use the same evaluation function across models.
- Persistence is evaluated on the same cutoff-safe validation events.

### Validation Results

| Rank | Model | RMSE | MAE | P90 Tail MAE | Improvement vs Persistence |
|---:|---|---:|---:|---:|---:|
| 1 | CatBoost | 5.3295 | 3.1594 | 8.6142 | 47.99% |
| 2 | XGBoost | 5.3663 | 3.1253 | 8.2456 | 47.63% |
| 3 | Transformer | 7.2685 | 5.1373 | 6.5745 | 29.07% |
| 4 | LSTM | 7.3646 | 5.0667 | 7.2139 | 28.13% |
| 5 | Persistence | 10.2470 | 5.8730 | 1.9036 | 0.00% |

### Additional Metrics

CatBoost:
- RMSE: 5.3295
- MAE: 3.1594
- R²: 0.5385
- Bias: +0.1118
- P90 Tail MAE: 8.6142
- P90 Tail RMSE: 10.1742

XGBoost:
- RMSE: 5.3663
- MAE: 3.1253
- R²: 0.5322
- Bias: +0.1887
- P90 Tail MAE: 8.2456
- P90 Tail RMSE: 9.7796

LSTM:
- RMSE: 7.3646
- MAE: 5.0667
- R²: 0.1189
- Bias: +2.7120
- P90 Tail MAE: 7.2139
- P90 Tail RMSE: 9.3916

Transformer:
- RMSE: 7.2685
- MAE: 5.1373
- P90 Tail MAE: 6.5745

### Current Interpretation

CatBoost is currently the strongest model on overall RMSE, achieving
approximately 48% improvement over the persistence baseline.

XGBoost is extremely close to CatBoost and has slightly lower overall MAE
and P90-tail MAE, but slightly higher RMSE.

The sequence models also outperform persistence, but remain substantially
behind the tree-based models:

- Transformer: ~29% RMSE improvement
- LSTM: ~28% RMSE improvement

Therefore, the CatBoost/XGBoost advantage persists even after correcting
the previous leakage problem.

This is evidence that the earlier CatBoost-vs-sequence performance gap was
not solely caused by target leakage.

### Important Status

These are the first model-comparison results considered valid for the
forward-looking 2-day forecasting task.

Previous results produced using the leakage-affected pipeline should not
be used for final model comparison or publication.