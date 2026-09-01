\# Build Reference — HTML Doc (Workflow, Stack, Cost, Timeline, Output, Prototype)

**Created:** 23 Aug 2026
**File:** `dam-break-build-reference.html` (shared with Nikhil as a downloadable artifact)
**Purpose:** Single consolidated reference pulling together everything decided across [[00-Foundation]], [[01-Theory-and-Prior-Art]], and [[02-System-Architecture]] into one visual document — workflow diagram, tech stack cards, computational cost table, build timeline, expected output mockup, and a prioritized prototype checklist.

## Why this exists separately from the three markdown notes

The markdown notes are the working/reasoning trail — decisions, open questions, sources, resolved-vs-open status. This HTML doc is the **presentable summary** of those decisions — the thing to actually show a teammate, mentor, or reference while building, without re-reading three separate notes. Treat the markdown notes as source-of-truth; treat this HTML doc as the synthesized output, to be regenerated/updated if the markdown notes change materially.

## What's inside

1. **Workflow** — the 9-step pipeline (dam selection → auto-fetch DEM/river channel → preprocessing → breach scenario → dual-engine simulation → export → GEE layer → dashboard), same logic as [[02-System-Architecture]]'s architecture diagram, redrawn as a step sequence with automated-vs-manual tagging
2. **Tech stack** — every tool from [[01-Theory-and-Prior-Art]] (DualSPHysics, Delft3D, Bhuvan, USGS SRTM, HydroSHEDS/HydroRIVERS, CWC, GEE) plus the Kaggle/college-GPU compute decision from the same note, as reference cards
3. **Computational cost** — DualSPHysics benchmark figures (tutorial scale → 1M particles → full terrain scale) plus the Delft3D multi-week learning-curve reality, sourced from the same research pulled into 01-Theory-and-Prior-Art
4. **Build timeline** — the three-phase split confirmed in [[00-Foundation]]'s "CORRECTED TIMELINE UNDERSTANDING" section: PPT round (understand/design only) → post-PPT build phase → pre-Finals integration
5. **Expected output** — a conceptual SVG mockup of the actual deliverable: overlapping SPH vs Delft3D flood-extent polygons on terrain contours with a river channel and dam marker — this is the target demo screen, not real simulation output
6. **Prototype checklist** — prioritized, 15-day-realistic items first (read precedent papers, small Kaggle SPH run, small CWC lookup table, real DEM/HydroRIVERS pull for 1-2 dams, architecture diagram, output mockup), full Delft3D build explicitly marked as out-of-scope for this window

## Status
Living reference — regenerate if scope, stack, or timeline decisions change materially in the source notes.

---
**Vault map for this project:**
[[00-Foundation]] → [[01-Theory-and-Prior-Art]] → [[02-System-Architecture]] → this note (synthesis)
