# 20261021 Idea: Crane Canvas — drop a tower crane on the model, see what it can lift, plan the site, replay every lift in 4D

## Entry 1

```
date-time:   2026-10-21 (scheduled run)
idea name:   Crane Canvas — "Drag a crane onto the plot. Every element turns green or red by whether the crane can
             lift it. Move it until the whole building is green."
             A web tool in cad2bim (works on any IFC, including structura/ETABS and Revit exports) plus a pyRevit
             button (AnonGee tab) that:
               (1) places tower cranes, a hoist, a concrete pump/boom, batching plant, rebar yard, formwork yard,
                   site roads and gates on the site plan (drag and drop, real footprints),
               (2) colours every element of the model by liftability: its weight (from IFC volume x density, or
                   bundle / precast / formwork panel weight) against the crane's load chart at that radius,
               (3) checks the swept area: jib over the neighbour's plot, road, power lines, school; two cranes
                   whose jibs overlap (height separation and slewing conflict), and hook height vs the tallest
                   level + rigging,
               (4) suggests the best crane position(s) and model (search over allowed positions and a crane
                   catalogue) for the fewest red elements and the lowest rental,
               (5) builds the site logistics plan per stage (excavation, substructure, superstructure, finishes)
                   and replays lifts in 4D from the schedule,
             then outputs the site logistics drawing set (PDF/DXF per stage), the lift study table (heaviest
             lifts, radius, capacity, % utilisation), crane hook-time per floor cycle and a 30-second 3D reel.
potential:   Very high for contractors, PMCs and tender teams on high-rise, precast, PEB and infra jobs. Today the
             "site logistics plan" for a tender or a mobilisation is drawn by hand in AutoCAD in 2-4 days: crane
             radius circles copied from a brochure, heaviest lift guessed, no link to the model or the schedule.
             Time saved: logistics plan + crane study from ~3 days -> ~1-2 hours (drop, check, optimise, export);
             every re-plan after a design or phasing change goes from a day to minutes.
             Risk saved: wrong crane size or position is one of the costliest site mistakes (a second crane, a
             re-erection, or rental for months of an oversized crane: Rs 4-8 lakh per month for a mid-size tower
             crane in India), and overload / jib-over-road incidents are a real safety and permit problem.
             Trending angle: "BIM for site logistics", "4D site planning" and "crane lift planning with digital
             twins" are regular LinkedIn topics in 2025-26 (contractors posting Synchro / Fuzor site videos), and
             crane time-lapse reels are among the most viewed construction content on Instagram. The reel here
             writes itself: building turns red/green as the crane is dragged, the jib sweeps, a red "overlap"
             flashes between two cranes, then the 4D replay lifts each floor's rebar and formwork in turn.
             Market: Synchro 4D (Bentley, paid), Fuzor (paid), Trimble SketchUp site logistics extensions (paid),
             Liebherr/Potain crane planners (free but brochure-level, no model, one brand only), 3D Lift Plan
             (paid, mobile cranes, no BIM), Navisworks TimeLiner (paid, no crane logic). Nothing free that reads
             the actual IFC, computes element weights, checks a load chart and draws Indian-style logistics
             sheets in the browser.
```

### Description

**The pain (daily life of a BIM team supporting a contractor or tender).**
- The planning engineer asks: "Which crane, where, and can it lift the transfer-girder cage / the precast
  staircase / the lift-core formwork?" The BIM modeller has the model but no tool to answer; they export a plan to
  CAD, draw circles from a brochure PDF and eyeball it.
- Weights of the heavy pieces (precast elements, rebar cages, steel trusses, shaft formwork, AC/DG units) are in
  the model as volumes and families, but nobody converts them to tonnes against a radius.
- Two cranes on the same plot, or a crane next to a neighbour's tower crane, need jib clearance checks; the
  jib swinging over a road or an HT line needs permissions. This is found when the crane arrives.
- The logistics plan changes per stage (excavation ramp, then rebar yard, then hoist and pump). Each stage is a
  separate hand-drawn sheet that never matches the latest model or schedule.
- Clients and site teams understand none of it from a 2D sheet; a 3D replay of the lifts would end the meeting
  in five minutes.

**The idea, in six parts.**

1. **Site in, building in.** Load the IFC (or the open Revit model). The plot boundary comes from the site/
   topography, a DXF of the survey (via C2B / cad2bim DXF reader), or is sketched. Neighbouring buildings,
   roads and power lines are drawn as simple extruded blocks with a height (or taken from an OSM building
   footprint import as a future add-on).

2. **Equipment library (editable Excel/JSON pack).** Tower cranes (hammerhead, luffing, flat-top) with load
   charts (capacity vs radius per jib length), mast sections, free-standing height and tie-in rules; mobile
   cranes (simple chart); hoist, concrete boom placer, stationary pump, batching plant, rebar cutting/bending
   yard, formwork yard, labour camp, site office, gates, wash bay, DG set. Each item is a real-size parametric
   block with clearance zones. Indian rental catalogue fields: monthly rent, erection/dismantling cost, power.

3. **Liftability map.** For each element (or each lift unit: one precast piece, one rebar cage, one formwork
   panel set, one steel member), weight = volume x density (from material) or a pack value; radius = horizontal
   distance from the crane mast to the element's centre of gravity; capacity from the load chart at that radius
   minus hook/rigging allowance. Element colour: green (< 75% utilisation), amber (75-100%), red (over capacity
   or out of reach), grey (not a crane lift: e.g. pumped concrete). Hook height is checked against the element's
   top + rigging + clearance, with the mast height raised automatically per stage (climbing / tie-in).

4. **Swept-area and clash checks.** The jib sweep is a disc/annulus; it is intersected with: plot boundary
   (overswing on neighbour or road -> amber, needs permission), power lines and exclusion zones (red), other
   cranes' sweep (if jibs overlap, check vertical separation per IS 14473 / BS 7121 practice and flag slewing
   conflict zones), and the building itself at every stage (luffing jib minimum angle near the core). Results are
   a list with plan snapshots, ready for the safety officer.

5. **Optimiser — "Find my crane".** The user marks allowed zones for the mast base (outside basement edge, inside
   a lift shaft, on a raft). The tool grids those zones, tries each catalogue crane at each point, and ranks
   options by: % of lift units green, worst utilisation, overswing area, number of cranes, and total rental for
   the schedule duration. Output: the top 3 options side by side with a cost line ("1 x 60 m flat-top at C3
   covers 98% of lifts, Rs 6.2 L/month; 2 x 45 m covers 100%, Rs 9.4 L/month").

6. **Stages, 4D and outputs.**
   - Stages are linked to the schedule (MS Project / Primavera XER / Excel; reuses Pour Planner's 4D data):
     excavation (ramp, dumper route, shoring from Dig Deep), substructure, each superstructure cycle,
     finishes (hoist, material landing platforms). Each stage gets its own layout and the tool checks access:
     truck turning radius at the gate, RMC transit mixer standing near the pump, rebar trailer reach.
   - **4D lift replay:** for every floor cycle the hook lifts rebar bundles, formwork sets and precast pieces
     from the yard to their final place; hook-time per lift (hoist speed, slew speed, trolley speed from the
     pack) adds up to **crane hours per floor cycle**, flagging when the crane, not the concrete, sets the cycle.
   - **Sheets:** site logistics plan per stage (A1/A3 PDF + DXF) with radius rings, capacity labels at key radii,
     legends, gates, roads, yards, safety zones; lift study table (top 20 heaviest lifts with radius, capacity,
     utilisation, rigging); crane foundation load summary (mast reactions from the catalogue) to hand to the
     structural engineer (structura / etabs-mcp can check the raft / pile cap under it).
   - **Reel:** one click renders the 30 s fly-through (reuses Model Reel) for the client and LinkedIn.

**Why it is a natural extension of this toolkit.**
- `cad2bim` already loads IFC in a worker, has the three.js viewer with picking, element quantities and material
  volumes (quantities and BBS to Excel), and the Revit bridge: liftability is a colour override on elements it
  already shows, and the weights come from quantities it already computes. BBS data gives real rebar cage and
  bundle weights.
- `structura` / `etabs-mcp` give member sizes, transfer girders and steel weights for steel/composite jobs, and
  can check the crane base slab or raft for mast reactions.
- `pyRevit-addons` (AnonGee tab) gives the in-Revit entry: same checks on the open model, results written to a
  shared parameter (`CC_Liftable`, `CC_Utilisation`) so it can be scheduled and filtered.
- It glues together earlier ideas: Pour Planner (schedule and cycles), Shutter Plan (formwork panel sets and
  their weights), BBS Weaver (cage weights), Dig Deep (excavation stage), Set-Out Pack (crane base setting-out
  points), Model Reel (video). Today each of them stops at "crane / truck planning to be done separately".

### Implementation

**Phase 1 — Web MVP: one crane, liftability map (2-3 weeks).**
- New panel "Site" in `cad2bim/apps/web`; equipment as simple meshes in the existing three.js scene, dragged on
  the ground plane (raycast to plane, snap to grid).
- `crane-pack.json`: a few generic tower cranes (40/50/60 m flat-top, 45 m luffing) with load chart arrays
  `[radius_m, capacity_t]`, interpolated linearly; editable in an Excel sheet and imported (xlsx already used).
- Weights: element volume from the engine's quantity code x density from material name (concrete 25 kN/m3,
  steel 78.5 kN/m3, timber/ply table) or explicit `Pset` weight; lift units grouped by storey + type.
- Colour override via the existing selection/highlight mechanism; summary bar: green/amber/red counts and the
  worst lift. Radius rings and capacity labels drawn as line meshes on the ground.

**Phase 2 — Checks, optimiser, sheets (3 weeks).**
- Swept-area checks with 2D polygon ops (clipper-lib or turf in the worker): sweep circle vs plot / neighbour /
  road polygons; crane-crane overlap with vertical separation rule.
- Optimiser: grid search over allowed zones (2 m step) x catalogue; score function as above; runs in a Web
  Worker, typically under a few seconds for a few hundred positions and a few thousand lift units.
- Sheets: plan projection + DXF writer (same approach as Drawing Autopilot / C2B DXF output), PDF via jsPDF
  (already used in structura). Lift table to Excel.
- pyRevit button: same logic in Python on `FilteredElementCollector` volumes, preview with
  `OverrideGraphicSettings`, crane as a generic model family with a jib-length parameter.

**Phase 3 — Stages and 4D (3-4 weeks).**
- Stage layouts saved as JSON in the project (one layout per stage, diffed for the sheet "what changed").
- Schedule import (Excel / MS Project XML / Primavera XER) mapped to storeys; lift replay animates hook path
  (slew + trolley + hoist) with a simple kinematic model; crane-hours per cycle chart.
- Export the stage layouts as IFC (`IfcBuildingElementProxy` / `IfcTransportElement` with a `Pset_CraneCanvas`)
  so the logistics plan travels with the model to Navisworks or any viewer.

**Tech notes / risks.**
- It is a planning assist, not a certified lift plan: every sheet carries "for planning; lift plan to be
  approved by the crane supplier / competent person", and the load chart source is printed on each sheet.
- Load charts vary by brand and configuration: the pack must be per crane model and editable; ship only generic
  examples and let the team add supplier charts.
- Weight from volume needs rebar included for cages and RC precast (use BBS data or a kg/m3 rate per element
  type from the pack).
- Keep the optimiser deterministic and explain its pick ("C3 chosen: 2 red lifts fewer than C1, Rs 1.1 L/month
  cheaper than 2 cranes") so planners trust it.

### Future update

- **Mobile crane mode:** outrigger footprint and ground bearing pressure check over basement slabs / retaining
  walls (link to structura for the slab check), for precast erection and steel trusses.
- **Wind and weather:** pull wind forecast for the site and mark days when the crane must stop (> 20 m/s), shift
  4D lifts and show the delay.
- **Live site link:** read the crane's anti-collision / load-moment data (or a simple app entry by the operator)
  to show actual lifts per day against the plan; feeds Bill Painter and Chat-to-Model's site diary.
- **Neighbourhood from maps:** import OSM building footprints and heights and HT line routes around the plot for
  a quick overswing study at tender stage.
- **Hoist and vertical transport:** passenger/material hoist capacity vs labour and material per floor per day,
  with a "people per hour" check for high-rise finishing stages.
- **Carbon and cost:** diesel/electric crane energy, rental and idle time per stage to compare a two-crane plan
  with a one-crane plan on cost and CO2 (a strong green-site LinkedIn post).
