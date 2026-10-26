# 20261026 Idea: Load Path X-ray — press one button and watch the load flow from every slab, through beams and columns, down to the footings: broken paths, floating columns and hidden transfer beams light up red, a quick load take-down sizes-checks every column and footing, and the numbers are cross-checked against ETABS

## Entry 1

```
date-time:   2026-10-26 (scheduled run)
idea name:   Load Path X-ray — "The modeller finishes the 3D model from the architect's DXF. Before it goes
             to ETABS or to the client, press X-ray. The building turns to dark glass. Blue light starts
             on every slab, runs to the beams, drips down the columns floor by floor and pools in the
             footings — brighter where the load is bigger. Three columns glow red: C14 stops on a beam
             at Level 2 (a hidden transfer the architect never mentioned), C22 lands on nothing at the
             podium (the footing was never modelled), and B7 on Level 5 spans onto a wall that is not
             there in the floor below. A side panel lists them with the fix. Column C9 at the ground floor
             carries about 3,100 kN factored on a 230 x 600 section: axial ratio 0.90, red. Footing F4 needs
             about 11.4 m² on 200 kN/m² soil and is drawn at 4 m². All of this in 20 seconds, in the
             browser, before anyone opens ETABS."
             One engine with four front ends:
               (1) cad2bim web app (IFC viewer and the CAD workspace's DXF -> 3D result): a new
                   "Load path" mode in the viewer and a "Load path" dock panel next to QA,
               (2) pyRevit button (AnonGee tab, Advance panel): run it on the live Revit model, colour
                   the elements with view overrides, write the take-down numbers into shared parameters
                   (LP_Pu_kN, LP_AxialRatio, LP_FootingArea_m2, LP_Status) for schedules and tags,
               (3) C2B utility 6 ("Cross-check: supports, continuity" — listed as "next" in C2B's
                   README): the same graph built from the normalised JSON, so a client DXF is checked
                   before any Revit model exists,
               (4) etabs-mcp / Structura / PlanWin AI Pro: compare the quick take-down with the real
                   column axial forces and base reactions from ETABS (or PlanWin's load take-down) and
                   flag any element where the two differ by more than a set % — the "did we model the
                   same building?" check.
             Outputs: an animated load-flow view (and a 20-second "X-ray reel" video), a list of broken
             or suspicious paths with element marks and the fix, a column load take-down table (per
             floor, cumulative, axial ratio), a footing size check table (required area vs modelled),
             a colour-by-load view for Revit, a BCF file with one topic per problem, and a one-page PDF.
potential:   Very high. Every RCC building the team models has a load path, and almost every bad model
             has a broken one somewhere: a column that does not continue, a beam resting on air, a
             footing missing under a column, a floating column on a beam that nobody flagged as a
             transfer. Today these are found late — when ETABS throws instability warnings, when the
             structural engineer reviews, or on site.
             Time today (one G+7 to G+15 residential building, measured by the way the office works,
             not timed with a stopwatch — estimates):
               - checking column continuity floor by floor by toggling levels in Revit / Navisworks:
                 2-4 hours, and still misses some,
               - finding the cause of ETABS "unstable / ill-conditioned" warnings after export:
                 0.5-2 days of back and forth between modeller and engineer,
               - a hand load take-down in Excel to sanity-check column sizes before analysis:
                 0.5-1 day (often skipped),
               - footing sizes checked only after the ETABS run: a redesign round if they overlap
                 or do not fit the plot: 0.5-1 day.
             With Load Path X-ray: about 20-60 seconds per model run, re-run after every fix.
             Expected saving: 1-3 days per building at the modelling -> analysis hand-over, and fewer
             wrong models reaching ETABS and the client. For a BIM team doing 4-8 buildings a month
             that is roughly 1-3 modeller-weeks a month (estimate).
             Wow effect: the building goes dark, then the load literally flows — glowing particles move
             along beams and drip down columns, gathering at the base like water. Thickness and
             brightness follow the load. A broken path is a red spark where the flow stops in mid-air.
             Click a footing and the "tree" of everything it carries lights up up to the roof; click a
             slab panel and you see exactly which columns it ends up in. That animation is exactly
             the kind of "load path explained" content that does very well on LinkedIn and Instagram
             (structural engineers share hand-drawn and animated load path posts all the time; BIM +
             "AI checks my model" posts are trending) — and here it comes from the team's own model,
             in one click.
             Market: ETABS, SAFE, RAM Structural System, Tekla Structural Designer, Robot, STAAD
             (all paid) compute loads but only after a full analytical model is built, and none shows a
             one-click animated load path on the BIM model. Revit's analytical model can show
             disconnected analytical members (Revit 2023+ analytical automation) but has no load
             take-down, no footing check and no animation. Solibri (paid) can check "column on column"
             rules but not loads. Navisworks clash detection does not know load paths. To our
             knowledge nothing free takes an IFC / DXF / Revit model -> support graph -> broken path
             finder -> quick take-down -> column and footing check -> ETABS cross-check in one tool.
```

### Description

**The pain (daily life of a BIM modeller).**
- The model is built from an architect's plan and a structural drawing that do not always agree. Columns get
  shifted to suit rooms, some stop at a podium, some start on a beam. Nobody writes this down.
- The modeller cannot easily see if every slab, beam and column is actually carried down to a footing. Revit
  shows geometry, not "what rests on what".
- Errors surface late: ETABS warnings after export, the structural engineer's red marks, or a site engineer
  asking "what is this column standing on?".
- Before analysis, a quick hand load take-down would catch undersized columns and footings that cannot fit,
  but it takes half a day in Excel, so it is skipped.

**What Load Path X-ray does.** It turns the model into a *support graph*: every element is a node and an edge
means "this element is carried by that one" (slab -> beam or wall or column, beam -> column or wall or beam,
column -> column or beam or footing, wall -> wall or beam or footing, footing -> soil). The graph is built from
geometry (contact and overlap within tolerances), not from Revit's analytical model, so it works on IFC, on the
DXF -> 3D result and on Revit alike. Then it:

1. finds every node with no path to soil (broken path),
2. finds suspicious paths (column on a beam = transfer; beam on a beam on a beam = long chain; cantilever
   longer than a set limit; column offset from the one below by more than a tolerance),
3. pushes a simple load (self-weight from the element's volume + floor finish + wall load + live load, all from
   an editable table) through the graph from top to bottom, splitting slab loads by tributary area (yield-line
   style 45° split for two-way panels, half-half for one-way), and
4. checks each column (factored axial load vs. section and grade: axial ratio Pu / (fck · Ac) against an
   office limit, default 0.4, editable) and each footing (service load + self-weight allowance / SBC vs. its
   modelled plan area).

Every number says what it is: "quick take-down estimate, not a design". It is a modelling QA and a sanity
check before analysis; the structural engineer's ETABS model stays the design.

**Daily use cases.**
1. *Before export to ETABS*: run X-ray, fix the red items, then export. ETABS instability warnings drop.
2. *DXF -> 3D from a client drawing*: the cad2bim pipeline or C2B output is X-rayed at once, so missing columns
   or footings in the client's drawing are reported back to the client the same day as an RFI.
3. *Architect moves a column*: re-run, and see if it now lands on a beam (transfer) and how much load that beam
   suddenly carries.
4. *Column size sanity before analysis*: the take-down table shows which ground-floor columns are likely to fail
   the axial ratio, so the engineer starts ETABS with realistic sizes.
5. *Footing layout check*: required footing areas drawn as ghost squares on the foundation plan show at once
   where footings overlap (go combined or raft) or cross the plot line.
6. *ETABS cross-check*: after analysis, pull column forces and base reactions from etabs-mcp; any column where
   ETABS differs from the take-down by more than, say, 25 % is flagged: usually a missing load, a wrong level
   or a modelling difference between Revit and ETABS.
7. *Client and site explanation*: the X-ray reel explains a transfer beam or a floating column to a client or a
   site team in 20 seconds.

### Implementation

**1. Read the model (no new readers).**
- cad2bim: IFC elements through `@cad2bim/engine` (category, level, bounds, solid geometry, mark, grade, volume,
  as the QA checks already use), and the Rust pipeline's DXF -> 3D output for drawings.
- Revit: pyRevit collects structural columns, framing, floors, walls, foundations with their solids and
  levels.
- C2B: normalised JSON (column stacks, beam spans between supports, slab panels, footings).

**2. Build the support graph.**
- Spatial index (BVH / grid on element bounding boxes) so a 2,000-element model is checked in well under a
  second in the browser (estimate; to be measured on a real model).
- Contact rules with named tolerances (shown in each finding, like the QA panel does):
  column bottom face within 25 mm of the top face of a column / beam / footing / wall below; beam end within
  its bearing zone of a column, wall or another beam; slab edge resting on a beam or wall top, or a slab
  directly on columns (flat slab, flagged as such).
- Edge weights for load sharing: slab -> supports by tributary polygons (45° split for two-way panels when the
  aspect ratio < 2, else one-way); beam -> its two supports by reactions of a simply supported span (point
  loads from supported beams included). Simple on purpose: it is a check, not an analysis.

**3. Findings.**
- *Broken path* (error): element with no route to a footing / soil.
- *Floating column / transfer* (warning): column carried by a beam; reports the beam, the load it receives and
  the beam's span and depth.
- *Offset column* (warning): column centre moved from the one below by more than a tolerance (default 50 mm).
- *Beam on beam chain* (note): more than two levels of beam-on-beam support.
- *Long cantilever* (warning): above an office limit (default 1.5 m for beams, 1.2 m for slabs — editable).
- *Column axial ratio* (error above limit, warning above 0.8 × limit).
- *Footing area short* (error) and *footings overlap / cross plot line* (warning).
- Each finding is a card with the element, the number, the tolerance, a "zoom to" button and a proposed fix
  ("extend C22 to a new footing", "flag B11 as transfer for the engineer"). BCF export for Revit / BIMcollab.

**4. Load table (data, not code).**
- One editable sheet: concrete unit weight, floor finish, wall load per m (by wall thickness, from the model's
  walls if present or a default per floor), live load by occupancy (residential, corridor, stair, terrace),
  load factors, number of floors carrying live load, SBC and footing self-weight allowance. Values filled with
  the office's usual defaults and the IS 875 (Part 2) occupancy values entered from the published code by the
  engineer before it ships (not typed from memory).

**5. The animation.**
- three.js: dim the model to glass, draw each graph edge as a tube, run GPU particles along edges from top
  floors down; particle density and tube width scale with cumulative load. Broken path ends get a red pulse;
  transfers get an amber ring. "Click to trace": up-tree (what this element carries) or down-tree (where its
  load goes).
- Recording: reuse the Model Reel idea's renderer to export a 20-second MP4 / WebM.

**6. Cross-check with ETABS.**
- etabs-mcp already exposes analysis results; pull column axial forces per story and joint reactions, match
  ETABS objects to model elements by label / mark and position, show the % difference as a heatmap.

**Tech stack.** TypeScript core in a Web Worker (graph, take-down, findings) inside `@cad2bim/engine`'s QA
layer, so it runs offline in the browser and nothing leaves the device; the same algorithm as a small Python
module for pyRevit and C2B (shared test cases in JSON so the two stay in step). three.js for the animation
(already used). No new outside services.

**MVP (3 weeks).**
1. Support graph from IFC (columns, beams, slabs, footings), broken path + floating column + offset column
   findings, cards in a "Load path" panel.
2. Quick take-down with a default load table; column axial ratio and footing area tables (Excel export).
3. The glowing flow animation with click-to-trace.
4. Test on three past office models (one with a known transfer, one with a known missing footing) and record
   the reel.

### Future update

- **Lateral path:** a second mode for wind / seismic: diaphragm -> shear walls / frames -> foundation, with the
  in-plane discontinuity check from cad2bim's QA design, and soft / weak storey hints.
- **Live in Revit:** re-run on every sync, with a small "load path health" badge on the ribbon (links to the
  Warning Doctor idea's health score).
- **Auto-fix proposals:** "extend column to footing", "add footing of size X", "mark beam as transfer and
  increase depth to span / 10" previewed as ghosts, applied as one undo step.
- **Footing auto-layout:** from the take-down, propose isolated / combined / raft footings and hand them to the
  Dig Deep idea for excavation quantities.
- **Construction stage path:** with Pour Planner's 4D sequence, show the load path at each stage (props and
  shuttering carrying the green slab), linking to Shutter Plan's prop layout.
- **AI explain:** ask "why is C14 red?" and the assistant answers from the graph with the chain of elements and
  numbers.
- **Retrofit mode:** for existing buildings (scan or old drawings), add a floor on top and see which columns and
  footings go red — a common Indian brief ("can we add one more floor?").

### Improvement suggestions for existing tools in the repos

- **cad2bim:** add a `support` relation to the engine's model (who rests on whom), computed once and reused by QA,
  BOQ (transfer beams), the viewer's isolate tools and this idea; the QA card's "starts 450 mm above its support"
  check already computes half of it.
- **C2B:** utility 6 "Cross-check: supports, continuity" can be this graph — same findings, same tolerances,
  written into the Excel review workbook as a new "Load path" sheet.
- **pyRevit-addons (AnonGee):** a generic "colour by parameter" view filter helper (gradient legend) so any tool
  that writes a number (load, axial ratio, cost) can show it as a heatmap in Revit.
- **etabs-mcp:** an MCP tool "column axial forces by story with labels" and "base reactions by joint with
  coordinates", so cross-checks need one call each.
- **Structura:** a "flow" display mode (animated particles along members) is worth reusing for load paths,
  4D sequences and even MEP flow.
- **PlanWin AI Pro:** its slab -> beam -> column take-down is the same idea in a different place; exporting it as
  JSON would let X-ray compare PlanWin and ETABS too.
