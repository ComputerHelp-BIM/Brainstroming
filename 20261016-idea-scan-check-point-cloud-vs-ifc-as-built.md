# 20261016 Idea: Scan Check — phone LiDAR / drone scan vs the IFC model: as-built deviation heatmap, plumb/level/flatness report and auto punch list

## Entry 2

```
date-time:   2026-10-16 (scheduled run, second idea of the day)
idea name:   Scan Check — drop in a point cloud or mesh scanned with an ordinary phone (iPhone/iPad LiDAR via Polycam /
             Scaniverse / 3D Scanner App, PLY / LAS / LAZ / E57 / OBJ) or a drone photogrammetry export next to the IFC
             model; the browser aligns them, colours every column, wall, slab and beam by "how far is the built thing from
             the model" (green / amber / red), reports verticality, level, flatness, size and position errors against
             tolerance (IS 456 / IS 14687 / ACI 117 / BS 8204 style, editable), and issues a punch list + BCF + 2D DXF
             deviation sheet for the site engineer
potential:   High. "Is it built the way we modelled it?" is checked today with a tape, a plumb bob, a dumpy level and a
             clipboard, element by element, after the formwork is struck (when fixing means chipping or grinding). Total-
             station / terrestrial-scanner "as-built vs BIM" tools exist (Trimble, Leica Cyclone, FARO As-Built, Autodesk
             ReCap + Revit, Verity, Cupix, Reconstruct) but are paid, need heavy desktop software, a native Revit/Navisworks
             model and a surveyor. Nothing free that runs in a browser, keeps the files on the PC, speaks IFC *and* the
             Computer-Help DXF/BBS world, and gives an engineer-readable verdict for a 40 m2 column tilt. Phone LiDAR scanning
             is a standing LinkedIn / Instagram trend ("I scanned the whole slab with my iPhone in 3 minutes"), and a 15-second
             reel of a model turning green/red over a real scan is highly shareable. Saves 1–3 days of manual checking per
             floor, catches pour defects (bulging formwork, tilted columns, slab dips, wrong sizes) before the next floor
             sits on them, and gives the contractor an *objective* record for disputes and handover.
```

### Description

**The pain (daily life of a BIM modeller / site-QA engineer).**
- The IFC model says the column is 300×600 at gridline C-4, plumb. On site nobody knows whether it is 285×620, 18 mm out of
  position and 12 mm out of plumb until a clash with the brick wall or lift guide rail appears.
- Slab level and flatness are checked at random points with a staff and level; floor screed, tile and false-ceiling
  quantities are then wrong by tens of percent.
- As-built drawings are redrawn by hand from tape measurements, so the "as-built BIM" handed over is really the *design*
  BIM with a new date.
- Phone scanners already give an accurate cloud (≈ 1–3 cm indoors), but the cloud is a pretty 3D toy: nobody turns it into a
  tolerance verdict against the model.

**The idea.** A **Scan Check** command in structura / cad2bim (browser, files stay on the PC):

1. **Input** — IFC (or DXF → 3D already in cad2bim) + one scan file (PLY / LAS / LAZ / E57 / OBJ / GLB). Large clouds are
   decimated by a voxel grid in a Web Worker, streamed in tiles; a 5–20 M point cloud stays interactive.
2. **Align** — 3 steps, no surveying knowledge:
   - *auto*: coarse alignment by up-axis detection (gravity from the scan) + dominant floor/wall plane matching, then ICP refine;
   - *assisted*: user clicks 3 pairs of matching corners (model corner ↔ scan corner);
   - *known control*: enter 2–3 grid-intersection coordinates measured on site.
   The residual of the alignment is shown, so the user knows when the comparison itself can be trusted.
3. **Segment by element** — for each IFC element, take the scan points inside its tolerance envelope, fit the right primitive
   (plane for wall / slab / beam soffit, rectangle for column section, line for beam axis) with a robust RANSAC fit, ignoring
   clutter (formwork props, scaffolding, people) via the fit residual and a minimum-support rule.
4. **Measure & judge** — per element, against editable tolerances (defaults from IS 456 / IS 14687 / ACI 117 / BS 8204, with
   the clause cited in the report):

   | Check | Output | Default tolerance |
   |---|---|---|
   | Position | dx, dy, dz of the fitted centroid / face vs model | ±12 mm column, ±10 mm wall |
   | Plumb | tilt over storey height in mm and mm/m | H/500 or 12 mm max |
   | Size | fitted width, depth, thickness vs modelled | −6 / +12 mm |
   | Level | slab surface mean vs modelled RL | ±10 mm |
   | Flatness | 3 m straightedge simulation / F-number style | 5 mm / 3 m |
   | Slab dip / deflection | min–max map vs plane | flagged by user limit |
   | Bulge (formwork blow-out) | local max normal deviation of a wall / beam face | 8 mm |

   Each element gets a **green / amber / red** verdict and a one-line reason ("column C-4 tilts 17 mm over 3.0 m: limit 6 mm").
5. **Show** — the model coloured by verdict; click a column and see a **deviation heatmap** painted on its faces and a section
   slider showing scan points vs model outline. Reports: element-level CSV, floor summary (% pass), and a **punch list** that is
   also exported as **BCF** (viewpoint at the element) and a **DXF deviation sheet** (plan with red dimension arrows "−18 mm" at
   every out-of-tolerance column and a slab-level contour map).
6. **Feed forward** — three uses beyond a QA report:
   - *Rebar & concrete quantities*: replace the design size with the fitted as-built size for the next floor's BBS / RMC order
     (cover up, concrete volume up, lap lengths unchanged).
   - *Next-floor setting-out*: one click prints the axis offsets that the next-floor column shuttering should be shifted by to
     recover plumb (a "kicker correction" table).
   - *Handover*: an **as-built IFC** with the real positions and sizes, plus an `Pset_AsBuilt` property set (tilt, level error,
     scan date, scanner) — the model that finally matches the building.

**Why it is practical every day.**
- Site engineers already own a phone; a 3-minute walk-through scan of a slab replaces a morning of staff-and-level readings.
- The modeller no longer redraws as-built; they review a heatmap.
- Disputes: "your column is 20 mm out" is now a dated, signed, reproducible number.
- It extends tools already in the toolkit (IFC viewer, rebar/BBS, Twin Check, Change Radar, Site QR Pass) rather than being a
  new silo.

### Implementation

**Where it lives.** `structura` (IFC viewer, section tools) hosts the 3D scene and the heatmap; `cad2bim` provides the DXF
deviation sheet; `planwin-ai-pro` supplies the design sizes for Twin-Check style comparison. No server: everything runs in the
browser with files local.

**Stack & modules** (all open source):
1. *Loaders*: `three.js` PLYLoader / OBJLoader / GLTFLoader; `laz-perf` WASM for LAS / LAZ; E57 via `libE57` compiled to WASM
   (phase 2; phone apps mostly export PLY / LAS / OBJ).
2. *Decimation & tiling*: voxel-grid downsample in a Web Worker, k-d tree (`kdbush` / custom) for nearest-neighbour; octree in
   an `OffscreenCanvas` / GPU buffer for rendering (`Potree`-style, or `three.js Points` with LOD for the MVP).
3. *Alignment*: gravity/up-axis from the scan (IMU where present, else dominant horizontal plane); coarse yaw by wall-direction
   histogram vs the model's wall-direction histogram (1D cross-correlation); point-to-plane ICP (small WASM / pure TS) on
   10–50 k sampled points; manual 3-point Kabsch fallback.
4. *Element fit*: for each IFC element, take its OBB inflated by a tolerance + capture band; RANSAC plane (walls, slabs,
   beam faces), RANSAC rectangle (columns: 4 planes with orthogonality constraint), least-squares line (beam axis). Return
   centroid, normal, size, residual RMS and inlier ratio; low-inlier elements are reported "not scanned / hidden", never silently
   green.
5. *Metrics*: pure functions (`tilt(normal, upAxis, height)`, `levelDelta`, `flatness3m(grid)`, `bulge(residualMap)`) with
   unit tests on synthetic clouds (known tilts / dips injected, e.g. +14 mm tilt must be reported 14 ±2 mm).
6. *Tolerance rule table*: JSON, editable in a side panel, versioned with the project; reports cite the rule id and code clause.
7. *Heatmap*: per-face texture / vertex colours from point-to-face signed distance; legend in mm with a diverging colormap
   (colour-blind-safe, per the dataviz guidance).
8. *Outputs*: CSV/XLSX, BCF 2.1 (viewpoint + camera), DXF sheet (layers `SCAN-DEV-OK/AMBER/RED`, dimension arrows, level
   contour polylines), as-built IFC (`IfcPropertySet`), PDF summary.

**Milestones.**
- *M1 (1–2 weeks)*: load IFC + PLY, manual 3-point align, slab level + column tilt/size for rectangular columns, green/red view, CSV.
- *M2 (+2 weeks)*: ICP auto-align, walls and beams, flatness and bulge maps, tolerance table UI, BCF + DXF deviation sheet.
- *M3 (+2 weeks)*: LAS/LAZ/E57, large-cloud tiling, as-built IFC export, "feed-forward" to BBS / RMC quantities.

**Risks & honest limits.**
- Phone LiDAR drifts over big spaces (≈ 1 cm per 5 m): the tool reports alignment residual and scan-stitch warning, and
  recommends scanning in zones with control points (a printed A3 target or QR sheet on columns, which Site QR Pass can generate).
- Clutter and shuttering occlude elements: coverage % is shown per element; low coverage is never marked green.
- It is a *screening* tool, not a replacement for a licensed survey when a contract demands one; reports say so and show the
  scan device and expected accuracy.

### Future update

- *Time-lapse*: compare Scan A (week 1) and Scan B (week 3) and the model to show **progress %** (feeds the 4D / Pour Planner
  and the RA bill) — "what has actually been built".
- *Rebar cage scan*: scan the bar cage before pour; count stirrups and measure spacing / cover against the BBS (links to Joint
  Congestion Doctor and Bar Optimizer) — closes the "was the steel placed as drawn?" loop.
- *MEP and finishes*: reuse the same engine for sleeve positions (Sleeve Punch) and wall flatness before plaster / tile
  quantity.
- *Auto-photo mode*: a handful of overlapping phone photos → on-device photogrammetry (WebGPU Gaussian-splat / SfM) when the
  phone has no LiDAR.
- *AI summary*: "Floor 3: 92 % columns in tolerance; C-4, D-7 tilt > 12 mm; slab dips 22 mm near gridline 5–6" generated in
  one sentence for the daily site report, plus a LinkedIn-ready before/after image of the heatmap.
- *Tolerance packs*: country-specific rule sets (IS, ACI, EN 13670, BS) switchable per project.
