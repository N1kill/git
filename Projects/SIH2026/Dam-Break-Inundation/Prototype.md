# Machhu-II Dam Breach Prototype — Full Build Pipeline (3D)

**Note on "2D vs 3D":** the hydraulic _solver_ below still runs HEC-RAS's 2D shallow-water engine — that's genuinely the right physics for a multi-km floodplain (real 3D CFD/Navier-Stokes tools like OpenFOAM or Flow-3D are used for spillway/gate-scale flows a few meters across, not a valley-wide flood; running true 3D CFD over the whole Machhu floodplain isn't something a student machine or timeline can realistically do). What changes for "3D" is the **output and prototype layer** — instead of flat 2D map images, you build an actual 3D terrain model with the flood surface draped over it, explorable and fly-through-able. That's what Phase 9.5 below covers, and it's the standard approach even in professional dam-break studies when a 3D deliverable is wanted. If you actually need the flow physics itself computed in 3D rather than just visualized in 3D, say so and I'll swap this out for an OpenFOAM-based approach instead — but that's a much heavier lift.

Everything from "I have the 8 raw datasets" → "I have a working inundation-map prototype." Follows the CWC's own tiered dam-break methodology (CWC 2018, "Guidelines for Mapping Flood Risks Associated with Dams") so your approach lines up with what CWC/NDSA actually expects — direct PDF: https://damsafety.cwc.gov.in/ecm-includes/PDFs/Guidelines_Developing_EAP_Dam.pdf

---

## Phase 0 — Tools to install first

|Tool|Use|Link|
|---|---|---|
|QGIS 3.34+|All GIS preprocessing, terrain conditioning, watershed delineation|https://qgis.org/download/|
|HEC-RAS 6.6 or 7.0.1|The actual 2D hydraulic/breach engine — this is the core of your prototype|https://www.hec.usace.army.mil/software/hec-ras/download.aspx|
|HEC-HMS (optional)|If you want a separate rainfall→runoff model instead of doing SCS-CN by hand|https://www.hec.usace.army.mil/software/hec-hms/download.aspx|
|WhiteboxTools|Free scriptable terrain hydrology (fill sinks, flow accum, watershed) — faster than doing it by hand in QGIS|https://www.whiteboxgeo.com/download-whiteboxtools/|
|Python stack|`pip install rasterio geopandas richdem numpy pandas matplotlib imdlib folium`|—|

---

## Phase 1 — DEM preprocessing (terrain conditioning)

1. **Reproject** DEM from WGS84 (EPSG:4326) → UTM Zone 42N (EPSG:32642). You need a projected CRS in meters or every slope/area calc downstream is wrong. QGIS: Raster → Projections → Warp.
2. **Mosaic** tiles if the Bhuvan/SRTM download gave you more than one tile covering the basin.
3. **Clip** to AOI: basin bbox (22.0–23.2°N, 70.4–71.3°E) + ~5km buffer.
4. **Fill sinks/depressions** — this is non-optional, real DEMs have noise pits that break flow routing. QGIS: SAGA → "Fill Sinks (Wang & Liu)", or WhiteboxTools `fill_depressions`.
5. **(Recommended) Stream-burn** the known river network (from Phase 2 below) into the DEM — Saurashtra terrain is flat, so unconditioned DEMs often route flow wrong without this. WhiteboxTools: `fill_burn`.
6. **Flow direction + flow accumulation** — WhiteboxTools `d8_pointer` + `d8_flow_accumulation` (or QGIS SAGA equivalents).
7. **Delineate the catchment**: set pour point at 22.82°N/70.84°E, snap it to the nearest high-accumulation cell, run watershed delineation.
8. **QC check**: your delineated area should come out close to **1,928 km²**. If it's off by more than ~10%, your pour point snapped to the wrong cell — nudge it and rerun.

## Phase 2 — River network processing

1. Extract streams from the flow-accumulation grid using a threshold (start around 1,000–5,000 cells, tune until the pattern looks river-like).
2. Cross-check/merge against the HydroRIVERS or WRIS vector layer you downloaded — use whichever is more accurate as your Phase-1 stream-burn input.
3. Compute Strahler stream order, reach lengths, and channel slopes — you'll need slope for time-of-concentration in Phase 6.

## Phase 3 — Rainfall preprocessing

1. Open the IMD NetCDF, clip to the catchment polygon from Phase 1.
2. Since it's 0.25° resolution (~27km cells) and the catchment is ~1,930 km², you'll likely only have 2–4 grid cells overlapping — use **Thiessen polygons** (area-weighted average) rather than simple averaging.
3. Pull the daily series for **9–12 Aug 1979** specifically (the storm that caused failure).
4. IMD data is daily — if HEC-RAS/HMS needs sub-daily resolution, disaggregate using the **Alternating Block Method** or an SCS Type-II design hyetograph shape scaled to your daily totals.

## Phase 4 — Inflow hydrograph (into the reservoir)

1. No gauge/telemetry exists for 1979, so build this from your Phase 3 rainfall + Phase 6 runoff (SCS-CN) instead of pulling a hydrograph directly.
2. Shape the runoff into a hydrograph using the **SCS Dimensionless Unit Hydrograph**, with lag time estimated from catchment length + slope (SCS lag equation or Kirpich formula).
3. Sanity-check the peak against the documented value: reported inflow/spillway numbers were on the order of **5,550–5,663 m³/s** — if your derived hydrograph peak is wildly off from that, revisit your CN or rainfall disaggregation.

## Phase 5 — LULC processing

1. Reclassify the Bhuvan LULC categories into standard **SCS-CN land-use classes** (fallow, row crops, pasture, forest, built-up, water).
2. Reproject/resample to match your DEM grid resolution and CRS (UTM 42N) so every layer overlays cleanly.

## Phase 6 — Soil → Hydrologic Soil Group (HSG)

1. From the NBSS&LUP soil map, extract texture/infiltration-class attributes for the catchment.
2. Assign each soil unit to **HSG A/B/C/D** using standard NRCS criteria (based on minimum infiltration rate / texture). Note: Saurashtra soils here are predominantly black cotton soil (Vertisols) — these typically classify as low-infiltration, high-runoff-potential soils, so expect the catchment to skew toward the higher-runoff HSG classes; confirm this against your actual soil map attributes rather than assuming it.
3. Reproject/resample to match the LULC/DEM grid.

## Phase 7 — SCS-CN curve number & runoff depth

1. Overlay LULC (Phase 5) × HSG (Phase 6) → assign a Curve Number to every cell using the standard SCS-CN lookup table (AMC-II baseline).
2. Adjust CN for **antecedent moisture condition** using the 5-day antecedent rainfall total from your IMD series (AMC I/II/III correction).
3. Compute area-weighted **composite CN** for the whole catchment.
4. Apply the SCS-CN runoff equation:
    - `S = (25400/CN) − 254` (mm)
    - `Ia = 0.2S` (initial abstraction — some use 0.05S per newer NRCS guidance, note which you used)
    - `Q = (P − Ia)² / (P − Ia + S)` for P > Ia, else Q = 0
5. Feed this excess-rainfall depth into the unit hydrograph from Phase 4 to get your final inflow hydrograph.

## Phase 8 — Breach parameters & breach hydrograph

1. Pull dam geometry from your NRLD entry: height (22.6m), crest length, and approximate storage-elevation curve (can derive from DEM + known reservoir extent if not published directly).
2. Compute breach parameters using **Froehlich (2008)** equations — this is what CWC itself recommends for Indian dams (per Dhiman & Patra 2019).
3. **Calibration step unique to choosing Machhu-II**: cross-check your Froehlich estimate against the actual historical/observed breach values documented for this dam in Wahl (1998) / the Indian dam-failure paper (links in the previous doc) — this lets you report a real accuracy comparison instead of a blind regression guess, which is the whole reason this dam was worth picking.
4. Generate the breach outflow hydrograph — either:
    - Do it directly in **HEC-RAS's built-in Breach Data editor** (enter breach width/side-slope/formation-time, it computes outflow via level-pool routing + broad-crested weir automatically), or
    - Compute manually in Python: breach growing linearly over the formation time, discharge via broad-crested weir equation, routed through reservoir storage.

## Phase 9 — 2D hydraulic model in HEC-RAS (the core prototype engine)

1. New HEC-RAS project → import your conditioned DEM (Phase 1) as terrain.
2. Create a **2D Flow Area** covering the downstream valley (Morbi city + the flooded villages — Togurupeta-style downstream extent, adjust names for Machhu/Morbi geography) — mesh cell size ~30–50m given your DEM resolution.
3. Add the dam as an inline structure / breach line at 22.82°N, 70.84°E.
4. Define the **Storage Area** upstream (the reservoir) with the storage-elevation curve from Phase 8.
5. Set the **inflow hydrograph** (Phase 7 output) as the boundary condition into the storage area.
6. Set a downstream boundary condition (normal depth using the channel slope near the outlet toward the Little Rann of Kutch).
7. Assign **Manning's n** roughness values per LULC class across the 2D mesh (built-up areas rougher than open agricultural land).
8. Enter your **breach parameters** (Phase 8) in the Breach Data editor and run the unsteady flow simulation.
9. Outputs you get directly from HEC-RAS Mapper: max depth grid, max velocity grid, flood arrival-time grid, and the inundation extent polygon.

## Phase 9.5 — 3D visualization of the results (the actual "3D" deliverable)

Three ways to do this, from zero-extra-effort to full interactive prototype. Do them in this order — each one builds on the last.

### Option A — HEC-RAS's own built-in 3D viewer (do this first, it's free and already there)

HEC-RAS Mapper has a native **3D Perspective Plot** mode — it drapes your Depth/WSE/Velocity results directly over the terrain, lets you rotate/pan/zoom through the scene, turn on velocity tracers, and even define a flight path so it auto-flies through the flooded valley. Zero extra tools needed. **Steps:** After your run finishes → View menu → 3D Perspective Plot → select Depth or WSE as the draped layer → use the navigation controls to fly through → optionally record a flight path animation for your presentation video. Docs: https://www.hec.usace.army.mil/confluence/rasdocs/rasum/latest/working-with-hec-ras (Chapter 8, "Viewing Results")

### Option B — QGIS 3D + Qgis2threejs (for a shareable web-viewable 3D model)

1. Load your conditioned DEM (Phase 1) into QGIS, open **3D Map View** (View → New 3D Map View), set it as the terrain elevation source, set a vertical exaggeration (1.5–3× is standard, disclose the value if you show it in your report).
2. Drape your HEC-RAS depth/WSE GeoTIFF output (exported from RAS Mapper as a raster in Phase 9) on top as a textured layer.
3. Install the **Qgis2threejs** plugin (Plugins → Manage and Install Plugins → search "Qgis2threejs") — https://plugins.qgis.org/plugins/Qgis2threejs/
4. Run it on your DEM+drape to export either: (a) a standalone web page you can open in any browser (view 3D terrain + flood layer, rotate/zoom, no install needed by whoever you show it to), or (b) a glTF/OBJ mesh file for use elsewhere (Blender, Cesium, Sketchfab).
5. For a flythrough video instead of a static/interactive model: QGIS 3D Map View → set camera keyframes at different points along the valley → export animation as MP4. Reference walkthrough: https://topostreets.com/beginners-guide-to-3d-map-creation-with-qgis-and-dem-to-mesh/

### Option C — CesiumJS (for a proper interactive 3D web-app prototype)

This is the heavier but most "final prototype"-looking option — a real geospatial 3D web app instead of a static export.

1. Get a free Cesium ion account: https://cesium.com/ion/ — this gives you globally-hosted terrain tiling so you don't have to build your own terrain server.
2. Upload your conditioned DEM as a custom terrain asset in Cesium ion (it tiles it automatically for streaming).
3. Export your HEC-RAS time-series depth/WSE grids (one raster per timestep from your unsteady run) and either: (a) convert each to a glTF "water surface" mesh at that timestep and animate by swapping meshes on a timer in your CesiumJS app, or (b) render depth as a semi-transparent colored overlay tile per timestep and step through them as a time-slider animation (simpler to build, still reads as "3D" since it's sitting on the 3D terrain, not a flat map).
4. Basic CesiumJS app is just an HTML file + a script tag pointing at the CesiumJS CDN + your Cesium ion access token — no backend needed for a prototype. Docs: https://cesium.com/learn/cesiumjs-learn/ and the terrain-from-raster discussion at https://community.cesium.com/t/generate-gltf-model-from-raster-dsm-instead-of-terrain/16531

### If you specifically need 3D _shapefiles_

If your plan is to keep everything in the GIS/shapefile ecosystem rather than a code-based 3D viewer: QGIS can extrude 2D flood-extent polygons by their depth attribute into **3D multipatch-style features** (Processing Toolbox → "Extrude" / use the 3D Map View's polygon extrusion), which you can then export back out as a Z-enabled shapefile or straight to glTF/OBJ per Option B above. This gets you a genuinely 3D dataset (not just a 3D-looking render) if that's a deliverable requirement for your project.

## Phase 10 — Validation

1. Compare your simulated inundation extent/depth qualitatively against documented accounts — e.g. reports of ~10 ft flood height reaching Morbi, the ASDSO case study narrative (https://damsafety.org/reference/dam-failure-case-study-machhu-dam-ii-gujarat-india-1979), contemporary flood-extent descriptions. There's no formal gauge network to validate against quantitatively for a 1979 event, so this stays qualitative — say so explicitly in your report rather than overstating confidence.
2. Run a **sensitivity analysis**: vary breach width/side-slope/formation-time by ±25%, 50%, 75% (this is literally the standard practice cited in the breach-parameter literature) and show how much your inundation extent/peak discharge shifts. This demonstrates you understand the uncertainty inherent in breach modeling — a strong point for an academic evaluation.

## Phase 11 — Turning it into an actual "prototype"

Since you're going 3D, build on Phase 9.5 rather than flat 2D maps:

- **Fastest**: Ship the QGIS Qgis2threejs web export (Option B) as-is — it's already a standalone 3D web page, viewable by anyone with a browser, no server needed.
- **More impressive**: CesiumJS app (Option C) with a time-slider scrubbing through your unsteady-flow timesteps, plus a dropdown to switch between your ±25/50/75% breach-parameter scenarios (Phase 10) — each scenario just swaps which precomputed set of depth tiles is loaded.
- **For your presentation/demo video regardless of which above**: record the HEC-RAS native fly-through (Option A) — it's the fastest way to get a compelling flood-progression animation without building any web app at all.

---

## Quick sanity-check list before you call it done

- [ ] Delineated catchment area ≈ 1,928 km² (±10%)
- [ ] Composite CN and resulting peak inflow are in a plausible range vs the documented ~5,550–5,663 m³/s
- [ ] Froehlich-estimated breach parameters compared explicitly against the historical/observed values (this comparison is your differentiator — don't skip it)
- [ ] Sensitivity analysis included, not just one deterministic run
- [ ] Validation section states clearly that it's qualitative (no gauge data existed in 1979) rather than implying quantitative validation you don't have