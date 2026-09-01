# System Architecture + Antigravity Build Brief

**Decided 27 Aug 2026.** Two calls made explicitly here that were left open in [[00-Foundation]] — flagging both for team veto, not silently assumed.

---

## Calls made (previously open questions)

**Screen-understanding model:** Transformers.js with a pretrained ViT-based screen/UI-understanding model — not a from-scratch or fine-tuned model.
Reasoning: 15-day PPT-round timeline, zero prior on-device ML experience on the team. PS explicitly says "ViT **or equivalent**" — a correctly-wired pretrained model satisfies this. Training/fine-tuning in this window is not credible.

**Server-side VLM:** Cloud-hosted open-weight VLM via API (Llama-3.2-Vision-class or Qwen-VL-class), not self-hosted.
Reasoning: PS explicitly permits cloud-hosted open-weight models during SIH. Self-hosting adds infra risk for zero scoring benefit in the PPT round.

**Still open, not decided here:** exact VLM provider/API (needs a cost + rate-limit check before demo day — flagged as a blocker risk in the Antigravity brief itself), team split (who owns client vs. server).

## Architecture — trust-boundary split

Two tiers, separated by a hard trust boundary. Client tier runs entirely in-browser (Manifest V3 extension): screen capture → DOM scanner + Screen ViT (Transformers.js/WebGPU) + BlazeFace (ONNX Runtime Web) running in parallel → redaction engine that cross-checks DOM and vision signals per region (both signals firing = high confidence; either alone = redact anyway, since false negatives cost more than false positives for PII) → outputs a redacted frame + JSON manifest. **Only the redacted frame + manifest cross the network** — this is the one architectural rule the whole PS pivots on, and Antigravity's brief calls it out as a critical-bug-not-style-issue.

Server tier: VLM reasoning API takes the sanitized frame + manifest + task, returns a structured action → action planner turns that into `{action, target}` JSON → returns to client. Client-side executor performs the action (click/scroll/fill) directly on the page DOM.

This directly implements two things locked in 00-Foundation: the hybrid DOM+vision redaction approach (innovation angle 02) and the redaction-scheme-aware server manifest (innovation angle 03) — both are now load-bearing parts of the architecture, not optional additions.

## Build priority — inverted from instinct, matches the rubric

Phase 1 (redaction pipeline) before Phase 2 (server reasoning) before Phase 3 (executor/wiring). This mirrors the scoring weights already locked in 00-Foundation: redaction-related metrics are 40% combined vs. 15% for latency, so the brief explicitly tells the agent to build redaction first even though a working "agent clicks a button" demo is the more instinctively satisfying thing to build first.

Phase 4 (visible resource-utilization overlay) is marked nice-to-have, only after 1–3 are solid — ties to innovation angle 04 from 00-Foundation but isn't load-bearing for a passing demo.

## Antigravity build brief — what it is, why it's shaped this way

**File:** `antigravity-brief.md`, shared as a downloadable artifact — meant to be pasted directly into Antigravity's Agent Manager as the task description for an autonomous build.

Written as a hard-bounded spec, not a conversational ask, because agentic coding tools drift without explicit constraints:
- **Non-negotiable constraints stated up front** — the "no raw pixels/DOM leave the browser" rule is stated as a critical-bug-severity constraint, not a nice-to-have, since this is the one thing that would silently break the entire PS's premise if an agent quietly optimized around it (e.g. sending the full frame "just to be safe" on the server side)
- **Tech stack marked as already decided** — explicit "do not re-litigate" framing, since an autonomous agent left to choose its own stack will often reach for something unfamiliar to the team or harder to demo live
- **ASCII architecture diagram inlined in the brief itself** — redundant with this note by design, since the brief needs to be self-contained if pasted into Antigravity without this vault attached
- **Phase-gated with a demo checkpoint per phase** — stops the agent from producing a large, unintegrated pile of code; each phase must run end-to-end before the next starts
- **Explicit out-of-scope list** — the same discipline used in the 15-day build checklist in [[03-Build-Reference-HTML]], now written for an agent instead of a human, since agents are equally prone to scope creep
- **Definition of done is a single testable scenario** — password field + face image test page, verified via Network tab that nothing raw left the browser — not a vague "build the privacy filter" ask
- **Repo structure specified** — prevents an agent from inventing its own layout that then needs to be reverse-engineered by teammates

## Status
Architecture locked, brief ready to hand to Antigravity. Two calls made above are subject to team veto before build starts. Regenerate this note if either call changes.

---
**Vault map for this project:**
[[00-Foundation]] → this note (architecture + Antigravity brief) → [[03-Build-Reference-HTML]] (PS synthesis)


---

## Addendum 27 Aug 2026 — rare feature ideas + model training clarification

### Model training — answered directly
**No model training or fine-tuning is needed anywhere in this build.** BlazeFace, the screen-understanding ViT, and the server VLM are all used pretrained/off-the-shelf. Already true in the original architecture (see "calls made" above) but stated more explicitly now as a hard constraint in the Antigravity brief, not just an implied default — added as constraint #4 alongside the trust-boundary rule.

Fallback if a pretrained model underperforms on test pages: swap to a different pretrained model, or lean more on the DOM-signal path. Never "train a better one" — not achievable credibly in 15 days with zero prior on-device ML experience, and not necessary given the PS's "ViT or equivalent" wording.

### Five rare feature ideas — researched, filtered against generic DLP-tool overlap

Searched the broader privacy/DLP tooling space (Strac, browser DLP products, federated/differential-privacy literature) to confirm which ideas are genuinely rare versus already commercially solved. Finding: **enterprise DLP tools already do OCR-based image/PDF redaction** — so "vision model reads pixels" alone (last session's wedge vs. consumer extensions) is less rare against the *enterprise* DLP category, though still rare against the *consumer browser extension* category researched previously. The five ideas below were filtered specifically against things generic DLP tools don't do, because DLP tools don't execute agent actions — that's the actual differentiation surface.

1. **Action-intent redaction** — redact not just visual content but the *reasoning trail* in the action manifest (e.g. send "primary submit action" instead of a spatially-derived description that could indirectly leak layout/content). Closes a leak class most teams won't think to close since it's not visual redaction.
2. **Fail-closed redaction under DOM mutation** — MutationObserver triggers a targeted re-scan on change; region stays treated as redacted until re-confirmed clean, never fails open. **Added to Phase 1 of the Antigravity brief as build item 7** — small enough to build now, not deferred.
3. **Redaction diffing across agent steps** — send only what changed since the last sanitized state on multi-step tasks, instead of a full re-scan each time. Answers both latency (15%) and resource utilization (20%) through an architecture choice rather than brute-force optimization. Judged too complex for 15 days — **kept as an explicit PPT roadmap item, not built**.
4. **On-device cryptographic audit trail** — tamper-evident hash-chain log of redaction events, stays local, never transmitted. Doesn't hit a scored metric directly; answers the off-script "how would I audit this actually did what it claims" question a defense-adjacent evaluator is likely to ask. **PPT roadmap item, not built.**
5. **Adversarial self-test harness** — synthetic test pages seeded with known-ground-truth PII (including PII rendered inside a canvas element specifically to test the vision path, not just the DOM path), triggered live in the popup, reporting real measured recall/precision instead of a scripted claim. **This is the one built into the brief as Phase 1.5**, inserted directly after the core redaction pipeline (Phase 1) and before server-side work (Phase 2).

### Why #5 got build priority over #1–4
Only #5 converts directly into live, on-demand evidence for the two highest-weighted rubric lines — accuracy of visual context (25%) and PII detection recall/precision (20%), 45% combined. A judge scoring those two metrics from a single scripted demo run is otherwise taking the team's word for it; the self-test harness removes that entirely and is something almost no team under hackathon time pressure builds unless explicitly planned for. #2 was cheap enough (a MutationObserver + a default-redacted state) to fold into Phase 1 directly rather than treat as a separate phase. #1, #3, #4 are strong pitch material — explicitly logged as PPT talking points in the brief's out-of-scope section so they're not lost, just not built.

### Antigravity brief — what changed
- Constraint #4 added: explicit no-training rule with a stated fallback path
- Phase 1 gained build item 7 (fail-closed mutation handling)
- New Phase 1.5 inserted: self-test harness, with its own demo checkpoint and an explicit "why this matters" line tied to rubric weights
- Out-of-scope list gained three roadmap-only items (#1, #3, #4 above) so the agent doesn't attempt them but the team doesn't lose the ideas either
- Repo structure gained `/extension/self-test`

## Status
Brief updated and re-shared. Ready to hand to Antigravity as-is.


---

## Correction 27 Aug 2026 — ViT model choice, resolved

Nikhil proposed `Xenova/vit-base-patch16-224` (or similar) via Transformers.js for the "screen-understanding ViT" role, intending it to detect PII rendered as pixels (text in images/canvas), separate from DOM scanning (structured fields) and BlazeFace (faces).

**Flagged as not acceptable as proposed — category mismatch, not a quality tradeoff.** `vit-base-patch16-224` is a 1000-class ImageNet image classifier: whole-image label output only, no localization, no text-reading capability. It cannot detect or locate PII in an image under any configuration. Assigning it this role would mean the "detect PII in pixels" path — the specific wedge identified in the competitive-landscape research (every consumer redaction extension fails on image/canvas content) — would not actually function at demo time.

**Resolved:** swap in **Tesseract.js** as a third parallel client-side detector, alongside DOM scanner and BlazeFace. Researched and confirmed: purely client-side, lightweight, no WebGPU/heavy-transformer-runtime requirement — this is NOT the heavy OCR pipeline Nikhil was trying to avoid (that would be TrOCR/Donut, which do need GPU). Tesseract.js runs only on regions not already claimed by DOM scanner or BlazeFace, keeping it cheap. OCR-extracted text passes through a simple pattern layer (email, phone, Aadhaar/PAN format, name pattern) — same detection logic as the DOM path, different source.

A ViT can still optionally be used, but only for a genuinely different purpose: a general visual-context embedding sent toward the server-side VLM alongside the manifest. Its output must never be treated as a redaction signal.

### What changed in the Antigravity brief
- Tech stack section rewritten: three parallel client-side detectors (DOM scanner, BlazeFace, Tesseract.js), with an explicit "do NOT use a general image classifier for PII detection" warning stating why
- Architecture diagram updated to show Tesseract.js in place of the ViT-as-detector, with the ViT demoted to an optional, clearly-labeled non-redaction role
- Phase 1 build steps updated: step 5 is now Tesseract.js integration + pattern matching; demo checkpoint (step 9) now explicitly includes a screenshot-style test image with embedded fake PAN/Aadhaar text, not just a password field + face
- Phase 1.5 self-test harness updated to reference "the Tesseract.js/OCR path" explicitly rather than "the vision path" generically

### Status
Resolved. Brief re-synced to outputs. This closes the last open item from the original "screen-understanding model choice" question in [[00-Foundation]] — the model choice is now concrete and correct, not just decided-but-wrong.
