
---
### 2026-08-25 — Additional dataset sources (open question, not yet decided)

**Trigger:** Nikhil asked about datasets beyond ESA Kelvins Collision Avoidance Challenge.

**Candidates found:**
- **Space-Track.org (transitioning to TraCSS, an 18 SDS -> Commerce Dept handoff)** — real CDM archive w/ RCS in single product. Requires operator/researcher registration; bulk historical CDM pull is not trivial, likely a slow scrape/API job, not a one-shot download.
- **CelesTrak SOCRATES / SOCRATES Plus** — free, live, TLE-based conjunction screening (~148k conjunctions/week in recent snapshot). NOT in ESA CDM schema (no covariance terms matching our 91 raw feature cols) — would need a new extraction pipeline, not a drop-in dataset swap.
- **ISRO/IS4OM public CDM archive** — not confirmed to exist; would need to check directly if we want India-specific data to match project framing.

**Claude's recommendation (not yet actioned):**
- Don't treat this as a training-data swap — schema mismatch means real engineering cost.
- Higher-value, lower-cost option: use live CelesTrak SOCRATES data as a **generalization sanity check** (unlabeled, does the trained model behave sanely on current real orbital geometry?) rather than a second training set.
- Flag for paper: if we do add SOCRATES validation, must NOT claim it in the paper until the same leakage/quality checks (NaN scan, outlier scan, 2-day cutoff leakage check) applied to ESA data are also applied here. Same status-tagging discipline as SRS (IMPLEMENTED vs PLANNED).
- Current known deferred items (NaN/outlier fix, orchestrator, unconfirmed CatBoost/XGBoost input format) take priority over any second-dataset expansion — this is scope expansion, not scope completion, until those are closed.

**Status:** Open question — no decision made. Revisit after backend input-format is confirmed and NaN/outlier fix is scheduled.
