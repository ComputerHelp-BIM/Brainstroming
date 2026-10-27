# 20261027 Idea: Tank Smith: type the flat count (or click the building), and cad2bim sizes the underground sump, the overhead tank, the fire reserve and the septic tank / soak pit to NBC and IS 1172 / IS 2470, designs them to IS 3370 (crack width, not just strength), places them in the 3D model with walls, slabs, haunches, manholes, ladders, pipes and the water-stop, and writes out the rebar, BBS, waterproofing and excavation BOQ and a drawing sheet. The client sees the tank fill with water in 3D, level by level, against the days of storage

## Entry 1

```
date-time:   2026-10-27 (scheduled run, UTC)
idea name:   Tank Smith. "A G+7 residential job, 28 flats, 1 lift. The modeller opens the
             cad2bim model and clicks Tanks. Tank Smith reads the building: 28 flats x 5 people
             = 140 people, 135 l/person/day (IS 1172 / NBC Part 9). It proposes: underground
             sump 2 days = 37.8 kl + fire reserve 50 kl (NBC Part 4, residential above 15 m:
             to be checked against the local fire NOC rules) = 88 kl in two compartments;
             overhead tank 1 day domestic = 18.9 kl + 5 kl terrace fire, on the lift-core
             shear walls; septic tank for 140 users with a 2-year desludging interval
             (IS 2470-1) plus a soak pit (IS 2470-2), or 'STP needed' if the local rule says so.
             The modeller drags the sump into the corner of the plot. It snaps outside the
             footing zone and warns: 'Sump 1.2 m from footing F7: the excavation crosses the
             footing's 45-degree load zone. Move it 0.8 m or design the shared wall.' One more
             click: walls 250 thick, base 300, top 175, haunches 150 x 150, M30, crack width
             0.2 mm (IS 3370-2, exposure: the soil side 'severe'), bars 12 @ 150 both faces in
             the walls. The 3D view shows the tank cut open, the rebar in it, a PVC water-stop
             at the kicker and the ladder. Then 'Fill': water rises in blue, a counter ticks
             'Day 1 ... Day 2 ... fire reserve kept', and the pump-on and pump-off levels light
             up. Out come the BBS, a drawing sheet (plan, two sections, rebar), the BOQ
             (excavation, PCC, concrete, steel, shuttering, waterproofing, pipes, ladder) and
             a design note PDF for the checking engineer. Twenty minutes for what normally
             takes two days across three people."
             One engine, five front ends:
               (1) cad2bim web app: a "Tanks" dock panel (size, place, design, fill, sheet);
                   tanks become IfcTank (shape) plus IfcWall / IfcSlab / IfcReinforcingBar
                   in the IFC, so QA, BOQ and BBS already work on them,
               (2) cad2bim CH template v2: the "water tanks, UGT / OHT" row in the template
                   2 scope (today tier C, walls and slabs only, by labels) becomes a real tank
                   rule. A layer TANK with a closed outline and a label like
                   "UGT 88KL 2C" is enough; Tank Smith fills in the rest,
               (3) Revit (bridge add-in + pyRevit AnonGee button): "Place tank" builds walls,
                   floors, openings and Structural Rebar from the same design, on the right
                   levels, with a schedule,
               (4) etabs-mcp: the OHT weight (water full + empty + sloshing mass, see below)
                   goes to ETABS as loads on the core walls; the sump wall pressures can be
                   checked as plates if the engineer wants a full model,
               (5) Structura: the tank shows in its 3D, BOQ and code-check views like any
                   other member group.
             Outputs: IFC (tank + members + bars), BBS Excel (IS 2502 shapes), drawing sheet
             (DXF + PDF), BOQ Excel, design note PDF (every formula and clause, every
             assumption in a box), Revit elements, ETABS load cases, a 20 s "fill" video for
             the client.
potential:   High, and very regular: almost every Indian building job, residential or
             commercial, has a UGT, most have an OHT, and many outside sewer areas have a
             septic tank or a small STP.
               - Today a tank is: the architect marks a box, the structural engineer runs an
                 Excel sheet (often an old one, strength design only, no crack width), the
                 draftsman draws plan, sections and rebar by hand in AutoCAD, the BIM modeller
                 builds walls and slabs again in Revit, and the QS measures it again. About
                 1.5-3 working days across 3-4 people per job (estimated from how offices
                 work, not measured).
               - Mistakes we see often (from experience, not counted): no crack-width check,
                 so the tank leaks and needs a costly repair; sump dug too close to a footing;
                 the OHT's water weight left out of the ETABS model; the fire reserve not kept
                 separate; no water-stop in the BOQ; septic tank sized by "rule of thumb".
             With Tank Smith (target, to be measured in the pilot): about 20-40 minutes per
             job for the first draft of all outputs, and one source of truth, so a change to
             the size updates the model, the BBS, the sheet and the BOQ together.
             Over a year: an office doing 20-40 jobs saves roughly 30-100 person-days
             (estimate).
             Trend: water storage, "leak-proof terraces and tanks", rainwater and water reuse,
             and "AI that designs and details for you" are steady LinkedIn and Instagram topics
             for Indian civil and BIM pages; short reels of "a building filling with water" or
             "rebar appearing in a cut-open box" do very well with engineers and with clients.
             Wow effect: you type one number (the flat count); the tank appears in the right
             place, cut open with its rebar, and fills with water day by day; the BBS, the
             drawing and the BOQ are already done.
             Market (to our knowledge, checked against what we know, not a full survey):
               - STAAD.Pro / STAAD Foundation, Midas, SAP2000: can analyse tanks as plates,
                 all paid, no sizing from occupancy, no detailing to BIM, no BBS.
               - Bentley RCDC: detailing for beams, columns, footings; not tanks, paid.
               - Revit with paid add-ins (e.g. rebar add-ins): you still model and size by
                 hand; no IS 3370 design.
               - Free Excel sheets for IS 3370: common, but strength-only or with old
                 IS 3370 (1965) tables, no model, no drawing.
               - Septic tank calculators online: size only, nothing in 3D.
             Nothing we know of goes from occupancy to a designed, detailed, BIM tank with
             BBS, sheet and BOQ in one step, free or paid.
```

### Description

**The pain (a normal tank on a normal job).**
- Size: the architect picks "a 50 kl sump" from memory. The fire officer asks for a separate fire reserve. The
  sump is re-sized late, after the excavation drawing has been issued.
- Design: the structural engineer opens an Excel sheet. Many sheets in circulation check strength only; IS 3370
  (2009 / 2021) asks for a crack-width check (0.2 mm, or 0.1 mm for severe exposure), which drives the steel up,
  often by 20-40% (rough figure from our own sheets, to be measured on real jobs).
- Drawing: a draftsman draws the plan, two sections and the rebar by hand. Corners, haunches and laps are where
  mistakes happen.
- BIM: the modeller builds the tank again in Revit from the drawing. The rebar is rarely modelled, so the BBS is
  made by hand from the drawing again.
- Loads: the OHT sits on the lift core; its full weight (often 20-40 t) and its sloshing in an earthquake are not
  always in the ETABS model.
- Site: the sump is dug, a footing next to it is undermined; the PVC water-stop and the waterproofing are not in
  the BOQ, so they are skipped or bought late.

**What Tank Smith does.**
1. **Size from the building.** It reads the model (or a short form): use (residential, office, hospital,
   school, mall), flats or people, floors, height, toilets. It applies the per-person demand (IS 1172 / NBC Part
   9), the storage days the office uses (a project setting: e.g. 1 day OHT + 1-2 days UGT), the fire reserve
   (NBC Part 4 table for the building's use and height; the local fire NOC rules override it, entered by the
   office) and the septic tank volume and soak pit area (IS 2470-1 and -2, by users and desludging interval;
   soil percolation from a short form). Every number shows the clause and can be changed by hand.
2. **Place.** Rules for where tanks go: UGT outside the footings' load zone, near the gate for the tanker, away
   from the septic tank (a minimum distance, e.g. 15-18 m from a well or source, set by the office from the local
   rule); OHT on the core walls or on its own columns, with clearance for the pump room and the lift machine room.
   Snaps and warns; never moves an element silently.
3. **Design to IS 3370.** Wall and slab thickness, grade, cover and bars from: water pressure (full tank) and
   earth plus surcharge pressure (empty tank, for the UGT), uplift with a high water table (check against
   flotation; add a toe or more base thickness), and the OHT's base slab and supporting walls. Plate moments from
   the IS 3370-4 coefficient tables (the method engineers already use by hand), then strength (IS 456) and the
   crack-width check (IS 3370-2). Results are in a design note with every step; the engineer of record signs it,
   Tank Smith does not.
4. **Detail.** Bars both faces, corner bars and L-bars at wall-wall and wall-base joints, haunch bars, laps
   staggered, manhole trimming bars, kicker with water-stop, puddle flanges for inlet, outlet, overflow, scour
   and vent, the ladder (MS or SS, inside or step irons), the partition wall for two compartments, the sump
   pocket for the pump suction.
5. **Model.** IFC: IfcTank for the tank as an object (with its capacity as a property), IfcWall and IfcSlab for
   the concrete, IfcOpeningElement for manholes and pipes, IfcReinforcingBar for every bar with its mark.
   Revit: walls, floors, openings, Structural Rebar, one schedule per tank.
6. **Sheet and BOQ.** Drawing sheet with plan, two sections, rebar, notes (grade, cover, crack width, test: a
   water-tightness test before plaster, as the code asks). BOQ: excavation (with working space and the slope or
   shoring from the Dig Deep idea), PCC, concrete, steel by diameter, shuttering, water-stop length,
   waterproofing area (inside crystalline coat or outside membrane, an office choice), pipes and fittings,
   ladder, manhole covers. BBS with IS 2502 shape codes.
7. **Fill reel.** A short animation in the viewer: the tank cut open, water rising, a counter showing days of
   storage, the fire reserve line, pump-on and pump-off levels, and the overflow. Recorded to MP4 / WebM for the
   client or the project's WhatsApp group.

**Daily use cases.**
1. *New residential job*: all four (UGT, OHT, fire, septic) in one go from the flat count.
2. *Municipal / fire NOC*: the fire reserve sheet with its clause and the tank drawing for the submission.
3. *Value engineering*: "two compartments or three? M25 or M30? 250 or 300 walls?" Change it and the steel,
   cost and crack width update live, side by side.
4. *Tank from a client DXF*: a box on layer TANK in the CH template becomes a full tank (cad2bim pipeline).
5. *Site check*: the BBS goes to the bar bender; the QR from the Site QR Pass idea opens the tank in 3D with its
   pour card (kicker first, water-stop in, then walls).
6. *Leaking old tank*: the Audit Eye idea finds a leak; Tank Smith re-checks the existing tank to today's crack
   width rules and gives the repair option (inside lining, extra wall).

**Not in the first version.** Circular and Intze tanks, elevated tanks on staging, and prestressed tanks; STP
process design (only its tank volumes from a supplier's table). These come later.

### Implementation

**1. A tank design core (one small library, used everywhere).**
- Pure functions, no UI: `size(building, rules) -> tanks[]`, `design(tank, soil, materials) -> members[]`,
  `detail(members) -> bars[]`, `boq(tank) -> rows[]`. TypeScript in `@cad2bim/engine` (next to `bbs/` and `qa/`)
  so the web app, the agent helper and Structura can call it; the Revit side calls it through the bridge.
- Code data as tables in JSON (per-person demand, fire reserve by use and height, IS 3370-4 moment coefficients
  by wall ratio and edge conditions, septic tank sizes), each row with its clause and edition, and the office can
  add a local-rule file. No code text is copied into the app, only the numbers engineers use, with their source.
- Tests: hand-worked examples (a 50 kl rectangular sump, a 20 kl OHT, a 100-user septic tank) checked against a
  known Excel sheet and the textbook examples, to the kg of steel and to 0.01 mm of crack width.

**2. Reading the building.**
- From IFC: IfcSpace / room names and counts, IfcBuildingStorey heights, the plot boundary if there is one, the
  footings (for the load-zone check) and the core walls (for the OHT). From the CH DXF: the flat count from the
  floor labels. Anything missing is a short form, never a guess shown as fact.

**3. Placement checks.**
- 2D on the site plan: distance to footings (with the 45-degree zone from the footing's founding level), to the
  septic tank, to the boundary; tanker access from the gate. Uses the engine's existing geometry and the QA rule
  framework (`qa/`), so a warning is a normal QA item with a fix suggestion.

**4. 3D and IFC.**
- Generate members with the same code the pipeline uses for walls and slabs (CH template v2's "tank rule" from
  the template 2 scope), so a tank from a DXF and a tank from the panel are the same thing. Bars as swept
  solids, as in BBS Weaver. IfcTank (IFC4) with `Pset_TankTypeCommon` (capacity) so other tools understand it.

**5. Outputs.**
- BBS: reuse the engine's `bbs/` export (shape codes, cut length with IS 2502 deductions).
- Sheet: the drawing generator from the Drawing Autopilot idea, or for the MVP a fixed DXF template filled in
  (plan, two sections, bar marks, notes) written by the engine's `dxf/`.
- Design note: a PDF of the calculation with inputs, clause, formula, value, check, OK / NOT OK.
- ETABS: through etabs-mcp, add "OHT water" load cases (full and empty) as area loads on the base slab outline
  and, for the seismic case, the impulsive and convective masses from IS 1893-2 for a rectangular tank (in a
  later step; the MVP adds the full water weight only and says so).
- Revit: bridge command `placeTank(design)`: walls, floors, openings and rebar by the design's bar list.

**6. Fill reel.**
- In the viewer: a section box through the tank, a water surface plane rising with a counter (litres and days
  of storage), lines for fire reserve and pump levels, recorded with the canvas recorder (the Model Reel idea's
  code), 15-25 s.

**MVP (3-4 weeks).**
1. Sizing (domestic, fire, septic) with the rule tables and a short form; the result sheet with clauses.
2. Rectangular UGT and OHT design to IS 3370 with crack width, IS 3370-4 coefficients, and the design note.
3. 3D in cad2bim (walls, slabs, haunch, manhole, bars), BBS, BOQ, and the fill reel.
4. Pilot: 5 past jobs from the office, compare Tank Smith's steel, concrete and BOQ with what was built and
   with the engineer's sheet; measure the time taken against their usual time.

### Future update

- **Circular, Intze and staging tanks** (elevated tanks on columns and braces), with IS 1893-2 sloshing and
  the staging designed as a frame in ETABS.
- **STP and ETP tanks** from a supplier's process sheet (volumes per chamber), with baffles and channels.
- **Swimming pools and lift pits** with the same water-retaining rules (they are tanks too).
- **Rain Run link:** the rainwater harvesting tank and recharge pits from the Rain Run idea use the same core.
- **Pump and pipe:** pump head and size from the OHT height and the pipe run, the pump room layout, and the
  Drain Line idea's pipe routing from OHT to the floors.
- **Water budget dashboard:** litres per day by use, reuse of STP water for flushing, savings in the tank size,
  for GRIHA / IGBC water credits (Carbon Lens idea's credit sheet).
- **Agent:** through the cad2bim agent helper (MCP), "add a 2-day sump for this building in the north-east
  corner" builds the whole thing as a proposal to accept.
- **Learning from the office:** keep a record of past tanks (size, steel, cost, leaks reported); show "similar
  tanks we built" with their actual kg/m3 next to the new design.

### Improvement suggestions for existing tools in the repos

- **cad2bim (CH template v2):** move "Water tanks, UGT / OHT" in `docs/design/ch-template-2-scope.md` from
  tier C (walls and slabs only, by labels) to a real tank rule: one closed outline plus a label
  ("UGT 88KL 2C", "OHT 25KL") makes a full tank with base, walls, top slab, haunch, partition and manhole.
  Sumps and septic tanks, listed there as later work, can use the same rule.
- **cad2bim (engine):** add IfcTank with its capacity property to the IFC writer and show capacity in the
  properties panel and in the BOQ; today a tank reads as loose walls and slabs.
- **cad2bim (QA):** a "near footing" rule for any excavation (sump, lift pit, septic tank) using the footing's
  founding level; useful beyond tanks.
- **Structura (`codecheck.js`):** add a water-retaining check mode (IS 3370 crack width) for walls and slabs
  tagged as tank members; it already makes its assumptions visible, which is what a tank check needs.
- **etabs-mcp:** a tool "add tank load" (full / empty water as area load, later impulsive and convective
  masses), so the OHT weight never goes missing from the analysis.
- **pyRevit-addons (AnonGee):** a "Tank" button that takes the design JSON and builds walls, floors, openings
  and rebar, for offices not using the bridge.
- **C2B:** recognise the tank outline and label on client DXFs and pass them through to cad2bim's tank rule.
