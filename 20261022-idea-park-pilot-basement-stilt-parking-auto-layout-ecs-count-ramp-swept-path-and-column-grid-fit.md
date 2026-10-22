# 20261022 Idea: Park Pilot — draw the basement outline, get the best parking layout in 3D with the ECS count, ramps, swept-path checks and a column grid that fits the cars

## Entry 1

```
date-time:   2026-10-22 (scheduled run)
idea name:   Park Pilot — "Give it the basement / stilt boundary, the column grid (or let it propose one),
             the core and ramp position and the city's parking rule. In about a minute it gives you several
             parking layouts, ranked by car count, with numbered bays, drive aisles, one-way arrows, ramps
             checked for slope and turning, the ECS count against the bye-law, and the whole thing in 3D in
             Revit and IFC."
             One Python engine (in C2B) with four front ends:
               (1) C2B: the basement outline, columns, cores and shear walls already come out of the DXF.
                   Park Pilot adds a PARKING layer and draws bays, aisles, arrows and numbers into the DXF,
               (2) pyRevit button (AnonGee tab, Essential panel): pick a level, pick the options, and get
                   native Parking components, aisle filled regions / floor finishes, ramp floors, bay tags and
                   a parking schedule,
               (3) PlanWin AI Pro: "Grid fit" mode. Show where a column falls inside a bay or an aisle and
                   suggest the column shift or bay spacing (for example 7.5 m / 8.1 m / 8.4 m) that gives the
                   most cars, before the structural grid is frozen,
               (4) cad2bim web viewer: a client link where the layout options can be flipped through in 3D,
                   with a car driving the path from the gate to any bay.
             Outputs: layout options ranked by car count and ECS, numbered bay plan (DXF/PDF), parking
             statement (Excel: car / two-wheeler / EV / accessible / visitor counts against required), ramp
             section sheet with slope and transition lengths, swept-path report, EV-charger and fire-tender
             notes, bay quantities (paint marking m, wheel stoppers, signage) for the BOQ.
potential:   Very high, every week. Every Indian residential or commercial building above G+3 has a stilt or
             1-3 basements, and parking count decides the saleable area: one lost car slot in Mumbai/Pune/
             Bengaluru is worth several lakh rupees. Parking layout is redone at every architectural or
             structural change.
             Time today (one basement of 3,000-6,000 m²):
               - first parking layout by hand in AutoCAD (bays, aisles, turning, ramps): 1-2 days,
               - each revision (column shifted, core moved, ramp moved): 4-8 hours,
               - counting ECS, two-wheeler, visitor and accessible bays for the area statement: 2-4 hours,
                 and often wrong by a few bays,
               - modelling the same layout in Revit (bays, tags, ramps): 1 day per level.
             With Park Pilot: ~15-30 minutes per level including review; revisions are a re-run. Over a
             project with 2 basements and 5-8 revisions this saves 2-3 weeks of architect + modeller time,
             and the "best" layout usually finds 3-8 % more cars than the hand layout.
             Quality saved: columns inside the door-opening zone of a bay, ramps steeper than 1:8 or without
             transitions, a 90° bay with a 5.0 m aisle where 6.0 m is needed, dead-end aisles with no turn,
             headroom under beams at the ramp, too few accessible/EV bays — all are routine fire-NOC and
             authority objections. Park Pilot checks each one.
             Trending angle: generative site and parking layouts (TestFit, Autodesk Forma, Archistar, Parkly)
             are among the most shared AEC posts on LinkedIn in 2025-26, and "AI found 12 more parking slots"
             reels go round on Instagram. The reel here: a grey empty basement; cars drop into bays row by row,
             the counter ticks 84 -> 97 -> 103 as three layouts flip; a red column flashes inside a bay,
             PlanWin nudges the grid 300 mm, the column turns green and two more cars appear; then a car
             drives down the ramp to bay B2-047 with its swept-path ribbon. 30 seconds.
             Market: TestFit (strong, US/global parking and site solver, paid around USD 5-10k per seat per
             year, no Indian bye-laws, no Revit native rebuild of PlanWin grid); Autodesk Forma (paid, early
             stage, site massing only); Archistar / Parkly AI (paid, web, early-stage design only); AutoTURN /
             Autodesk Vehicle Tracking (paid swept path only, no layout); Dynamo/Grasshopper parking scripts
             (one-off, fragile, no ECS, no ramp check). Nothing free that goes basement DXF -> best layout with
             Indian ECS rules -> column grid fit -> ramp and swept-path check -> Revit + IFC + parking statement
             in one step.
```

### Description

**The pain (daily life of a BIM modeller in an Indian architecture / structure / BIM-services office).**
- The architect hands over a basement outline with a column grid, two cores and "ramp here". The modeller
  places bays by hand: 2.5 x 5.0 m cars, 6.0 m aisles for 90°, then finds a column in the middle of a bay and
  starts again. Every column shift by structure means re-placing whole rows.
- The bye-law count is a spreadsheet nobody trusts: ECS per tenement by carpet area, visitor %, two-wheeler
  ratio, accessible bays, EV-ready % (now mandatory in many cities), mechanical/stack parking credits.
- Ramps are drawn in plan with an arrow. Slope (1:8 / 1:10), transition slopes at top and bottom, width for
  one-way / two-way, inner and outer turning radius on curved ramps and headroom under the beam at the ramp
  top are checked late, often by the fire officer.
- Structure and parking fight each other: the structural engineer picks a 7.0 m grid that fits the flats
  above; that grid wastes a car per bay row below. Nobody runs "what grid gives the most cars and still works
  for the tower" because it takes days.
- In Revit, parking is a pile of loose components and filled regions; the count in the schedule and the count
  in the area statement do not match.

**The idea, in six parts.**
1. **Rule packs.** YAML per city / code: bay sizes (car, compact, SUV, two-wheeler, accessible), aisle widths by
   angle (90°, 60°, 45°, parallel), one-way / two-way, ramp slopes and transitions, headroom, ECS formula,
   visitor %, EV %, accessible %. Ship generic NBC 2016 + examples (Mumbai DCPR-style, Pune UDCPR-style,
   Bengaluru-style); a new city is a new file, not new code.
2. **Layout solver.** Input: boundary polygon, obstacles (columns with size, shear walls, cores, shafts, ramps,
   DG/STP rooms), entry/exit points, rule pack. The solver tries aisle directions and offsets, bay angles and
   mixes (big/compact/two-wheeler infill in leftover strips), builds the aisle network as a graph and keeps
   only layouts where every bay is reachable from the gate without dead ends. Score = cars (ECS) first, then
   fewer reversing moves, shorter drive length, clean column zones. Returns the top 3-5 options.
3. **Column grid fit (the unique part).** Every column is scored against the bays: inside the door zone = bad,
   at the bay corner / between bay pairs = good. Park Pilot proposes small shifts (within limits set by the
   engineer, e.g. ±300 mm) or a better bay pitch and shows "+N cars" for each. PlanWin AI Pro reads the
   suggestion, re-runs load take-down and design, and flags if a beam span or column size breaks. Parking and
   structure agree before drawings go out.
4. **Ramps and swept path.** Straight and curved ramps generated from the start/end levels and the slope rule,
   with transitions; checks width, inner/outer radius and headroom against beams from the model. A design car
   (and a fire tender for the ground-level drive) is driven along the aisle graph; its swept-path ribbon must
   stay clear of columns and walls. Every bay gets a "park-in" check (one or two moves).
5. **3D and drawings.** Revit: Parking family instances with bay number / type / EV / accessible parameters,
   aisle floor finish, arrows (detail items), ramp floors with sloped arrows, slab openings; parking schedule
   that equals the statement. IFC: IfcSpace (PredefinedType PARKING) per bay with property sets, ramps as
   IfcRamp / IfcRampFlight, so the cad2bim viewer and structura's BOQ see it. DXF: numbered bay plan with
   dimensions, a ramp section and the statement table.
6. **Parking statement + BOQ.** Excel/PDF: required vs provided per category per level, option comparison,
   plus quantities for paint marking (running m), wheel stoppers, convex mirrors, signage, EV chargers and
   cable trays — fed into the Area Statement Pro and Bill Painter ideas.

**Why it fits this toolkit (extends what already exists).**
- **C2B** already parses the basement DXF (columns, walls, cores, shafts) and exports DXF/Revit JSON; Park
  Pilot is a new stage on the same geometry, not a new importer.
- **PlanWin AI Pro** owns the column grid and has an AI prompt box; "fit grid for parking" becomes one prompt
  ("shift grid C by up to 300 mm for more cars") with immediate design re-check.
- **pyRevit-addons (AnonGee)** gets a high-visibility button with the existing WPF UI templates and `bridge/`.
- **cad2bim web** viewer already shows IFC; option flipping and the car drive animation are a viewer plugin.
- **structura** BOQ picks up bay marking, stoppers and ramp concrete from the IFC.
- Shares the rule-pack approach with Area Statement Pro (ECS count goes straight into the FSI sheet), Exit Map
  (fire tender path and basement egress) and Model Reel (the drive-through video).

**Wow moments.**
- Three layout options appear side by side with a live car counter; click one and it is in Revit in 3D.
- "Grid fit": a red column inside a bay, one click, the grid moves 300 mm in PlanWin, the design check stays
  green and the counter shows +6 cars.
- A car drives from the gate down the ramp to any clicked bay, leaving a coloured swept-path ribbon; a tight
  corner turns red and offers "widen aisle to 6.3 m" or "cut column corner".

### Implementation

**Phase 1 — Solver + rules + DXF (3 weeks).**
- Python package `c2b.parking`: `load_rules(yaml)`, `solve(site: Site, rules) -> [Layout]`, `check(layout,
  rules) -> [Finding]`, `statement(layout, rules, program) -> Table`. Geometry with Shapely (polygons, buffers,
  clearance), aisle graph with NetworkX (reachability, dead ends, drive length).
- Search: sweep aisle direction (0/90° + boundary-aligned angles), offset step 100 mm, bay angle set; fill
  strips with a greedy row packer, then local improvement (swap bay types, flip one-way direction). Optional
  OR-Tools CP-SAT pass for the last 2-5 % on large basements.
- DXF writer via ezdxf on C2B's export: layers PARKING-BAY, PARKING-AISLE, PARKING-ARROW, PARKING-TEXT.
- CLI: `c2b parking basement.dxf --rules profiles/parking_pune.yaml --program flats.csv --out b1_parking.dxf`.
- Unit tests with hand-checked basements (rectangular, L-shaped, with ramp and two cores) and fixed counts.

**Phase 2 — Ramps, swept path, grid fit (3 weeks).**
- Ramp generator (straight / curved / helical) from levels + slope; transitions; headroom vs beams read from
  C2B/PlanWin model.
- Swept path: bicycle model of the design vehicle (wheelbase, track, overhangs, min radius from the rule pack)
  along aisle-graph polylines; envelope polygon tested against obstacles.
- Grid fit: per-column penalty field; search column offsets within engineer limits; output a JSON "grid
  proposal" PlanWin AI Pro imports, re-runs take-down and design, and returns pass/fail per member.

**Phase 3 — Revit + IFC + web (3 weeks).**
- pyRevit button: WPF form (level, rule pack, options, entry points) using AnonGee UI templates; places Parking
  family instances by type with parameters `Bay_No`, `Bay_Type`, `EV`, `Accessible`, `Level`; aisle floors;
  arrows as detail items; ramp floors via `Floor.Create` with slope arrow; parking schedule + statement sheet.
- IFC writer (IfcOpenShell): IfcSpace PARKING per bay with `Pset_SpaceParkingCommon` + custom pset, IfcRamp.
- cad2bim viewer plugin: option switcher, car-drive animation along the path, swept-path ribbon, counter; a
  Model Reel preset records the 30-second clip.

**Tech notes / risks.**
- Rule interpretation varies by authority and changes by circular; packs carry a version and source note, and
  the statement says which pack was used. Output is a design aid for the architect to approve.
- Solver time on very irregular 10,000 m² floors: cap search, show the best-so-far progressively.
- Revit Parking families vary by office; map by type name in a JSON config, ship a default family set.
- Mechanical / stack / puzzle parking counted as a bay type with its own footprint and pit depth, not modelled
  mechanically in v1.

### Future update
- Mechanical parking mode (stack, puzzle, tower) with vendor footprints and pit/headroom checks.
- Basement MEP fit: keep sprinkler mains, ventilation ducts and jet fans above aisles, not over bay ends,
  linked to Sleeve Punch for beam openings.
- Basement ventilation and jet-fan placement check (CFD-lite airflow arrows), CO sensor layout.
- EV charging plan: charger positions, load per DB and cable tray route, handed to Live Wire.
- Surface parking and site layouts for malls, hospitals, IT parks; truck and bus swept paths for logistics.
- 4D: basement excavation and ramp sequence with Dig Deep and Pour Planner; Site QR Pass bay tags for the
  society handover (bay number -> flat number allotment sheet).
- AI mode: "give me 110 cars with the ramp on the east side" — the AI box sets constraints and runs the solver.
