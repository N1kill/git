---
tags:
  - prahari
  - decisions
last-updated: 2026-08-20
---

# Decisions Log

Dated entries. Each decision keeps its rationale — don't overwrite, append.

---

## 2026-08-XX — Proceed with current model weights, defer NaN/outlier fix

**Decision:** Continue backend/frontend build using current model weights as-is. NaN/outlier remediation and diagnostic re-run deferred to a later pass.

**Rationale:** Backend/frontend build is the active priority. Blocking on a full remediation pass would stall integration work. The RMSE gap this creates is understood and documented (see DQ-01 in `Model-Status.md`), not a mystery being ignored.

**Risk accepted:** LSTM/Transformer vs CatBoost comparison is not currently clean/defensible for a paper-level architectural claim. Must be re-run post-fix before publishing any "CatBoost is architecturally better" claim.

---

## 2026-08-XX — Calibration and SHAP built against CatBoost only

**Decision:** Build calibration layer and SHAP explainability against CatBoost specifically, not gated behind the full orchestrator/meta-model.

**Rationale:** Calibration and explainability are per-model, post-hoc concerns. Since CatBoost is the confirmed primary model, there's no architectural reason to wait for multi-model routing to exist first. This unblocks progress rather than being a shortcut.

---

## 2026-08-XX — `gate_weights` field made nullable in `schemas.py`

**Decision:** Lock `backend/schemas.py` early as the shared contract, with `gate_weights` nullable.

**Rationale:** Allows incremental component integration (calibration, explainability, dashboard) without breaking the API contract once a real orchestrator is trained later.

---

## Template for new entries

```
## YYYY-MM-DD — [short decision title]

**Decision:** what was decided

**Rationale:** why

**Risk accepted:** (if any) what tradeoff this creates and what would need to happen to close it
```

---

## 2026-08-20 — MVP serving model and uncertainty policy locked

**Decision:** The first real Prahari serving release will use **CatBoost only**. XGBoost, LSTM, and Transformer remain comparison/research models. The trained multi-model orchestrator is explicitly deferred until its prerequisites are met.

**Decision:** **Conformal prediction** is the finalized uncertainty method for the MVP. It will produce a documented prediction interval around the CatBoost output at a declared target coverage. The UI must show a conformal interval or coverage claim only after the method has been fitted on a held-out calibration split and empirically verified.

**Rationale:** CatBoost is the confirmed best current model and gives the project one dependable, explainable serving path. A real learned orchestrator needs stable base models and out-of-fold prediction data; the demo's hard-coded gate weights are not a substitute. Conformal prediction is selected because the operator output requires an uncertainty interval with measurable coverage, rather than a visual heuristic spread.

**Risk accepted:** This MVP does not yet demonstrate that combining model families improves on CatBoost. Any mapping from the model's regression target to a displayed probability must be defined and versioned separately from the conformal interval; conformal prediction supplies uncertainty quantification, not a standalone probability-calibration curve.



## 2026-08-19 — DQ-01 diagnosis revised: NaN handling alone does not explain the gap

**Decision:** Treat DQ-01's original root cause ("CatBoost handles NaNs natively, LSTM/Transformer do not") as superseded, not resolved. Three separate, independently verified fixes were applied to the LSTM/Transformer pipeline and none closed the RMSE gap:

1. Missing-value handling + `risk_at_floor` flag for ESA's -30 risk-floor clamp (~25% of training rows) — no measurable change.
2. Floor-aware weighted loss, replacing the flat 90th-percentile upweighting scheme — no measurable change, slightly worse on some runs.
3. Standardization stats bug fix — `get_feature_stats()` was computing mean/std from `build_summary_features()` output (last-CDM-only, a biased subsample), not the raw per-CDM sequence data the models actually train on. Confirmed via direct comparison: off by 60–75%+ on features like `miss_distance`. Fixed via new `get_raw_feature_stats()`. No measurable change.
4. LSTM padding bug fix — `self.lstm(x)` was running over the full zero-padded tensor with no `pack_padded_sequence`, meaning the LSTM computed through padding on most events (real sequences average ~12.4 CDMs against `max_len=21`, so 40–70%+ of most sequences was padding). Fixed via `pack_padded_sequence`/`pad_packed_sequence`. No measurable change; LSTM test RMSE moved 7.89 → 8.15 (noise-level, not improvement).

**Rationale for recording this as a decision, not just a log entry:** four fixes in a row, each independently verified correct in isolation (confirmed via direct numeric checks, not just code review), with zero net effect on the gap, is itself evidence — not a bug-hunting failure. This shifts the working hypothesis from "there's a hidden bug" to "the gap is likely structural": ~13,150 training events may genuinely be insufficient for LSTM/Transformer to beat CatBoost/XGBoost on this task, consistent with prior published work on this exact dataset (see base paper notes).

**Risk accepted:** Continuing to hunt for a fifth hidden bug has diminishing expected value. The honest next fork is either (a) a diagnostic scan of raw feature values for pathological columns (cheap, ~an afternoon) or (b) accept CatBoost as primary and report LSTM/Transformer as a genuine, literature-consistent underperformance — not a failure, a finding. Neither has been executed yet as of this entry.

**Also decided:** Domain-agnostic pretraining (generic tabular/sequential data, then fine-tune) was considered and rejected — no meaningful transfer mechanism from unrelated domains to orbital CDM features. Domain-adjacent pretraining was evaluated against real candidate datasets (NASA CMAPSS turbofan degradation: ~21.5K rows, same sequential-degradation-to-failure shape but smaller than our own training set, fails the scale test that makes pretraining worth doing; Home Credit Default Risk's `installments_payments` table: 13.6M sequential rows, genuinely passes the scale test, but domain overlap with orbital mechanics is nil) — not pursued for now, lower priority than (a)/(b) above.


## 2026-08-19 — Domain-Adjacent Pretraining (Considered, Currently Parked)

> This is an idea under consideration, not a build decision. The formal evaluation and rejection-of-generic-pretraining decision is logged in `Decisions.md` (2026-08-19 entry). This section exists to keep the idea itself, and the real datasets behind it, visible and findable — not to reopen the decision.

#### Decision:
Instead of training the sequential models (LSTM, Transformer) from scratch only on our ~13,150 real CDM events, pretrain them first on a **larger, structurally similar dataset from a different domain** — repeated, sequential measurements per entity, trending toward an eventual risk/failure outcome — then fine-tune the pretrained weights on our real CDM data.

This is explicitly **not** generic tabular/sequential pretraining (housing prices, weather, etc.) — that was evaluated and rejected, since there's no meaningful transfer mechanism from an unrelated domain to orbital features. The idea that remains live is narrower: pretrain on data that shares our actual *task shape* — sequential entity-level records escalating toward a risk outcome — even if the domain itself (finance, aerospace maintenance) has no direct connection to orbital mechanics.

### Why this might help, and the real constraint that decides whether it's worth doing

Pretraining is only worth doing when the pretraining dataset is **substantially larger** than the target dataset — that's the entire mechanism by which it works (broad exposure before fine-tuning on the smaller, specific target). A pretraining source smaller than or comparable in size to our own 162,634-row / 13,150-event dataset gives no real advantage — it's just training on a different, irrelevant-domain dataset first, for no benefit. This is the single filter every candidate below is checked against.

### Candidate datasets evaluated — with links

**1. Home Credit Default Risk — passes the scale test**
- Kaggle (canonical source): https://www.kaggle.com/c/home-credit-default-risk
- Hugging Face mirror: https://huggingface.co/datasets/mohameddhameem/home-credit-default-risk
- Shape: 58.5M total rows across linked tables. The relevant sequential component is `installments_payments` (13.6M rows) and `POS_CASH_balance` (10M rows) and `credit_card_balance` (3.8M rows) — repeated, time-ordered entries per applicant, building toward a binary default-risk outcome. `application_train` (307.5K rows) is the tabular/summary-feature equivalent.
- **Scale check: 13.6M sequential rows vs our 162,634 CDM rows — passes.** This is the one candidate where real pretraining scale exists.
- Domain overlap with orbital mechanics: none. The transfer hypothesis is purely about sequential-escalation-to-risk task shape, not shared features.

**2. NASA CMAPSS Turbofan Engine Degradation — fails the scale test**
- Kaggle: https://www.kaggle.com/datasets/behrad3d/nasa-cmaps
- Hugging Face: https://huggingface.co/datasets/nominal-io/nasa-turbofan-degradation
- Shape: ~21,500 rows, 21 sensor channels per engine per cycle, run-to-failure trajectories — structurally the closest match to CDM sequences of anything found (repeated sensor readings over time, trending toward a failure/danger outcome).
- **Scale check: ~21.5K rows vs our 162,634 — fails.** Smaller than our own dataset, so no pretraining advantage exists despite the strong structural similarity. Kept here for completeness and because it's the best conceptual match, not because it's actionable.

**3. Hugging Face general risk/tabular datasets — checked, not viable**
- Search covered generic "risk prediction" and churn/credit-default terms on Hugging Face directly (`hf://datasets` search). Results were consistently small (hundreds to tens of thousands of rows), single-snapshot tabular (no sequential structure), low-traction. None cleared either the scale bar or the sequential-structure requirement. Not linked individually — none are strong enough candidates to be worth listing as options.

### Current status

Parked, not pursued. Even the one candidate that passes the scale test (Home Credit) requires a real, separate research effort to validate the transfer hypothesis (does sequential-escalation-to-risk structure actually transfer across domains this different — untested, not assumed). Lower priority than resolving the open LSTM/Transformer question (see `Model-Status.md` "Post-Fix Results") — pretraining on more data doesn't address a gap that four independent, verified fixes already failed to close, if that gap turns out to be structural rather than data-volume-limited.


## 2026-08-20 — Leakage-safe CatBoost pipeline built and validated; old comparison numbers voided

**Decision:** Adopt the new `preprocessing.py` / `feature_extraction.py` / `train_catboost.py` pipeline as the correct, leakage-safe methodology going forward. Treat every model comparison number produced before this date (CatBoost 1.89/2.88, and all LSTM/Transformer numbers from the DQ-01 and Post-Fix-Results entries) as **void for architectural comparison purposes** — not deleted from the record, but no longer usable to claim any model is "better" than another.

**Rationale:** The terminal-risk leakage confirmed in the "Saved Artifact Verification" entry (2026-08-20, `Model-Status.md`) was real. `delta_risk` and `risk_slope` — the artifact's two highest-importance features — were computed from the same terminal `risk` value being predicted. The new pipeline enforces a genuine cutoff boundary (`context = g[g[TIME_COL] >= terminal_time + horizon_days]`) so historical risk trend remains a legal feature while the terminal answer never enters the input.

**Also corrected:** the claim that six covariance columns were exact duplicates (previously used to justify dropping them from `RAW_FEATURE_COLS`) was checked directly against the real `train_data.csv` and found false. The new pipeline keeps all 97 valid numeric CDM inputs.

**New honest baseline:** CatBoost on the leakage-safe task: validation RMSE 5.391, MAE 3.222, vs a persistence-baseline RMSE of 10.247 (~47% reduction). This is a harder, honestly-posed task than the old one, so it is not a regression — it is the first trustworthy number this project has produced.

**Risk accepted / carried forward:** LSTM and Transformer have NOT been retrained on this corrected pipeline. Their old numbers were computed on data with the same class of leakage risk (no cutoff boundary applied in the old `build_sequences()`). Until they're retrained on the new pipeline, there is no valid comparison between model families — the "structural, dataset-size-limited" hypothesis from the 2026-08-19 DQ-01 revision entry is now unconfirmed pending this retrain, not disproven, just no longer resting on trustworthy numbers.

**Process note:** test_data.csv remains untouched (`test_used: false` in the model manifest, SHA256 fingerprint recorded). The experimental ladder going forward: persistence baseline → current-risk-only CatBoost → full cutoff-safe CatBoost (done, this entry) → + feature ablations → best model → conformal calibration → freeze → test_data.csv once → final result. LSTM/Transformer retrain on the corrected pipeline should happen before or alongside the ablation step, not after.


## 2026-08-20 — Persistence baseline recomputed per horizon, every time

**Decision:** The persistence baseline (predict terminal risk = last observed risk within the allowed context) is never a single fixed number. It must be recomputed at the exact same `horizon_days` cutoff as whatever model experiment it's being compared against, every time.

**Rationale:** A baseline computed at a different horizon than the model it's being compared to isn't a valid comparison — an easier (smaller horizon, more recent context) or harder (larger horizon, staler context) baseline would make any RMSE improvement claim meaningless. Given horizon is expected to be experimented with (2d/1d/12h cutoffs per the original evaluation plan), this has to be a standing rule, not a one-time calculation.

**Implementation rule:** every results table/comparison going forward — CatBoost, XGBoost, LSTM, Transformer, at any horizon — includes a persistence-baseline row computed at that identical horizon. No model result is reported without its matching-horizon baseline alongside it.



---

## 2026-08-21 — Decision: Adopt Leakage-Safe 2-Day Benchmark

### Decision

The corrected cutoff-safe pipeline is now the official development benchmark for Prahari.

All model comparisons must use the same:

- event-level train/validation split
- 2-day forecast horizon
- cutoff rule
- preprocessing
- missing-value handling
- sequence normalization
- target definition
- evaluation metrics

### Benchmark

Persistence baseline @ 2 days:

- RMSE: 10.246984

Best current model:

- CatBoost
- RMSE: 5.329520
- Improvement vs persistence: 47.989%

### Model comparison

CatBoost and XGBoost are considered closely matched.

CatBoost:
- lowest RMSE
- highest R²

XGBoost:
- slightly lower MAE
- slightly lower P90-tail MAE

No claim of statistically significant superiority will be made until seed stability / repeated evaluation is performed.

### Sequence models

Transformer currently outperforms LSTM in RMSE:

- Transformer: 7.268497
- LSTM: 7.364551

Both remain behind the tree-based models on overall RMSE.

### Evaluation policy

`test_data.csv` must remain untouched during development and hyperparameter selection.

It may only be used for final held-out evaluation after the model configuration is frozen.


## 2026-08-21 — Corrected Pipeline Becomes Authoritative Evaluation Method

### Decision

The leakage-safe cutoff pipeline is now the authoritative methodology for
Prahari model evaluation.

For a forecasting horizon of `HORIZON_DAYS = 2.0`:

cutoff_time = terminal_time + 2 days

context = CDMs where time_to_tca >= cutoff_time