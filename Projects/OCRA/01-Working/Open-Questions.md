---
tags:
  - prahari
  - tracking
last-updated: 2026-08-20
---

# Open Questions

Unresolved items. Move to `Decisions.md` once resolved, with the resolution recorded there — don't just delete from here.

## Blocking / Needs Input

## Blocking / Needs Input

- [ ] **Real training data location:** RESOLVED — `train_data.csv` (162,634 rows, 13,154 events) and `test_data.csv` (24,484 rows, 2,167 events) are both confirmed present and correct. See `Decisions.md` 2026-08-20 entry.
- [x] **Serving-feature parity and target leakage:** RESOLVED 2026-08-20. New cutoff-safe pipeline (`preprocessing.py`, `feature_extraction.py`, `train_catboost.py`) fixes this. See `Decisions.md` 2026-08-20 entry and `Model-Status.md` "Leakage-Safe Retrain" section for full detail.
- [ ] **LSTM/Transformer retrain on corrected pipeline — NEW, blocking.** Every LSTM/Transformer number recorded to date was computed on the old pipeline, which had the same class of leakage risk (no cutoff boundary in `build_sequences()`). Must retrain both on the new `feature_extraction.py`/`preprocessing.py` cutoff-safe examples before any CatBoost-vs-sequence-model comparison can be trusted. See `Model-Status.md` "Leakage-Safe Retrain" section.
- [ ] **Conformal calibration artifact:** the current saved CatBoost model has no accompanying calibration split, fitted conformal artifact, target coverage, or empirical held-out coverage record. These are required before the UI may show a conformal interval. Still open — the new CatBoost candidate is a validation-only run, not yet frozen for calibration.
- [x] **Real RMSE/MAE numbers** for CatBoost, XGBoost, LSTM, Transformer — recorded, but see above: LSTM/Transformer numbers are stale/pre-leakage-fix and need re-running.

## Deferred (known, not urgent)

- [ ] NaN/outlier fix (46 NaN features, 21 outlier features, sentinel values) — deferred in favor of backend/frontend build. Re-run LSTM/Transformer after fix for a clean architectural comparison.
- [ ] Owner/timeline column for SRS evaluation plan — waiting on calibration split date being locked.
- [ ] Interaction owner column for SRS evaluation plan — same blocker as above.

## To Decide

- [ ] Whether `Evaluation-Plan.md` gets built before or after calibration split date is locked (chicken-and-egg — could stub it now with TBD dates).
