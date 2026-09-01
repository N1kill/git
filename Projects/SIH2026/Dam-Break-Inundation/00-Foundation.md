\# Dam Break Inundation Modelling — Foundation Notes

**PS:** SIH26161 — Dam Break Inundation Modelling Using Hydrodynamic Modelling of any River
**Org:** National Technical Research Organisation (NTRO)
**Owner:** Nikhil (solo until Railway/Browser-Agents teammates free up)
**Team:** 6 total — 1 on Railway Block Planning, 2 on Browser Agents, 2 on Single-Pass Drone 3D, 1 (me) on this
**Timeline:** Finals in December. PPT pitch round ~15 days from today (23 Aug 2026 → early Sept).
**Reality check:** Zero prior experience with SPH, Delft3D, or hydrodynamic modelling going in. Plan is built assuming that, not pretending otherwise.

---

## 1. What the PS actually is (plain English)

A dam breaks, or a natural lake (formed by landslide/glacier) suddenly bursts. A wall of water rushes downstream.
**We're building software that predicts: which areas flood, how deep, how fast the water arrives** — given the dam/lake location, water volume, and the shape of the land downstream (terrain).

Real cited events: Rishi Ganga (Uttarakhand, Feb 2021), Wapriyang river (Nov 2021), Phuktal river near Sumdo J&K (Mar 2015), Kosi river (2008).

This is for disaster preparedness (HADR — Humanitarian Assistance and Disaster Relief) — authorities need "what if" flood maps *before* a break happens, to plan evacuation.

## 2. What the PS literally asks for (from the brief)

i. A modelling framework predicting dam break/river blockage flood extent, using **both SPH and Delft3D**, comparing the two
ii. A customizable tool to generate flood inundation scenarios from different input datasets
iii. A dashboard (GUI) for input + output visualization, output as `.shp` or `.kml`
iv. A near-real-time flood analysis layer using Google Earth Engine (GEE)
v. Final demo must use a real Indian river + dam, open-source data

## 3. The two named methods, explained

### Smoothed Particle Hydrodynamics (SPH)
- Water modelled as thousands of virtual "particles," each with mass/velocity/pressure
- Physics equations govern how nearby particles push/pull each other
- Run forward in time-steps → collective particle motion = the flood
- Same family of technique used in some CGI water effects / scientific fluid sims
- Good: naturally handles chaotic flow (breach, obstacles) without hand-coding every case
- Hard: computationally expensive at scale, numerically unstable if not tuned carefully

### Delft3D
- Existing professional software (Deltares, Netherlands) used by real hydrologists/coastal engineers worldwide
- Grid-based, not particle-based: divide the map into cells, solve flow equations per cell, update each time-step
- Good: mature, trusted, you're *configuring* it, not building simulation math from scratch
- Hard: real learning curve — built for professional hydrologists, not hackathon speed; heavy software to even install/set up correctly

**PS wants both, compared side by side. This is the single biggest scope risk on this project.**

## 4. Other terms explained

- **DEM (Digital Elevation Model):** land shape as data — a grid where each point has a height value. Simulation needs this to know which way water flows downhill.
- **GIS (Geographic Information System):** any software/data tied to real-world map locations. Our dashboard = a GIS dashboard.
- **Google Earth Engine (GEE):** free Google cloud platform with huge satellite imagery archives + compute, no need to download massive files yourself. "Near real-time" layer = pull current data automatically instead of using static files.
- **.shp / .kml:** standard map-data file formats so our output flood map works in other GIS tools (QGIS, Google Earth), not locked into our own app.

## 5. Dataset access — honest picture

**Terrain (DEM): available, no blockers**
- Bhuvan (ISRO's geospatial portal) — Indian DEM data
- USGS SRTM — global elevation data, free, well-documented
- Google Earth Engine — also hosts DEM directly, doubles as terrain source + near-real-time tool

**Dam/river data:** PS says "any river and dam of India, open source." We choose. Central Water Commission publishes public dam data (location, approx. reservoir capacity, river geometry) — not as precise as a real operator's internal data, but real and legitimate.

**What does NOT exist:** no pre-made "dam break scenario" dataset. We generate scenarios ourselves by feeding real terrain + a hypothetical breach into the simulation. That's the actual task, not a gap.

**Open question for us:** does anyone on the team (now or arriving later) have Delft3D institutional access or any SPH library experience? Determines whether we're configuring existing software or building simulation logic more from scratch.

---
## Status: FOUNDATION LOCKED. Next → 01-Scope-Decision.md (SPH vs Delft3D lead, dam selection)

## CORRECTED TIMELINE UNDERSTANDING (23 Aug 2026)

- **Next 15 days (→ early Sept):** PPT round only. Goal = understand the domain deeply enough to design + pitch credibly. NOT building the working system yet.
- **Between PPT round and Finals (December):** actual build phase, likely with Railway/Browser-Agents teammates joining once their tracks wrap.
- So the next 15 days = theory, existing-software study, architecture design, dam/region selection, and a strong PPT — not code.

Status: FOUNDATION LOCKED. Next → 01-Theory-and-Prior-Art.md


---
**→ See [[03-Build-Reference-HTML]] for the consolidated visual reference (workflow diagram, tech stack, cost table, timeline, output mockup, prototype checklist) pulling together this note + 01 + 02.**
