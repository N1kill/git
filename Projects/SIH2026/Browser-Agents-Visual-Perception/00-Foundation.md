# On-Device Visual Perception for Browser Agents — Foundation Notes

**PS:** SIH26171 — On-device Visual Perception for Light-weight Browser Agents
**Org:** Indian Space Research Organisation (ISRO), Dept. of Space
**Category/Theme:** Software / Smart Automation
**Owner:** Nikhil + 2 teammates (per Dam-Break note: "2 on Browser Agents")
**Team status:** Flagged by Nikhil as "most tractable" of the 4 locked PS
**Timeline:** PPT round ~15 days out (finals December)
**Reality check:** No prior on-device ML / WebGPU / browser-extension-ML build behind us going in — flagging that now, not pretending otherwise. Concepts are documented, gap is hands-on build time.

---

## 1. What the PS actually is (plain English)

Build a browser extension that "looks at" the user's screen using a small AI vision model running **on the user's own machine** (not a server). Before anything about that screen is sent to a cloud AI for reasoning/decision-making, the extension must **blank out anything private** — passwords, faces, personal text — locally, so the cloud server only ever sees a sanitized version. The cloud AI reasons over the sanitized version and sends back an action ("click this," "scroll here") which the extension executes.

Two AI systems working together, split by trust boundary:
- **Client (browser):** sees everything, but only ever *shows*/sends sanitized data outward. Fast, small model.
- **Server (cloud):** never sees raw sensitive data, only sanitized context + does the heavy reasoning.

## 2. What the PS literally asks for (from the brief)

**Client-side (extension/JS):**
1. Local Vision Processing — client-side vision model (ViT or equivalent) running in-browser (WebGPU-accelerated) evaluating current screen state
2. Privacy-Preserving Filter — sanitizes sensitive/personal visual data via bounding-box redaction, semantic obfuscation, or masking; must dynamically detect + redact (blur faces, black out passwords, mask PII)

**Server-side:**
3. Server-side integration — transmit anonymized visual context to a centralized LLM/VLM, which interprets sanitized data and returns either processed data (re-ingested locally) or a UI action ("click submit," "scroll down") the client executes
4. Any offline-deployable open-source/open-weight model allowed server-side; cloud-hosted versions of the same models permitted during SIH itself
5. Must demonstrate an end-to-end task assisting the user

**Evaluation weights (this is the rubric — build order should follow this):**
| Metric | Weight |
|---|---|
| Accuracy of visual context from screen | 25% |
| Recall/precision of sensitive/PII detection | 20% |
| Precision of redaction | 20% |
| Client-side resource utilization | 20% |
| End-to-end task latency | 15% |

**Key read: redaction-related metrics (detection + precision) = 40% combined. Latency, the "flashy demo" metric, is only 15%.** Build priority should be inverted from instinct: nail redaction quality before optimizing for a snappy agent demo.

## 3. Terms explained

- **ViT (Vision Transformer):** an image-classification/understanding model architecture (like BERT, but for images cut into patches instead of words). Smaller variants exist specifically for edge/browser deployment.
- **WebGPU:** browser API giving JS direct access to the GPU for compute, not just graphics — this is what makes real-time in-browser ML feasible. Chrome 113+ and Edge support it well; Firefox support is present but slower (see research below).
- **ONNX Runtime Web / Transformers.js:** the two dominant JS libraries for running pretrained ML models (vision or language) inside a browser tab, with automatic WebGPU→WebAssembly fallback.
- **DOM tags/sanitization:** the PS explicitly floats DOM-tag-based sanitization as one option — i.e., not purely pixel-level redaction, but also leveraging the page's own structure (e.g., an `<input type="password">` tag) to know what to hide, combined with the vision model for things the DOM doesn't label (e.g., a face in an embedded image, PII rendered inside a canvas/screenshot element).

## 4. Technical grounding (researched 27 Aug 2026 — real numbers, not guesses)

### Browser ML inference performance (current, 2026)
- Transformers.js v4 (shipped Feb 2026) rewrote the runtime in C++ with a WebGPU backend — **3-10x speedup over v3**.
- WebGPU vs WASM on single-image classification: RTX 3060 discrete GPU ~18-22ms (WebGPU) vs 35-45ms (WASM). Integrated GPUs (M2, Intel Iris Xe): gap narrows to near-parity, ~30-40ms either way.
- Shader compilation cold-start tax: **3-10 second first-use initialization window** for WebGPU (WGSL shader compile) — must be hidden behind a loading state in the demo, not ignored.
- Chrome/Dawn WebGPU implementation is consistently the most stable across platforms; Firefox WebGPU support exists but is substantially slower — **build/demo on Chrome primarily**, mention Firefox as a stated-but-secondary target per the PS wording ("popular browsers").
- Realistic guidance for interactive/responsive tasks: keep models ≤2B params (irrelevant for us — our vision task doesn't need anywhere near that; classification/detection-sized models are tens of MB, not GB).

### Face/PII detection model — concrete pick
- **BlazeFace (Google Research, MediaPipe origin)** — small, fast (30+fps on most devices), pre-converted ONNX versions exist on Hugging Face (e.g. `garavv/blazeface-onnx`) and GitHub. 128×128 input, outputs bounding boxes + 6 facial landmarks + confidence score. This is the standard credible answer for real-time in-browser face detection — directly usable via ONNX Runtime Web.
- Post-processing note: BlazeFace's raw ONNX output needs NMS (non-max suppression) + anchor decoding — some community forks bake this into the ONNX graph itself, worth using one of those rather than hand-rolling it under time pressure.

### Competitive landscape check — IMPORTANT, changes the pitch
Searched the existing Chrome extension ecosystem for "redact PII on screen" tools. Found many: Blur It, Smart Blur, Privacy Blur, PII Blur, Obscuro, BlurShield, ScreenMask, PrivacyScrubber. **Every single one of them operates on DOM text content via regex/keyword matching.** BlurShield's own limitations section explicitly states: *"Image and Canvas Content: BlurShield operates on DOM text content. Sensitive data rendered within images, canvas elements, or embedded PDFs will not be detected or blurred."*

**This is the wedge.** None of the existing tools do actual vision-model-based screen understanding. A ViT/vision-model approach catches:
- PII rendered as pixels inside images, canvas elements, embedded PDFs, screenshots-within-pages
- Faces in embedded photos/video thumbnails
- Anything a regex can't see because it's not in the text DOM at all

Positioning for the PPT: **"Existing tools redact what's in the DOM's text. We redact what's on the screen — including what's rendered as pixels, which is exactly the gap every existing extension admits it has."** This is a stronger, more specific claim than generic "we built a privacy-preserving browser agent."

## 5. Crowd-risk assessment

Moderate-high. "Browser AI agent" is one of the most-touched buzzphrases in 2026 hackathons — expect real competition on the *agent* half of this PS. The **redaction pipeline half is less crowded** and is also 40% of the score (redaction recall/precision + redaction precision) vs. 15% for latency. Competitive strategy: don't try to out-flashy other teams' agent demos — out-engineer them on the privacy filter, which is also the part ISRO/defense-adjacent evaluators will care about most (this is literally a privacy-preservation PS from a space agency, not a generic productivity-agent PS).

## 6. Open questions for the team

- Which vision approach for the *screen-understanding* half (distinct from PII detection): full ViT running via Transformers.js, or a lighter object/text-region detector? Need to scope this against the "accuracy of visual context from screen" metric (25%, the single largest weight) — this deserves its own research pass, not a quick pick.
- Server-side VLM choice: PS explicitly allows cloud-hosted versions of open-weight models during SIH. Need to pick one (e.g. a small open VLM) and confirm API access/cost isn't a blocker for the demo.
- DOM-tag sanitization vs. pure pixel redaction — PS explicitly mentions DOM tags as a valid method. Likely a **hybrid** is strongest: DOM tags for structured fields (password inputs, labeled forms) + vision model for anything DOM-invisible (images/canvas/screenshots). Confirm this hybrid framing gets called out explicitly in the pitch, since it directly maps to both bullet points in the PS's "Privacy Preserving Filter" ask.
- Which 2 teammates are on this track, and what's each person's current skill overlap with WebGPU/extension dev vs. ML/model-conversion work — determines who owns client vs. server side.

---
## Status: FOUNDATION LOCKED (research pass done, no build yet). Next → 03-Build-Reference-HTML.md for the consolidated visual reference.
