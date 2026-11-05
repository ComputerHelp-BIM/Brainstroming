# 20261105 Idea: Yard Smith — survey levels and a plot outline become the whole outside of the project in 3D: roads, kerbs, storm drains, paving, compound wall, gates, STP and landscape, with cut/fill, a gradient and drain-fall check, an external-works BOQ, and a "rain flows to the drain" replay

## Entry 1

```
date-time:   2026-11-05 (scheduled run, second entry of the day)
idea name:   Yard Smith (External Development / Site Works Modeller: Survey Spot Levels + Plot Outline ->
             Ground Surface in 3D -> Internal Roads, Kerbs, Paving, Storm-Water Drains, Compound Wall,
             Gates, Underground Tanks / STP Pits, Trees and Landscape Zones -> Gradient, Drain-Fall and
             Cut/Fill Checks -> External-Works BOQ -> Rain-Flow Replay -> IFC (IfcGeographicElement /
             IfcCivilElement / IfcWall) and a Clean Site Plan DXF)
potential:   High, and a gap in the whole idea folder. Every idea so far models the building: columns,
             slabs, basements, terraces, ceilings, lifts. Nothing models what is OUTSIDE the building line,
             and every real project has it: 5 to 15 percent of the cost of a residential township or
             industrial plot, a large share of the site drawings, and a weak spot of most BIM models (the
             site is a flat grey plane in Revit, or a separate CAD file nobody links). The work is also
             repetitive and rule-driven (road width, kerb, camber, drain gradient, wall spacing and
             pillars), which makes it a good fit for automatic layout. Time saved (an estimate, not
             measured): a modeller spends about 1 to 3 days on external works for one building plot in
             Revit and a further day on its quantities; a first pass from this tool could be minutes plus
             a review. Most of the value is not speed alone: gradients that do not drain, a drain that runs
             uphill into the road level, and a gate that does not clear the fire tender are found in the
             model, not at the first monsoon.
```

### Description

**The pain (what the BIM modeller, site engineer and architect actually do today).**
- The building gets the care; "external development" gets a few lines in a CAD file: a road outline, a
  compound wall line, hatch for paving, a note "drain as per detail". Levels come as a survey spreadsheet
  or a DXF full of small spot-height texts.
- In Revit, toposurface from survey points is possible but slow to prepare, and the roads, kerbs, drains
  and compound wall are drawn one by one. Drain gradients are checked by eye. Cut and fill is a separate
  spreadsheet, if it exists.
- The usual site-level failures: the plinth is lower than the road crown so water flows in; a storm drain
  with no fall because two spot levels were mis-read; a gate leaf that hits the pillar; a bend too tight for
  a fire tender; the STP pit sitting on the neighbour's side; paving quantities taken from a rough area
  hatch. These show up at construction, or in the first heavy rain.
- The external-works BOQ (PCC and paver, kerb stone, drain length by depth, compound wall running metre,
  gate count, earthwork) is done by hand from a DXF, so it is one of the least reliable items of a tender.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Dig Deep (2026-10-19) does **foundation and basement** earthwork and shoring,
  not site grading; Rain Run (2026-10-24) slopes **terraces and podiums** and sizes their outlets; Plot Pilot
  (2026-10-25) makes **massing options** inside a plot; Park Pilot (2026-10-22) lays out **basement and stilt
  parking**; Gap Guard (2026-10-31) mentions **compound-wall movement joints**; Crane Canvas (2026-10-21) is a
  site-logistics plan. None of these makes the **site surface and the external works as elements** with
  levels, falls and a BOQ. Checked by searching the folder for compound wall, paver, contour, kerb, storm
  water, spot level and landscape; only passing mentions were found.
- In cad2bim (read from `CLAUDE.md` and `docs/`): the CAD engine reads DXF/DWG in the browser (Rust to
  WebAssembly), the CH template defines layers and labels, the pipeline writes IFC, and the viewer, QA, BOQ
  and BBS already exist. The CH template is **structural-concrete only**; external works are not in it, so
  this would be a **new, separate opt-in layer set** (for example a "CH-Site" family), never a change to the
  v1 or v2 readers (`CLAUDE.md` forbids changing v1's `Entity` or `analyze` for v2, and the same care applies
  here).
- In the market (not exhaustively checked): civil design tools such as Civil 3D, 12d and similar do grading,
  corridors and storm networks, but they are **paid and heavy**, and are a civil engineer's tool, not a
  building BIM modeller's quick pass. Revit has toposurface and site tools but no automatic rule-based layout
  of roads, drains or compound wall. A free, browser-based, no-upload tool that turns a survey sheet and a
  plot line into a checked 3D site and its BOQ for building teams was not found. Treat this as unverified.

**The idea.** A **Site** workspace in the cad2bim web app. Nothing leaves the device.

1. **Bring in the site.** Three inputs, any of them alone:
   - the **plot boundary** (a closed polyline on a named layer, or traced from the CH template's plot line);
   - **survey levels**: a CSV/Excel with `x, y, z` (and optional code such as `GL`, `ROAD`, `DRAIN`), or the
     spot-height texts and points of a DXF;
   - the **building footprint and plinth level** taken from the open model, so the building and the site are
     one scene.
2. **Ground surface.** Build a triangulated terrain from the spot levels, with the plot boundary as the edge.
   Show contours (1.0 m, 0.5 m, 0.25 m), a slope colour map (flat, gentle, steep) and a high/low point list.
   Spot levels that disagree with their neighbours by more than a set amount are flagged as likely typing
   errors ("102.35 beside 120.35") before they make a bad surface.
3. **Draw the roads with a few clicks.** The person sketches a road centre-line (or accepts an auto route from
   the gate to the building entrance and the fire-tender approach). Choose a type: width, kerb, camber (for
   example 2 percent crossfall), footpath, layers (subgrade, WBM, PCC, tile or paver). The tool builds the
   road as 3D elements that follow the ground, with kerb stones and dished kerb at gates and a bend radius
   check for the fire tender (a rule value to be entered by the user, from the local code; the tool does not
   guess it).
4. **Storm drains that actually fall.** The tool proposes drain runs along kerbs and low points, sized by
   area and rainfall (user-entered rational-method inputs), with a start and end invert level for every
   segment, chambers at turns and at a set spacing, and a **fall check**: every segment must fall by at least
   the minimum gradient; any run that rises, flattens or ends above its outfall is red, with the exact
   segment named and a suggested fix (deepen the start chamber by 150 mm, or re-route). Outfall to the
   municipal drain or a recharge pit is a choice, not an assumption.
5. **Paving, parking surface and landscape zones.** Fill an area with paver, PCC or grass; set slopes to
   direct water to gullies; mark tree positions from a species list (canopy size shown so trees do not hit
   buildings or overhead lines) and landscape beds. Open space and tree counts are counted for approval
   forms where a rule needs them (the rule values are entered by the user; the tool does not state a legal
   minimum).
6. **Compound wall and gates.** Pick a style: wall height, pillar spacing, plinth beam, expansion-joint
   spacing (an input), with or without railing. The tool follows the plot boundary, adds pillars at corners
   and gates, cuts the wall at gates, places gate leaves with swing arcs, and runs a **swing and clearance
   check** (leaf against pillar, against a parked car line, against the footpath). Each gate gets an
   ID for the joinery and the BOQ.
7. **Service pits.** UG water tank, sump, septic tank or STP, rainwater harvesting pit, transformer pad and
   DG pad can be dropped as boxes with the right clearances (user-entered setbacks), checked against the
   plot line, the building and the tree canopy, and given their earthwork volume. (Their structural design is
   a different idea in this folder: Tank Smith.)
8. **Cut and fill.** Compare the existing ground to the proposed surface and give cut and fill volumes by
   zone, the net balance (how much soil leaves or arrives), and a haul summary to feed Dig Deep's truck
   numbers. A quick "balance the site" slider moves the formation level and shows the effect at once.
9. **Plinth and entry checks.** Compare the building plinth level to the nearest road crown and the highest
   flood or water-logging mark the person gives. A plinth lower than the road is a high-priority finding.
10. **Rain-flow replay (the wow).** A button releases rain on the 3D site: blue streaks run downhill over the
    ground, roads and paving, gather in the drains, and either reach the outfall (green) or pool (red).
    Ponds are named and sized. The same replay can be saved as a 10-second clip for a post or a client
    meeting. The model is simplified and says so on the screen; it is a visual check, not a hydraulic study.
11. **Outputs.** (a) IFC with the site as `IfcGeographicElement` / `IfcCivilElement` / `IfcWall` / `IfcPipe` and
    sensible property sets; (b) an **external-works BOQ** (road area by layer, kerb length, paver area, drain
    length by depth band, chambers, wall running metre and pillars, gates, trees, earthwork cut and fill) with
    the usual "click a line to see it in 3D" proof; (c) a clean **site plan DXF** with levels, falls, drain
    arrows and gate schedule; (d) a one-page **site checks report** (what drains, what does not).

**Why it will be popular and shareable.** The clip: drop a survey sheet and a plot line, and a terrain, road,
wall, gate and drains appear; press Rain, and the water finds the drains, with one pond glowing red where a
level is wrong. The same page then prints the external-works BOQ. Suggested post: "Survey sheet in. Roads,
drains, compound wall and BOQ out. Then I let it rain." It ties to the trend of simple AI-assisted site and
infrastructure workflows and of "digital twin" demos, but this is not a claim of what is trending today: the
news and reels were not checked in this session (no web access to LinkedIn or Instagram).

### Implementation

- **Where it fits.** `packages/cad-engine` (read plot boundary, spot-height texts and points from DXF/DWG;
  the CSV/Excel path needs only the web side); a **new package, `@cad2bim/site-engine`**, with its own version
  starting at 0.1.0 (terrain triangulation, contours, slope, road and drain generators, cut/fill, checks) so
  it does not grow the IFC engine or the pipeline; `packages/engine` (viewer: terrain, extrusions along a
  path, streak animation, the IFC writer for the site classes); `apps/web` (Site workspace, tool palette,
  results panels, BOQ). The `cad` branch's rule applies: reading by the CAD core's Rust crates, drawing by
  cad2bim's own three.js, UI from the design system, no GPL code.
- **Terrain.** Delaunay triangulation of the spot levels inside the plot boundary (plain computational
  geometry; write it or use a permissive-licence library after checking its licence against `CLAUDE.md`).
  Add the boundary as constraint edges and break-lines for road edges and drains so the surface honours them.
  Contours by marching over the triangles. Large sites (100,000 spot levels) are thinned by a tolerance, and
  the result shows how much was thinned.
- **Road and drain generators.** A path is a polyline with a profile (cross-section as a small list of
  offsets and levels) swept along it, draped on the terrain or set by a design profile; the cross-section is
  editable in a small table, not code. Drain runs are a graph (nodes: chambers and outfalls; edges:
  segments with invert levels); the fall check is a graph walk from each node downstream, which is easy to
  test.
- **Rules as data.** Every number that comes from a code, a bylaw or a municipal standard (minimum gradient,
  road widths, tree rules, setbacks, fire-tender turn radius) lives in a user-editable **site rules file**
  (JSON), shipped with *placeholders marked "set by you"*, never silently built in. The report prints the
  rule values used, so a reviewer can see what was assumed.
- **Rain-flow.** Steepest-descent flow on the triangulated surface plus the drain graph; animation is a
  particle trail along the computed flow lines (no fluid physics). Ponds are local minima not connected to an
  outfall, with an area and a depth estimate.
- **Cut/fill.** Prismoidal volumes between two triangulated surfaces on a common grid or by triangle
  prisms; report the grid size and the method used.
- **Outputs.** Reuse the BOQ panel and the "click a quantity to see it in 3D" idea (Measure Right, Trace
  Lens, 2026-11-03); the IFC writer already exists for the building and is extended with the site classes;
  the DXF site plan reuses the clean-drawing writer ideas of Drawing Autopilot (2026-10-15).
- **Tests.** (1) A synthetic tilted plane gives exact slope, contour spacing and a cut/fill that matches the
  hand calculation. (2) A drain with one uphill segment is flagged on that segment only. (3) A pond is found
  at a built-in dent and gone after the dent is filled. (4) A gate leaf overlapping a pillar fails the swing
  check. (5) A typing error in one spot level (120.35 for 102.35) is flagged. (6) The IFC opens in two other
  viewers and keeps the classes. (7) Round-trip of the rules file. Real survey files and drawings stay in a
  scratch folder, never in git.
- **Measure before claiming.** Time a real modeller on two real plots (one small residential, one large
  industrial) by hand and with the tool, and record: minutes, number of manual corrections, and quantity
  differences against the hand BOQ. Publish those, not guesses.
- **Risks.** (1) Survey data is often wrong or in mixed units and datums: the import shows a table to confirm
  units and the origin before building. (2) Wrong rule values produce wrong checks: placeholders must be
  filled, the report prints them, and there is a clear "not a design; review by a qualified civil engineer"
  line. (3) The rain replay can be mistaken for a hydraulic design: it is labelled a visual check. (4) Large
  terrains and long roads can slow the viewer: level of detail and a thinning step. (5) Local municipal
  drawings differ a lot; start with one city's conventions and make templates, not code, for others.
- **First 2 weeks.** CSV + plot boundary to terrain, contours, slope map and cut/fill against a flat
  formation level; a simple road sweep and a compound wall around the boundary with the BOQ lines for both.
  Drains, gates, trees, rain replay and the DXF site plan come after. Record time and quantity differences
  from the two pilot plots.

### Future update

- **Link to Plot Pilot**: the massing option's footprint and setbacks flow straight into the site, so a change
  of building shape updates the road, wall and BOQ.
- **Link to Dig Deep and Crane Canvas**: site cut/fill feeds the truck and haul numbers; temporary roads,
  crane pads and the storage yard are drawn on the same terrain, and the 4D shows the site being formed.
- **Link to Rain Run and Tank Smith**: terrace and podium outlets connect to the site drain graph; tank pits
  from Tank Smith sit on the site with their pipes.
- **Link to Sun Compass and Sun Roof**: shadows of trees and the compound wall on solar panels and gardens.
- **Phone or drone survey** (later): a photo-based terrain from a drone flight, or a total-station CSV, in
  place of a hand survey sheet; the as-built surface compared with the design (ties to Scan Check and Set-Out
  Pack).
- **Utility corridors**: water, sewer, power and fibre lines with clearance checks between them.
- **Multi-plot layouts**: a township or layout with many plots, internal roads and shared services, with a
  plot-wise yield table.
- **Client "walk the site" reel**: a drone-style flythrough of the finished site with trees and lighting, in
  the same spirit as Model Reel (2026-10-18).
