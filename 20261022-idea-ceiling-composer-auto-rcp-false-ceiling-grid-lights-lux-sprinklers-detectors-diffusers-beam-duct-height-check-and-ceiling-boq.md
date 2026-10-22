# 20261022 Idea: Ceiling Composer — one click from rooms to a coordinated RCP: false-ceiling grid, lights to lux, sprinklers, smoke detectors, AC diffusers and speakers, all on grid and clear of beams and ducts, with ceiling-height check, RCP sheets and ceiling BOQ

## Entry 1

```
date-time:   2026-10-22 (scheduled run)
idea name:   Ceiling Composer — "Pick the rooms, pick a ceiling style. In two minutes every room gets
             its false-ceiling grid centred and balanced, lights placed to hit the target lux, sprinklers
             and smoke detectors at code spacing, AC diffusers / cassette units and speakers dropped into
             free tiles, nothing landing on a beam, a duct or a grid line, and the ceiling height checked
             against the deepest beam + duct + slope above. Out come the RCP sheets, the device schedules
             and the GI-channel / board / tile BOQ."
             One Python engine (shared by C2B and the AnonGee pyRevit tab) with four front ends:
               (1) pyRevit button (AnonGee tab, MEP / Interior panel): reads rooms, ceilings, beams, ducts,
                   pipes, cable trays and linked models; writes Revit ceilings with grid, lighting /
                   sprinkler / detector / diffuser families, bulkheads where needed, tags and RCP sheets,
               (2) C2B: on a client DXF plan, rooms and beams from layers give a 2D RCP DXF with device
                   blocks and a ceiling BOQ the same day, before a Revit model exists,
               (3) cad2bim web viewer: "look up" mode. Walk the flat or office in the browser, ceiling and
                   devices in 3D, a lux heatmap on the floor, sprinkler spray cones and detector coverage
                   discs as overlays, tap a device for its card,
               (4) structura / Bill Painter: ceiling and device quantities into the BOQ and RA bills.
             Outputs: Revit ceilings with grid origin set, placed and tagged devices per room, lux /
             uniformity table per room (Eh avg, Emin/Eavg, W/m² vs ECBC LPD), sprinkler and detector
             coverage report (spacing, wall distance, obstruction, coverage %), ceiling-height clash list
             (room, beam / duct / pipe, clear height, shortfall), bulkhead proposals, RCP sheets with
             dimensions to grid, and a ceiling BOQ (gypsum / mineral fibre tiles, GI perimeter, ceiling
             section, intermediate channel, hangers, cut tiles, wastage, cove light length).
potential:   Very high, every day, on every interior fit-out, office, hospital, mall, school, hotel and
             premium residential project. The RCP is the drawing that the architect, interior designer,
             electrical, fire and HVAC consultants all fight over, and the BIM modeller is the one who has
             to make it agree.
             Time today (one office floor of ~1500 m² or a 3BHK interior):
               - ceiling grid setting out and balancing (no slivers at walls), by hand: 2-4 hours,
               - light layout: export to DIALux / Relux, calculate, copy positions back by eye: 4-8 hours,
               - sprinkler layout to IS 15105 / NFPA 13 spacing, by hand in CAD: 4-6 hours,
               - smoke / heat detector layout to IS 2189 coverage, by hand: 2-3 hours,
               - diffusers, speakers, access panels squeezed in, then "everything moved" when
                 coordination finds a beam or duct: 4-8 hours, repeated each revision,
               - ceiling-height vs beam + duct check: done late, in the clash meeting: 3-6 hours,
               - ceiling BOQ from the drawing with a scale: 2-4 hours.
             With Ceiling Composer: 30-60 minutes per floor including review, and each revision is a
             re-run. Over a fit-out with 6-10 revisions this saves 3-6 weeks of BIM + MEP drafting time.
             Quality saved: lights centred on a tile but over a beam; sprinkler 2.1 m from a wall where the
             max is half the spacing; detector inside a beam pocket it cannot see out of; diffuser on a grid
             line; a 2.4 m ceiling promised in the brochure that is really 2.25 m under the main duct; BOQ
             tiles counted as whole when one third of them are cut.
             Trending angle: "false ceiling design ideas" and cove-light reels are evergreen top content on
             Indian Instagram interiors pages; "auto layout in Revit with Dynamo / AI" and "MEP coordination
             before / after" posts are some of the most shared BIM content on LinkedIn, and fire-safety
             audits after recent hospital and mall fires have pushed sprinkler / detector compliance into
             the news. The reel: an empty grey office floor, looking up; the grid draws itself from the
             centre and snaps balanced; lights pop in row by row while a lux heatmap on the floor turns
             from blue to even green, "312 lux, U0 0.62, 6.8 W/m² PASS"; sprinklers drop with their spray
             cones overlapping just right; one cone turns red at a beam, the sprinkler slides 300 mm and
             goes green; detectors fill their discs; diffusers land in free tiles; a red band shows a duct
             under the ceiling line, a bulkhead grows around it; the RCP sheet and BOQ fly out. 30 seconds.
             Market: DIALux evo / Relux (free, lighting only, separate model, positions copied by hand);
             ElumTools / AGi32 (paid lighting inside or beside Revit, no sprinklers or detectors);
             AutoSPRINK / HydraCAD / MagiCAD Sprinkler Designer / SprinkCAD (paid, expensive, sprinklers
             only, hydraulics focus); Revit itself (manual ceilings, manual device placement, ceiling grid
             origin by hand); Dynamo graphs from blogs (one task each, brittle, no codes). Nothing free
             that does grid + lights to lux + sprinklers + detectors + diffusers together, coordinated
             against structure and ducts, with height check, RCP sheets and BOQ in one step, and nothing
             with IS 15105 / IS 2189 / NBC 2016 / ECBC rule packs out of the box.
```

### Description

**The pain (daily life of a BIM modeller on a fit-out or MEP coordination job).**
- The architect gives a room plan and says "600 x 600 grid, 2.7 m ceiling". The modeller sets the grid
  origin by trial and error so the cut tiles at both walls are equal and more than half a tile. Every room.
- The electrical consultant sends a light layout made in DIALux on a different base plan. The modeller
  copies positions by eye; half of them land on grid lines or over beams.
- The fire consultant sends a sprinkler layout on yet another base plan. Spacing and wall distance are
  checked by measuring. Beam pockets and deep beams that block spray are found late, or on site.
- Smoke detectors are dropped "one every 60 m²" without checking beam depth, pockets, or distance to
  diffusers (supply air blows smoke away from detectors).
- HVAC wants four-way cassettes and linear slots in specific tiles, the interior designer wants a cove
  and a gypsum island in the middle, and everything collides.
- Two weeks before the ceiling contractor starts, the clash run shows a 400 mm duct below the ceiling line
  in the corridor. The ceiling drops 150 mm, the brochure height is lost, and the RCP is redrawn.
- The ceiling contractor's BOQ is measured off the RCP with a scale, without cut tiles, hangers or GI
  quantities by section, and it never matches the bill.

**The idea, in seven parts.**
1. **Read the room and what is above it.** For each room: boundary, finished floor, target ceiling
   height, function (office, cabin, corridor, toilet, bedroom, ward, retail), and everything between the
   ceiling line and the slab: beams (from the structural model / link), ducts, pipes, cable trays, the
   sprinkler main. Build a "what is above" depth map per room (clear height at every point).
2. **Ceiling height check first.** For the target ceiling height, flag every point where beam bottom /
   duct bottom + insulation + hanger + ceiling build-up (from a rule pack, e.g. 100 mm for grid, 150 mm
   for gypsum) does not fit. Show the shortfall map in colour and offer three fixes: lower the whole
   ceiling, add a bulkhead / drop along the obstruction (auto-drawn with its width), or flag the duct for
   re-routing (BCF issue to the MEP team via the IDS / BCF tooling).
3. **Grid composer.** Per room and ceiling style (600 x 600 grid, 600 x 1200, plain gypsum, gypsum with
   cove border, island + grid, baffle / open-cell): set the grid origin so cut tiles at opposite walls are
   equal and >= half a tile, align with the neighbouring corridor grid where they meet, and keep the main
   light row on the room's long axis. Gypsum styles get the cove profile and the border width.
4. **Lights to lux (lighting engine).** Target illuminance and uniformity per room function from a rule
   pack (IS 3646 / NBC 2016 Part 8 values, editable: office 300-500 lux, corridor 100, toilet 150,
   bedroom 150 ...). Choose fittings from an office catalogue (IES / LDT photometry files linked to Revit
   types). Solve rows x columns on tile centres (or a free layout on gypsum), compute point-by-point
   illuminance on the working plane with the lumen method for a first guess and a direct + simple
   inter-reflection calculation for the check. Report Eavg, Emin / Eavg, W/m² against ECBC lighting power
   density. Emergency lights and exit signs placed by a separate rule.
5. **Sprinklers and detectors (life-safety engine).** Sprinklers by hazard class (IS 15105 / NBC Part 4,
   NFPA 13 as an alternate pack): max coverage area, max spacing, max and min wall distance, min distance
   between heads, beam rule (distance from beam vs beam depth below deflector), obstruction check against
   lights and diffusers, pendent vs concealed, and heads in the above-ceiling void where the depth and
   fire load require it. Detectors by IS 2189: coverage radius by type, ceiling height and beam depth
   (beam pockets treated as separate compartments when deep), distance from supply diffusers, and toilets /
   kitchens with heat detectors. Every device is placed in a tile centre or on a quarter point when the
   grid allows, never on a grid line.
6. **Diffusers, speakers, access panels.** From HVAC data (air flow per room, or a simple per-m² rule):
   number and type of diffusers / cassettes, placed in free tiles away from detectors and lights;
   return grilles; PA speakers to a coverage rule; access panels under valves, dampers and junction boxes
   found in the "what is above" model. A placement solver resolves all device types together with
   priorities (sprinkler and detector code rules first, lights second, diffusers third, speakers last)
   and reports any rule it could not meet.
7. **Sheets, BOQ and viewer.** RCP views per floor with ceiling heights, grid origins dimensioned,
   device tags and a legend; a coordinated device schedule; the ceiling BOQ by type: tiles (whole and cut),
   GI perimeter angle, main runners / ceiling section, cross tees / intermediate channels, hangers at code
   spacing, gypsum board area, cove length, bulkhead area, plus device counts. The cad2bim viewer shows the
   ceiling and devices in 3D with the lux heatmap and coverage overlays, for client and consultant sign-off.

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)**: a high-visibility interior / MEP button on the existing WPF UI templates
  and `bridge/`. Revit already holds rooms, ceilings, linked structure and MEP; the tool only needs to read
  them and write ceilings and families.
- **C2B** already reads client DXF plans and layer profiles; beams and room boundaries from the structural
  and architectural layers are enough for a 2D RCP and a BOQ on day one.
- **cad2bim web** already renders IFC with three.js; a "look up" camera, floor heatmap texture and
  coverage decals are a viewer plugin, and the sign-off link replaces a printed RCP meeting.
- **structura** BOQ, **Bill Painter** (RA bills from painted progress) and **Wall Wizard** (room finish
  schedule) consume the ceiling quantities; **Live Wire** gets light points and circuits from the placed
  fittings; **Sleeve Punch** gets the sprinkler drops and access panels; **Exit Map** gets emergency lights
  and exit signs; **Sun Compass** daylight results reduce artificial light near windows (daylight zones);
  **Model Reel** records the 30-second clip; **IDS QA / BCF** carries the duct re-route requests.

**Wow moments.**
- The ceiling grid draws itself from the room centre and snaps balanced, with equal cut tiles at the walls.
- The floor lux heatmap turns from patchy blue to even green as lights pop in, and the LPD counter flips to
  PASS.
- Sprinkler spray cones and detector discs cover the floor; one goes red at a beam, slides, goes green.
- A red band appears under the corridor duct; a bulkhead grows around it, and the clear-height table shows
  the brochure height is kept everywhere else.
- RCP sheet, device schedule and BOQ come out in the same click, and the client walks the ceiling on a
  phone.

### Implementation

**Phase 1 — Room / obstruction reader, height check, grid composer, BOQ (3 weeks).**
- Python package `c2b.ceiling` (shared by pyRevit through the bridge):
  `read_rooms(model) -> [Room]`, `above_map(room, beams, ducts, pipes, trays) -> HeightGrid`,
  `height_check(room, target, rules) -> [Shortfall]`, `compose_grid(room, style, rules) -> Grid`,
  `ceiling_boq(rooms, grids, rules) -> Boq`.
- Geometry with Shapely (room polygons, grid clipping, cut tile classification) and numpy height grids at
  50-100 mm resolution; obstruction solids from Revit geometry or IFC (IfcOpenShell) projected to plan
  with their bottom levels.
- Rule packs in YAML: `ceiling_buildup.yaml`, `grid_styles.yaml`, `boq_rates.yaml`, each with version and
  source note.
- CLI: `c2b ceiling plan.dxf --style grid600 --height 2700 --rules profiles/ceiling_default.yaml`.
- Tests: rectangular, L-shaped and rotated rooms; hand-checked balanced origins; BOQ against a measured
  sample RCP from a past project.

**Phase 2 — Lighting and life-safety engines, placement solver (4 weeks).**
- Photometry: parse IES LM-63 and EULUMDAT (LDT) files; point-by-point direct illuminance on a working
  plane grid plus a simple inter-reflection term (room surface reflectances from the rule pack); lumen
  method for the first layout guess. Validate against DIALux results on 5 sample rooms (target within
  10 %).
- Sprinkler rules (IS 15105 / NBC Part 4, NFPA 13 alternate): spacing, coverage, wall distance, beam rule
  table, obstruction distances. Detector rules (IS 2189): coverage by type and height, beam pocket logic,
  distance from supply diffusers.
- Placement solver: candidate points = tile centres / quarter points (or a fine lattice on gypsum);
  integer or greedy + local search with hard code constraints and soft aesthetic ones (symmetry, alignment
  across rooms, rows on the long axis). Explain every unmet rule.
- Output: a JSON "ceiling proposal" (grid origins, devices with type and position, bulkheads, findings).

**Phase 3 — Revit write-back, sheets, viewer, downstream (3 weeks).**
- pyRevit button: WPF form (rooms, style, targets, rule packs, fixture catalogue, preview); creates ceilings
  by room boundary with the grid origin set by moving the ceiling pattern alignment, places hosted
  (face-based) light, sprinkler, detector, diffuser and speaker families from a type-mapping JSON, draws
  bulkheads as ceilings / soffits, tags devices, creates RCP views and sheets with a legend and the
  schedules. All changes in one transaction group with a single undo; re-run updates existing elements by
  a `CC_Id` parameter instead of duplicating.
- cad2bim viewer plugin: "look up" camera preset, lux heatmap texture on the floor, spray cone and
  detector disc overlays, device cards, BCF export of open findings; Model Reel preset for the clip.
- Export quantities to the structura BOQ; light points to Live Wire; drops and panels to Sleeve Punch.

**Tech notes / risks.**
- Codes differ by authority and project (local fire NOC, client standards, NFPA for MNC clients). Every
  rule lives in a versioned pack named on each report; the output is a design aid for the fire / MEP
  consultant to approve and sign, not a replacement for the hydraulic calculation or their stamp.
- Lighting accuracy: the simple calculation is for layout; the tool also exports the room and fittings to
  DIALux / Relux format (or IFC) for the consultant's final calculation.
- Revit ceiling grid alignment via the API is fiddly (pattern alignment references on the ceiling face);
  fall back to placing the ceiling sketch with an offset when direct alignment fails, and verify by reading
  the pattern back.
- Linked models: obstruction reading must handle link transforms and phase / design options; cache per
  link version.
- Office families vary; ship a default family set and map by type name in JSON.

### Future update
- Sprinkler hydraulics-lite: pipe sizing from the most remote area by pipe schedule method, branch line
  routing to the main, and a node table to hand to the fire consultant's hydraulic software.
- Circuiting: group lights into circuits and switching zones (with occupancy / daylight sensors) and hand
  them to Live Wire's DB schedule and SLD.
- Acoustic mode: speaker coverage and ceiling tile NRC choice per room function (classrooms, meeting rooms).
- Hospital and clean-room packs: ward light / nurse call, operating theatre laminar flow ceiling, HEPA
  module grid.
- AI mode: "make the ceiling feel premium within ₹ X / m²": the AI box proposes cove + island + grid
  combinations, re-runs lux and BOQ, and shows the cost delta.
- Site check: Site QR Pass on each room opens the RCP and device list; Scan Check compares installed device
  positions from a point cloud or phone photos with the model.
- Retail / mall packs: shop-front bulkhead, signage zones and tenant ceiling handover drawings generated from
  the base-build RCP.
