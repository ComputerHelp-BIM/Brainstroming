# 20261021 Idea: Live Wire — rooms in, electrical points, slab conduits, circuits, balanced DB, SLD and cable BOQ out

## Entry 1

```
date-time:   2026-10-21 (scheduled run)
idea name:   Live Wire — "Pick a flat. Press Wire. Watch the lights pop in, the conduits run across the slab,
             and the DB balance itself."
             A pyRevit button (AnonGee tab) + a web/DXF/IFC mode in cad2bim that takes rooms (Revit rooms,
             cad2bim-converted DXF room labels, or IFC spaces) and, from an editable room-type rule pack:
               (1) places light, fan, switchboard, 6A/16A socket, AC, geyser, TV/data points at rule heights,
               (2) groups them into circuits (lighting / power / AC / geyser) within point and load limits,
               (3) routes concealed conduits in the slab (ceiling points) and walls (switches, sockets) back to
                   the flat DB, with a slab-congestion check against the pour,
               (4) balances circuits across R/Y/B phases and sizes MCB / RCCB / cable per circuit,
             then outputs the slab conduit layout sheet (the drawing site needs BEFORE the pour), the DB
             schedule, an auto single-line diagram (SLD), and a wire + conduit + accessory BOQ.
potential:   Very high for Indian residential, hostel, hotel and office fit-out jobs. Electrical is the MEP trade
             that is modelled last and least: modellers place hundreds of point families one by one, circuits
             are rarely made in Revit, conduits are never modelled, and the "slab conduit drawing" is hand-drawn
             by the electrical contractor the night before casting.
             Time saved: a 3BHK flat point layout + circuiting by hand ~4-6 h -> ~5 min (rules + review);
             a 15-storey tower with 4 flat types goes from ~5-7 days of electrical modelling + SLD/DB schedule
             drafting to about one day including checks. The slab congestion check prevents a common defect:
             3-4 conduits crossing in a 125 mm slab at the DB entry, honeycombing and cover loss (conduit OD
             should stay under 1/3 of slab thickness, IS 456 cl. 12.4), found only after de-shuttering.
             Trending angle: "generative MEP" / "auto electrical layout with AI" and "Revit electrical circuits in
             one click" are hot LinkedIn/Instagram topics in 2025-26 (Revit 2025/26 electrical analytical and
             circuit-naming updates brought fresh attention). The reel writes itself: empty floor plan -> points
             pop into every room -> orange conduits draw across the slab to the DB -> three phase bars R/Y/B
             even out live -> SLD appears. That is the wow for both modellers and site electricians.
             Market: Revit's own circuits (manual, no layout, no conduit-in-slab, no phase balancing beyond a
             panel schedule), MagiCAD Electrical, Trimble/DDS-CAD, ElumTools, Electrical Design add-ins (all
             paid, priced per seat per year, built around EU/US practice). Free DIALux does lighting only.
             Nothing free that knows Indian residential practice (modular switchboards, 6/16A points, flat DB
             with RCCB, IS 732 / NBC Part 8 point loads, concealed PVC conduit in slab) and connects the layout
             to the slab pour and the cable BOQ.
```

### Description

**The pain (daily life of a BIM modeller on an Indian residential / commercial job).**
- The electrical consultant sends a marked-up PDF (or nothing). The modeller places every light, fan, switch,
  socket family by hand, room by room, flat by flat, then repeats it for mirrored flats.
- Circuits are almost never created in Revit because it is tedious; so panel schedules, load totals and phase
  balance are done in Excel by someone else and never match the model.
- Conduits in the slab are not modelled at all. The slab conduit layout is drawn by the contractor on site, often
  on the shuttering itself. Congestion near the DB and lift lobby, conduits over beams, and missing sleeves are
  found during or after the pour (and Pour Planner / Site QR Pass from earlier ideas cannot check what is
  not in the model).
- SLDs and DB schedules are drawn in AutoCAD from scratch for every flat type; the cable and accessory BOQ is
  estimated with thumb rules ("x m of 1.5 sq mm per point").

**The idea, in six parts.**

1. **Read the rooms.** Input: Revit rooms (name + area + boundaries), cad2bim DXF with room text labels
   (BED, LIVING, KITCHEN, TOILET...), or IFC `IfcSpace`. Doors and windows are read to keep the switchboard on the
   latch side of the door and sockets off window walls. The flat DB location is picked once (or found from an
   `IfcElectricDistributionBoard` / a DB family / a "DB" block).

2. **Place points from a room-type rule pack.** An Excel sheet (same Excel bridge style as the AnonGee tools):
   per room type, the number of lights per m², fans, 6A and 16A sockets, AC point, geyser point, TV/data,
   heights (switchboard 1200, sockets 300, AC 2100, geyser 1800 mm...), and clearances. Lights are placed on a
   grid fitted to the room (or a simple lux target per room using the lumen method for offices). Points are real
   Revit families with the project's types, so they tag, schedule and export to IFC as `IfcLightFixture`,
   `IfcOutlet`, `IfcSwitchingDevice`.

3. **Circuit and balance.** Points are grouped into circuits by rules (e.g. lighting max 10 points / 800 W,
   power 16A one per circuit, AC dedicated), nearest-neighbour so circuits stay compact. Each circuit gets MCB
   rating, cable size (1.5 / 2.5 / 4 / 6 sq mm by load, length and voltage drop) and an RCCB group. A small
   balancing step assigns circuits to R/Y/B so the phase loads are within a set spread; the panel shows three
   bars that even out live (the "wow" moment). Circuits are created as real Revit electrical systems so Revit's
   own panel schedule fills itself.

4. **Route conduits and check the slab.** Ceiling points are joined by a tree of concealed conduits in the slab
   back to the DB (grid router on 50 mm, 0/90° only, prefer straight runs, avoid columns, cross beams only at
   allowed zones with a sleeve). Wall points drop from the slab to switchboards. Then the slab check:
   - conduit OD vs slab thickness (OD <= 1/3 of the slab, 25 mm in 125 mm OK, 32 mm flagged),
   - number of conduits stacked / crossing at one spot (> 2 crossing flagged, typical at the DB and corridor),
   - spacing from rebar and from the slab edge, conduits running over a beam top, conduits in a sunken slab
     (links to Drain Line), conduits through a post-tension or shear zone.
   Results are painted green / amber / red in a 3D view with labels ("4 conduits crossing at DB, split into 2
   routes"), and red items become BCF issues for structure.

5. **Outputs the site actually uses.**
   - **Slab conduit layout sheet** per slab pour (conduit size, number, run, junction boxes, DB entry), A3 PDF + DXF,
     ready the day before casting; a QR on it opens the 3D view of that pour (reuses Site QR Pass).
   - **DB schedule** per flat type: circuit no., description, points, load, MCB, cable, phase.
   - **Auto SLD** (main incomer -> meter -> flat DB -> RCCB groups -> MCBs) as DXF/PDF, drawn from the circuits,
     never redrawn by hand.
   - **BOQ to Excel:** wire metres per size and colour (from routed lengths + drops + allowance), conduit metres per
     size, bends, junction boxes, modular plates by module count, switches, sockets, MCBs, RCCBs, DBs.
   - **Load summary** per flat, floor and building (connected load, diversity, maximum demand) for the
     electricity board application.

6. **Preview, then commit.** As in Toolsmith and Drain Line, everything is first a preview (temporary coloured
   graphics + tables). "Commit" creates all Revit elements in one undo group. Re-running after the architect moves
   a wall updates only the affected rooms.

**Why it is a natural extension of this toolkit.**
- `cad2bim` already maps electrical IFC classes (`IFCLIGHTFIXTURE`, `IFCOUTLET`, `IFCSWITCHINGDEVICE`,
  `IFCELECTRICDISTRIBUTIONBOARD`, `IFCCABLECARRIERSEGMENT` in `packages/engine/src/model/categories.ts`) and
  0.91.0 was optimised on the owner's heavy ELECTRICAL model, so the web viewer can show points, conduits and
  phase colours and export IFC4 without Revit.
- `pyRevit-addons` (AnonGee tab) has the one-click bulk pattern (AutoLevel, Bulk Tool, StrucBIM) and the Excel
  bridge for an editable rule pack.
- `structura` / `etabs-mcp` know slab thickness, beam positions and PT/shear zones, exactly what the slab conduit
  check needs; `C2B` already reads DXF layers and text, which gives the no-Revit room input.
- It closes the MEP set started by Sleeve Punch (openings) and Drain Line (plumbing): after Live Wire, every
  wet and dry room gets its services in minutes, and the pour check covers both pipes and conduits.

### Implementation

**Phase 1 — Revit MVP: points + circuits for one flat (2-3 weeks).**
- `LiveWire.pushbutton` in `AnonGee.tab/Essential.panel` (or a new `MEP.panel` shared with Drain Line).
- Rule pack `livewire_rules.xlsx`: room-type table, heights, family-type mapping, circuit limits, cable table.
- Placement: room boundary via `Room.GetBoundarySegments`, light grid fitted to the room's largest rectangle,
  wall-hosted points by `doc.Create.NewFamilyInstance(face, point, dir, symbol)` on the chosen wall face,
  door swing read from `FamilyInstance.FacingOrientation` / `HandOrientation`.
- Circuits: `Electrical.ElectricalSystem.Create(doc, connectorIds, ElectricalSystemType.PowerCircuit)`, then
  `SelectPanel(db)`; MCB rating / cable size written to shared parameters (circuit rating, wire size).
- Phase balancing: greedy largest-first assignment to the lightest phase, then a few swap passes; written to a
  phase parameter used by the DB schedule.
- Preview with `DirectContext3D` or temporary model lines + `OverrideGraphicSettings`; one `TransactionGroup`.

**Phase 2 — Conduits, slab check, outputs (3 weeks).**
- Conduit tree: same deterministic grid router idea as Drain Line (shared module), created with
  `Electrical.Conduit.Create(doc, conduitTypeId, p1, p2, levelId)` and standard fittings (0/90° only so fitting
  creation is reliable).
- Slab check: slab thickness from the floor type, beams from structural framing bounding boxes; crossings counted
  on a 100 mm grid; OD rule and cover rule from the rule pack.
- Sheets: plan view per pour zone (scope box), auto-tag conduits, export PDF/DXF; DB schedule as a Revit schedule
  + Excel; SLD drawn as DXF with `ezdxf` from the circuit table (no Revit drafting needed).
- BOQ export in the existing Excel BOQ style; BCF for red items (reuse the IDS-QA BCF writer).

**Phase 3 — Typical floors and web mode (3-4 weeks).**
- Flat fingerprint (room types + relative positions, mirrored allowed) to propagate to typical and mirrored flats,
  with circuit and DB naming per flat (F-1203-L1...).
- Port placement + circuiting + router to TypeScript in `cad2bim/packages/engine` (rule pack shared as JSON);
  input DXF with room labels -> points and conduits in the browser -> IFC4 (`IfcLightFixture`, `IfcOutlet`,
  `IfcCableCarrierSegment`, `IfcDistributionCircuit`) + SLD PDF.

**Tech notes / risks.**
- Rules vary by consultant and client: everything must come from the Excel pack, nothing hard-coded, and the pack
  is versioned per project.
- It is a design assist: the DB schedule and SLD carry "for coordination; electrical consultant to approve" and
  list the rule values used (diversity, voltage drop limit, cable table).
- Keep the router and the balancer deterministic so a re-run after a small change gives a small diff (pairs well
  with Change Radar).

### Future update

- **Lighting by lux:** a quick lumen-method / point-by-point check per room with a false-colour lux map, and an
  optional export to DIALux for the consultant.
- **Common areas and risers:** lobby, staircase and basement lighting, the rising main / bus duct in the shaft,
  floor DBs and the building-level SLD up to the transformer and DG set.
- **Low current systems:** CCTV, data, fire alarm detectors (spacing per IS 2189) and PA speakers with the same
  place-circuit-route flow, plus cable tray routing in ceilings with tray fill check.
- **Solar + EV ready:** rooftop PV string layout from the roof slab and EV charger points in parking, with the
  load added to the building summary (trending topic for green-building posts).
- **Site progress hook:** conduit laid / wiring done / fixtures fixed per flat feeds Bill Painter (RA bill) and
  Model Reel (4D video), and the slab conduit sheet sign-off becomes a checklist item in Pour Planner.
- **Photo-to-points:** read the consultant's marked-up PDF or a site photo of a switchboard layout with a vision
  model and use it as placement hints instead of the default rules.
