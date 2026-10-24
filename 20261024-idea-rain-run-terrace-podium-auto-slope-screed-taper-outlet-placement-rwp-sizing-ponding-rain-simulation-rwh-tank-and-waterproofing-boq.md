# 20261024 Idea: Rain Run — terrace / podium / balcony drainage from the model: auto slope screed (tapered brick-bat-coba / foam concrete) with ridge-and-valley lines, auto rainwater outlet (khurra) placement, rainwater-pipe sizing and down-take routing, a 3D "rain on the model" ponding simulation, rainwater-harvesting pit / tank sizing, and a waterproofing + screed BOQ

## Entry 1

```
date-time:   2026-10-24 (scheduled run)
idea name:   Rain Run — "Press one button and it starts raining on your Revit model: water runs down the
             terrace, collects in blue puddles wherever the slope is wrong, and drains into outlets that
             Rain Run has just placed. Then the slope screed is drawn as a contoured 3D taper, the
             rainwater pipes size and route themselves down the shaft, the harvesting pit is sized for
             the city's bylaw, and the waterproofing BOQ is ready — before monsoon leaks become an RFI."
             Extensible version of the existing Drain Line idea (2026-10-20), which lists
             "storm water & terrace drainage" only as a future item. Rain Run makes that item a
             full tool and re-uses the Drain Line router.
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Wet" stack in Essential.panel next to
                   StrucBIM): Find Roofs, Auto Slope, Place Outlets, Size & Route RWP, Rain Test,
                   RWH Size, Waterproof BOQ,
               (2) cad2bim web viewer: animated rain/ponding replay on the IFC model with an
                   intensity slider (25 / 50 / 100 mm/h, or "Mumbai 26 July cloudburst"), share link
                   for client, consultant and waterproofing contractor,
               (3) etabs-mcp bridge: extra dead load of screed + ponding water + RWH tank on the
                   slab, and beam-depth check where the down-take pipe crosses a transfer beam,
               (4) earlier engines reused: Drain Line (pipe router, fall check), Sleeve Punch (slab
                   sleeves for outlets and RWP), Hanger Hero (pipe clamps on the shaft), Warning
                   Doctor (roof / floor health), Model Reel (MP4 rain clip), Set-Out Pack (outlet
                   and screed level points for site), Site QR Pass (outlet QR with test log).
             Outputs: terrace slope plan DXF (spot levels, ridge/valley lines, arrows, screed thickness
             contours), screed thickness heat-map, outlet schedule, RWP sizing sheet (catchment per
             outlet, rainfall intensity, pipe dia, count) as per NBC 2016 Part 9 / IS 1742 / IS 2527
             (editable tables), RWP riser schematic, ponding report with screenshots, RWH pit/tank
             design sheet per city bylaw, waterproofing + screed + coving + parapet-flashing BOQ, and
             a one-page "monsoon readiness" report for the client.
potential:   High — every building has a terrace, podium, balcony, sunken slab or basement ramp, and
             roof leakage is the number-one post-handover complaint in India (and a big one in the
             Gulf and South-East Asia). Monsoon flooding and leakage videos trend on LinkedIn and
             Instagram every June-September; clients and RERA complaints both punish it.
             Time today (one G+12 tower with 4,000 m² terrace + podium + 48 balconies):
               - slope design by hand: architect marks arrows on a PDF, site supervisor decides the
                 brick-bat-coba thickness on site: 1-2 days drawing, often "not modelled at all",
               - outlet positions and RWP count: rule of thumb ("one per 40 m²"), 0.5-1 day, often
                 under-sized at podium/drop-off where the tower roof also discharges,
               - modelling tapered screed in Revit: slab shape editor point by point, 1-3 days per
                 terrace, and redone when a staircase headroom, OHT or solar frame moves,
               - RWP down-take routing and sleeves: 1-2 days, clashes found at shaft stage,
               - RWH pit / tank size: separate Excel per bylaw, 0.5 day, often missing for approval,
               - screed / waterproofing BOQ: 1 day, screed volume typically off by 20-40 %
                 (average-thickness guess vs real taper).
             With Rain Run: 20-40 minutes for the full set from the existing model; re-run in 2-3
             minutes after any change. Expected saving: 5-9 working days per project for the BIM team,
             15-30 % screed saving through optimised ridge/valley layout (thinnest screed for the
             required fall), and far fewer leakage callbacks.
             Wow factor: the rain test. Rain particles fall on the 3D model, run along the slope arrows,
             pool as blue puddles in low spots (e.g. behind the lift machine room or under a solar
             frame) and spiral into the outlets; a counter shows "L/s per outlet" and turns red if a
             pipe would overflow. Fix with one click → re-run → no puddles. A 20-second reel is perfect
             for LinkedIn / Instagram (#BIM #Waterproofing #Monsoon #Revit #Plumbing #ConstructionTech).
             Market check: Revit has no auto taper — Slab Shape Editor and "Variable thickness" layers
             are manual point editing. Tapered-insulation design from roofing manufacturers is a
             vendor service tied to their foam boards and not for brick-bat-coba / foam-concrete Indian
             practice. Siphonic roof-drain tools (e.g. Geberit Pluvia, Wavin QuickStream design
             software) are free but brand-locked to siphonic systems and do not slope the slab. Paid
             add-ins (e.g. some tapered-roof Revit plug-ins, MagiCAD / Stabiplan drainage modules) are
             per-seat, Europe-centric, and do not simulate ponding or size RWH per Indian bylaws.
             A free, BIM-native slope + outlet + RWP + rain simulation + RWH + BOQ chain does not exist.
```

## Description

**The problem.** The terrace is the most leak-prone surface of a building and the least modelled one. The
structural slab is usually flat in the model; the slope screed, outlets and rainwater pipes are "site
decisions". As a result: water ponds behind parapets, near OHT legs and under services; outlets are too few at
the podium where the tower roof also discharges; the screed is 200 mm thick at one end (extra dead load the
structural engineer never checked); and the waterproofing BOQ is guessed.

**What Rain Run does.**

1. **Find drainable surfaces.**
   - Reads Floors / Roofs / Slabs tagged by level name or rule (Terrace, Podium, Balcony, Sunken, Utility,
     Ramp) or picked by the user; in IFC via cad2bim, IfcSlab with PredefinedType ROOF/FLOOR + IfcSpace names.
   - Finds obstacles and boundaries: parapets, OHT / LMR / staircase headroom walls, solar frames, equipment
     plinths (ODUs from Cool Load), expansion joints, openings and skylights, door thresholds (to keep the
     finished level below the door sill).

2. **Auto slope (ridge-and-valley design).**
   - Builds a 2D polygon of each surface with holes, chooses outlet positions (step 3) and computes a slope
     field so that every point drains to an outlet with fall ≥ minimum (1:100 default, editable per finish:
     1:80 terrace, 1:60 balcony, 1:50 ramp gutter).
   - Uses a straight-skeleton / Voronoi partition around outlets to draw ridge and valley lines like a hand
     drafter would, then minimises total screed volume subject to minimum thickness (e.g. 25 mm at outlet,
     max allowable at ridges) and door-sill / parapet-flashing height limits.
   - Writes it back to Revit as a separate "Slope Screed" floor with Slab Shape Editor points (or a sub-layer
     with variable thickness), plus spot elevations, slope arrows and ridge/valley detail lines in the plan.

3. **Auto outlet (khurra) placement.**
   - Rule-based seed: corners and parapet edges, min/max spacing, max catchment per outlet from pipe capacity,
     away from beams/columns (structural link) and above a shaft or along a façade drop line.
   - Iterates with step 2 to balance catchment area per outlet (so pipes are equal size), and adds an overflow
     spout through the parapet for each catchment (NBC good practice).

4. **RWP sizing and routing.**
   - Catchment area per outlet × design rainfall intensity (city table: e.g. Mumbai, Chennai, Kolkata, Pune,
     Delhi, Bengaluru, Dubai… editable) × runoff coefficient → flow in L/s; pipe dia from an editable
     NBC 2016 Part 9 / IS 1742 / IS 2527 capacity table; cascading flow where the tower roof discharges onto the
     podium is added automatically.
   - Routes the down-take pipe with the Drain Line router: through shafts or façade, around transfer beams,
     with offsets limited to 45° bends; places sleeves (Sleeve Punch), clamps (Hanger Hero), cleanouts at each
     bend and at ground, and connects to the storm-water chamber at plinth.

5. **Rain test (ponding simulation).**
   - Rasterises the finished top surface (slab + screed + finish) into a height grid (e.g. 50 mm cells),
     rains a given intensity for N minutes and runs a simple D8 / shallow-water flow to the outlets;
     cells that hold water above 5 mm after rain stops are "ponding". Pipe capacity limits outflow, so an
     under-sized outlet shows overflow.
   - Reports ponding area, max depth, location (with a screenshot and a BCF topic), and "time to drain".
     Works also on the as-built: import Scan Check point cloud of the screed to find real low spots before
     the waterproofing membrane is laid.

6. **Rainwater harvesting.**
   - Roof + paved catchment from the model; storage / recharge pit size by city bylaw rule table (editable:
     litres per m² of roof, pit dimensions, filter chamber), or by the rational method for first-flush and
     peak; places a pit/tank family in the site model and checks clearance to building foundation and
     boundary; writes an approval-ready sheet next to Area Statement Pro.

7. **Waterproofing + screed BOQ.**
   - Real screed volume from the taper (m³), brick-bat-coba / foam concrete / PCC options compared, membrane
     area with upturns at parapets (300 mm default) and coving lengths, outlet grating count, overflow
     spouts, RWP by dia (m), bends, clamps, cleanouts, sleeves, RWH items. Exported as Excel and as a
     "dead load added" note for the structural team (kN/m² map to etabs-mcp).

**Who uses it every day.** BIM modeler (slope and outlets in minutes instead of days of shape editing),
architect (terrace plan with arrows and levels), plumbing / MEP coordinator (RWP sizes, risers, sleeves),
structural engineer (screed + water load check), QS (BOQ), site engineer (spot level points, outlet set-out),
waterproofing contractor and client (rain-test reel and monsoon-readiness report).

## Implementation

**Data model (shared JSON, same pattern as the other engines).**
```
surface   { id, kind: TERRACE|PODIUM|BALCONY|SUNKEN|RAMP, level, polygon[], holes[], obstacles[],
            slab_top_m, finish_mm, sill_limits[{pt, max_ffl}], min_fall, min_screed_mm, max_screed_mm }
outlet    { id, surface_id, pt, catchment_m2, flow_lps, rwp_dia_mm, overflow_spout_pt, riser_id }
slope     { surface_id, ridges[], valleys[], points[{x, y, z_top}], screed_vol_m3, max_thk_mm }
riser     { id, outlets[], path[], dia_mm, bends, cleanouts[], sleeves[], discharge_to }
rain_run  { intensity_mm_h, duration_min, grid_mm, ponding[{cell_bbox, depth_mm}], overflow_outlets[],
            drain_time_min }
rwh       { catchment_m2, rule, required_l, pit[{type, dims, location}], ok }
catalogue { rainfall[{city, i_mm_h}], rwp_capacity[{dia_mm, max_area_m2_at_i}], runoff_c[{surface, c}],
            screed_types[{name, density_kn_m3, rate}], membrane_types[], rwh_bylaws[{city, rule}], rates[] }
```

**Core (pure Python, unit-tested, no Revit dependency) — `lib/py3/rainrun/core/`.**
- `partition.py`: polygon + holes → catchments per outlet (weighted Voronoi clipped to polygon, using
  `shapely`), straight-skeleton ridges/valleys.
- `slope.py`: z-field on a triangulation (constrained Delaunay) solved as a small linear program
  (min screed volume s.t. fall ≥ min along steepest-descent paths, thickness bounds, sill limits) —
  `scipy.optimize.linprog`; fallback to analytic planar facets per catchment for the MVP.
- `outlets.py`: seed + Lloyd-style relaxation to balance catchments, spacing and structural no-go zones.
- `hydraulics.py`: rational method, cascade roofs, pipe capacity table lookup.
- `ponding.py`: numpy height-grid flow (priority-flood depression filling + D8 routing); fast enough for
  10,000 m² at 50 mm in a few seconds; the same algorithm ported to TypeScript / WebGL for the viewer.
- Tests with known shapes (rectangle with one corner outlet, L-terrace with LMR, podium with tower discharge).

**Revit side (pyRevit-addons).**
- New `Wet.stack` in `Essential.panel` (AnonGee tab) with the buttons listed above; WPF UI with the AnonGee
  theme XAML (`Resources/*.xaml`), engine in `lib/py3/rainrun/` (numpy, shapely, ezdxf, openpyxl already
  vendored or easy to add).
- Writes the screed as a Floor with `SlabShapeEditor.AddPoint` / `ModifySubElement` (or `DrawPoint` +
  elevation) for each solved vertex; spot elevations, slope arrows (`SpotSlope`), detail lines on the
  terrace plan view; Outlets and RWP as plumbing fixtures / pipes on a "Storm Water" piping system, so normal
  Revit schedules and filters work.
- Shared parameters on outlets (RR_Catchment_m2, RR_Flow_Lps, RR_RWP_Dia) and on the screed floor
  (RR_MaxThk_mm, RR_Volume_m3).

**Web side (cad2bim).**
- "Rain Run" panel in `apps/web` using `@cad2bim/engine` for IFC slabs; rain particle + ponding shader in the
  existing 3D viewer; intensity slider; puddle depth colour scale; click a puddle → BCF issue.
- MCP tools in `packages/mcp`: `rainrun.slope`, `rainrun.outlets`, `rainrun.rain_test`, `rainrun.rwh`,
  `rainrun.boq` — so an agent can answer "will the podium flood at 100 mm/h and how many RWPs do we need?".

**Structural link (etabs-mcp).** Push screed + ponding + RWH tank loads as area loads per slab panel; read
beam depths and transfer-beam locations to keep RWP routes and outlets clear.

**Outputs.** ezdxf (slope plan with levels, arrows, ridge/valley, thickness contours; RWP riser schematic),
openpyxl (outlet schedule, RWP sizing, RWH sheet, BOQ), HTML-to-PDF monsoon-readiness report, IFC export
(IfcCovering for screed with Pset_RainRun, IfcFlowTerminal outlets, IfcPipeSegment RWP), MP4 via Model Reel.

**MVP (2-3 weeks).**
1. Pick terrace floor → auto outlets at corners/edges by rule → planar-facet slope per catchment.
2. Write slope screed floor with shape points + spot levels + arrows in Revit.
3. RWP sizing sheet from a city rainfall table + outlet schedule + screed volume BOQ in Excel.
4. Simple ponding check (grid + depression filling) with a coloured result view.

**V1 (next 4-6 weeks).** Ridge/valley optimiser (min screed), cascade podium flows, RWP routing with Drain
Line router + sleeves + clamps, rain animation in cad2bim, RWH pit sizing with 4-5 city bylaws, etabs-mcp load
push, slope plan DXF set, BCF export of ponding spots, Model Reel rain clip.

## Future update

- **As-built screed check:** phone LiDAR / total-station spot levels (Set-Out Pack, Scan Check) compared
  with the designed taper before the membrane goes on — "fix this 6 m² low spot now, not after the first
  monsoon".
- **Balcony and toilet sunken slab mode:** same slope engine for balconies, wash areas, sunken toilets and
  kitchens with floor-trap position (Drain Line link) and door-sill limits.
- **Basement & ramp storm water:** ramp gutters, basement sump sizing and pump selection, and a "street
  flood level" check for the entry ramp crest — a direct answer to urban-flooding headlines.
- **Green / blue roof:** detention-roof design (store and slowly release rain), green-roof soil load and
  drainage board, IGBC / GRIHA water-efficiency points.
- **AI from a photo:** upload a site photo of a ponding terrace → match it to the model and propose new
  outlets / screed correction for repair and maintenance teams (Plan Tracer / Scan Check tech).
- **Siphonic mode:** for large malls / warehouses / PEB roofs, gutter + siphonic outlet sizing with a
  brand-neutral hydraulic check.
- **Improvements to existing tools:** (a) Drain Line — move its path-finder into a shared `lib/py3/mep_router`
  so Drain Line, Cool Load, Live Wire and Rain Run share one router; (b) Warning Doctor — add a "wet area
  health" check (terrace floors without slope screed, outlets not connected to a system, balconies with
  finished level above the door sill); (c) Area Statement Pro — append the RWH sheet to the approval set
  automatically; (d) Model Reel — add a "rain" camera preset so every project gets a monsoon clip for
  social media.
