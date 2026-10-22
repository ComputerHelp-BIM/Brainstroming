# 20261022 Idea: Kit Cutter — turn a cast-in-situ model into a precast kit: panels, joints, lifting inserts, mould-bed nesting, truck loads and an erection 4D, in one click

## Entry 1

```
date-time:   2026-10-22 (scheduled run, second idea of the day)
idea name:   Kit Cutter — "Your building is modelled as normal walls, slabs, beams and columns. Press Cut.
             Watch it split into numbered precast pieces, each one under the crane and truck limit, with
             joints, lifting inserts, connection details, a shop drawing, a mould-bed plan and a truck
             loading list — then watch it go up, piece by piece, in 4D."
             One Python engine (in C2B) with four front ends:
               (1) pyRevit button (AnonGee tab, Structure panel): pick walls / slabs / beams / stairs, pick a
                   precast system (solid wall panel, sandwich wall, hollow-core slab, solid / half slab
                   (filigree), precast beam, column, staircase, parapet / chajja), and get native Parts or
                   precast families with piece marks, weights, joints and inserts,
               (2) C2B: a DXF plan with a PRECAST layer (piece outlines, marks, joint lines) and an Excel piece
                   register, straight from the architectural / structural DXF, before any Revit model exists,
               (3) PlanWin AI Pro / etabs-mcp: joint positions and connection types pushed back to the
                   analysis model (pinned / moment / wet joint), so the design matches the kit,
               (4) cad2bim web viewer: an exploded "kit" view and an erection replay, with a QR per piece that
                   the yard and the site scan (re-using the Site QR Pass idea).
             Outputs: piece register (Excel: mark, type, size, volume, weight, concrete grade, rebar kg,
             inserts, lifting points, casting date, truck no., erection day), one shop drawing per piece type
             (DXF/PDF: elevation, section, openings, inserts, rebar, lifting points), joint and connection
             detail sheets, mould-bed / casting-yard nesting plan, truck loading plans with axle and height
             checks, crane pick list (hand-off to Crane Canvas), erection 4D sequence and video.
potential:   Very high and growing fast. Precast and "modern methods of construction" are being pushed hard in
             India (PMAY-Urban / Light House Projects, CPWD and state housing boards, data centres,
             warehouses, metro and railway station buildings, big developer townships), and every
             cast-in-situ firm that bids for this work has to "convert" its design to precast first.
             Time today (one G+14 residential tower, ~2,500-3,500 pieces):
               - precast panelization by hand (where to cut walls and slabs, piece sizes vs crane / truck /
                 mould): 1-2 weeks for an engineer + 2 modellers, redone at every architectural change,
               - shop drawings per piece type (150-400 types): 3-6 weeks of drafting,
               - piece register, weights, insert counts, BOQ: 3-5 days in Excel, often wrong,
               - truck loading and erection sequence: done on site from a printout, by experience.
             With Kit Cutter: first panelization in ~30 minutes including review, 80-90 % of shop drawings
             generated, register/BOQ always equal to the model. Saves 4-8 weeks per tower and lets a
             normal BIM team bid and deliver precast jobs without buying a precast-specific CAD suite.
             Quality saved: pieces heavier than the crane can lift at that radius, panels too tall for the
             truck under a bridge (>3.8-4.2 m loaded height), openings too near a panel edge, joints through
             a door head or a beam seat, lifting inserts not over the centre of gravity, duplicate marks,
             missing grout tubes / shear keys — all routine yard and site failures.
             Trending angle: "precast / modular / off-site construction", "DfMA" and "factory-built
             buildings" are among the most shared AEC topics on LinkedIn in 2025-26 (Light House Projects,
             Katerra-style lessons, data-centre speed builds), and time-lapse reels of precast towers going up
             a floor in 3-4 days are viral on Instagram. The reel here: a normal grey Revit tower; press Cut;
             thin red joint lines race across every wall and slab, the building explodes into hundreds of
             colour-coded pieces with marks and tonnage labels; one red panel ("6.4 t > 5.2 t at 32 m")
             splits itself in two and turns green; pieces fly onto mould beds, then onto trucks, then the
             tower re-assembles floor by floor in 4D with a day counter. 30 seconds.
             Market: Tekla Structures precast (excellent, paid, ~USD 4-6k per seat per year, a separate
             modelling world from Revit), Allplan Precast / Planbar (paid, Europe-centric), AGACAD Precast
             Concrete for Revit (paid add-in, per-seat licence), Revit's own "Precast" tools (Structural
             Precast extension: limited wall/slab split, European rule sets, no Indian truck / crane / yard
             logic, little control over shop drawings), Idat / Unitechnik / Nemetschek MES (factory-side,
             paid, enterprise). Nothing free that goes cast-in-situ DXF or Revit model -> rule-based
             panelization with Indian truck, crane and IS 15916 / IS 11447 conventions -> shop drawings ->
             yard nesting -> truck loading -> erection 4D, and that also feeds the joint types back into the
             ETABS / PlanWin analysis.
```

### Description

**The pain (daily life of a BIM modeller in an Indian architecture / structure / BIM-services office).**
- The client says "we are going precast for towers B and C". The design is already done as cast-in-situ walls
  and slabs in Revit / AutoCAD. Someone now has to decide where every wall and slab is cut, by hand, on a
  printed plan, with a calculator for weights.
- Every limit fights every other: the crane lifts 5 t at the far corner but 10 t near the mast, the truck bed is
  12 m x 2.5 m with a height limit, the mould beds are 12 m x 3.6 m, the architect does not want joints through
  windows, and the structural engineer wants joints at known positions for the analysis.
- One architectural change (a window moved 300 mm) can change 40 pieces, their marks, their drawings and the
  register. Nobody wants to touch it, so the model and the factory drift apart.
- Shop drawings for precast are repetitive (same elevation, section, openings, inserts, rebar, lifting points)
  but are drawn one by one in AutoCAD.
- On site, the erection order is decided on the morning of the lift. Trucks arrive with pieces in the wrong order
  and the crane waits.

**The idea, in seven parts.**
1. **System and rule packs.** YAML per precast system and per project: max piece size and weight, crane chart
   (or link to Crane Canvas), truck types (bed size, payload, max loaded height, A-frame or flat), mould-bed
   sizes, min distance from openings to edges, preferred joint positions (on grid, at wall junctions, beside
   openings), joint types (vertical wet joint, horizontal grouted sleeve, shear key, dowel, welded plate,
   corbel), insert catalogue (lifting anchors, sockets, grout sleeves, electrical boxes). Ship packs for solid
   wall + solid slab housing, hollow-core + precast frame, sandwich facade panels, and precast staircases.
2. **Auto-panelization (the core).** Each wall / slab face becomes a 2D region with openings and junctions.
   The engine proposes cut lines that respect all rules and minimises: number of pieces, number of distinct
   types (more repeats = fewer moulds and drawings), joint length, and over-limit pieces. It snaps to the grid
   and to repeated floor patterns so floors 2-14 reuse the same marks.
3. **Live checks with colours.** Every piece shows green / amber / red for weight vs crane at its install
   radius, size vs truck and mould, edge distance to openings, centre of gravity vs lifting points, slenderness
   for handling and demoulding. A red piece gets one-click fixes: "split at 2.4 m", "move joint beside
   window", "change to half slab (filigree) + topping".
4. **Joints and connections.** Joints are real objects: each has a type from the pack, a detail drawing, and
   quantities (grout litres, sleeves, sealant m, dowels). Joint types are exported to etabs-mcp / PlanWin as
   releases or links so the analysis model is the precast model.
5. **Pieces in Revit and IFC.** Revit Parts (or precast families) with `Piece_Mark`, `Piece_Type`, `Weight_kg`,
   `Volume_m3`, `Grade`, `Rebar_kg`, `Mould_Bed`, `Truck_No`, `Erection_Day`, `QR_Url`. IFC:
   IfcElementAssembly per piece with IfcWall / IfcSlab parts, IfcMechanicalFastener for inserts, Pset for
   precast data — so the cad2bim viewer, structura BOQ and other tools read the same pieces.
6. **Shop drawings and BOQ.** Per piece type: elevation with openings and inserts, sections, rebar (re-using
   the Detail Drop and BBS Weaver logic for mesh and bars), lifting points, weight, volume, casting table; one
   PDF/DXF per type, auto-revised when the piece changes. Piece register, insert schedule, joint schedule,
   concrete / rebar / grout BOQ.
7. **Yard, trucks and erection 4D.** Mould-bed nesting (pieces packed onto beds per casting day, with curing
   time), truck loading plans (pieces per truck in erection order, axle and height check, A-frame side), and an
   erection sequence per floor (stability rules: walls before slabs, braced panels, max open joints). The 4D
   replay plays in the cad2bim viewer and exports a Model Reel video.

**Why it fits this toolkit (extends what already exists).**
- **C2B** already reads walls, columns, openings and grids from DXF; Kit Cutter adds a panelization stage so a
  precast tender can be priced from a DXF in hours, before Revit.
- **pyRevit-addons (AnonGee)** gets a showcase button using the existing WPF UI templates and `bridge/`;
  Revit Parts give a clean, reversible split of the original walls and slabs (the design model is not lost).
- **etabs-mcp / PlanWin AI Pro** receive joint types and piece boundaries so the analysis matches the kit, and
  return member forces that size the connections.
- **cad2bim web** viewer shows the exploded kit, the QR per piece and the erection replay to the client and
  the factory without a Revit licence.
- **structura** BOQ and costing read precast pieces, inserts and grout from the IFC.
- Re-uses other ideas instead of duplicating them: Crane Canvas (crane chart and pick plan), Site QR Pass
  (piece QR tags), Detail Drop / BBS Weaver (rebar), Change Radar (which pieces changed after a revision),
  Model Reel (the video), Bill Painter (pay by pieces erected).

**Wow moments.**
- Press Cut: joint lines sweep across the whole tower and the counter shows "2,964 pieces, 186 types, max
  6.1 t" in under a minute.
- Move a window in the architect's model: only the 12 affected pieces flash, re-cut, re-mark and re-draw;
  Change Radar lists exactly which shop drawings were revised.
- The "type squeezer" slider: drag towards "more repeats" and watch types drop from 186 to 121 as the joints
  shift a few hundred millimetres — fewer moulds, lower cost, shown live in rupees.
- Scan the QR on a real panel in the yard: the viewer opens with that piece glowing in the tower, its truck,
  its crane pick and its erection day.

### Implementation

**Phase 1 — Panelization engine + DXF + register (3-4 weeks).**
- Python package `c2b.precast`: `load_pack(yaml)`, `faces(model) -> [Face]` (wall / slab regions with openings,
  junctions and grid), `panelize(face, pack) -> [Piece]`, `check(pieces, pack) -> [Finding]`,
  `register(pieces) -> Table`.
- Geometry with Shapely; candidate cuts from grid lines, junctions and opening jambs (+/- edge distance);
  choose cuts by dynamic programming along each face (1D for walls, strip-wise for slabs), then a global
  pass that unifies similar pieces into types (hash of size + openings + inserts, with tolerance).
- Weight = volume x density (+ rebar estimate per type), centre of gravity from the polygon minus openings;
  lifting points from the insert catalogue placed symmetric about the CoG.
- CLI: `c2b precast tower_b.dxf --pack profiles/precast_solid_wall_housing.yaml --crane crane.yaml --out
  tower_b_precast.dxf --register tower_b_pieces.xlsx`. Tests with hand-checked walls (with door, window, two
  junctions) and slabs (with shaft opening) and fixed piece counts.

**Phase 2 — Revit + IFC + shop drawings (3-4 weeks).**
- pyRevit button: WPF form (selection, system pack, crane, truck, preview); split host elements with
  `PartUtils.CreateParts` + `PartMaker` divisions (keeps the design model intact), write piece parameters,
  colour by check status with view filters; or place precast families when the office prefers families.
- Joints as detail / generic model families with type parameters; inserts as face-based families.
- Shop drawings: per type an assembly (`AssemblyInstance.Create`) and `AssemblyViewUtils` elevation / section
  / schedule views on a sheet template, auto-dimensioned (re-using Drawing Autopilot logic); DXF/PDF export.
- IFC via IfcOpenShell: IfcElementAssembly + parts + IfcMechanicalFastener + custom `Pset_PrecastPiece`.

**Phase 3 — Yard, trucks, 4D, analysis link (3 weeks).**
- Mould-bed nesting: 2D bin packing (rectangles + clearance) per casting day with curing and bed reuse; truck
  loading: 3D-lite bin packing with axle load and loaded-height check, ordered by erection sequence.
- Erection sequence: per floor graph of "must stand before" rules (walls -> braces -> slabs -> joints grouted),
  durations from the pack, crane cycles from Crane Canvas; output a 4D schedule (CSV / MS Project XML / IFC
  IfcTask) and a cad2bim viewer replay.
- etabs-mcp: push joint types as releases / links, pull forces for connection checks; flag a joint whose force
  exceeds the connection capacity in the pack.

**Tech notes / risks.**
- Precast design rules differ by manufacturer; everything is a pack with a version and source note, and the
  output is a design aid for the precast engineer to approve, not a stamped design.
- Very irregular architecture gives many unique types; the type squeezer and "report non-standard pieces"
  make that visible early instead of hiding it.
- Revit Parts can be slow on very large models; process per level and cache faces.
- Rebar for precast (mesh + edge bars + lifting reinforcement) starts as rule-based typical details in v1;
  full design-driven rebar comes with Detail Drop integration.

### Future update
- Sandwich and architectural facade panels: insulation layer, ties, textured finishes, colour mock-ups for the
  client.
- Hybrid mode: "precast walls + cast-in-situ slab" or "aluminium-formwork core + precast facade", with a cost
  and time comparison per option for tenders.
- Factory link: export to MES / CSV for the casting yard (mould-bed booking, concrete per day, rebar cages per
  day) and a simple factory dashboard in the cad2bim web app.
- Site tracking: scan piece QR on arrival, on lift and after grouting; the 4D model turns grey -> yellow -> green
  and feeds Bill Painter RA bills.
- Modular / volumetric (pod bathrooms, room modules) and PEB steel kits with the same rule-pack engine.
- AI mode: "cut tower B for a 6 t crane and 12 m trucks, keep joints out of the living room walls" — the AI box
  sets the pack and constraints and runs Cut.
- Embodied-carbon and cost per piece type, so the team can show "precast vs cast-in-situ" savings on LinkedIn
  with real numbers.
