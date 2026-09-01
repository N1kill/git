\# System Architecture — Generalized, Auto-Fetching Design

**Decided 23 Aug 2026.** This corrects/replaces the earlier framing of "pick one dam to build around" — that was about *demonstrating* the system, not *limiting* it. The actual system must be generalized: user selects any dam, system auto-fetches everything else.

---
## The core design principle

**We are NOT building a single-dam case study. We are building a generalized tool, demonstrated on a small number of real dams for the PPT/demo — because running full dual-engine (SPH + Delft3D) simulations across all ~123 major Indian dams simultaneously isn't a compute-feasible hackathon deliverable, not because the tool itself is limited to one location.**

This directly matches what the PS actually asks for: "a generalized modelling framework" (point i) and "customized tool/framework... using different input datasets" (point ii) — NOT a single fixed case study.

## Answering Nikhil's core question: can the system fetch DEM + river channel itself, without manual upload per dam?

**Yes. This is the correct architecture, not a stretch goal.** Here's why it's genuinely achievable:

### The key insight: everything is queryable by coordinates, not just manually browsable

1. **Pre-built dam database (one-time setup effort, not per-user)**
   - Scrape/compile CWC's major dam list ONCE into our own database: dam name, coordinates (lat/long), reservoir capacity, associated river name
   - Source: CWC Reservoir Bulletin (123 major reservoirs) — https://www.cwc.gov.in/reservoir-level-storage-bulletin
   - This becomes our own lookup table. User's dropdown/map-pick queries THIS table — instant, no live scraping per request.

2. **User selects a dam** (dropdown or map pin) — this is the ONLY manual input needed from the user

3. **System auto-fetches, using that dam's stored coordinates:**
   - **DEM**: query USGS SRTM or Bhuvan CartoDEM programmatically for a bounding box around the dam's coordinates (API-based fetch, not manual portal browsing — this is a standard GIS workflow)
   - **River channel path**: trace downstream from the dam's coordinates using a river-network dataset (see below) — determines which direction/channel the water travels, not just raw terrain shape
   - **Reservoir volume**: already have it, from our own pre-built dam database (step 1)

### River network data source — CONFIRMED

**HydroSHEDS / HydroRIVERS** (WWF/USGS collaborative project) — this is the answer to "how do we get the river channel automatically":
- Free, global coverage including India, vectorized river network with ~8.5 million river reaches worldwide
- Includes **flow direction data** — literally tells you which way water flows at any point, which is exactly what's needed to auto-trace a channel downstream from a dam
- Derived from the SAME underlying SRTM elevation data as our DEM source — meaning our terrain and river-network data will be naturally consistent with each other, not two mismatched datasets
- Also directly available inside Google Earth Engine (as a queryable layer), which is convenient since we're already using GEE for the near-real-time requirement
- Download portal: https://www.hydrosheds.org/downloads-core-data-v2
- HydroRIVERS product specifically (the vectorized river lines): https://www.hydrosheds.org/products/hydrorivers

**Real precedent — someone already did almost exactly this for India:** A 2023 published research paper (Environmental System Science Data journal) built a dataset called "GHI" (Geospatial dataset for Hydrologic analyses in India) that combined **CWC gauge station data + HydroSHEDS catchment/river-network tracing** — literally the same combination of data sources we're proposing. Worth reading this paper closely; it may even be directly citable/usable as methodology validation in our PPT.
- Paper: https://essd.copernicus.org/articles/15/4389/2023/

## Revised architecture diagram (conceptual)

```
[User selects a dam from dropdown/map]
              ↓
[System looks up dam in our pre-built CWC database]
   → coordinates, reservoir capacity, river name
              ↓
     ┌────────┴────────┐
     ↓                 ↓
[Auto-fetch DEM]  [Auto-trace river channel]
(USGS/Bhuvan API   (HydroSHEDS/HydroRIVERS,
 for bounding box   flow-direction from same
 around coords)     coordinates)
     └────────┬────────┘
              ↓
   [Terrain + channel + reservoir volume
    = complete simulation input, zero
    manual data hunting per dam]
              ↓
   [User defines breach scenario:
    breach width, opening time, etc.]
              ↓
   [Run SPH (DualSPHysics) AND Delft3D
    on the same auto-fetched inputs]
              ↓
   [Compare outputs, export .shp/.kml,
    display on dashboard]
```

## Why this is a STRONGER pitch than a single fixed case study

"Watch me pick any dam from this list and the system just works" is a dramatically more impressive live demo than "here's our one pre-built Ukai Dam simulation." It directly proves the "generalized framework" requirement the PS explicitly asks for, rather than us having to argue we could generalize it later.

## What demo dams to actually showcase (not a limitation, just what we highlight)

For the PPT/demo, we'll still highlight 2-3 real dams as **example runs** — likely ones with:
- Good DEM coverage
- Real historical precedent we can reference (e.g., near one of the flood events cited in the PS background: Uttarakhand, Assam, J&K) for validation credibility
This is a demo-scope decision, NOT an architecture limitation. The system itself works for any dam in our database.

---
## Status: Architecture decided — generalized, auto-fetching system confirmed as the correct design.
Next → pick 2-3 example dams for the demo showcase (not "the one dam we build for") + start reading the GHI paper methodology in detail.


---
**→ See [[03-Build-Reference-HTML]] for the consolidated visual reference (workflow diagram, tech stack, cost table, timeline, output mockup, prototype checklist) pulling together this note + 00 + 01.**
