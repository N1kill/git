\# Theory + Existing Software to Study — 15-Day Learning Plan

**Goal for these 15 days:** understand this domain deeply enough to design the system and pitch it credibly. NOT building the working system — that's for the Finals build phase (Dec).

---
## Existing software already solving this — study these, don't reinvent

### For the Delft3D / grid-based hydrodynamic side

**HEC-RAS** (US Army Corps of Engineers, free) — the most important one to know about, arguably MORE useful to study than Delft3D itself:
- Industry-standard river hydraulics + floodplain inundation + dam-break simulation software, free and widely used
- **Directly relevant precedent: HEC-RAS has been used to simulate the 2006 Ukai Dam flood event in India** (Patel et al., 2017) — this is a real Indian dam-break case study we can reference directly in our PPT, and potentially use as a partial validation benchmark later
- Also used for the famous 1976 Teton Dam failure simulation (US) — a well-documented benchmark case worth reading about to understand what a "good" dam-break simulation study looks like
- **Why this matters for us:** even though the PS specifically names Delft3D, HEC-RAS is the more accessible, better-documented, more widely-taught equivalent — reading HEC-RAS case studies teaches the same underlying grid-based hydrodynamic modelling concepts Delft3D uses, and is a more realistic first software to actually touch

**Delft3D** (Deltares, the PS's actual named tool):
- Free but with a real learning curve — professional-grade, used for storm surge, river hydraulics, coastal engineering worldwide
- Confirmed via research: Delft3D-FLOW is commonly paired with HEC-RAS in real published studies (one simulates ocean/storm surge, other simulates river/floodplain) — this pairing pattern is worth understanding since it shows how professionals actually combine these tools in practice

**TELEMAC-2D** — another real open-source alternative in this same category, used in comparative studies against HEC-RAS. Worth knowing this exists as a third reference point, not necessarily to use.

### For the SPH / particle-based side

**DualSPHysics — THIS IS THE ANSWER to "is there similar software already doing this"**
- Free, open-source (GPLv3 license), actively maintained SPH solver
- Written in C++ with CUDA (GPU) and OpenMP (multi-core CPU) support
- **Dam-break is literally one of its core, most-validated use cases** — used in real published research for dam-break impact, wave interaction, offshore structures
- GPU acceleration is dramatic: one benchmark shows a dam-break case with 1 million particles taking ~15 min on a GPU vs 6 hours on 8 CPU cores
- This is very likely what we should actually build on top of, rather than writing SPH physics from scratch — configuring/adapting DualSPHysics for our specific dam scenario is a realistic build target; writing SPH equations from zero is not
- Link: http://dual.sphysics.org/

**SPHysics** — the academic predecessor/sibling project to DualSPHysics (Johns Hopkins, University of Vigo, University of Manchester) — worth knowing exists, but DualSPHysics is the more modern, GPU-accelerated, actively developed option.

**GeoClaw** — another real open-source option, originally built for tsunami simulation but extended specifically to dam-break flooding (validated against the real 1959 Malpasset dam failure and 1976 Teton dam rupture). Different numerical method (finite volume, not SPH), but a legitimate alternative worth knowing about.

---
## Real published dam-break case studies to read (these teach the theory better than a textbook)

1. **Ukai Dam, India (2006)** — simulated with HEC-RAS (Patel et al., 2017). MOST IMPORTANT for us — real Indian precedent.
2. **Teton Dam failure, USA (1976)** — simulated with both GeoClaw and HEC-RAS, results compared against real historical observations (Spero et al., 2022). Excellent case study for understanding what "validating a dam-break model" actually looks like in practice.
3. **Malpasset Dam failure, France (1959)** — classic benchmark case, validated against real field data and scaled lab experiments (George, 2011). This is practically the "hello world" of dam-break simulation validation — worth understanding well.

---
## Key evaluation metrics used in real research (useful for OUR eventual validation + PPT credibility)

From the comparative literature: r², IA (index of agreement), NSE (Nash-Sutcliffe Efficiency), KGE (Kling-Gupta Efficiency), RMSE — these are the standard metrics real hydrodynamic modelling papers use to compare simulated vs observed flood depth/extent. Knowing these terms and citing them correctly in our PPT will read as informed, not just enthusiastic.

---
## What this means for OUR approach (draft thinking — refine after more reading)

- **We are very unlikely to be writing SPH numerical methods from scratch.** DualSPHysics exists, is free, GPU-accelerated, and dam-break is a core validated use case for it. Our job is likely: configure it for our chosen dam + terrain, not reimplement fluid dynamics equations ourselves.
- **HEC-RAS may be a more realistic stand-in to actually learn hands-on than Delft3D itself**, even though the PS names Delft3D specifically — same category of tool, better documentation, wider community, free. Worth discussing with team/mentor whether presenting HEC-RAS as our grid-based comparison engine (citing it as functionally equivalent to Delft3D, more accessible) is an acceptable framing, OR whether we need to specifically use Delft3D because the PS names it.
- **The Ukai Dam (India, 2006) HEC-RAS study is our best reference case** — real Indian dam, real precedent, real published methodology we can cite and potentially loosely follow the approach of.

## Open questions to resolve before/during the 15 days
- [ ] Does the PS require Delft3D specifically (grading criteria), or would a credibly-argued equivalent (HEC-RAS) be acceptable? Worth checking official PS clarification channels if SIH provides one.
- [ ] Confirm: does anyone on the team have any GPU access (personal or institutional) for DualSPHysics later in the build phase?
- [ ] Read the Ukai Dam (2017) and Teton Dam (2022) papers properly — not just abstracts — before finalizing our own dam/region choice.

---
**Next note → 02-Dam-Region-Selection.md** (once above reading is done)

## RESOLVED — Delft3D vs HEC-RAS question (23 Aug 2026)

**CLOSED, not open anymore.** Re-read the actual PS text (pasted by Nikhil in full) — it names "Smooth Particle Hydrodynamics" and "Delf3D" model explicitly, TWICE, including in the deliverables section ("compare the scenario" using these two named models). This is not "a hydrodynamic model of your choice" — Delft3D is specifically required for the deliverable.

**Implication:** HEC-RAS remains valuable as a *learning reference* (the Ukai Dam India case study is still worth citing for PPT credibility, and HEC-RAS documentation teaches the same grid-based hydrodynamic concepts Delft3D uses) — but our actual build target must be Delft3D itself, not a substitute. Don't present HEC-RAS as our engine in the final pitch.

## RESOLVED — GPU access (23 Aug 2026)

- **College GPU**: requested, available. Use for: sustained/iterative work, especially during Finals build phase (Dec). Ask specifically for CUDA-capable (NVIDIA) access with decent VRAM — DualSPHysics needs CUDA specifically, not AMD.
- **Kaggle (30 hrs/week, free tier, P100/T4x2)**: sufficient for THESE 15 days — learning DualSPHysics, running small tutorial-scale example simulations (thousands of particles). NOT sufficient for iterative real-scale simulation work later — 9-12 hr session caps + weekly reset will slow down real tuning work at millions-of-particles scale.
- **Plan**: Kaggle now for learning + small tests. Push college GPU access to be sorted well before December build phase for the real iterative work.

---
## Confirmed Dataset Links (23 Aug 2026)

### DEM / Terrain data

**Bhuvan (ISRO/NRSC) — India-specific DEM, CartoDEM (Cartosat-1, 30m resolution)**
- Main download portal: https://bhuvan-app3.nrsc.gov.in/data/download/
- Free satellite data & DEM download wiki/guide: https://bhuvan.nrsc.gov.in/wiki/index.php/Free_Satellite_Data_Download
- Requires free registration on Bhuvan
- Also listed on India's Open Government Data portal: search "Digital Elevation Model (DEM) generated from Cartosat-1" on data.gov.in

**USGS EarthExplorer — SRTM DEM (30m or 90m resolution), global coverage**
- Portal: https://earthexplorer.usgs.gov/
- Requires free USGS/EROS account registration
- Path once logged in: Data Sets tab → Digital Elevation → SRTM → SRTM 1-Arc-Second Global (30m, most commonly used)
- Well-documented, many tutorials available (search "USGS EarthExplorer SRTM DEM download tutorial" if the UI is confusing at first)

**Recommendation:** Bhuvan for India-specific accuracy/context (it's literally ISRO's own data, extra credibility for judges), cross-check against USGS SRTM as a second source if needed.

### Dam / Reservoir data

**Central Water Commission (CWC) — official Indian dam/reservoir authority**
- Main hydrological data page: https://cwc.gov.in/en/hydrological-data
- Reservoir monitoring (weekly bulletins, live storage by dam): https://cwc.gov.in/en/reservoir-monitoring
- Reservoir Level & Storage Bulletin (123 major reservoirs, daily/weekly): https://www.cwc.gov.in/reservoir-level-storage-bulletin
- Reservoir Dashboard: https://cwc.gov.in/wm_dashboard
- National Register of Large Dams — exists under CWC's Water Info section (dam-by-dam technical specs)

**Open Government Data Platform — same CWC data, structured/downloadable format**
- https://www.data.gov.in/resource/daily-data-reservoir-level-central-water-commission-cwc

**AIKosh (India AI dataset platform) — also hosts CWC reservoir data**
- https://aikosh.indiaai.gov.in/home/datasets/details/daily_data_of_reservoir_level_of_central_water_commission_cwc.html

**Note:** CWC data gives water LEVELS and storage volumes (good for estimating reservoir capacity for our breach scenario), not detailed dam engineering specs (breach geometry etc. — we'll need to estimate/assume reasonable values for that part, standard practice per the case-study papers we read).

### Google Earth Engine (for the "near real-time" requirement)

- Main platform: https://earthengine.google.com/
- Requires free account (Google account + GEE access approval, usually fast for students/research use)
- Hosts DEM datasets directly too (can double as a second terrain source) plus current/recent satellite imagery — this is literally built for the "near real-time" layer the PS asks for in point (iv)

---
## Status: Dataset access fully mapped. Both software-choice and GPU questions resolved.
Next → 02-Dam-Region-Selection.md (pick the actual dam + river to build around, using CWC's major-reservoir list + Bhuvan DEM coverage)


---
**→ See [[03-Build-Reference-HTML]] for the consolidated visual reference (workflow diagram, tech stack, cost table, timeline, output mockup, prototype checklist) pulling together this note + 00 + 02.**
