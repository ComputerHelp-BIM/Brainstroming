# 20261021 Idea: Stair Smith — pick the stair core, get a code-checked staircase in 3D, its rebar, its section sheet and its BBS

## Entry 1

```
date-time:   2026-10-21 (scheduled run)
idea name:   Stair Smith — "Click the stair box on the plan. Pick dog-legged, open-well, three-flight or
             scissor. Get the stair in 3D on every floor, with steps that fit the floor heights, headroom and
             NBC checks done, waist slab and landing rebar placed, the section sheet drawn and the BBS made."
             One engine (Python, in C2B) with three front ends:
               (1) C2B: the stair rectangle in the architect's DWG/DXF (layer STAIR, or the closed polyline
                   marked "STAIR" that C2B today turns into a shaft cut-out) becomes a real staircase instead
                   of a hole. Today C2B's mapping has "stairs": False; this switches it on,
               (2) pyRevit button (AnonGee tab, Essential panel): select a room or draw a box, pick the type,
                   get native Revit Stairs + landings + railings on every storey, plus Structural Rebar in the
                   waist slab and landings,
               (3) structura / cad2bim web: drop the same stair into the IFC so the structural model, BOQ and
                   viewer show it (IfcStair / IfcStairFlight / IfcSlab landings / IfcReinforcingBar).
             Outputs: 3D stair per floor, per-floor riser / tread table (riser count and height change with
             floor height), stair plan + section + rebar detail sheet (DXF/PDF), BBS (Excel, IS 2502 shape
             codes), concrete / shuttering / tile / railing quantities, and a check report.
potential:   Very high, every day. Every building with more than one floor has 1-6 staircases, and in Indian
             residential / commercial work the staircase is the single most re-drawn element: architects change
             a floor height or a landing and the riser count, the section, the rebar and the BBS all change.
             Time today (one stair core, G+10, with typical + odd floors):
               - modelling in Revit (stairs by component, landings, railings, per level): 3-6 hours, longer
                 when floor heights differ (lobby, podium, mezzanine), because each run is redone,
               - stair section + rebar detail in AutoCAD: 3-5 hours,
               - BBS for waist slabs (cranked bars, nosing bars, landing beams): 1-2 days by hand in Excel,
                 and it is the BBS most often wrong on site (crank lengths and development lengths),
               - checks (riser/tread/headroom/width vs NBC 2016 Part 4) done by eye or not at all.
             With Stair Smith: ~10-20 minutes per core including review; every floor-height change is a
             re-run. Across a typical project with 2-3 cores and 3-5 design revisions this saves 1-2 weeks of
             modeller + draughtsman time.
             Quality saved: headroom clashes with the beam above the landing, riser heights that differ on
             the last step (a trip hazard and an NOC objection), wrong crank lengths in the BBS, and missing
             stair openings in the slab model are routine site RFIs. Stair Smith checks all of them.
             Trending angle: "generative design", "parametric BIM" and "one-click Revit automation" posts
             are among the most shared on LinkedIn in 2025-26, and short reels of Dynamo/Grasshopper stairs
             assembling themselves do very well on Instagram. The reel here: a 2D CAD plan, a click on the
             stair box, flights rise floor by floor, the riser table counts up, rebar threads into the waist
             slab in orange, a red headroom cone flashes under a beam and turns green after one click, and
             the section sheet + BBS slide out. 30 seconds, no explanation needed.
             Market: Revit's own Stair-by-Component (manual per level, no rebar, no code check); Dynamo
             community stair graphs (fragile, no rebar/BBS); Autodesk Revit "Structural Precast" and
             paid add-ins like Naviate / Ideate (no Indian code, no BBS); Tekla / Allplan / Graitec stair
             rebar macros (paid, expensive, not for Revit-only teams); RCDC / STAAD staircase design (paid,
             no 3D BIM stair, no drawing from the model). Nothing free that goes CAD box -> 3D stair on every
             floor -> IS 456 / NBC checks -> rebar -> section sheet -> IS 2502 BBS in one step.
```

### Description

**The pain (daily life of a BIM modeller in an Indian architecture / structure / BIM-services office).**
- The architect's plan shows the stair as a rectangle with a few lines and "UP". The modeller has to work out
  riser count, tread, mid-landing level, flight lengths and which side goes up, separately for every floor
  because the ground floor is 4.2 m, the typical is 3.0 m and the terrace is 3.3 m.
- Revit stairs are slow: Stair-by-Component per level, then landings that do not join the floor slab,
  railings that break at the landing, "multistorey stair" that fails when heights differ. Many offices simply
  model the stair as a sloped slab plus generic steps, which then gives wrong quantities.
- Structurally the stair is a separate world: waist slab thickness from span/depth, main bars that crank at
  the landing, distribution bars, nosing bars, landing beam or slab-supported landing. Nobody models this rebar
  in 3D, so the section is drawn in AutoCAD and the BBS is typed by hand, and they drift apart.
- Checks are manual: riser ≤ 150-190 mm by occupancy, tread ≥ 250-300 mm, 2R + T rule, max 12-15 risers per
  flight, clear width by occupancy, headroom ≥ 2.2 m (2.1 m min) under beams and the flight above, handrail
  height, landing ≥ width. Fire NOC drawings get rejected on exactly these numbers.
- When a floor height changes in revision 4, everything above is redone by hand.

**The idea, in five parts.**
1. **Stair Box -> Stair Layout solver.** Input: the stair box (from DXF polyline, Revit room/box, or IFC
   opening), the level list with floor-to-floor heights, stair type, "up" direction/side, occupancy (residential
   / commercial / assembly / educational per NBC Part 4) and preferred riser. The solver finds, per floor, the
   riser count and equal riser height, tread, flight split (equal or 2/3 + 1/3 for open-well), mid-landing
   levels and widths, and the well gap, to fit the box. If it cannot fit, it says why and by how much
   ("needs 350 mm more length or 1 more flight").
2. **3D everywhere.** Revit: native Stairs (runs + landings) and Railings per level, landings joined to the
   floor slab edge, stair opening cut in each slab. IFC/web: IfcStair with IfcStairFlight and IfcSlab
   (LANDING) — same data, so cad2bim viewer, structura BOQ and C2B models all agree.
3. **Code and clash checks.** NBC 2016 Part 4 (width/riser/tread/flight/headroom/handrail), IS 456 (waist slab
   span/depth, cover, min steel, spacing), headroom cone swept along every flight against beams, ducts, the
   flight above and lintels — each failure as a coloured 3D marker and a line in the report, with a one-click
   fix where possible (change riser count, shift landing, raise beam soffit flag for the structure team).
4. **Rebar + BBS.** Waist slab designed from the effective span and occupancy live load (IS 875 Part 2): thickness
   and bar dia/spacing from a simple IS 456 one-way slab check (engineer can override). Places main bars with the
   crank at the landing (correct 45/90 degree development per IS 456 Cl. 26.2), distribution bars, nosing/top
   bars, landing slab or landing beam bars, starter bars into the floor beam. Exports BBS in IS 2502 shape codes
   (same format as the BBS Weaver / Bar Optimizer ideas already in this repo, so it flows straight into cutting
   optimisation).
5. **Sheet out.** One DXF/PDF sheet per stair: plan at each distinct floor, longitudinal section with levels,
   riser table, rebar detail section with bar marks, BBS table, general notes. Uses the same DXF writer as C2B's
   template_dxf export, title block from the office template.

**Why it is a natural extension of this toolkit.**
- **C2B** already reads the architect's DWG/DXF and turns closed polylines labelled STAIR into shaft cut-outs
  (`revit/mapping.py`: `shaft_from = ["LIFT","SHAFT","STAIR"]`, `"stairs": False`). Stair Smith turns that
  hole into the actual stair and keeps the slab opening — the missing piece in "2D CAD to 3D Revit".
- **pyRevit-addons (AnonGee)** gets a high-value one-click button; the Revit API (`StairsEditScope`,
  `StairsRun.CreateStraightRun`, `StairsLanding.CreateSketchedLanding`, `Railing.Create`,
  `Rebar.CreateFromCurves`) supports all of it.
- **structura** already understands sloping slabs / stair flights at their own level in its parser and has
  `rebar.js` + `boq.js`; the IFC stair gets rebar quantities and BOQ for free.
- **cad2bim web** shows the stair in the IFC viewer, so the client review link shows it.
- Shares BBS format with the BBS Weaver and Bar Optimizer ideas and quantities with Bill Painter.

**Wow moments.**
- A whole G+12 staircase with 3 different floor heights appears in under a minute, every riser equal on every
  floor, landings meeting the floor slabs.
- Headroom cone: a translucent green tunnel above every flight; anything that pokes into it turns red.
- Change floor height of level 5 from 3.0 to 3.15 m -> stair above re-solves, rebar and BBS update, the sheet
  revision cloud appears automatically.

### Implementation

**Phase 1 — Solver + checks + DXF sheet (2 weeks).**
- Python package `c2b.stairs`: pure functions `solve_layout(box, levels, type, rules) -> StairPlan` (riser count
  search, flight split, landing levels) and `check(stair_plan, rules) -> [Finding]`. Rules in a YAML profile
  (`profiles/stairs_nbc2016.yaml`) so other codes (IBC, BS, UAE) are a new file, not new code.
- DXF sheet generator reusing C2B's DXF export (ezdxf): plan, section, riser table, notes.
- CLI: `c2b stairs plan.dxf --levels levels.csv --type dogleg --out stair_sheet.dxf`.
- Unit tests on known cases (e.g. 3000 mm floor, 150 mm riser -> 20 risers, 2 x 10; 3150 mm -> 21 risers at
  150 mm, 11 + 10 with landing shift).

**Phase 2 — Revit + rebar + BBS (3 weeks).**
- pyRevit button: pick room/box, form (WPF from AnonGee UI_Templates) with type, side, occupancy; calls the same
  solver (IronPython/CPython bridge, or the solver ported to a shared JSON contract via the existing `bridge/`).
- Revit creation inside `StairsEditScope`; railings by type; slab opening via `NewOpening`.
- Waist slab design + rebar curves generated from the StairPlan; `Rebar.CreateFromCurves` with shape matching;
  BBS export to Excel (openpyxl) in IS 2502 format.
- Headroom check: swept solid along each flight at 2.2 m, `ElementIntersectsSolidFilter` against beams/ducts.

**Phase 3 — IFC/web + C2B pipeline (2 weeks).**
- C2B: set `"stairs": True` path — STAIR polylines feed the solver and emit stairs in the Revit plan/JSON.
- IFC writer (IfcOpenShell): IfcStair/IfcStairFlight/IfcSlab LANDING + IfcReinforcingBar, so structura and
  the cad2bim viewer pick it up; web panel to tweak type/side and download the sheet and BBS.
- Demo reel recording script (camera path + timed reveal) for LinkedIn/Instagram.

**Tech notes / risks.**
- Revit multistorey stairs vary by version (2023+ have `MultistoreyStairs`); start with one stair per level,
  then group.
- Rebar design is a starting design for the engineer to approve, clearly marked "for review", never final.
- Odd boxes (curved, quarter-turn with winders) are Phase 4; winders are restricted by NBC for exits anyway.
- Keep the solver deterministic and code-free in rules so QA can verify by hand.

### Future update
- Winders, spiral and helical stairs, ramps (1:12 / 1:20 accessibility per Harmonised Guidelines 2021) using the
  same solver.
- Fire-escape staircase mode linked to the Exit Map idea: stair width from occupant load, and egress replay
  uses the real stair geometry.
- Precast stair mode: split flights into precast units, weight per unit (feeds Crane Canvas liftability),
  lifting inserts and shop drawings.
- Steel stair mode (stringers, chequered plate treads) with connection plates and fabrication DXF.
- Formwork for waist slab and landings feeding Shutter Plan; concrete pour sequence feeding Pour Planner.
- Finish takeoff: tread/riser tiles, skirting, nosing strips, handrails in running metres for the Wall Wizard
  finish BOQ.
- AI mode: point at a scanned PDF plan (Plan Tracer) and it finds the stair box and "UP" arrow itself.
