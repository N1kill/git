
---
### 2026-08-26 — Isolation test complete: architecture effect on tail accuracy CONFIRMED (partially independent of loss weighting)

**Trigger:** Confirmed via bootstrap CI that LSTM/Transformer beat CatBoost/XGBoost on high-risk tail MAE (previous entry), but LSTM/Transformer's `train_sequence_model` uses `high_risk_weight=5.0`/`floor_weight=0.3` loss reweighting that CatBoost/XGBoost never received — needed to isolate whether the tail advantage was architecture or just reweighting.

**Method:** Retrained CatBoost and XGBoost with identical sample-weighting rule (same floor/high-risk threshold logic, same 0.3/5.0 weights) as `train_sequence_model`'s loss function. Compared bootstrap 95% CIs on tail MAE across all six model variants (orig CatBoost, reweighted CatBoost, orig XGBoost, reweighted XGBoost, LSTM, Transformer).

**Results:**

| Model | Watch MAE | High-bucket MAE (95% CI) |
|---|---|---|
| CatBoost (orig) | 2.55 | 8.61 [7.72, 9.53] |
| CatBoost (reweighted) | 3.10 | 7.06 [6.27, 7.88] |
| XGBoost (orig) | 2.56 | 8.25 [7.39, 9.13] |
| XGBoost (reweighted) | 2.72 | 8.36 [7.50, 9.25] (no improvement — got slightly worse) |
| LSTM | 4.33 | 4.95 [4.27, 5.72] |
| Transformer | 4.11 | 4.64 [3.95, 5.43] |

**Verdict:** CatBoost(reweighted) high-bucket CI [6.27, 7.88] does NOT overlap LSTM [4.27, 5.72] or Transformer [3.95, 5.43]. **LSTM/Transformer still win the tail even after matching the loss-weighting treatment.** Reweighting alone explains only part of the original gap (closed CatBoost's gap by ~1.5 MAE points, did essentially nothing for XGBoost) — architecture contributes something real beyond the reweighting.

**Important nuance — do not oversimplify in the paper:** Reweighting CatBoost traded watch-bucket accuracy for tail accuracy (watch MAE worsened 2.55→3.10 to buy the tail improvement) — same trade-off sequence models already make via their loss weighting. So this is NOT "LSTM wins outright" — CatBoost (either variant) still has better watch-bucket (average-case, 90% of events) accuracy than LSTM/Transformer. The honest claim is a precise, bounded one:

**Paper-ready claim (verified across 3 checks — bootstrap CI, sample-size robustness via n=142/1277, and loss-weighting isolation):** "Sequence architecture (LSTM/Transformer) provides a statistically confirmed tail-accuracy advantage on high-risk conjunctions that persists after controlling for loss-function reweighting, though CatBoost/XGBoost retain better average-case (watch-bucket) accuracy under either weighting scheme. This suggests sequence-aware modeling is complementary rather than strictly superior — motivating the orchestrator/gating approach (route high-risk-leaning events to sequence models, route routine events to tree models) rather than picking a single best model."

**Caveat to retain from earlier entry:** CatBoost/XGBoost evaluated on `yva` (tabular examples), LSTM/Transformer on `yva_seq` (sequence examples) — same cutoff rule, independently built, not guaranteed identical event sets. Note in paper limitations.

**Status:** This is now a validated, defensible finding — safe to build into the paper's results section and to use as the motivating evidence for the orchestrator/gate design (previously PLANNED, not yet built). This is the first concrete data-driven justification for why the orchestrator should exist at all, rather than just picking whichever single model has the best overall RMSE.

---
### 2026-08-26 — Event-alignment check: PASS, no rework needed

**Trigger:** Before building anything on top of today's bucket/bootstrap-CI findings, verified that `build_cutoff_examples` (tabular path, used for CatBoost/XGBoost) and `build_cutoff_sequences` (sequence path, used for LSTM/Transformer) select the same event set from `val_proc` — since both apply the same 2-day cutoff rule but iterate/filter independently, nothing had structurally guaranteed agreement.

**Result:** 1419 events in both, 0 only-in-tabular, 0 only-in-sequence. Row counts match `yva` (1419) and `yva_seq` (1419) exactly.

**Conclusion:** All bootstrap CI comparisons and the reweighting isolation test from the last two entries are confirmed valid — comparing the same event population throughout, no rework needed. The caveat noted in those entries ("not guaranteed identical event sets") is resolved — can drop that caveat from future write-ups of this finding.

**Plan of action agreed with Nikhil (in order):**
1. ~~Event-alignment check~~ — DONE, passed.
2. Build orchestrator — starting rule-based (route by whether CatBoost's own prediction crosses the same 90th-percentile threshold already used in loss weighting: above -> defer to sequence model, below -> use CatBoost), not the full learned meta-model yet. Justified now by the confirmed complementary-not-weaker finding.
3. Learned meta-model upgrade (gating features: cdm_count, time_to_tca, covariance_trend_slope, risk_estimate_variance) — after rule-based version works end-to-end.
4. Run once on test_data.csv — only after orchestrator is locked, not before. Test set has been deliberately untouched so far (confirmed by explicit print statement in v3 notebook Section 5).
5. Write paper results — from test-set numbers as headline, val-set + bucket/CI analysis as supporting methodology evidence.
6. Calibration (MAPIE/crepes) and SHAP explainability — deliberately sequenced AFTER orchestrator, since both should sit on top of the final prediction path, which isn't decided until the orchestrator exists.

**Status:** Ready to start orchestrator (step 2).

---
### 2026-08-26 — Rule-based orchestrator v1: built, tested, hit a real ceiling — motivates learned meta-model

**Built:** Simple binary router — if CatBoost's own predicted risk crosses a threshold (90th percentile of CatBoost's val predictions), defer to Transformer; else use CatBoost. Deliberately simple/auditable v1, not the learned meta-model.

**Result at 90th percentile threshold (-15.69):**
- Orchestrated RMSE: 5.45 — WORSE than CatBoost alone (5.33)
- Orchestrated tail MAE: 7.60 — better than CatBoost alone (8.61) but far short of Transformer alone (4.64)

**Root cause diagnosed:** Routing precision/recall against the TRUE top-10% high-risk bucket (n=142) was only 52.8%/52.8% — of the 142 events routed to Transformer, only 75 were actually high-risk; 67 were false-routes, and 67 true high-risk events were MISSED (stuck with CatBoost's weaker tail prediction). CatBoost's predicted-risk RANK is a noisy, close-to-coinflip proxy for true high-risk status (consistent with R²=0.54 — the model's ranking isn't precise enough for hard thresholding).

**Sensitivity sweep (routing quantile 90%->70%, i.e. routing more events):**

| Quantile routed | n routed | Recall of true high-risk | RMSE | Tail MAE |
|---|---|---|---|---|
| 90% | 142 | 52.8% | 5.45 | 7.60 |
| 85% | 213 | 66.9% | 5.74 | 6.99 |
| 80% | 284 | 78.9% | 6.03 | 6.08 |
| 75% | 355 | 85.9% | 6.20 | 5.49 |
| 70% | 426 | 92.3% | 6.47 | 4.99 |

**Finding: this is a genuine trade-off curve, not a missing sweet spot.** Widening the routed net monotonically improves tail MAE and monotonically worsens overall RMSE — every additional routed event costs average-case accuracy (Transformer's watch-bucket MAE 4.11 vs CatBoost's 2.55) to buy tail accuracy. No threshold on CatBoost's raw prediction alone resolves this; the noise is inherent to using one model's single scalar score as a routing signal.

**Conclusion:** Rule-based v1 has done its job — it's not a discarded failure, it's the evidence needed to justify skipping further threshold-tuning and going straight to the learned meta-model (plan step 3), which can use richer per-event signals (cdm_count, time_to_tca, covariance_trend_slope, risk_estimate_variance) instead of a single noisy scalar. This is a legitimate, reportable ablation for the paper: "a naive threshold-based router on top of CatBoost's own prediction achieves only 53% routing precision, motivating a learned gating approach."

**Status:** Rule-based v1 code complete and evaluated (`orchestrator_v1_rule_based.py`, `orchestrator_diagnosis.py`). Next: learned meta-model using the gating features already scoped in the implementation plan.

---
### 2026-08-26 — Learned meta-model (v2 gate): built, fixed a real bug, still a clean negative result

**Built:** Logistic regression gate using 4 features (num_cdms_so_far, cutoff_time_to_tca, covariance_trend, observed_risk_std), trained to predict "will Transformer beat CatBoost on this event," using a held-out gate-train/gate-eval split within val_proc (1184/790 events) so gate performance isn't measured on data it was fit to.

**Bug found and fixed:** First run had all 4 coefficients at exactly 0.0000, AUC=0.4941 (coin-flip) — root cause was unscaled features (covariance_trend spans ~±6.4e7, num_cdms_so_far spans 1-16) crushed by sklearn's default L2 penalty. Fixed with StandardScaler. Confirmed via univariate separation check (Step 2) that this was genuinely a scaling bug, not zero signal — standardized mean differences of +0.04 to +0.13 SD exist per feature, just small.

**After fix:** AUC improved 0.494 -> 0.576 (weak-but-real signal, confirmed via C-sweep, stable across regularization strengths). Threshold-swept the gate's routing probability from 0.30 to 0.70.

**Result — clean negative:**

| Threshold | % routed | RMSE | Tail MAE |
|---|---|---|---|
| 0.30-0.40 | 100% | 7.20 | 3.87 |
| 0.45 | 88.6% | 6.72 | 4.89 |
| 0.50 | 56.9% | 5.72 | 7.72 |
| 0.55 | 7.1% | 5.43 | 8.55 |
| 0.60-0.70 | 0% | **5.41** | 8.55 |

Best RMSE point in the entire sweep is threshold=0.60, which routes 0% of events — i.e. **the gate's own best answer is "never use the sequence model."** Every threshold that routes anything makes RMSE worse than CatBoost alone (5.41).

**Diagnosis:** AUC 0.58 is too weak to route profitably. CatBoost's tail MAE (8.55) is ~2.2x worse than Transformer's (3.87) on true high-risk events — closing that gap requires real routing precision, and these 4 features (standardized effect sizes 0.04-0.13 SD) don't carry enough signal to hit it. Consistent with rule-based v1's earlier finding (53% precision on CatBoost's own prediction rank was also insufficient) — this is the SECOND independent routing approach to hit the same wall, which strengthens confidence this is a real limit of current signal, not one approach's flaw.

**Conclusion — two legitimate paths forward, no default pick made yet:**
1. Accept the RMSE cost as the price of tail-recall, and explicitly frame the design goal as "catch more true high-risk events, not minimize average error" (a legitimate operational stance, but must be a deliberate stated choice, not backed into).
2. Engineer richer gating signal before concluding orchestration doesn't work — e.g. combine CatBoost's own prediction WITH the 4 existing features (ensemble of signals), or add an explicit CatBoost-vs-Transformer prediction-disagreement feature (large disagreement between two models is often a stronger uncertainty signal than any single input feature).

**Status:** Orchestrator (both rule-based v1 and learned v2) currently does NOT beat CatBoost alone on RMSE with information available today. Not deployed. Awaiting decision on which path (1 or 2) to pursue, or whether to proceed toward test_data.csv with CatBoost as the sole production model (with LSTM/Transformer kept as documented, tail-superior alternatives, not blended) while paper documents this ablation as a real finding: two independent routing approaches were tried and both failed to beat the strongest single model, which is itself informative about how hard it is to safely blend models with a ~50%-recall level of pre-prediction signal.

---
### 2026-08-26 — Third orchestrator attempt (disagreement gate): third independent confirmation of the same ceiling. Stopping this line of attack.

**Built:** Gate using |CatBoost_pred - Transformer_pred| (model disagreement) as a signal — tested alone, then combined with the 4 existing v2 features. Different hypothesis than v1 (CatBoost's own score) and v2 (event-level features): disagreement between independently-trained models doesn't require either model to be individually well-calibrated.

**Result:**
- Disagreement alone: standardized separation -0.27 (STRONGEST single-feature separation across all 3 attempts — stronger than any v2 feature's 0.04-0.13). AUC 0.5647.
- Combined (4 features + disagreement): AUC 0.5797 (barely above v2's 0.5755 alone — disagreement adds almost nothing on top of the existing features).
- Threshold sweep: disagreement-only's best RMSE point was 5.4077 at threshold 0.55 (11.2% routed) vs CatBoost alone's 5.4083 — a difference of 0.0006, i.e. noise, not a real improvement. Correcting an overly naive pass/fail check in the script that initially reported this as "beats CatBoost" — it does not, at any meaningful effect size. Every other threshold in the sweep is clearly worse than CatBoost alone, same monotonic-cost shape as v1 and v2.
- Combined gate's actual best answer was again 0% routed (same as v2's core finding).

**Conclusion — three independent signal types, three confirmations of the same ceiling:**
1. v1: CatBoost's own predicted score (53% routing precision) — failed
2. v2: 4 pre-cutoff event features, AUC 0.58 — failed (best threshold routes 0%)
3. v3: model disagreement, alone or combined, AUC 0.56-0.58 — failed (best real improvement is statistical noise)

This is no longer "haven't found the right signal yet" — three qualitatively different signal types (single-model confidence, event characteristics, cross-model disagreement) all converge on the same wall. **Real finding for the paper:** at the current average-case accuracy gap between CatBoost and Transformer (2.55 vs 4.11 MAE on the ~90% watch-bucket), no cheap available signal is strong enough to route profitably on overall RMSE. This is itself a legitimate, citable methodological result — not a wasted line of investigation.

**Decision: stopping further orchestrator-signal attempts on the RMSE-optimization framing.** Two paths remain, still undecided, both previously identified:
1. Ship CatBoost alone as the production model; document LSTM/Transformer's confirmed tail-superiority as a separate, real finding (not blended); move to test_data.csv.
2. Explicitly reframe the goal as tail-recall (not RMSE) and accept the routing cost as a deliberate operational trade-off, if that framing is judged to better match Prahari's actual purpose (catching dangerous events matters more than average-case accuracy).

**Status:** Orchestration work paused pending this decision. All three attempts' code is preserved (`orchestrator_v1_rule_based.py`, `orchestrator_v2_learned_gate.py`, `gate_scaling_fix.py`, `gate_threshold_tuning.py`, `orchestrator_v3_disagreement_gate.py`) — useful as documented negative results for the paper's methodology/ablation section regardless of which path is chosen next.

---
### 2026-08-26 — Decision: orchestrator will be cutoff-aware (single pooled gate, not 4 separate gates)

**Trigger:** Cutoff sweep revealed the CatBoost-vs-Transformer RMSE gap changes shape dramatically across cutoffs (0.59 at 0.5d -> 1.79 at 3.0d) — see Model-Status.md entry same date for full numbers. All orchestrator attempts today (v1 rule-based, v2 learned gate, v3 disagreement gate, v4 soft-blend in progress) were built/evaluated only at the 2.0-day cutoff.

**Decision:** Rather than building 4 separate cutoff-specific orchestrators, add `cutoff_days` as an explicit feature to the existing gating feature set, and train ONE gate on pooled data across all 4 cutoffs (0.5/1/2/3 days). Lets the gate learn how routing should shift as the gap shape changes, without the maintenance burden of 4 separate models. Evaluate per-cutoff-bucket after training (same discipline as the earlier bucket/bootstrap-CI work) to confirm it isn't just good on average while being wrong at the extremes (0.5d or 3.0d specifically).

**Status:** Not yet implemented. v4 soft-blend script (`orchestrator_v4_soft_blend.py`) currently only uses 2.0-day data — needs this update before its results can be trusted as representative of the full system.

---
### 2026-08-26 — Orchestrator v5: cutoff-aware soft-blend built, running

**Built:** Final orchestrator attempt of the day, incorporating every lesson from v1-v4:
- Soft blend (sigmoid-weighted combination), not hard switch — v1-v3 all failed with hard switching, where any wrong routing decision cost the full RMSE penalty
- `cutoff_days` as an explicit gating feature, pooled training across all 4 cutoffs (0.5/1/2/3 days) — motivated by the cutoff sweep finding that the CatBoost-vs-Transformer gap changes shape across cutoffs (0.59 at 0.5d -> 1.79 at 3.0d)
- 4-model disagreement (std + range across CatBoost/XGBoost/LSTM/Transformer predictions), not just the CatBoost/Transformer pair
- Continuous regression target (catboost_err - sequence_err), not binary classification — richer signal than v2's approach
- Evaluated PER-CUTOFF-BUCKET with bootstrap CI, not just pooled average — designed specifically so a good pooled result can't hide a real loss at one cutoff (this was a gap in earlier framing)

**Status:** Code complete (`orchestrator_v5_final_cutoff_aware.py`), submitted for execution. Retrains 8 models across 2 splits (gate-train/gate-eval) x 4 cutoffs — compute-heavy, similar cost to the full cutoff sweep. Results pending.

**Context reminder for continuity:** this is the 4th distinct orchestration approach tried today (after v1 rule-based threshold, v2 learned classification gate, v3 model-disagreement gate — all three failed to beat CatBoost alone on RMSE). If v5 also fails to win cleanly across all cutoffs, the recommended default per today's accumulated evidence is: ship CatBoost alone as the production model, document LSTM/Transformer's confirmed tail-accuracy advantage (holds across all 4 cutoffs, not just 2.0d) as a separate finding, and treat the four orchestration attempts collectively as a rigorous negative-results contribution for the paper's methodology section.

**Will update with final per-cutoff-bucket results and verdict once the run completes.**



---
### 2026-08-26 — Orchestrator v5 RESULT: no win at any cutoff. Also fixed a real bug (undefined GATING_FEATURE_COLS — v5 assumed v2's kernel state; made self-contained).

**Result (bootstrap CI, per-cutoff-bucket, as designed):**

| Cutoff | CatBoost RMSE (95% CI) | Blend RMSE (95% CI) | Verdict |
|---|---|---|---|
| 0.5d | — | — | CI overlap, no real difference |
| 1.0d | — | — | CI overlap, no real difference |
| 2.0d | 5.449 [4.996, 5.895] | 5.495 [5.037, 5.950] | CI overlap, no real difference |
| 3.0d | — | — | CI overlap, no real difference |

**0/4 cutoffs showed a confirmed win. 0/4 showed a confirmed loss. 4/4 showed "no real difference" via bootstrap CI.** Gate coefficients were dominated by `four_model_disagreement_std` (-3.01) and `range` (+1.77) — the event-level features (cdm_count, time_to_tca, covariance_trend, risk_estimate_variance) contributed almost nothing, consistent with v2/v3's earlier finding that those features carry weak signal (AUC 0.56-0.58).

**This is the fourth and final independent orchestrator hypothesis, and the fourth confirmed failure to beat CatBoost alone on RMSE:**
1. v1 (CatBoost's own score, hard threshold) — failed, 53% routing precision
2. v2 (4 event features, learned logistic gate) — failed, AUC 0.58, best answer routes 0%
3. v3 (model disagreement, alone + combined) — failed, apparent win was 0.0006 RMSE noise
4. v5 (cutoff-aware, 4-model disagreement, soft-blend, pooled across cutoffs) — failed, CI overlap at every cutoff

**Decision: orchestration line of work is CLOSED as of this entry.** Four qualitatively different hypotheses (own-confidence, event features, pairwise disagreement, 4-model disagreement + cutoff-awareness + soft blending) all converge on the same wall, at every horizon tested. This is no longer treated as "haven't found the right gate yet" — it's a rigorous negative result in its own right.

**Reframe for the paper (agreed with Nikhil):** the project's novelty was never "we shipped a working orchestrator." It's the rigorous characterization of the tabular-vs-sequence tradeoff itself — a confirmed, CI-backed tail-accuracy advantage for sequence models (Isolation Test entry, same date) PLUS an honest, methodologically serious investigation into why that advantage resists cheap automation. A silently-successful gate would have been a *less* interesting contribution than this — it would report a number with no insight into why it works. Four rigorous, independent, correctly-null results are a stronger methods section than one unexamined success.

**Status:** CatBoost ships as the sole production model. LSTM/Transformer remain documented, tail-superior alternatives (not blended). All five orchestrator scripts (`orchestrator_v1_rule_based.py` through `orchestrator_v5_final_cutoff_aware.py`, plus `orchestrator_diagnosis.py`, `gate_scaling_fix.py`, `gate_threshold_tuning.py`) preserved as citable negative-results code for the paper's ablation/methodology section.

---
### 2026-08-27 — Oracle ceiling check: confirms real, non-trivial signal exists — but is invisible to every feature tried so far

**Trigger:** Before fully closing the orchestration line, ran one more check that required no retraining: if a perfect oracle gate existed (picks the best-performing model per-event using the true label, in hindsight — not deployable, but establishes the theoretical ceiling), how much would RMSE improve over CatBoost alone? Answers whether the four failed gates lacked the right features, or whether the ceiling itself is too small to be worth chasing.

**Method:** `oracle_ceiling_check.py` — per-event minimum-absolute-error selection across CatBoost/XGBoost/LSTM/Transformer val predictions, bootstrap 95% CI (2000 resamples) on the ceiling improvement, at the 2.0-day cutoff.

**Result:**
- Oracle RMSE: 3.9687 [3.698, 4.241] vs CatBoost alone 5.3295 [5.041, 5.632] — CIs do NOT overlap. **Ceiling improvement: 25.5% [22.6%, 28.7%] — real, not noise.**
- Oracle switches away from CatBoost on 73.9% of events. Per-model pick breakdown: XGBoost 38.5%, CatBoost 26.1%, Transformer 18.4%, LSTM 17.0%.

**Split test (isolates WHERE the ceiling comes from — tabular noise-averaging vs genuine tabular-vs-sequence complementarity):**
- Oracle restricted to CatBoost+XGBoost only: RMSE 4.9895, improvement over best-tabular = **6.4%** (the "boring" ensemble-averaging effect, two correlated tabular models canceling independent noise).
- Full 4-model oracle improvement: 25.5%. **Incremental gain from having LSTM/Transformer available: 19.2 percentage points.**
- **Conclusion: the ceiling is primarily a real tabular-vs-sequence effect (19.2 of 25.5pp), not primarily tabular noise-averaging (6.4pp).** This confirms the four failed gates were not chasing a mirage — genuine exploitable signal exists in principle.

**Status:** Ceiling confirmed real and substantially attributable to sequence-model complementarity. Motivated one more diagnostic before any 6th gate attempt (see next entry) — look at what actually characterizes the events where the oracle picks a sequence model, using the oracle's own choices as ground truth, instead of guessing at features blind like attempts 1-5 did.

---
### 2026-08-27 — Oracle switch diagnosis: NO available feature explains the oracle's routing pattern. Orchestration closed, this time with direct evidence, not just repeated failure.

**Trigger:** Given the oracle ceiling confirmed real signal (previous entry), ran a direct diagnostic — for every val event, label it "sequence-favored" (best of LSTM/Transformer beats best of CatBoost/XGBoost, using true labels) or "tabular-favored," then compare the two groups across every engineered feature already computed in the pipeline (`ENGINEERED_COLS`), via Cohen's d. This uses the oracle's actual choices as ground truth — something none of the five prior gate attempts had, since they all had to guess at relevant features blind.

**Method:** `oracle_switch_diagnosis.py`. Features tested: `num_cdms_so_far`, `cutoff_time_to_tca`, `covariance_trend`, `observed_risk_std`, `miss_distance_slope`, `observed_risk_slope`, `delta_miss_distance`, `delta_observed_risk`, `risk_floor_fraction`, `sequence_length`, `nan_feature_count` — effectively the full `ENGINEERED_COLS` set plus missingness count.

**Result (2.0-day cutoff, n=1419, 502 sequence-favored / 917 tabular-favored):**

| Feature | Cohen's d |
|---|---|
| cutoff_time_to_tca | 0.126 |
| observed_risk_std | -0.109 |
| risk_floor_fraction | -0.102 |
| delta_observed_risk | -0.088 |
| num_cdms_so_far / sequence_length | -0.059 |
| nan_feature_count | -0.057 |
| miss_distance_slope | 0.055 |
| delta_miss_distance | 0.034 |
| covariance_trend | 0.034 |
| observed_risk_slope | -0.011 |

**Every single feature falls below the 0.15 threshold for "negligible" (Cohen's convention).** The largest effect (`cutoff_time_to_tca`, d=0.126) is still in negligible territory. None of the 11 candidates — which is effectively the complete engineered feature set the tabular and sequence models both already have access to — separates sequence-favored events from tabular-favored events by any meaningful margin.

**Interpretation:** The 25.5% ceiling (19.2pp of it attributable to genuine sequence-model complementarity, per the prior entry) is real, but is invisible to every scalar summary statistic currently computed. This is a coherent explanation, not just another dead end: the signal likely lives in sequence *shape* (pattern across timesteps — e.g. autocorrelation, curvature, non-monotonicity) rather than any linear trend/variance/timing summary, which is exactly the class of structure an LSTM/Transformer would pick up on but a hand-engineered scalar feature would not.

**Decision: orchestration is closed, now on stronger grounds than repeated failure alone.** Going further would require engineering genuinely new shape-based features (autocorrelation, second-derivative curvature, DTW-style pattern features) for a 6th gate attempt — a substantial new engineering effort for a ceiling that is real but modest (25.5% total, 19.2pp of practical interest). Diminishing returns are steep; not pursued further.

**Final paper framing (locked):** Three independent, escalating lines of evidence, not one: (1) five gate architectures across two full sessions, all failing to beat CatBoost alone on RMSE; (2) an oracle ceiling check confirming the failure was not due to a nonexistent signal — 19.2pp of real, structural tabular-vs-sequence complementarity exists; (3) a direct feature-separation diagnosis showing that signal is invisible to every scalar feature currently in the pipeline, pointing at sequence *shape* as the likely missing ingredient. This is a substantially stronger, more rigorous negative-results contribution than "we tried a gate and it didn't work" — it characterizes *why*, using the true labels as ground truth once cheaply available, rather than stopping at repeated failure.

**Status:** Orchestration CLOSED. CatBoost ships as sole production model. LSTM/Transformer remain documented tail-superior alternatives. All orchestrator scripts (v1-v5) plus `oracle_ceiling_check.py` and `oracle_switch_diagnosis.py` preserved for the paper's methodology/ablation section. Next: proceed to `test_data.csv` run per the standing plan (step 4, entry 2026-08-26 "Event-alignment check"), then calibration + SHAP.


---
### 2026-08-28 — Orchestration REOPENED: attempt #6 scoped as shape-based soft-blend gate, negative-result framing kept as fallback

**Status:** Script delivered to `/mnt/user-data/outputs/orchestrator_v6_shape_gate.py`, fully tested against both false-positive and true-positive synthetic scenarios. NOT yet run on real data. Next action: wire into the notebook (raw_channels, seq_lengths, yva, pred_cat, pred_lstm, pred_tf already in scope from the diagnostic session) and run the pre-committed bootstrap evaluation for real. Given the modesty of the underlying signal (d=-0.237, single feature) and how conservatively the gate now behaves on borderline cases, a FAILURE verdict (CI includes zero) on real data would not be surprising and should be treated as a legitimate, informative outcome per the original fallback agreement — fold into the negative-result framing as the sixth attempt if so.

---

### FINAL RESULT, 2026-08-28 — Attempt #6 CLOSED: FAILURE, CI includes zero

**Real run:** gate correctly excluded 43 train-fold events with undefined curvature (too-short sequences), fit on the remainder, and the internal holdout safety check found **no (a, b) combination beat pure CatBoost even on gate-train data** — triggered the fallback to effectively-pure-CatBoost before ever reaching final evaluation. Final gate-eval bootstrap: **0.00% RMSE improvement over CatBoost alone, 95% CI [-0.00%, +0.00%]** — a clean, unambiguous null, not a marginal or noisy one.

**Verdict, per the pre-committed bar agreed before this attempt began:** FAILURE. CI includes zero. Closed on the same terms as v5 — no exception for the effort spent finding, confound-checking, and confirming `observed_risk_curvature_normalized` as a real (if modest) separator. A statistically detectable Cohen's d (-0.237 at n=1419) did not translate into gate-usable per-event routing signal - the effect size was real but too small to safely act on, which is itself informative, not a contradiction.

**Attempt #6 summary for the record:**
- Reopened 2026-08-28 on Nikhil's explicit instruction ("build orchestrator at any cost" → scoped to shape-based features, the one gap left open by the 08-27 diagnosis)
- Full sweep: 3 feature families (autocorrelation, curvature, DTW), 15+ individual features, all confound-checked against `sequence_length`/`cutoff_time_to_tca`
- One survivor: `observed_risk_curvature_normalized`, d=-0.237, confirmed independent (grew under normalization, unlike DTW which shrank)
- Gate built with v5's soft-blend mechanism, two real implementation bugs caught and fixed during testing (grid-search overfitting; oracle-label leakage in a test harness) before trusting it on real data
- Final gate-eval bootstrap: null result, CI excludes nothing but zero

**Orchestration status: CLOSED, sixth independent confirmation.** Per the standing 2026-08-27 framing (kept as agreed fallback throughout this attempt): six failed/ruled-out attempts (v1 rule-based, v2 learned classifier, v3 disagreement gate, v5 cutoff-aware soft-blend, oracle switch diagnosis on 11 scalar features, v6 shape-based soft-blend) plus two rigorous diagnostics (oracle ceiling check confirming a real-but-inaccessible 25.5% ceiling; oracle switch diagnosis + shape sweep confirming no available feature family - scalar, autocorrelation, curvature, or DTW - carries enough per-event signal to exploit it) is a stronger, more defensible methods contribution than a silently-working orchestrator would have been. Recommend: ship CatBoost alone as production (unchanged, confirmed since Model-Status.md v1.1), document the six-attempt/two-diagnostic negative-result arc as the paper's orchestration section, and close this line of investigation.

### DTW result, 2026-08-28 (same session)

Real DTW run (post-fix): `dtw_dist_to_seq_favored_ref` d=0.192, `dtw_dist_to_tab_favored_ref` d=0.189 — both borderline-positive, both flagged. **Confound-checked**: the two DTW features correlate with each other at r=0.991 (essentially one signal, not two) and with `sequence_length` at r=0.276/0.337 (meaningfully more shared variance than curvature's confound, ~11% vs ~4.4%). Normalized by sequence_length and rerun: d dropped to 0.162/0.158 — shrank toward the threshold (unlike curvature, which grew under normalization). **Verdict: DTW is mostly a repackaging of sequence_length, weak/redundant support at best, not an independent second pillar.**

**Full sweep verdict (15+ features across 3 families, scalar/autocorr/curvature/sign-changes/DTW):** exactly ONE robust, confound-checked, independent finding — `observed_risk_curvature_normalized`, d=-0.237. Everything else negligible or collapses under confound-checking.

**Nikhil's call:** build a small v6 gate using ONLY curvature_normalized, accepting it as a modest single-feature signal, not a strong one.

---

### orchestrator_v6_shape_gate.py — built, debugged, delivered

**Design (agreed before building):** reuse v5's sigmoid soft-blend mechanism exactly (not hard-pick — soft blend degrades gracefully, matching why v1-v3 failed and v5 was designed the way it was). Single feature only. Evaluation: bootstrap CI on a held-out gate-eval split, same rigor as v5, pre-committed success bar fixed BEFORE seeing results — **SUCCESS only if the RMSE-improvement-over-CatBoost-alone bootstrap CI excludes zero**, even if the point estimate is small. CI-includes-zero closes v6 the same way it closed v5, no exception for effort spent.

**Two real bugs found and fixed during testing (script was NOT trusted on real data until both were resolved):**

1. **Overfitting in gate parameter search.** Original `fit_gate` did blind grid-search minimizing RMSE directly on the full gate-train split — confirmed via synthetic no-signal test that this produces a false SUCCESS (spurious pattern-matching to noise, since with ~2500 grid points *something* will look like it helps on train even with zero true signal). **Fixed:** parameter selection now uses an internal train/holdout split nested inside gate-train, requires the selected (a, b) to beat a null/pure-CatBoost baseline on that inner holdout, and falls back to effectively-pure-CatBoost (a=0, b=-10) if nothing does — pushing the real decision entirely onto the outer gate-eval bootstrap step, which is what actually determines ship/no-ship.

2. **Leakage in the synthetic test harness's baseline (not the real code path)** — an earlier version of the wiring example blended `best_tab_pred`/`best_seq_pred` (oracle-selected per-event using true `yva`, matching how `oracle_switch_diagnosis` itself defines "best" for ANALYSIS) as the actual blend INPUT for the gate. That's valid for diagnosis but not for a deployable gate comparison — it silently gave the gate access to ground-truth-informed predictions no production system would have. **Fixed:** wiring example now blends `pred_cat` (the real, single, confirmed production model) against `sequence_pred = (pred_lstm + pred_tf) / 2` (a real, fixed, deployable averaging policy — no oracle selection), matching what would actually ship.

**Validation after both fixes:** synthetic no-signal tests (5 seeds, correlated-error harness matching real model behavior — shared per-event difficulty affecting all models, not independent noise) now correctly report ~0% improvement / FAILURE or the explicit fallback-triggered warning. Synthetic real-signal tests (3 seeds, curvature genuinely predictive of which model wins, matching the real d=-0.237 direction and magnitude in spirit) show the gate correctly detects and cautiously reports on the signal — including one seed where it conservatively reports FAILURE (CI includes zero) despite a positive point estimate (+3.69%), which is the correct, honest behavior for a modest real-world effect size under sampling noise, not a bug.

**Status:** Script delivered to `/mnt/user-data/outputs/orchestrator_v6_shape_gate.py`, fully tested against both false-positive and true-positive synthetic scenarios. NOT yet run on real data. Next action: wire into the notebook (raw_channels, seq_lengths, yva, pred_cat, pred_lstm, pred_tf already in scope from the diagnostic session) and run the pre-committed bootstrap evaluation for real. Given the modesty of the underlying signal (d=-0.237, single feature) and how conservatively the gate now behaves on borderline cases, a FAILURE verdict (CI includes zero) on real data would not be surprising and should be treated as a legitimate, informative outcome per the original fallback agreement — fold into the negative-result framing as the sixth attempt if so.
---
### 2026-08-28 — Attempt #7 opened: attention-weight diagnostic (near-zero-cost, genuinely new lens)

**Next action:** paste the wiring block (in the script's docstring) into the notebook, run for real, read Cohen's d same as every prior table (0.15 negligible threshold, 0.15-0.25 caution, confound-check against sequence_length before trusting anything that clears the bar - same discipline as curvature/DTW). If this also comes back negligible, move to candidate #2 (meta-learner on model outputs) per the original cheapest-first ordering; if a real signal survives, it feeds either a v7 gate or informs which of the remaining three candidates is worth the larger investment.

**Result:** clean negative across all three attention features - `attn_entropy` d=0.052, `attn_peak_weight` d=0.085, `attn_peak_position` d=-0.076. No borderline case this time, no confound-check needed (nothing cleared even the caution band). Sanity check (`pred_tf_check` vs `pred_tf`) passed, so the result is trustworthy, not an artifact of misalignment.

**Substantive finding, beyond the null gating result:** `attn_entropy` sits at ~0.978-0.979 for BOTH groups - the Transformer's attention is nearly uniform across the sequence on almost every event, not selectively concentrating on particular CDMs in an event-dependent way. This is close to uniform-average pooling with minor noise, not sharp, informative attention. Genuinely answers Nikhil's original "how do CatBoost and Transformer really compare" question at a mechanistic level: the Transformer isn't learning to pick out specific informative observations per-event the way its architecture would allow - whatever advantage it has on certain events isn't coming from selective attention. Worth keeping for the paper's model-comparison section regardless of the orchestration outcome.

**Attempt #7, attention-diagnostic phase: CLOSED, clean negative.** Per the cheapest-first ordering agreed before starting, moving to candidate #2: meta-learner (stacking) on model output values/interactions - never tried across any of the six-plus prior attempts, which all gated on raw-input features or attention, never on the base models' actual output values.
---
### 2026-08-28 — Attempt #7, candidate #2: meta-learner (stacking) on model outputs — built, debugged, delivered

**Status:** Script delivered to `/mnt/user-data/outputs/meta_learner_v7.py`. Code path confirmed correct via the sanity check (meta-learner legitimately beats CatBoost-alone once real signal is present, in both synthetic no-signal and signal-planted scenarios) - the specific synthetic magnitude of improvement (~55-59%) is a harness-realism artifact, not something to expect on real data, so no numeric expectation should be read into it. NOT yet run on real data. Next action: wire into notebook using `pred_cat`, `pred_xgb`, `pred_lstm`, `pred_tf`, `yva` already in scope; read the `diagnose_routing_vs_averaging()` output FIRST (states whether any gain is real routing vs. simple averaging) before trusting the final bootstrap CI verdict.

---

### FINAL RESULT, 2026-08-28 — Attempt #7 candidate #2 CLOSED: FAILURE, CI includes zero

**Real run:** CatBoost alone RMSE 5.2749, fixed linear blend RMSE 5.3101 (slightly WORSE), meta-learner RMSE 5.3703 (worse still). `diagnose_routing_vs_averaging()` correctly reports case (b) is not even in play - neither the meta-learner nor the simple linear blend beats CatBoost alone, so there is no ensembling gain to speak of, let alone real per-event routing. Final bootstrap: **-1.81% RMSE "improvement"** (i.e. a slight loss) over CatBoost alone, 95% CI [-4.17%, +0.42%] - includes zero, and the point estimate itself is negative.

**Verdict, per the pre-committed bar:** FAILURE. This is a cleaner and more informative null than a marginal positive would have been - it rules out BOTH the ambitious claim (genuine per-event routing exists in the output-value interactions) AND the modest fallback claim Nikhil explicitly accepted going in (a simple global ensembling gain). Neither holds. CatBoost alone is not just the best single model - it's difficult to improve on even via a flexible learned combination of all four models' outputs, reinforcing that the real predictive signal concentrates in CatBoost specifically rather than being distributable across the ensemble.

**Attempt #7 status so far:** two candidates run (attention-weight diagnostic: clean negative, all d<0.09; meta-learner/stacking: clean negative, CatBoost alone wins outright). Two candidates remain from the original four (per-cutoff specialized gates; two-stage hard-event isolation) - both materially more expensive to build/test than the two just closed. Given the accumulating pattern - every mechanism tried across seven attempts (rule-based, learned classifier, disagreement gate, cutoff-aware soft-blend, shape-based soft-blend, attention-based, output-stacking) and every feature family (11 scalar, autocorrelation, curvature, DTW, attention distribution, model-output interactions) has failed the same pre-committed bar - the orchestration negative-result case is now substantially stronger than it was even after attempt #6. Recommend revisiting with Nikhil whether the two remaining, more expensive candidates are worth pursuing before the SIH PPT deadline, or whether this is the natural point to close the full orchestration investigation and write it up.


---
### 2026-08-29 — Attempt #8: two-loss gate (TESTAM-adapted, published mechanism) — CLOSED: confirmed FAILURE, worse than doing nothing

**Trigger:** After attempt #7's two candidates both closed negative, Nikhil asked whether published research existed on this exact problem (routing between architecturally different models when the gate signal is weak) rather than trying another guessed feature. Literature search surfaced a real, named failure mode — "expert/routing collapse," where a gate trained on a single classification objective learns to route everything to one side — and a specific published fix: TESTAM (Lee & Ko, ICLR 2024, arXiv:2403.02600) trains the gate on TWO separate losses simultaneously (worst-route avoidance + best-route selection) rather than one, and their own ablation shows the single-loss version barely helps while the two-loss version does. This is mechanically different from every v1-v7 attempt, all of which used exactly one classification/regression objective for the gate.

**Honest gaps flagged before building:** TESTAM's 3 experts are structurally similar (same Transformer backbone); this is 2 architecturally different experts (CatBoost tree ensemble vs Transformer sequence model) — a harder routing problem by construction. TESTAM trains on ~1800+ roads x thousands of timesteps; this gate has ~1400 events total. A null result was flagged in advance as plausible and informative, not a sign of a broken build.

**Validation process caught three real bugs before real data was touched — worth recording in detail since each is a generalizable lesson, not gate-specific:**

1. First "no-signal" synthetic scenario used a shared per-event "difficulty" term added to both models' predictions, which accidentally made `disagreement = |pred_cat - pred_tf|` correlate with Transformer's error (r=0.45) as a pure construction artifact, not real signal — caught by the validation assertion firing.
2. Removing the shared-difficulty term did NOT fix it — `disagreement` and `err_tf` are correlated **by mathematical identity** for any two independent error sources (confirmed algebraically: `disagreement = |noise_cat - noise_tf|` is definitionally entangled with `err_tf = |noise_tf|`, r~0.57-0.59 held even under fully independent noise). **This is a retroactive concern for v3** ("disagreement gate"), which used exactly this feature — v3's null result likely still stands (it failed even WITH this unsafe-but-informative-looking input available), but the feature itself was never a clean one. `disagreement` was removed entirely from this gate's feature set rather than partially trusted.
3. The deeper bug: comparing the learned gate's blended output against "CatBoost alone" is the wrong test. ANY blend of two models with different variances (CatBoost sigma 5.3 vs Transformer sigma 7.0) beats the worse single model for free, via basic variance reduction — nothing to do with routing. Confirmed directly: a fully RANDOM, unlearned routing weight near 0.5 showed the same false "improvement" the real gate did (bootstrap CI [0.79%, 2.37%], excluding zero, from pure noise). **Fixed by changing the baseline to the best possible FIXED (non-adaptive) blend weight**, found via 1D search on gate-train only — this baseline already captures 100% of the "know the two models' relative average quality" free lunch, so only improvement beyond it reflects genuine per-event learned routing. This is a retroactive concern worth checking against v4/v5 (also soft-blends, also compared against CatBoost-alone in their original evaluation) — not asserted as definitely wrong there, but the same trap was structurally available.

**Real run result:**
- Gate-train internal check: gated RMSE 5.8142 vs CatBoost-alone 5.4073 — **gate did not even beat CatBoost on its own training data**, a strong advance warning.
- Training loss curves for both objectives were flat and near the random-guessing baseline across 300 epochs (L_worst 0.6930→0.6862, L_best 0.6932→0.6898, vs ln(2)≈0.693) — the gate did not find learnable structure in its inputs, confirmed by the loss curves themselves, not inferred after the fact.
- Best FIXED blend weight found on gate-train: **w=0.99** (99% CatBoost, 1% Transformer) — even the most generous static combination the data supports is essentially "just use CatBoost."
- Held-out gate-eval: CatBoost alone RMSE 5.2108, best fixed blend (w=0.99) RMSE 5.2139, two-loss gate RMSE 5.7509. **Gate is worse than both.**
- Bootstrap 95% CI vs best-fixed-blend baseline: **[-15.70%, -5.11%]** — confidently, confirmedly WORSE, not just "no improvement."

**Verdict:** FAILURE, and the most decisive one of the whole investigation — not "no signal found" (the flat/negligible pattern of every prior attempt) but "the published mechanism specifically designed to escape single-objective gate collapse converged on a harder version of that same collapse (w=0.99), and per-event routing on top of that near-total collapse made things actively worse." A mechanism grounded in real, peer-reviewed research, properly adapted, and validated through three real bugs before ever touching real data, still landed on the same wall — harder than any single-objective gate did.

**Full tally, nine mechanisms across two sessions, one wall:**
1. v1 — CatBoost's own score, hard threshold — failed, 53% precision
2. v2 — 4 event features, learned classifier — failed, AUC 0.58, best answer routes 0%
3. v3 — model disagreement — failed, apparent win was noise (and used a feature now known to have a built-in mathematical entanglement with error)
4. v5 — cutoff-aware, 4-model disagreement, soft-blend — failed, CI overlap at every cutoff
5. v6 — shape/curvature soft-blend gate — failed, internal safety check found nothing, 0.00% CI [-0.00%, +0.00%]
6. v7-attention — Transformer's own internal attention pattern — failed, all d<0.09 (real side-finding: attention is nearly uniform, entropy ~0.978, not selectively focusing on informative CDMs)
7. v7-meta-learner — flexible stacking on all 4 models' output values — failed, worse than CatBoost alone, CI [-4.17%, +0.42%]
8. Two-loss gate (TESTAM-adapted) — failed, confirmed worse than even a fixed blend, CI [-15.70%, -5.11%]

Plus two independent, clean ceiling diagnostics (4-model oracle split test: 19.2pp genuine sequence-complementarity component; CatBoost-vs-Transformer-only oracle: 19.1% [16.3%, 22.1%] clean ceiling) confirming a real, modest, structurally-present signal that no mechanism across event-features, disagreement, shape, attention, output-stacking, or published two-loss training has been able to safely exploit.

**Decision: orchestration investigation is CLOSED, for the final time, on the strongest possible grounds.** Eight distinct mechanisms, spanning every reasonable category (rule-based, learned classifiers, disagreement, cutoff-aware blending, shape-based, model-internal/attention-based, output-stacking, and a published loss-design specifically engineered to avoid the exact failure mode seen repeatedly) have converged on the same result. The last and most sophisticated attempt failed harder, not softer, than the ones before it — the two-loss mechanism's own best fixed-weight answer was near-total collapse to CatBoost (w=0.99), and adaptive routing on top of that made it worse. This is not "we haven't found the right approach" — this is confirmed, from every reasonable angle available, that no cheap, learnable, per-event routing signal exists between CatBoost and Transformer at this dataset's size and structure.

**Final status: CatBoost ships as the sole production model, unchanged since 2026-08-26.** LSTM/Transformer remain documented, tail-superior alternatives (per the 2026-08-26 Isolation Test entry — that finding stands independently and is unaffected by any orchestration result). All nine orchestrator/gate scripts and both oracle diagnostics preserved for the paper's methodology/ablation section — the eight-mechanism, two-diagnostic negative-result arc, including the specific mathematical/statistical traps discovered and corrected along the way (disagreement-error identity, fixed-blend-vs-naive-blend distinction), is a substantive and defensible contribution in its own right. Proceed to `test_data.csv` run per the standing plan, then calibration + SHAP.



---
### 2026-08-29 — LSTM cross-check: same ceiling, same gate failure, confirming the wall is architecture-independent

**Trigger:** All 8 prior gate attempts and both oracle checks used Transformer, never LSTM, as the sequence-side model. Before finalizing anything, checked whether LSTM's different overall error pattern (worse raw RMSE than Transformer throughout: 7.05 vs 7.01 at 2.0d) might nonetheless be more learnable for routing purposes — genuinely untested ground, not a repeat.

**Method:** Generalized `catboost_vs_transformer_oracle.py` to `catboost_vs_second_model_oracle.py` (same validated core mechanism, parameterized model name/predictions instead of hardcoded Transformer — confirmed via the same validation scenarios, identical pass). Then reran the two-loss gate (`two_loss_gate.py`, confirmed model-agnostic — every use of the "pred_tf" parameter is pure arithmetic, no Transformer-specific logic anywhere) with `pred_lstm` substituted for `pred_tf`.

**Oracle result — CatBoost vs LSTM only:** 19.0% [16.3%, 21.9%], CI excludes zero. **Essentially identical to CatBoost vs Transformer's 19.1% [16.3%, 22.1%]** — same lower bound to the decimal, upper bounds a tenth of a point apart. This is a real and interesting finding in its own right: the ~19% ceiling is not specific to Transformer's architecture — it appears to be a structural property of "CatBoost vs any sequence-based model" on this dataset, not an artifact of which sequence architecture was chosen.

**Two-loss gate result — CatBoost vs LSTM:**
- Gate-train internal check: gated RMSE 5.7806 vs CatBoost-alone 5.4073 — same advance-warning pattern as the Transformer run.
- Training loss curves flat near the random-guessing baseline (L_worst 0.6932→0.6859, L_best 0.6931→0.6798 across 300 epochs) — same as Transformer's run.
- Best fixed blend weight: **w=0.98** (vs Transformer's w=0.99) — again, essentially "just use CatBoost."
- Held-out bootstrap CI vs best-fixed-blend baseline: **[-14.58%, -4.90%]** (vs Transformer's [-15.70%, -5.11%]) — confirmed loss, nearly identical magnitude.

**Conclusion:** The two-loss mechanism fails against LSTM by essentially the same margin, in the same way, as it failed against Transformer. This rules out "maybe this is a Transformer-specific problem" as an explanation for any of the ten attempts' failures — the wall is not about which sequence architecture was chosen. It's a structural property of the CatBoost-vs-sequence-model relationship itself at this dataset's size and structure, independent of which sequence model sits on the other side.

**Decision: orchestration investigation is CLOSED, now confirmed architecture-independent, for the final time.** Ten mechanisms (v1, v2, v3, v5, v6, v7-attention, v7-meta-learner, two-loss-vs-Transformer, plus the LSTM oracle check and two-loss-vs-LSTM just completed) have converged on the same wall, now checked against both available sequence architectures. **Proceeding to `test_data.csv` per the standing plan** — deliberately run exactly once, after this decision is fully frozen, as the final clean confirmation. CatBoost ships as the sole production model; LSTM and Transformer both remain documented, tail-superior alternatives (the 2026-08-26 Isolation Test finding, which used Transformer specifically, stands independently of this cross-check and is unaffected by it).

