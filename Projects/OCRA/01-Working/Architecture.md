---
tags:
  - prahari
  - architecture
  - conceptual
status: high-level — not implementation-accurate, see note
last-updated: 2026-08-19
---
	
# Architecture (High-Level / Conceptual)

> This is the conceptual shape of the system, confirmed to roughly match. It is NOT a 1:1 map of the actual repo/modules — for that, a separate implementation-accurate doc should be built later referencing real files (`backend/schemas.py`, `orchestrator/gate.py`, `orchestrator/meta_features.py`, `calibration/conformal.py`, `explain/shap_layer.py`, `explain/reason_text.py`) once the build stabilizes.

## Pipeline Shape

```
ESA CDM Archive
      │
      ▼
Data Ingestion  (schema validation, event_id grouping)
      │
      ▼
Sequence Builder  (sort CDMs per event by time-to-TCA, ragged sequences)
      │
      ▼
Feature Store  (summary features + tensors, versioned by cutoff + split)
      │
      ├──────────────┬──────────────────┐
      ▼              ▼                  ▼
Baseline Models   Core Model(s)    Sequence Models
(naive latest-CDM) (CatBoost —     (LSTM / Transformer —
                    primary,        comparison, currently
                    confirmed best  affected by DQ-01 NaN
                    RMEE)           confound, see Model-Status)
      │              │                  │
      └──────────────┴──────────────────┘
                      │
                      ▼
          Conformal Uncertainty Layer
       (conformal prediction interval around CatBoost,
        with declared target coverage and empirical
        held-out verification; no heuristic CI claims)
                      │
                      ▼
               Triage Engine
          (Low / Watch / High-Priority / Critical
           via documented thresholds)
                      │
                      ▼
             Explainability Layer
          (SHAP TreeExplainer on CatBoost →
           plain-language reason text)
                      │
                      ▼
         Operator Dashboard / API
   (ranked queue, risk trend, explanation, audit detail)
```

## Key Architectural Decisions (confirmed)

- **CatBoost is the primary production model.** LSTM/Transformer are comparison/research models, not currently in the serving path. XGBoost is also a comparison baseline. See `Model-Status.md` for the full picture and the DQ-01 confound.
- **Conformal uncertainty and SHAP explainability are built against CatBoost only**, not gated behind a full orchestrator. This is intentional and architecturally correct — per-model, post-hoc uncertainty/explainability doesn't need to wait for multi-model routing to exist.
- **`backend/schemas.py` is the shared contract.** Built and pydantic-validated. `gate_weights` is nullable so the orchestrator can be slotted in later without breaking the API contract — this is deliberate forward-compatibility, not an oversight.
- **The orchestrator/meta-model does not exist yet as a trained component.** Per-event routing between models is currently a placeholder (hardcoded CDM-count thresholds in the demo file `gateWeightsDemo`) — this is DEMO-ONLY and does not reflect a real trained gate. Per-event disagreement and missingness are being logged now specifically to enable training a real orchestrator later.
- **Two-file dataset loading**: `train_data.csv` + `test_data.csv` loaded separately. The ESA test set is hand-curated to over-represent high-risk events — this matters for how metrics should be interpreted (test set is not a random sample of the true class distribution).

## Explicitly Not Yet Built

- Trained orchestrator / meta-model for per-event model routing
- Conformal uncertainty layer (interval generation and held-out coverage verification) + SHAP explanation service
- **New:** research-mode and simulation-mode frontend files (`prahari_research_mode.html`, `prahari_simulation_mode.html`) exist and correctly build the full real feature-vector pipeline, but both still call clearly-marked placeholder risk functions (`placeholderRiskModelA`, `placeholderRiskModel`) instead of a real trained-model endpoint. Swap points are explicitly commented in both files. Needs a `/predict/*` serving layer (trained model + calibration + SHAP + explanation text) before either produces real numbers — not a second model, a serving/formatting layer around the existing pipeline.

## Relationship to Demo Files

This doc (`Architecture.md`) and `Model-Status.md` are the source of truth going forward, not the demo HTML.





## Future Development — [[Future Development — Orchestrator]]

