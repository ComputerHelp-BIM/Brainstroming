# 20261027 Idea: Basement Box: draw the basement outline once, and cad2bim builds the whole below-ground box, i.e. propped retaining walls, raft, lift pits, sumps, ramp and waterproofing, checks it for soil, surcharge and groundwater, and writes out the rebar, BBS, BOQ and a dewatering plan

## Entry 1

```
date-time:   2026-10-27 (scheduled run, UTC)
idea name:   Basement Box. "A B+G+12 job in a city plot with a high water table. The modeller
             has the basement outline in a DXF (or the plot lines in the model). In cad2bim they
             click Basement, pick the outline, and type three things: number of basements (2),
             floor-to-floor (3.6 m), and the soil / water-table level from the bore log
             (GWT 2.5 m below ground). Basement Box builds the box in 3D in seconds: perimeter
             retaining walls propped by the basement slabs, the raft, the lift pits and
             sumps, the ramp, and the waterproofing layers. The modeller turns on 'Earth and
             water' and sees coloured pressure arrows on the walls: soil, 20 kN/m2 vehicle
             surcharge from the road side, and water below the table. The checks run live:
             'Wall W3: at-rest pressure at the raft 94 kPa, wall thickness 300 is too thin for
             the 7.2 m unpropped height at the ramp opening. Add a 400 wall or move the ramp
             prop slab.' One click on 'Fix' and the wall thickens to 400 only on that bay.
             Then 'Flood test': water rises outside the box and the tool shows uplift on the raft
             when the pit is dewatered and the building is still light, with a warning on the
             dewatering stage: 'Uplift 1.1 x ... self weight at the raft is short by
             ... kN at the end of stage 2: keep the dewatering on until the 4th floor slab is
             cast.' Out come the wall and raft rebar, the BBS, the BOQ (excavation, PCC,
             waterproofing, concrete, steel, shuttering, dewatering pump-days), a drawing
             sheet set (plan, two sections, wall elevation with bar marks, ramp detail)
             and a one-page design note for the checking engineer. Two days of basement
             set-up for three people becomes one afternoon for one modeller."
             Why it is a "wow": the person sees the water and the earth push on their own
             model, and sees a red warning turn green after one click.
             One engine, four front ends:
               (1) cad2bim web app: a "Basement" dock panel (outline, levels, soil, checks,
                   stages, sheet). Output is IfcWall / IfcSlab / IfcFooting / IfcReinforcingBar
                   and IfcBuildingElementProxy for pits, so QA, BOQ and BBS already work,
               (2) cad2bim CH template v2: a closed polyline on a layer BASEMENT-LINE with a
                   label such as "B2 3.6 GWT-2.5" is enough input; the pipeline's v2 reader
                   passes the outline and values to the Basement engine,
               (3) Revit (bridge add-in): "Place basement" creates walls, floors, pits, ramp
                   and Structural Rebar on the right levels, with a schedule,
               (4) etabs-mcp: the earth and water pressures go to ETABS as wall pressure
                   loads (load cases EP, SUR, WT) so the engineer can run the full model.
potential:   HIGH, and I think it is a real gap, but I have not done a market search in this
             session, so treat the 'not in the market' line below as an estimate.
             - Daily use. Every urban RCC project of more than G+6 in an Indian metro has one
               or more basements. It is the slowest, most error-prone part of the model: walls
               change thickness with depth, openings for ramps and lift pits break the
               wall layout, and every redesign by the structural engineer moves the walls.
             - Time saved (estimate, not measured): setting up a 2-basement box by hand in
               Revit is about 1.5 to 2 days, plus another 1 to 2 days for the rebar of the
               walls and the BBS. Expected with Basement Box: 3 to 4 hours for the first
               pass and minutes for a redesign. To be measured on two real jobs in the pilot.
             - Money. Basement mistakes are the costliest ones on site: wrong wall thickness
               found after casting, a lift pit that does not match the lift maker's drawing,
               a ramp that does not clear a beam, and waterproofing that is missed. One
               avoided rework on a basement wall is worth more than a year of the tool.
             - Trend on LinkedIn and Instagram (the search term is only my reading of the
               feeds, to be confirmed): basement collapses and flooding in monsoon, 'excavation
               and shoring' reels, and 'waterproofing failure' site videos get a lot of views
               in Indian construction accounts. 'See the water push on your model' is a clip
               that people will share.
             - Uniqueness. Each piece exists on its own: retaining wall design in general
               structural design software, shoring in geotechnical software, waterproofing
               takeoff in estimating packages, most of them paid and none of them in the BIM
               model. What I do not know of is one tool, free to the team, that goes from an
               outline to the 3D box with checks, rebar and BBS in an IFC. Inside our own
               toolkit it extends Dig Deep (earthwork and shoring), Pile Pilot (soil strata),
               Rain Run (water), Bar Optimizer (BBS) and Tank Smith (water-retaining design).
description: The problem in one paragraph. A basement is not a set of walls. It is a box that
             has to hold back soil and water, stay put when the water pushes it up, let a
             car in, let a lift go down to a pit, and drain what leaks. In the office the
             pieces are done by different people with different tools: the architect draws
             the ramp, the structural engineer designs the wall in a calculation sheet, the
             modeller draws it again in Revit, the geotech writes the water table in a PDF,
             and the site plans the dewatering by experience. When any one of them changes,
             the others find out late.

             What Basement Box does, step by step.
             1. Input. Outline (from DXF, or drawn, or picked from the model's plot),
                number of basements, floor-to-floor heights, ground level, soil layers (from
                Pile Pilot's bore-log strata if present, or typed: unit weight, friction angle,
                cohesion) and ground water level. Defaults are written plainly and can be
                changed. Everything the user types is saved in the .c2b project file.
             2. Build. Perimeter wall on the outline offset inward by the wall thickness,
                basement slabs at each level, raft or footings under the box (user chooses; raft
                is the default under a high water table), ramp with the user's slope and width
                (the slope limit is read from a local rule table, e.g. 1:8 to 1:10 for
                parking ramps in many by-laws, to be checked per city), lift pits at every
                lift core with the depth taken from a lift-maker table the user can edit
                (typical 1.4 to 1.6 m, to be checked against the maker's drawing), sumps and
                a drain channel with a fall, and a column drop panel where the raft needs it.
             3. Loads (shown, not hidden). Active, at-rest (the default for a wall held by
                slabs) or passive earth pressure by the standard Rankine / Coulomb formulas;
                surcharge (vehicle load strip along the road side, building next door as a
                line load); water pressure below the water table. Each wall bay gets its own
                pressure from its own depth. Every number has a 'show working' link.
             4. Check. Wall as a one-way slab spanning between the basement slabs, or as a
                two-way plate where it is held by cross walls (simple, conservative checks,
                clearly labelled 'pre-design, to be confirmed by your structural engineer'):
                thickness, bending, shear, and crack width on the water side (IS 3370 rule
                for water-retaining cases, where used). Raft: uplift, bearing, punching.
                Overturning and sliding for the non-propped stage (before the slabs exist).
                Results are traffic lights per wall bay and per raft panel.
             5. Stages. The sequence of excavate, dewater, cast raft, cast wall, cast slab,
                backfill, stop dewatering. At each stage, the tool checks uplift and wall
                stability, and writes down the earliest date that dewatering may stop
                (feeds the 4D timeline and the Pour Planner).
             6. Detail. Wall and raft rebar by bay: vertical bars on the earth face, horizontal
                bars both faces, corner and T-junction bars, extra bars around the ramp opening
                and the pit, dowels to the raft, and lap lengths from Bar Optimizer's rule
                table. The result are real IfcReinforcingBar in the IFC, so BBS and QA run as
                they do for any other rebar in cad2bim.
             7. Waterproofing and drainage. Raft: PCC, membrane, protection screed; wall:
                membrane or crystalline coat, drainage board, backfill; the joints (kicker,
                construction, expansion) with water-stops. Added as layers and a schedule, so
                the BOQ prices them by area and by metre.
             8. Output. Drawing set (plan, sections, wall elevation, ramp detail), BBS, BOQ,
                the stage plan, and a design note PDF listing every input and every formula.
implementation:
             Built on what the repo already has; no new file format and nothing leaves the
             device. All of this is my plan after reading the roadmap and the design notes; I have
             not built or run anything yet.
             Phase 0 (about 1 week): rules and inputs, no 3D yet.
               - A new engine module `basement` in `packages/engine` with plain TypeScript
                 functions: earth pressure, surcharge, water pressure, wall bay checks, uplift.
                 Unit tests against hand calculations from two textbook examples (to be
                 checked by a structural engineer on the team before it is shown to anyone).
               - Rule tables as JSON with a source line on each row (soil defaults, ramp
                 slope by city, lift pit depth by lift maker). Editable in the app, saved in
                 the project.
             Phase 1 (2 to 3 weeks): the box in 3D.
               - From outline to IfcWall, IfcSlab, IfcFooting through the IFC writer the app
                 already uses (docs/design/ifc-writer-parameters.md); lift pits and sumps as
                 simple extrusions; ramp as a sloped slab with an opening in the slab above.
               - Dock panel with the inputs from step 1 and a live 3D update.
               - Pressure arrows as a three.js overlay, drawn per wall bay, colour by size.
             Phase 2 (2 to 3 weeks): checks, stages, fix button.
               - Traffic-light overlay per wall bay and raft panel; a 'Fix' action that
                 changes only the thickness or the bar spacing of that bay and re-checks.
               - Stage table (excavate, dewater, raft, wall, slab, backfill) with the uplift
                 and stability result at each stage; hand the dates to the 4D timeline.
               - Flood-test animation: a transparent water plane that rises in the 3D view.
             Phase 3 (2 to 3 weeks): rebar and drawings.
               - Wall and raft rebar by bay through the rebar design (docs/design/rebar.md),
                 reusing Bar Optimizer's lap and cutting rules and the BBS already in the app.
               - Sheet set through the 3D to 2D drawing path (see Drawing Autopilot).
             Phase 4 (2 weeks): CH template v2 and Revit.
               - Layer BASEMENT-LINE and its labels in the v2 reader, and 'Place basement' in
                 the Revit bridge from the same engine output.
               - etabs-mcp load cases EP, SUR and WT as wall pressures.
             How we know it works: (a) the unit tests above; (b) 3 finished basements from
             real jobs, rebuilt with Basement Box and compared with the real model on wall
             thickness, bar weight and BOQ (target within 5 to 8 percent on steel, to be
             measured; the first pass will probably be worse); (c) a time test with two
             modellers, Basement Box against by hand, on the same outline.
             Risks and how to handle them: (1) Structural responsibility: the output is a
             pre-design and a model start, never a stamped design; every sheet says so, and
             the engineer of record signs the design note. (2) Soil data: wrong inputs give
             wrong answers, so the soil table shows its source and a 'not from a bore log'
             warning in red. (3) Local rules: ramp slope, setbacks and fire rules differ by
             city, so they are tables with a city name, not hard-coded. (4) Scope: start with
             rectangular and L-shaped outlines and straight ramps; curved ramps later.
future update:
             - Shoring and excavation: soldier piles, secant or contiguous piles and
               king-post and strut layout as the temporary stage, linked with Dig Deep and Pile
               Pilot; the same box becomes a diaphragm or pile wall for the permanent stage
               in tall buildings.
             - Live monitoring: a CSV of the site's piezometer and inclinometer readings drawn
               on the wall as dots; alert when a reading crosses the design value; a
               monsoon mode that replays last year's rain on the dewatering plan.
             - Dewatering design: well-point or sump pumping count from the soil permeability
               and the inflow, pump electricity and diesel cost in the BOQ.
             - Stack parking: car lifts and mechanical stackers as an alternative to the
               ramp, with the pit depth and the cost compared in one table (links to Park Pilot).
             - Basement MEP: sump pumps, fire tank and the drain from the ramp as a start for
               the MEP models (links to Drain Line and Sleeve Punch).
             - Carbon and cost: a Carbon Lens swap for the wall mix (cement replacement) and
               a cost-time slider for 2 versus 3 basements, to show the client what the extra
               level really adds.
             - Client reel: the 'water rises outside, the box holds' clip as a 15-second
               video for the office's LinkedIn page, through Model Reel.
```
