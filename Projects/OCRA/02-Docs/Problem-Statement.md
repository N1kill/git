---
tags:
  - prahari
  - problem-statement
  - fixed
status: LOCKED
last-updated: 2026-08-20
---

# Problem Statement

> Prahari Machine-Learning Early-Warning Triage for High-Risk Satellite Debris Conjunctions

## Context

Satellite operators today rely heavily on Conjunction Data Messages (CDMs) — the standardized alerts issued when two tracked objects in orbit are predicted to pass close to each other. As tracking improves over the days leading up to the predicted close approach (Time of Closest Approach, TCA), multiple CDMs are issued for the same event, each refining the risk estimate.

In practice, operational triage tends to lean on the **most recent CDM** as the effective risk signal, even though the full sequence of CDMs for an event carries trend information — how miss distance is shrinking, how covariance is converging, how relative velocity is behaving — that a single snapshot discards.

This matters at national scale too: India's own space situational awareness apparatus (ISRO's IS4OM, operating through ISTRAC) runs recurring conjunction screening (Space Object Proximity Analysis, SOPA) and faces the same snapshot-vs-sequence tension as international operators.

## Problem

High-risk conjunctions are rare relative to the total volume of CDM alerts generated. Analysts must triage a large number of events under time pressure, and current snapshot-based approaches:

- Discard trend information present across a CDM sequence
- Risk late detection of genuinely dangerous events (waiting for the final CDM before acting)
- Provide risk scores without calibrated confidence or human-readable justification

## Proposed Solution — Prahari

Prahari is a sequence-aware machine learning triage system that treats each conjunction event as a **full CDM sequence**, not a single final message, to:

1. Predict collision risk **earlier** than the final CDM (using partial sequences truncated at pre-TCA cutoffs)
2. Beat the naive latest-CDM baseline on rigorous imbalanced-classification metrics
3. Produce **calibrated** risk probabilities an analyst can trust
4. **Explain** high-priority flags in terms of physically meaningful drivers (miss-distance trend, covariance convergence, relative velocity, etc.)

## Dataset

ESA Kelvins Collision Avoidance Challenge dataset — real CDM sequences, event-grouped, published under CC BY 4.0 (Zenodo DOI 10.5281/zenodo.4463683).

## Explicitly Out of Scope

- Real-time official operational feed integration
- Automated maneuver execution / COLA decision-making
- Legal/regulatory certification
- Satellite control-system integration

Prahari is **decision support**, not an autonomous collision-avoidance authority.

## Expected Outcome

For each conjunction event, Prahari outputs:

- A **ranked queue** of active conjunction events, each with a calibrated risk score, triage label (Low / Watch / High-Priority / Critical), miss distance, and relative velocity
- A **calibrated confidence interval** around the risk score (conformal prediction, target coverage reported and empirically verified — not a decorative band)
- A **plain-language explanation** for Watch-and-above events, backed by ranked SHAP feature drivers (miss-distance trend, covariance convergence, relative velocity, CDM sequence consistency)
- A **lead-time gain metric**: how much earlier Prahari flags a high-risk event, in days, versus the latest-CDM baseline — measured across a cutoff sweep (2d / 1d / 12h / final)
- An **audit record** per prediction: model version, cutoff, input features, score, label, explanation, and timestamp

At the system level, the deliverable also includes the evaluation artifacts proving the above: PR-AUC and recall-at-precision-floor tables per cutoff, pre/post-calibration Brier scores, and the degradation curve showing performance as prediction is made earlier relative to TCA.

This is what the current backend schema, dashboard, and evaluation plan (see SRS §4, §6) are built toward — not all of it is implemented yet. Calibration, SHAP explanation, and the orchestrator/gate are still planned; see the SRS status tags for what's live today.


