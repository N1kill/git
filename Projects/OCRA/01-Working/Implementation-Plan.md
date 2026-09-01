---
tags: [prahari, implementation, plan]
status: active
last-updated: 2026-08-20
---

# Prahari Implementation Plan

## Locked MVP decisions

- **Serving model:** CatBoost only.
- **Comparison models:** XGBoost, LSTM, and Transformer remain offline research/comparison models.
- **Orchestrator:** deferred. `gate_weights` remains nullable in the API contract; demo gate weights must not be represented as a trained component.
- **Uncertainty:** conformal prediction intervals, fitted on a held-out calibration split and reported with target and empirical coverage.
- **Explainability:** actual CatBoost SHAP values, converted into operator-readable reason text.

## Delivery sequence

1. **Data and target audit**
   - Assemble and validate source CDM files with a documented data manifest.
   - Preserve event-level train/validation/test separation.
   - Construct cutoff-specific examples (2d, 1d, 12h, final) without allowing future CDMs or the terminal target into input features.
   - Add checks for schema compatibility, split overlap, target leakage, and deterministic feature ordering.

2. **Reproducible CatBoost release**
   - Train CatBoost from one approved preprocessing and feature-building pipeline.
   - Compare with the latest-CDM baseline at each cutoff.
   - Store the model, feature order, fitted preprocessing values, cutoff, metrics, dataset/split identifiers, and model version as one release artifact.

3. **Serving layer**
   - Implement validated `POST /predict/research` input for a full CDM history.
   - Build features on the server, run CatBoost, add conformal interval, generate SHAP drivers and reason text, apply triage policy, and return audit metadata.
   - Do not require the ground-truth `risk` target in inference input.

4. **Frontend integration**
   - Replace the research-mode placeholder model with the real research endpoint.
   - Preserve demo status for simulation mode until its reduced physical feature vector has a compatible trained model and endpoint.
   - Keep visual output and wording aligned with the returned model metadata; no fabricated routing or coverage claims.

5. **Evaluation and documentation**
   - Produce cutoff-sweep performance, calibration/coverage, lead-time, and explanation examples.
   - Update Model Status and Architecture only with verified results.

## First active task

Audit the existing CSV/data assembly and feature builder before retraining. In particular, distinguish historical CDM `risk` values that are legitimately available at a cutoff from the terminal risk target being forecast. No input feature may be computed from a CDM after the cutoff or from the target label being evaluated.

### Audit note — 2026-08-20

The four locally available `CDM_data_part*.csv` files are synthetic/demo fixtures, not the training source. They are excluded from model development. The actual model source is the ESA Kelvins Collision Avoidance Challenge data.

The saved CatBoost artifact was then inspected directly. It is a healthy 1,997-tree artifact that expects 189 named features, including missingness flags and six engineered summary features. The current checked-in feature builder does not reproduce that full schema. It also calculates `delta_risk` and `risk_slope` using the final `risk` field while setting that same final field as the regression target. This is not a valid live-prediction contract: a terminal target is unavailable at inference time. The next implementation task is therefore a cutoff-safe, artifact-versioned feature builder and CatBoost retrain using the real event-identified ESA data.

### Test-data verification — 2026-08-20

`test_data.csv` is a real ESA evaluation artifact and is now available locally. It contains 24,484 CDM rows, 2,167 unique `event_id` values, 18 missions, and 103 columns. Its time-to-TCA values span 6.9932 down to 2.0002 days, so it is reserved as the final **2-day early-warning** test set. Every raw feature required by the saved CatBoost artifact is present. It must remain read-only: no model fitting, missing-value fitting, threshold tuning, or conformal calibration may use it. The matching `train_data.csv` is still needed for the next build step.

## Cutoff-Safe 189-Feature Rebuild Plan

### Target definition

For a prediction made at a cutoff (initially 2 days before TCA):

- **Input:** only CDMs issued at or before the available-history boundary.
- **Target:** the event's terminal/final risk, held separately from the input dataframe.
- **Rule:** no feature may use a CDM issued after the cutoff or the terminal target value.

### Phase A — Rebuild the feature contract

1. Start from the raw ESA columns and define three explicit groups:
   - identifiers (`event_id`, `mission_id`) — grouping/audit only, never model inputs;
   - labels (`risk` used as terminal target, plus any derived evaluation labels) — never direct model inputs;
   - permitted physical/OD/CDM measurements — model candidates.
2. Retain the saved-artifact pattern: 91 selected numeric raw fields, `c_object_type`, and a missingness flag for every numeric field.
3. Fit numeric fill values from the training partition only; apply them unchanged to validation, calibration, and test.
4. Build sequence-summary features using only history available at the cutoff:
   - CDM count and observed time span;
   - miss-distance and covariance trends;
   - any observed-risk trend calculated between historical CDMs only, never against the terminal target.
5. Save the ordered feature list as a versioned manifest and assert that training and inference produce the same columns in the same order.

### Phase B — Construct valid learning examples

1. Group `train_data.csv` by `event_id`.
2. For every event with sufficient history, construct a 2-day-cutoff example from CDMs available before that cutoff.
3. Attach its separately stored terminal label as `y` after all features have been built.
4. Split by event into training, validation, and calibration partitions; no event may span partitions.
5. Add automated tests that deliberately try to pass a post-cutoff CDM or target column and must fail.

### Phase C — Train and validate CatBoost

1. Train CatBoost using the rebuilt feature table and event-level training partition.
2. Select the iteration count/hyperparameters only from the validation partition.
3. Compare against a declared latest-CDM baseline using the identical target and cutoff.
4. Store the model, feature manifest, fill values, training configuration, dataset fingerprint, and validation metrics together as one model release.

### Phase D — Fit conformal uncertainty

1. Freeze the selected CatBoost model.
2. Use the dedicated calibration partition to fit the conformal interval method at the selected target coverage (initial target: 90%).
3. Save the conformal artifact and its calibration metadata with the model release.
4. Report empirical coverage and interval width on validation/calibration only; the test set remains untouched.

### Phase E — One final test and serving promotion

1. Run the frozen model bundle once against `test_data.csv` at the 2-day cutoff.
2. Report RMSE/MAE, high-risk classification metrics, conformal coverage, interval width, and lead-time context.
3. Promote only if the input manifest exactly matches, no leakage test fails, and test results are recorded.
4. Load that model bundle in `POST /predict/research`; derive real SHAP values and explanation text from the same served feature vector.


