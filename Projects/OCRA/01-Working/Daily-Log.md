---
tags:
  - prahari
  - journal
last-updated: 2026-08-20
---

# Daily Log:

## 2026-08-21

Completed leakage-safe 2-day model comparison. CatBoost achieved the best validation RMSE (5.3295), followed closely by XGBoost (5.3663). Transformer (7.2685) slightly outperformed LSTM (7.3646). Persistence baseline was 10.2470. CatBoost achieved 47.99% RMSE improvement over persistence. Confirmed that previous ~1.89 RMSE results were from the leakage-affected pipeline and are no longer valid. Corrected the earlier false claim that position covariance determinant columns were exact duplicates of sigma columns. Next: test seed stability/ensembles, investigate P90-tail persistence behavior, then freeze the model and perform final evaluation on untouched ESA test_data.csv.
## 2026-08-20

- Real leakage confirmed and fixed. New pipeline built: `preprocessing.py`, `feature_extraction.py`, `train_catboost.py` — enforces a genuine cutoff boundary (features only from CDMs ≥2 days before an event's terminal CDM), historical risk trend allowed, terminal risk never touches input features.
- Corrected a false claim from earlier in the project: six covariance columns previously asserted as exact duplicates (and dropped) are NOT duplicates in the real data — checked directly, kept in the new pipeline.
- New honest CatBoost validation numbers: RMSE 5.391 / MAE 3.222, vs persistence baseline RMSE 10.247 (~47% reduction). Not comparable to the old 1.89 number — different, harder, honestly-posed task.
- Voided the old CatBoost/XGBoost/LSTM/Transformer comparison table for architectural-claim purposes — logged in `Decisions.md`. test_data.csv still untouched (`test_used: false`, SHA256 fingerprint recorded).
- New blocking item opened: LSTM/Transformer need retraining on the corrected pipeline — their old numbers share the same leakage-class risk and are no longer trustworthy for comparison.
- Updated `Model-Status.md`, `Decisions.md`, `Open-Questions.md`.



## 2026-08-19

- Ran four independent, verified fixes on LSTM/Transformer (missing-value/floor handling, floor-aware loss, standardization stats bug, padding/pack_padded_sequence bug) — none closed the RMSE gap vs CatBoost. Full detail in `Decisions.md`.
- Revised DQ-01: superseded the single-cause "NaN handling" diagnosis. Working hypothesis now structural (dataset size), not a lingering bug — two open forks recorded (diagnostic scan vs accept-and-report).
- Reviewed two new frontend files: `prahari_research_mode.html` (CSV upload, full 103-feature panel) and `prahari_simulation_mode.html` (live 3D orbital sim). Both correctly build the real feature pipeline but still call explicitly-marked placeholder risk functions — confirmed via code, not assumed. Need one `/predict/*` serving layer, not a second model.
- Evaluated and rejected domain-agnostic pretraining (no transfer mechanism). Evaluated domain-adjacent candidates (NASA CMAPSS — fails scale test; Home Credit `installments_payments` — passes scale test, zero domain overlap) — parked, lower priority than resolving the RMSE gap.
- Updated `Architecture.md` and `Model-Status.md` to reflect all of the above.


Running dev journal. Newest entry on top. Keep entries short — this is for "what happened," `Decisions.md` is for "why."

---

## 2026-08-18

- Vault initialized. Created `Model-Status.md`, `Problem-Statement.md` (draft), `Architecture.md`, `Open-Questions.md`, `Decisions.md`.
- Flagged: `OCRA.html` / `prahari_final.html` are demo-only, not authoritative — vault docs supersede them.
- Flagged: no real RMSE/MAE numbers recorded yet anywhere.




<!-- Add new entries above this line, newest on top -->
