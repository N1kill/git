# Build Reference — HTML Doc (What/How/Innovation)

**Created:** 27 Aug 2026
**File:** `browser-agents-build-reference.html` (shared with Nikhil as a downloadable artifact)
**Purpose:** Single consolidated reference pulling together [[00-Foundation]]'s research into one visual document — the PS breakdown, scoring-weight priority order, competitive-landscape finding, concrete technical stack, four innovation angles, open questions, and a 15-day-realistic build checklist.

## Why this exists separately from the foundation note

Same pattern as Dam-Break: the markdown note is the working/reasoning trail (research, sources, open questions). This HTML doc is the **presentable synthesis** — what to actually show a teammate or read while building, without re-reading the full research note. Treat 00-Foundation as source-of-truth; treat this HTML as the regenerate-if-decisions-change output.

## What's inside

1. **Core idea + signature visual** — an animated mock "screen scan → redaction" viewport showing the client/server trust boundary in one glance (face + password field getting redacted locally before a green "safe" action gets sent to the server)
2. **PS breakdown** — the 4 required components split client/server, plus the 5 scoring weights as visual bars — flags that redaction-related metrics (detection + precision) = 40% combined vs. 15% for latency, and states the build-order implication directly
3. **Competitive landscape table** — researched Chrome extension ecosystem (BlurShield, ScreenMask, Obscuro, PrivacyScrubber, etc.), all confirmed DOM-text/regex-only, none doing vision-based screen understanding. Includes the direct finding from BlurShield's own limitations section admitting it can't see image/canvas content — this is the pitch wedge.
4. **Technical stack** — concrete picks, not option lists: Transformers.js v4 + WebGPU, BlazeFace-ONNX for face detection, Chrome-primary browser target, hybrid DOM+vision redaction approach — each with real numbers (latency, cold-start tax) from the research pass
5. **Four innovation angles** — confidence-gated redaction, DOM-vision cross-validation as a two-signal system, redaction-scheme-aware server manifest, visible on-screen resource-utilization indicator — each tied explicitly to why it scores against the PS's own rubric
6. **Open questions** — screen-understanding model choice (still needs its own research pass), server VLM pick, team split
7. **15-day build checklist** — prioritized, explicitly scoped to what's achievable before the PPT round, with an explicit out-of-scope list (no from-scratch ViT training, no full Firefox parity, no general-purpose agent)

## Design notes
Deep space-navy + signal-amber palette (not the generic purple/violet "AI startup" look) — signal-amber chosen because it's the real color convention for detection/redaction bounding boxes, ties directly to subject matter. Monospace/technical type pairing since this is an engineering PS. Signature element is the live scan/redaction viewport — it's the one visual that embodies the whole PS in a single glance rather than decorating around the content.

## Status
Living reference — regenerate if scope, stack, or model-choice decisions change materially in [[00-Foundation]].

---
**Vault map for this project:**
[[00-Foundation]] → this note (synthesis)
