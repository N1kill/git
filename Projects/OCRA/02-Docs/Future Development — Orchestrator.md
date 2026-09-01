> Everything in this section is a plan, not a built component. Nothing here should be read as current system behavior — see `Model-Status.md` and the "Explicitly Not Yet Built" section above for what's actually running today.

### The idea

Right now the pipeline has one production model (CatBoost) and two under-evaluation sequence models (LSTM, Transformer) that are not in the serving path. The planned orchestrator is a **learned meta-model that sits above the trained base models**, combining their individual predictions into one final output per event — rather than the system always relying on a single hardcoded model.

This is a binding layer across the two model families that currently exist side by side but never interact:

- **Tabular family** — CatBoost (primary), XGBoost (comparison). Both consume engineered summary features: one row per event, built by collapsing a CDM sequence into deltas, trends, and the most recent message's values.
- **Sequential family** — LSTM, Transformer. Both consume the raw, ordered, padded CDM sequence directly — every message in the event, not a collapsed summary.

These two families currently produce two independent, uncombined predictions per event. The orchestrator's job is to bind them into one.

### How it's planned to work

1. **Inputs to the orchestrator, per event:**
   - The point prediction from each trained base model (CatBoost, XGBoost, LSTM, Transformer)
   - A small set of meta-features describing the event itself: number of CDMs received so far, time-to-TCA at the current cutoff, covariance trend, and per-model prediction variance/disagreement

2. **What it outputs:** one final combined risk prediction per event — either a learned weighted combination of the base models' outputs, or (the current leaning, pending a build decision) a small trained meta-model (logistic/linear regression) that learns the combination weights from data rather than having them hand-set.

3. **Why meta-features matter, not just the four raw predictions:** the orchestrator needs some signal for *when* to trust the tabular family over the sequential family and vice versa. Early in an event (few CDMs received), the sequence models have very little real sequence to work with — the tabular models' engineered trend features may be more reliable at that point. Later in an event (many CDMs, closer to TCA), the sequence models have a fuller history to draw on. The meta-features give the orchestrator the context to weigh this per event, rather than applying one fixed global weighting to every event regardless of how much sequence exists.

4. **Training data for the orchestrator:** requires the four base models' predictions and true errors, recorded per validation event — which is why per-event disagreement and missingness are already being logged now (see "Key Architectural Decisions" above), specifically so this data exists once the orchestrator is actually built. The orchestrator cannot be trained before the base models are stable and their outputs trustworthy.

### Why this is sequenced after, not alongside, the current work

The orchestrator needs stable, trusted base-model outputs to learn from. Given the open LSTM/Transformer question (see "Post-Fix Results" in `Model-Status.md` — four fixes applied, gap not resolved, likely structural), building the orchestrator now would mean training a meta-model on top of two base models whose reliability is still an open question. The honest order is: resolve or accept the sequence-model gap first, then build the orchestrator on top of whatever the final, trusted base-model set turns out to be — which may end up being all four models, or may end up being just the tabular pair if the sequence models are ultimately not included in the serving path at all.

### What this replaces

The current `gateWeightsDemo` function (hardcoded CDM-count thresholds, demo-only) is the placeholder this orchestrator is meant to replace — same as `placeholderRiskModelA`/`placeholderRiskModel` are placeholders for the eventual `/predict/*` serving layer. Not a new idea being introduced from scratch; this section makes concrete a binding mechanism that was already implied by `schemas.py`'s nullable `gate_weights` field.


### Build Workflow (Training Time) — Step by Step

This is the order the orchestrator actually gets built in, once its prerequisite (stable, trusted base models) is met:

**Step 1 — Freeze the base models.**
CatBoost, XGBoost, LSTM, Transformer are each fully trained and locked. No orchestrator work starts before this — its whole training set depends on these four being stable, since retraining a base model later would invalidate every orchestrator training example collected before that point.

**Step 2 — Generate the orchestrator's training set.**
Run all four frozen base models against every event in the validation split. For each event, record:
- Each model's raw point prediction (4 numbers)
- The event's meta-features at that cutoff: CDMs received so far, time-to-TCA, covariance trend, inter-model prediction variance (how much the 4 models disagree with each other)
- The true `risk` value for that event (the label)

This produces one training row per validation event — not CDM rows, event-level rows — where the input is `[catboost_pred, xgboost_pred, lstm_pred, transformer_pred, cdm_count, time_to_tca, covariance_trend, prediction_variance]` and the target is the same true `risk` value the base models were trained against.

**Step 3 — Train the meta-model on that dataset.**
Fit a small model (logistic/linear regression, per the current leaning) on the Step 2 dataset. This is a much smaller, simpler training problem than any of the four base models — 8 or so input features, one output, trained on however many events are in the validation split. The meta-model learns, from real data, when the tabular family's prediction should dominate the final output versus when the sequential family's should.

**Step 4 — Evaluate the orchestrator against the same baselines the base models were judged against.**
Run the trained orchestrator on the held-out `test_data.csv` split. Compare its RMSE against: the naive latest-CDM baseline, CatBoost alone, and (if included) the sequence models alone. The orchestrator only earns a place in the real pipeline if it beats the best single base model — if it doesn't, that's a legitimate, reportable finding (the base models may already be too correlated for combining them to add anything), not a failure to hide.

**Step 5 — Lock and version the orchestrator, same discipline as the base models.**
Once trained and evaluated, the orchestrator's weights get saved and versioned the same way `Model-Status.md` tracks the four base models today — so its own performance is recorded, not just assumed.

### Runtime Workflow (Inference Time) — What It Looks Like Live

This is what actually happens per event, once the orchestrator is wired into the serving layer:

```
New CDM arrives for an event
        │
        ▼
Feature Store builds BOTH representations
  (summary features AND padded sequence)
        │
        ├─────────────┬─────────────┬─────────────┐
        ▼             ▼             ▼             ▼
    CatBoost      XGBoost         LSTM       Transformer
   (tabular)      (tabular)   (sequential)   (sequential)
        │             │             │             │
        └─────────────┴──────┬──────┴─────────────┘
                              ▼
                   4 raw predictions +
                   meta-features for this event
                   (CDM count, time-to-TCA,
                    covariance trend, disagreement)
                              │
                              ▼
                        Orchestrator
                (trained meta-model — combines
                 the 4 predictions into ONE
                 final risk value, weighted by
                 what the meta-features say about
                 which family to trust right now)
                              │
                              ▼
                     Calibration Layer
                ()
                              │
                              ▼
                Triage Engine → Explainability Layer
                              │
                              ▼
                  Serving Layer → Dashboard
```

**What actually changes from today's pipeline:** right now, a new CDM only ever flows through CatBoost, straight to calibration. Under this plan, it flows through all four base models in parallel, then through the orchestrator, and only the orchestrator's single combined output continues on to calibration — the four individual model outputs are consumed by the orchestrator and never shown to the dashboard directly. The dashboard and everything downstream of calibration doesn't need to change at all; from its perspective, it's still just receiving one risk value per event, same as today.

**What the explainability layer gains, optionally:** since the orchestrator's meta-features include which family it leaned on for a given event, the explanation text generation could eventually say something like "flagged primarily on sequence-based signal, given the event's long CDM history" — a genuine, real reason grounded in the orchestrator's own weighting, not an invented detail. This isn't required for a first version, but it's a natural extension once the orchestrator itself exists.