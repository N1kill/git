---
tags:
  - prahari
  - build
  - plan
status: active
last-updated: 2026-08-21
---

# Build — Backend & Frontend Plan

> Derived from `Model-Status.md` (v1.1), `Architecture.md`, `Decisions.md`, and `Implementation-Plan.md`. This doc turns the locked decisions into an actual build sequence. Don't duplicate rationale here — link back to `Decisions.md` for "why."

## Starting Position (as of 2026-08-21)

- **Serving model: CatBoost only**, leakage-safe pipeline, validation RMSE 5.3295 (47.99% over persistence). This is the first trustworthy number the project has produced — build against it.
- **XGBoost/LSTM/Transformer** stay offline comparison models. Not in the serving path.
- **Uncertainty method: conformal prediction** — locked, but not yet fitted. No calibration artifact exists yet (`Open-Questions.md`).
- **Explainability: SHAP TreeExplainer on CatBoost** — locked, not yet built.
- **Orchestrator: deferred.** `gate_weights` nullable in `schemas.py`. Do not build real routing logic yet.
- **Two frontend files already exist** (`prahari_research_mode.html`, `prahari_simulation_mode.html`) and already build the correct feature pipeline client-side — they just call placeholder risk functions (`placeholderRiskModelA`, `placeholderRiskModel`). This is a **swap-in job for research mode**, not a rebuild.
- **test_data.csv is sealed.** Do not touch it during this build. It's used exactly once, at the end, per `Decisions.md`.

---

## Backend

### 1. Freeze the model release artifact
Before writing any serving code, package the current CatBoost result as one versioned release:
- Trained model (`catboost_model.cbm`)
- Feature manifest (189-feature list, ordered)
- Fitted fill values (train-partition only)
- Cutoff config (2 days)
- Dataset fingerprint (SHA256, already recorded)
- Validation metrics (RMSE 5.3295 / MAE 3.1594 / R² 0.5385)

This is the thing `/predict/*` loads. Don't let serving code reach into loose training scripts.

### 2. Lock `backend/schemas.py` as the contract (already decided)
- `gate_weights` stays nullable.
- Add response fields now for: prediction, conformal interval (nullable until Phase D below), SHAP drivers (nullable until built), triage tier, model version/manifest hash, `test_used: false` provenance flag.
- Building the schema with nullable future fields means calibration/SHAP can slot in later without breaking the contract — same pattern already used for the orchestrator.

### 3. Build `POST /predict/research`
Per `Implementation-Plan.md` Phase E:
- Input: full CDM history for one event (matches what `prahari_research_mode.html` already sends).
- Server builds the 189-feature vector using the same `feature_extraction.py`/`preprocessing.py` used in training — not a reimplementation. Import the training code, don't duplicate it.
- Reject any payload containing the terminal `risk` target — this must be structurally impossible, not just undocumented.
- Run CatBoost → raw risk prediction.
- Stub conformal interval and SHAP fields as `null` initially (Phase D/E below fill these in) rather than blocking the endpoint on their completion.
- Apply the triage policy (Low/Watch/High-Priority/Critical) via documented thresholds on the point prediction.
- Return audit metadata: model version, feature manifest hash, cutoff used, timestamp.

### 4. Conformal calibration service (unblocks the interval field)
This is the current top blocker per `Open-Questions.md`.
- Use the dedicated calibration partition (never validation, never test) to fit a conformal predictor at 90% target coverage.
- Persist the conformal artifact alongside the model release, versioned together.
- Report empirical coverage on the calibration split — this must be a measured number before the UI is allowed to display a coverage claim (per the 2026-08-20 decision).
- Wire the fitted interval into `/predict/research`'s previously-null field.

### 5. SHAP explainability service
- SHAP `TreeExplainer` against the frozen CatBoost model only — not gated behind the orchestrator, per the standing architectural decision.
- Convert raw SHAP values into operator-readable reason text (top-N contributing features in plain language).
- Wire into the previously-null SHAP field in the response.

### 6. Leave open, do not block on
- LSTM/Transformer retrain on the corrected pipeline — real blocker for *publishing* a model comparison claim, **not** a blocker for backend build (per the 2026-08-XX "proceed with current weights" decision).
- Orchestrator/meta-model — explicitly deferred, `gate_weights` null is the correct state right now.
- NaN/outlier remediation (DQ-01) — deferred, doesn't affect CatBoost serving since CatBoost handles NaNs natively.

**Backend build order:** release artifact freeze → schemas lock → `/predict/research` with nulled uncertainty/explainability → conformal service → SHAP service → wire both into the endpoint.

---

## Frontend

## Frontend

### 0. Visual direction — decided 2026-08-21, not yet in Architecture.md

**Decision:** The dashboard is not a plain admin panel. Full 3D scene (planets/orbits, purple space theme) runs throughout the app, with dashboard panels (queue, risk trend, explanation, audit detail) floating over/within the 3D view rather than the 3D being a hero-only backdrop. Tech: **Three.js**.

**Open item — verify before building:** `prahari_simulation_mode.html` already has a live 3D orbital sim per `Architecture.md`/`Daily-Log.md` 2026-08-19, but this vault doesn't record what it's actually built with. Check the real file at `.../3rdproject/OCRA/prahari_simulation_mode.html` before starting the 3D layer — if it's already Three.js, reuse its scene/camera/orbit setup as the shared base per your instinct here rather than building a second one from scratch. If it's something else (raw WebGL, a different library), that's a decision point: port it or run two rendering approaches side by side.

**Implication for build order below:** the 3D shell (scene, camera, planet/orbit field, purple lighting/material pass) becomes its own workstream, roughly parallel to backend work, since dashboard panels now need to be built as overlays on top of it rather than a standalone page. Recommend building the panel logic (data fetching, tables, charts) decoupled from the 3D shell so either can be swapped/iterated without blocking the other — panels consume `/predict/research` regardless of what's rendering behind them.

### 1. Research mode (`prahari_research_mode.html`) — swap, don't rebuild
- Already does CSV upload and builds the full 103-feature panel correctly (confirmed via code, per `Daily-Log.md` 2026-08-19).
- The only change: replace the call to `placeholderRiskModelA` with a real fetch to `POST /predict/research`. Swap point is already commented in the file.
- Once the backend returns non-null conformal/SHAP fields, render them — interval band on the risk trend, reason text under each prediction. Do not fabricate a coverage claim or interval in the UI before the backend actually returns a fitted one (standing rule from `Decisions.md`).
- Ranked queue / operator dashboard view: consumes the same endpoint across multiple events, sorts by triage tier then risk.
- **New:** this view now needs to be restructured as panels over the 3D scene rather than a standalone page — see §0.

### 2. Simulation mode (`prahari_simulation_mode.html`) — stays demo-only for now
- Uses a reduced physical feature vector (live 3D orbital sim), not the full CDM history schema.
- Per `Architecture.md`: this needs its own compatible trained model/endpoint before it can go live — that's a separate, later effort, not part of this build pass.
- No action needed on its model logic beyond leaving its placeholder clearly marked as-is. Its 3D scene, however, is now relevant as a possible shared base — see §0.

### 3. Dashboard-level requirements
- Any displayed number must trace back to a real field from `/predict/research` — no hardcoded gate weights, no synthetic coverage numbers, no LSTM/Transformer numbers presented as production-comparable (they're pre-retrain, per `Open-Questions.md`).
- Model metadata (version, feature manifest hash) should be visible somewhere in the UI (footer/audit panel) so the dashboard is traceable to the exact model release it's calling.
- Purple space theme + 3D panel-overlay layout applies across research mode and any shared shell — keep the visual language consistent rather than styling research mode separately from simulation mode.

**Frontend build order:** verify simulation mode's existing 3D stack → stand up the shared Three.js scene/shell (planets, orbits, purple lighting) → build panel components against `/predict/research` decoupled from the 3D shell → wire panels into the shell as overlays → wire research mode to the real endpoint with nulled fields first (unblocks end-to-end testing immediately) → add interval rendering once conformal service ships → add reason text once SHAP service ships → leave simulation mode's model logic untouched, evaluate merging its 3D scene into the shared shell.

## Suggested Sequencing (backend + frontend interleaved)

1. Freeze CatBoost release artifact.
2. Lock `schemas.py` with nullable uncertainty/explainability fields.
3. Ship `/predict/research` returning real predictions, null interval/SHAP.
4. Swap frontend research mode to call the real endpoint — get end-to-end working with nulls visible/hidden gracefully.
5. Build + wire conformal calibration service.
6. Build + wire SHAP explainability service.
7. Frontend renders interval + reason text once both are non-null.
8. Only after all of the above: LSTM/Transformer retrain (unblocks a real model-comparison claim, doesn't block serving) → freeze → one-time `test_data.csv` run → final evaluation write-up.

## Explicit Non-Goals for This Build Pass

- No orchestrator / trained gate.
- No DQ-01 NaN/outlier remediation.
- No simulation-mode real model.
- No touching `test_data.csv`.
