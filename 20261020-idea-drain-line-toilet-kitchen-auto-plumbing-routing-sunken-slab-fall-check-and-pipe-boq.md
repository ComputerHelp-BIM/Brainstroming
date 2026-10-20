# 20261020 Idea: Drain Line — click a toilet, get sloped drainage + water supply routed to the shaft, checked against the sunken slab, with isometric and pipe BOQ

## Entry 1

```
date-time:   2026-10-20 (scheduled run, fourth entry of the day)
idea name:   Drain Line — "Pick the wet rooms. Press Route. Watch the pipes fall to the shaft."
             A pyRevit button (AnonGee tab) + a web/IFC mode in cad2bim that takes placed plumbing fixtures
             (WC, wash basin, floor trap, shower, kitchen sink, washing machine point) and the nearest shaft/duct,
             and automatically routes:
               (1) soil and waste lines with real gravity fall (1:40 / 1:60 / 1:100 by diameter),
               (2) CPVC / PPR hot & cold water supply from the shaft riser,
               (3) vent connections, cleanouts and the vertical stack down every typical floor,
             then checks every drain run against the SUNKEN SLAB depth / false-ceiling void it must fit in,
             paints any run that does not fit red ("fall needs 285 mm, sunken depth is 250 mm"),
             and outputs a plumbing isometric sheet, sleeve list for structure, and a pipe + fitting BOQ.
potential:   Very high for Indian residential / hotel / hospital projects, where 30-70 % of rooms are wet rooms
             and the plumbing model is the most skipped, most clashing, most re-drawn part of a BIM job.
             Today a modeller draws each pipe by hand in Revit (routing preferences, slope on, connect, fail,
             delete, redraw), typically 1.5-3 h per toilet layout, then copies it to typical floors and fixes
             the floors that are not typical. The fall-vs-sunken-depth problem is usually found on site by the
             plumber when the slab is already cast, the result is chiselling, raised floors or extra steps.
             Time saved: one toilet 2-3 h -> ~3-5 min (route + review); a 12-storey tower with 4 toilet types
             x 4 flats per floor goes from ~4-5 working days of plumbing modelling to under one day including
             checking. The sunken-slab check alone prevents the single most common wet-area site rework.
             Trending angle: "generative MEP routing" and "auto-route pipes in Revit" clips are among the most
             viewed MEP-BIM posts on LinkedIn/Instagram in 2025-26; the 15-second reel of a bare toilet plan
             -> pipes "growing" and dropping to the shaft with a fall animation -> red flag on one run -> fixed
             is exactly the post format that gets shared, and the isometric sheet appearing at the end is the
             "wow" for site engineers who still draw isometrics by hand.
             Market: MagiCAD, Stratus/Autodesk Fabrication, Plumbing Design add-ins (e.g. "Revit MEP auto-route"
             paid tools), Revit's built-in "Generate Layout" (unreliable, no slope reasoning, no sunken check),
             Dynamo graphs (one-off, need a scripter). Nothing free that understands Indian practice (sunken
             toilets, Indian/EWC WC, Nahani trap, shaft-based stacks, NBC 2016 Part 9 / IS 1742 fixture units)
             and checks the drain fall against the structure.
```

### Description

**The pain (daily life of a BIM modeller on a residential / hospitality job).**
- Architects hand over a plan with fixtures drawn; the MEP consultant gives a schematic riser and maybe a
  hand-drawn isometric photo on WhatsApp. The BIM modeller has to turn that into real pipes.
- Revit pipe modelling is click-heavy: set routing preferences, set slope, draw, the connection fails on the
  P-trap, delete, redraw. Copying to typical floors breaks connections at every floor where the layout differs.
- The structural slab has a sunken portion (often 200-300 mm) for the toilet. Nobody checks that the longest
  WC-to-shaft run at 1:40 actually fits inside it. Site finds out after casting.
- Sleeves through beams and walls for these pipes are not communicated to structure in time (the Sleeve Punch
  idea of 2026-10-16 needs the pipes to exist first; Drain Line is what creates them).
- Plumbing BOQ (pipe length per diameter, bends, tees, traps, cleanouts) is estimated from thumb-rules because
  the model is incomplete.

**The idea, in six parts.**

1. **Read the wet rooms.** Input is any of: Revit fixtures already placed; a cad2bim-converted DXF where fixture
   blocks (WC, WB, FT, SINK) are recognised by block name / layer; or an IFC with `IfcSanitaryTerminal`
   (cad2bim already maps it to `PlumbingFixture`). The tool finds each room, its fixtures, the nearest shaft /
   duct (by room name "SHAFT"/"DUCT" or a picked element), and the slab sunk zone (sunken floor in Revit, or a
   region on a "SUNKEN" layer in DXF).

2. **Route with rules, not guesses.** A small grid/graph router (A* on a 50 mm grid inside the sunk zone /
   ceiling void, along walls, avoiding beams and columns) connects:
   - WC -> soil line (100 mm), basins / sinks / floor traps -> waste line (50-75 mm) via Nahani / floor trap,
   - branches join with 45° Y-tees in flow direction, never against it,
   - each branch gets the fall from a rule pack by diameter, and its invert levels are computed back from the
     stack connection point,
   - vents tie into the vent stack; cleanouts at every bend > 45° and at the head of each branch,
   - supply: CPVC/PPR from the shaft riser with a stop valve per toilet, hot line to geyser point,
     concealed in wall at standard heights (rule pack: WB 600, shower mixer 1000, WC flush 1200 mm etc.).
   Everything is created as real Revit pipes and fittings with the project's pipe types and systems, so it
   tags, schedules and exports to IFC like hand-made work.

3. **Sunken-slab & void check (the "wow" safety net).** For every drain run the tool computes the depth needed
   at its deepest point = invert drop over length + pipe OD + fitting allowance + floor finish/bedding, and
   compares it with the available sunk depth / ceiling void. Results are painted in a 3D view: green fits,
   amber < 25 mm spare, red does not fit, with a label "needs 285, has 250". It suggests the cheapest fix:
   reroute closer to the shaft, move the floor trap, change to 1:60 where allowed, or raise sunk depth by X mm
   (sent as a BCF issue to structure, reusing the IDS-QA idea's BCF writer).

4. **Typical-floor stack.** Route once on a typical floor, then "Propagate" copies the layout to every floor
   that has the same toilet type (matched by room/fixture fingerprint), joins the vertical stack through each
   floor, and only re-routes floors that differ, listing them for review.

5. **Outputs the site actually uses.**
   - Plumbing isometric sheet per toilet type (auto 30° iso view, dims, invert levels, diameters, fall arrows),
     printable A3 PDF and DXF for the plumber.
   - Sleeve list (beam / wall penetrations with size and centre level) to feed the Sleeve Punch coordinator.
   - Pipe + fitting BOQ by diameter and material (m of 100 mm uPVC SWR, nos. of 100 mm 87.5° bends, Y-tees,
     traps, cleanouts, CPVC 20/25 mm, valves) to Excel in the same format as the existing BOQ exports.
   - Fixture-unit and stack sizing summary per NBC Part 9 / IS 1742, so the consultant can tick it.

6. **Review before commit.** Like the Toolsmith idea, routing is first a preview (temporary coloured geometry
   + a table of runs, falls and fit status). "Commit" creates the Revit elements in one undo group.

**Why it is a natural extension of this toolkit.**
- `cad2bim` already parses DXF and maps `IfcSanitaryTerminal`, `IfcPipeSegment`, `IfcPipeFitting`, valves
  (`packages/engine/src/model/categories.ts`), so the web mode can show fixtures and routed pipes in its IFC
  viewer and export IFC4 pipes without Revit at all.
- `pyRevit-addons` (AnonGee tab) already has the pattern of one-click bulk tools (AutoLevel, Bulk Tool, StrucBIM)
  and an Excel `bridge`, ideal for the rule pack (falls, heights, diameters) being an editable Excel sheet.
- The structural side (Structura / ETABS Bridge) knows beam depths and slab levels, which is exactly what the
  fit check and the sleeve list need.

### Implementation

**Phase 1 — Revit MVP, single toilet (2-3 weeks).**
- `DrainLine.pushbutton` in `AnonGee.tab/Essential.panel` (or a new `MEP.panel`).
- Inputs: pick room(s) or select fixtures; pick shaft/stack point; read sunk depth from a sunken floor's
  thickness offset or ask once.
- Router in pure Python (IronPython/CPython via pyRevit): build a 2D occupancy grid of the room from walls,
  columns, beams (bounding boxes) at 50 mm; A* from each fixture connector to the stack with turn penalty and
  "prefer along walls" cost; merge branches into a tree (Steiner-like greedy join toward the stack).
- Levels: walk the tree from the stack outward, assign inverts by fall per diameter; compute required depth.
- Revit API: `Plumbing.Pipe.Create(doc, systemTypeId, pipeTypeId, levelId, p1, p2)`, then
  `doc.Create.NewElbowFitting / NewTeeFitting` (or `NewTakeoffFitting`) on connectors; slope set by the
  endpoints themselves. Wrap in `TransactionGroup` for one undo.
- Rule pack: `drainline_rules.xlsx` (diameter by fixture, fall by diameter, wall-concealed heights, allowances).
- Fit-check view: a 3D view with `OverrideGraphicSettings` green/amber/red and text notes.

**Phase 2 — Multi-floor, outputs (2-3 weeks).**
- Room/fixture fingerprint (fixture types + relative positions rounded to 50 mm) to group typical toilets;
  propagate and stack.
- Isometric sheet: duplicate 3D view, orient to iso, crop to the toilet, auto dimension inverts with spot
  elevations, place on an A3 titleblock; export PDF/DXF.
- BOQ export: reuse the existing Excel export style; sleeve list CSV that the Sleeve Punch idea consumes.
- BCF for red runs (reuse IDS-QA BCF writer).

**Phase 3 — Web / no-Revit mode in cad2bim (3-4 weeks).**
- Same router ported to TypeScript in `packages/engine` (shared rule pack as JSON).
- Input: DXF plan with fixture blocks + SUNKEN / SHAFT layers -> routed pipes in the browser 3D viewer with the
  fall animation -> export IFC4 (`IfcPipeSegment` / `IfcPipeFitting`, `IfcDistributionSystem`) and the iso PDF.
- This gives small firms and site engineers the tool without a Revit seat.

**Tech notes / risks.**
- Fitting creation in Revit fails on odd angles: router only allows 0/45/90° turns, and the tree joins at
  45° Y only, which keeps fittings standard and creation reliable.
- Keep the router deterministic (same input -> same layout) so review and re-runs are predictable.
- It is a design assist, not a design sign-off: the summary sheet says "for coordination; MEP consultant to
  approve" and lists every rule value used.

### Future update

- **Storm water & terrace drainage:** rainwater outlets, terrace slope check and down-take pipes routed to the
  external chamber, with roof area per outlet per NBC.
- **External network:** inspection chambers, gully traps and sewer line to the municipal connection with
  invert levels from the site levels (links to the Dig Deep earthwork idea for trench quantities).
- **HVAC drain & fire sprinkler mode:** the same router + fit check for AC condensate lines and sprinkler
  branches in the ceiling void (ceiling-height check, "false ceiling must drop 75 mm here").
- **Photo-to-layout:** read the consultant's hand-drawn isometric photo (WhatsApp) with a vision model and use
  it as routing hints.
- **4D / Model Reel hook:** plumbing first-fix sequence per floor feeds the 4D video and the RA-bill painter
  (pipe metres installed this month).
- **Learning from fixes:** when modellers adjust a routed layout, store the edit as a preference
  (e.g. "this client wants floor traps near the door") and apply it next time.
