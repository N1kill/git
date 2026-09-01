# Oracle — Model Status

## Version history
| Version | Accuracy | Notes |
|---|---|---|
| v0 | 65% | baseline |
| v1.0 | 78% | overfit |
| v1.1 | 74.48% | more honest, walk-forward validated |

## Current issue
Real production verification data shows 7d and 30d accuracy **well below training estimates**. Both models are overdue for retraining — substantial new verified rows are now available.

**Recommendation:** immediate retraining with updated hyperparameters.

## Data quality bugs fixed (in `prediction_verification.py`)
1. Direction-blind correctness logic
2. UK stock pence/pound price mismatch
3. Missing extreme return filter
4. Batch ticker assignment bug

**Key learning:** most production accuracy issues traced back to subtle data handling errors (unit mismatches, direction-blindness) — not model architecture problems. Data bugs are expensive and easy to miss.

## Philosophy
Oracle explicitly frames outputs as **probabilities, not predictions**. This shapes architecture: calibration steps, honest validation, directional correctness checks all exist to protect this distinction. Calibration over raw accuracy — a well-calibrated 60% beats a false 80%.
