# 20261028 Idea: Lift Lord: type the flat count and the floor heights (or click the lift core), and cad2bim sizes and counts the lifts, builds the shaft, pit, overrun and machine room in 3D to a lift maker's clearances, puts the guide-rail brackets, landing-door sills, embeds and slab openings in the model before the slab is cast, checks the fire lift, the pit and the head room against the beams, and writes out the shaft rebar, BBS and BOQ and a lift-core drawing sheet. The client "rides the lift" in 3D from basement to terrace

## Entry 1

```
date-time:   2026-10-28 (scheduled run, UTC)
idea name:   Lift Lord. "A G+14 residential tower, 4 flats a floor, 2 basements. The modeller
             clicks the lift core box on the typical plan and presses Lifts. Lift Lord reads
             the building: 60 flats x 5 people = 300 people, 17 floors served, 3.0 m floor
             height, 54 m of travel. It tries 2 x 8-person (544 kg) lifts and shows 'five
             minute handling: 9.8 % of the population, waiting interval 46 s' against
             the office's targets; it tries 3 x 8-person and shows 13.1 % and 31 s. The
             modeller picks 3, one of them marked as the fire lift. Lift Lord then builds the
             core: three shafts with the clear plan size (width x depth) and the pit depth
             and overhead height from the lift-maker table the office has loaded (editable,
             each row with its source), 200 mm shear walls, the machine room on top (or a
             machine-room-less overhead), the pit slab, the landing door openings at every
             floor, the guide-rail bracket embeds at every 2 m or 3 m, the buffer pits, the
             sump in the pit. It then warns: 'Beam B-17-04 comes down 450 mm at the terrace
             machine room slab: the overhead is 4.2 m, the maker needs 4.4 m for 1.0 m/s
             rated speed. Raise the slab 200 mm or drop the speed to 0.63.' 'Fire lift
             shaft opens into the lobby without a protected ventilated lobby: add the
             lobby per the fire rules (check the local rules).' 'Pit depth 1.4 m, lift
             maker's table says 1.5 m for 1.0 m/s: pit below the basement raft level by 0.1
             m, conflict with Basement Box's raft thickness at the lift core: add a
             local pit drop of 400 mm (it shows the extra concrete and the waterproofing).'
             One more click: shaft wall rebar, door-opening trimming bars, embed plates
             with anchors, the pit and the machine-room slab, all with BBS and BOQ. And
             'Ride': a camera sits in the car and rises floor by floor with the floor
             numbers ticking, a door opening on each landing, the shaft wall and rebar
             visible in a cut-away of the lift core beside it.
             One engine, five front ends:
               (1) cad2bim web app: a "Lifts" dock panel (size, count, place, check, ride),
                   producing IfcTransportElement (type ELEVATOR) with the shaft as
                   IfcWall / IfcSlab / IfcOpeningElement and the embeds as IfcDiscreteAccessory,
                   so QA, BOQ and BBS already work on them,
               (2) cad2bim CH template v2: a layer LIFT with a closed rectangle and a label
                   such as "LIFT 8P 1.0MS" (or the lift core as part of a core layer) makes
                   the whole lift, using the same rule,
               (3) Revit (bridge add-in + pyRevit AnonGee button): "Place lift" builds
                   the core with the lift family's own pit/overrun parameters, openings
                   and embeds, plus a lift schedule, from the same design,
               (4) etabs-mcp: the lift core walls get a note on the openings (the door opening
                   sizes and the stiffness loss); the machine-room slab gets the machine load
                   from the maker table (reaction at the hoist beam) as a load case,
               (5) Structura: the core shows with its openings in the code-check and BOQ views.
             Outputs: IFC (lifts + shaft + embeds + bars), BBS Excel, lift-core drawing sheet
             (plan at each type of floor, section through the shaft, pit and machine room
             detail; DXF + PDF), BOQ Excel, a one-page "lift data sheet" to hand to the lift
             vendor (the clear sizes, loads on the pit and hoist beam, openings and embeds),
             a design note PDF, Revit elements, ETABS load cases, a 20 s "ride" video.
potential:   High, and very regular: every building over about four floors has at least one
             lift, and in India NBC asks for a lift above G+3 (check the local rule), so
             almost every residential, office and hospital job has a lift core. The lift is
             also the one thing that crosses every discipline (architecture, structure,
             fire, MEP, finishes) and the one that is changed late.
               - Today a lift is: the architect marks a box on the plan from a rule of thumb
                 ("two lifts for this tower"), the lift vendor gives a drawing weeks later
                 with the real shaft size, pit and overhead (which differ by maker and by
                 speed), and the structural and BIM team then change the core, the pit
                 and the machine room slab again. In practice the shaft is often redrawn two
                 or three times. About 1-2 working days of rework per job across 3-4 people
                 (estimated from how offices work, not measured).
               - Mistakes we see often (from experience, not counted): shaft too small for
                 the lift maker's car, pit too shallow after the raft is cast, overhead
                 short because a beam or the water tank sits above, fire lift without the
                 lobby, missing guide-rail embeds so brackets are drilled into the wall later
                 (which cuts rebar), door openings that do not line up floor to floor, a lift
                 that stops where the stair headroom or a ramp is, lift pit in the water
                 table without waterproofing.
             With Lift Lord (target, to be measured in the pilot): about 20-30 minutes per
             job for the first draft of all outputs, a vendor-ready data sheet on day one,
             and one source of truth, so a change of lift count or speed updates the model, the
             openings, the BBS, the sheet and the BOQ together. Over a year: an office doing
             20-40 jobs saves roughly 20-60 person-days (estimate).
             Trend: lift and "vertical transport" content, "AI that checks your design"
             and "clash detection that actually finds the late changes" are steady LinkedIn
             topics for civil and BIM pages. We did not verify a specific trend by search in
             this run (the quick search found nothing recent on lifts), so treat this as an
             opinion, to be checked. Short reels of a lift riding through a building
             in 3D do well with engineers and with clients.
             Wow effect: you type the flat count; the right number of lifts appear, with the
             core around them cut open, the rebar in the walls, the embeds at every bracket,
             the pit and the machine room; a beam that is too low is flagged; and the client
             rides the lift from the basement to the terrace.
             Market (to our knowledge, checked against what we know, not a full survey):
               - Lift makers' own tools (the big makers offer free planning and traffic
                 analysis web tools and Revit/BIM lift families): free, but each for that
                 maker only, no structure, no rebar, no BBS, no fire lift or beam check, no
                 India rules.
               - Paid traffic-analysis programs (e.g. lift consultants' simulation software):
                 handling capacity and waiting time only, no BIM.
               - Revit with a lift family: the family is placed by hand; the shaft, pit
                 and overrun are on the user, not checked against the structure.
               - Generative design in Revit / Dynamo: can be set up for cores, but needs a
                 specialist to build the graph, nothing ready for lifts.
             Nothing we know of goes from occupancy to lift count, shaft, embeds, checks,
             rebar, BBS and the vendor sheet in one step, free or paid. (Lift makers' tools
             are the nearest, and the pilot should compare against them first.)
```

### Description

**The pain (a normal lift on a normal job).**
- Count and size: the architect picks "two lifts" from memory. The tenant or the fire officer
  asks for a third or a service lift. The core is redrawn after the plan is frozen.
- Maker data: the real clear shaft size, pit and overhead depend on the maker, the rated load
  (544 kg, 884 kg, 1,088 kg), the speed (0.63 to 2.5 m/s and more), the door type (side- or
  centre-opening) and whether it is a machine-room-less (MRL) lift. The numbers come after the
  structure is drawn.
- Structure: the pit and the machine room slab are the last things the engineer checks.
  The lift pit is a mini-tank in the water table, the machine room slab takes the hoist beam
  load, and the shear walls around the shaft are weakened by the door openings.
- Site: brackets are drilled into a cast wall because embeds were not in the drawing. The pit is
  dug under a footing. The lift maker rejects the shaft because it is 20 mm short.
- Fire: the fire lift needs its own protected lobby, a separate power feed, and to reach every
  floor, including the basements, within the time the rule gives. This is often found late.

**What Lift Lord does.**
1. **Size and count from the building.** It reads the model (or a short form): use (residential,
   office, hospital, hotel), population per floor (flats x people, or area / persons per m2),
   floors served, floor heights, basements served or not. It runs a simple traffic calculation
   (five-minute handling capacity as a percent of the population, and the average waiting
   interval) for 1 to 6 lifts and a few car sizes and speeds, and shows a small table with
   the office's own targets (e.g. 10-12 % handling for residential and an interval under 60 s
   - these are set by the office, not built in, and they can be changed). It also shows the
   service/stretcher lift and the fire lift as marked options.
2. **Place.** Rules for where lifts go: grouped in one core, near the entrance lobby, away
   from bedrooms where noise matters (a rule the office can turn off), landing opening
   aligned with the lobby door, no column or beam in the landing clear width.
3. **Maker table, not a made-up size.** The shaft clear size, pit depth, overhead height,
   machine room size (or MRL), rail bracket spacing, buffer loads and hoist beam loads come
   from a table of rows the office loads (or picks from a few common generic rows),
   each row with its source (a maker's catalogue, a vendor drawing) and its date. Lift Lord does not ship
   a maker's private drawing. A generic "typical 8-person / 1.0 m/s" row is provided as a starting point,
   marked as a placeholder to be replaced with the vendor's real sizes when they arrive.
   When the vendor sends their drawing, the table row is replaced and every check re-runs.
4. **Build in 3D.** Shaft walls, pit slab, local pit drop, sump in the pit, door openings
   with lintels, sill and sill beam, guide-rail bracket embeds at the spacing, hoist beam
   and machine room (or MRL pocket), ladder and light box in the pit, and the car and
   counterweight volumes as clearance boxes. Each landing door is a real opening with the
   right size, aligned floor to floor.
5. **Check (the part that saves the late change).** Each check is a normal QA item with a
   fix suggestion and a clash view:
   - overhead height versus beams, slabs, the water tank above the machine room,
   - pit depth versus the footing, raft, water table (adds waterproofing) and the basement
     level,
   - shaft clear size versus the lift maker's row, with the tolerance the maker asks,
   - fire lift: separate shaft or fire-rated separation, protected lobby, access to every
     floor, power feed marked (the actual rule is taken from a rule file the office
     sets for the city - Lift Lord shows the clause, it does not decide),
   - door alignment floor to floor, and landing against stairs and ramps,
   - headroom for the machine room door and the hoist beam path,
   - travel (top floor to pit) against the rated speed and the rope length.
6. **Detail the structure.** Shear wall bars with trimming bars around each door opening,
   pit slab and walls with the water-retaining rules (shared with Tank Smith's core), embed
   plates with anchors, the machine-room slab with the hoist beam reaction, with bars,
   BBS and BOQ. The engineer of record signs the design note; Lift Lord does not.
7. **Hand off.** A one-page vendor sheet: the clear sizes, pit and overhead, the loads
   (pit buffer reaction, hoist beam reaction, rail bracket pull-out), the openings and the embeds,
   in the vendor's usual order. The vendor replies with a marked-up copy or a new row, and
   the model follows.
8. **Ride.** A section box through the core and a camera inside a car. The car rises with
   floor numbers, doors open on the landings, and the shaft wall, rebar and pit stay visible in a cut-away.
   15-25 s, recorded with the canvas recorder (the Model Reel idea's code).

**Honest limits.** The traffic calculation is a quick estimate for choosing a count, not a
lift consultant's simulation. The maker's numbers are only as good as the table. Fire rules differ by city
and change; Lift Lord shows the clause and asks, and never says "compliant" on its own.

### Implementation

**1. A lift core library (pure functions, no UI).**
- `size(building, rules, makerTable) -> options[]` (handling %, interval, car size, speed),
  `build(option, core) -> members[]`, `check(members, model) -> issues[]`,
  `detail(members) -> bars[]`, `boq(lift) -> rows[]`. TypeScript in `@cad2bim/engine` next to
  `qa/` and `bbs/`, so the web app, the agent helper and Structura can call it; Revit calls it
  through the bridge.
- Data as JSON tables: maker rows (shaft width and depth, pit, overhead, machine room, loads,
  bracket spacing, with source and date), traffic targets (set by the office), fire rules by city
  (each with clause and edition, and a local-rule file). No private maker drawings in the repo.
- Tests: three worked cases (a G+7 with 2 lifts, a G+14 with 3 lifts and a fire lift, a hospital with
  a stretcher lift) with known handling figures from a textbook, and a hand-checked pit and overhead
  against a catalogue sheet, to the millimetre.

**2. Reading the building.**
- From IFC: IfcBuildingStorey elevations, IfcSpace for flats and use, the lift core walls, the
  raft and footings (pit conflict), beams above the shaft top (overhead). From the CH DXF: the
  LIFT layer and label. Anything missing is a short form, never a guess shown as a fact.

**3. Checks as QA rules.**
- Use the existing `qa/` framework; each rule yields an item with element ids, a message and
  a fix suggestion. The overhead and pit checks need a vertical ray from the shaft rectangle:
  reuse the engine's geometry and the section box. A "near footing" rule is shared with Tank Smith and
  Basement Box.

**4. 3D and IFC.**
- Generate members with the pipeline's wall and slab code (CH v2), so a lift from a DXF and a
  lift from the panel are the same thing. IfcTransportElement (IFC4, predefined type
  ELEVATOR) with `Pset_TransportElementElevator` (capacity, speed, travel, number of stops).
  Bars as swept solids, as in BBS Weaver. Embeds as IfcDiscreteAccessory with an anchor list.

**5. Outputs.**
- BBS reuses the engine's `bbs/` export. The sheet uses the Drawing Autopilot generator, or
  for the MVP a fixed DXF template filled in (plan, section, pit detail, machine room detail).
  The vendor sheet is a one-page PDF from the same data. ETABS: openings noted and the machine
  load added through etabs-mcp.

**6. Ride video.** Camera path along the shaft axis with the section box, recorded as in the
Model Reel idea; floor numbers are an overlay; doors are animated by a simple scale of two door boxes.

**MVP (3-4 weeks).**
1. Traffic estimate and the lift table with the generic starting rows, a short form, and a result
   sheet.
2. Shaft, pit, overhead, machine room and door openings in 3D from a click on the core box; IFC.
3. Overhead, pit, shaft-size and door-alignment checks as QA items; the vendor sheet.
4. Shaft rebar, BBS, BOQ and the ride video.
5. Pilot: five past jobs from the office. Compare Lift Lord's count and sizes with what was built,
   how many of the late changes it would have flagged, and the time taken against the usual time.

### Future update

- **Escalators, travelators and dumbwaiters** with the same table approach (openings, truss
  loads, pits).
- **Lift groups with destination control** in the traffic estimate, and a compare with the
  estimate of the vendor's own program.
- **Real vendor families:** import a maker's Revit/IFC lift and read its clearances into the table,
  so the "maker table" fills itself.
- **Core optimizer** with Tower Stacker and Load Path X-Ray: slide the core, resize the walls,
  and show the effect on stiffness, flat sizes and carpet area (Area Statement Pro).
- **Exit Map link:** fire lift, evacuation lift and refuge floors in the egress replay.
- **Construction link:** lift pit and shaft pour in the 4D pour planner, the lift installation
  (rails, car, test) as 4D tasks, and the temporary construction hoist for the crane/logistics
  plan (Crane Canvas).
- **Handover Hub link:** each lift as an asset with QR tag, AMC dates and the annual
  inspection due date in the O&M manual.
- **Agent:** through the cad2bim agent helper (MCP), "add a fire lift to the north core and check
  the overhead" proposes the whole change to accept.
- **Learning from the office:** a record of past lifts (count, vendor, final shaft size, change
  count) shown as "similar jobs we built".

### Improvement suggestions for existing tools in the repos

- **cad2bim (CH template v2):** add a lift rule in `docs/design/ch-template-2-scope.md`: a
  closed rectangle on a LIFT layer with a label ("LIFT 8P 1.0MS", "FIRELIFT") makes the shaft,
  pit, openings and machine room; today a lift core is just walls by labels.
- **cad2bim (engine):** write IfcTransportElement (ELEVATOR) with its property set and show
  capacity and speed in the properties panel and the BOQ.
- **cad2bim (QA):** a vertical-clearance rule (ray from a footprint up and down to the nearest
  beam, slab or footing) usable for lifts, stairs headroom, tanks and ramps.
- **Basement Box:** take the lift pit from Lift Lord's maker table instead of its own, so the
  pit depth and the raft drop come from one place.
- **Stair Smith / Exit Map:** read the lift lobby and fire lift for the egress check; one core,
  one set of doors.
- **etabs-mcp:** a tool "add core openings" (door openings as wall openings with lintel stiffness)
  and "add machine room load".
- **pyRevit-addons (AnonGee):** a "Lift" button that takes the design JSON and builds the shaft,
  openings, embeds and the lift schedule, for offices not using the bridge.
- **C2B:** recognise a lift rectangle and label on client DXFs and pass them to cad2bim's lift rule.
