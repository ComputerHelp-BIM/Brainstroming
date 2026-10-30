# 20261030 Idea: Column Compass — from the architect's plan to three ready structural column + beam layouts, scored on cost, parking and headroom, and dropped into the model as a frame in one click

## Entry 3

```
date-time:   2026-10-30 (scheduled run, UTC)
idea name:   Column Compass (Architect Plan -> 3 Structural Layout Options -> IFC Frame Start).
             "Every project starts with the same blank-page hour: where do the columns go?
              Let the tool draw three honest answers, and show what each one costs."
potential:   High. Every RCC building starts with the structural engineer or senior modeller marking
             columns on the architect's plan by hand, over several rounds. It takes hours to days, it is done
             with a pencil, a grid in the head and the fear of a column in a parking bay or a bedroom door.
             The result decides steel and concrete quantity, headroom, carpet area and parking count for the
             whole life of the job, yet the first layout is rarely compared with a second one because
             there is no time. A tool that draws three valid options in a minute, with kg/m2 and m3/m2 beside
             each, moves that decision from "first idea wins" to "best of three".
             Estimated saving (an estimate, not measured): 3-8 hours per building at concept stage, plus fewer
             column-shift rounds with the architect later (each round is typically a day of re-modelling).
```

### Description

**The pain (daily life of a modeller / structural engineer).**
The architect sends a plan (DWG, DXF or PDF). The first structural job is the column layout. The engineer tries
a grid, finds a column in the middle of a car bay, nudges it, finds it now hits a door, nudges again, then
checks that columns line up with the floor above and the basement below. Then the beam layout follows the
columns. Each shift means the architect redraws and the modeller rebuilds. The effort is mostly *geometric
search under rules*, which is exactly what a computer does well, and exactly what is repeated on every project.

**Why now (trending).** The AEC feeds on LinkedIn and Instagram are full of *AI-generated layouts*,
*generative design*, *text-to-BIM* and *AI structural engineer* posts. Most are demos of floor-plan generation
(architecture side). The structural side, "plan in, columns out, with numbers", gets far fewer real tools. A
before/after reel (bare plan, then columns popping in, then a cost bar) is the type of post that travels.

**The idea.** In the browser, open the architect's plan (the DXF/DWG reader and the plan-tracing pipeline in
cad2bim already turn lines into walls, doors, windows, cores and parking bays). Press **Column Compass**. It
returns **three layout options** (for example *Economy*, *Parking-friendly*, *Flush-with-walls*), each with a
grid, columns, primary and secondary beams, and a one-line scorecard.

1. **Read the constraints from the plan (not typed by hand).**
   - Walls (to hide columns inside), door and window openings (no column in an opening), lift and stair cores
     (candidate shear-wall positions), shafts, parking bays and drive aisles (keep columns on bay lines), and the
     outer boundary and setbacks.
   - Optional: the basement or stilt plan (so columns stack), and a prior floor (so columns continue).
2. **Rules, in a short editable table (a "rule pack").** Preferred span range (for example 4.0-6.5 m for
   slab-beam, up to 8 m with deeper beams), maximum slab panel aspect ratio for two-way action, minimum column
   size from the storey count, cantilever limit, edge-column rule (inside the wall thickness), torsion balance
   (centre of stiffness near centre of mass), and clearance for car bays. Every rule has a "hard / prefer / off"
   switch. The default pack is for Indian low and mid-rise RCC; a team can keep its own.
3. **Solver.** A grid-and-snap search: propose a small number of candidate grids from the walls and bays,
   place columns at wall intersections and bay lines, remove those that hit openings, add columns where a
   span or cantilever limit is broken, and score the result. Three different weightings give three options
   (cost first, parking first, wall-flush first). This is a deterministic constraint search with a score, not a
   black box, so the same input gives the same answer and every choice has a reason.
4. **Stacking through floors.** Run on the typical floor, then check the floor above and below: columns that do not
   continue are shown as red "transfer" flags with a count, and the solver tries to remove them. A tower with a
   lift core gets shear-wall suggestions at the core, with a simple stiffness-balance indicator.
5. **Beams and slab panels.** Primary beams along column lines, secondary beams only where the slab panel would
   exceed the limit, slab thickness from span/depth ratio (IS 456 style, editable). Beam depths come from span,
   and a **headroom check** compares each beam soffit with the door and window head heights (the Flush Fit
   idea checks this after modelling; Column Compass avoids it at the start).
6. **Scorecard per option, live.** Concrete m3/m2, steel kg/m2 (using a typical ratio table the user can edit,
   clearly marked as an *estimate* until detailing is done), formwork m2/m2, number of columns, number of
   unique column sizes, parking bays kept, column area lost from carpet area, and number of transfer flags.
   A slider "cost weight vs parking weight" re-ranks the options live.
7. **Pick, tweak, hand off.** Click an option, drag a column and watch spans, headroom and the score update;
   press **Make frame** to write a clean IFC structural frame (grids, columns, beams, slabs as placeholders with
   sizes and marks) and a short rationale sheet for the architect: "columns moved to the wall line at C4, C7 to
   keep bays 14-18 clear".
8. **Architect round-trip.** Export a transparent overlay (DXF with columns on their own layer, plus a PDF) so the
   architect sees the proposal on their own plan and can reply with markups (the Markup Mirror idea reads them
   back). The same file can be given to ETABS/STAAD users as a starting grid.

**The wow moment (a 30-second reel).** A flat architect plan, grey. A tap. Columns pop in along the walls like
pins on a map, red flags flicker and clear, beams draw between them. Three thumbnails appear (Economy 3.9 t,
Parking +6 bays, Flush) with bars for steel and concrete. The viewer drags a slider and the ranking reshuffles.
A final tap turns the winner into a 3D frame that orbits. A block of work that normally fills a morning is
shown start to finish in half a minute, and the numbers on screen (kg/m2, bays kept) are what a client or
senior engineer actually asks.

**Why this is unique / extends what is already here.**
- It uses what cad2bim already does: DXF/DWG reading, plan-to-3D understanding, IFC output, quantity (BOQ)
  and QA engines. The new part is the *structural layout solver and the scorecard*.
- It fills a gap between existing ideas without repeating them: *Plot Pilot* makes massing and unit options,
  *Park Pilot* lays out bays inside a given column grid, *Flush Fit* checks columns and beams against the
  architecture after both exist, *Tower Stacker* repeats a typical floor, *Load Path X-Ray* checks a layout
  that is already modelled. Column Compass is the step in between: it proposes the first grid that those
  other tools then refine. Park Pilot and Column Compass can also run as a loop (columns on bay lines, bays
  around columns) to find the best pair.
- Market check (a search at the time of writing, not an audit): research papers and a few start-ups
  describe automated structural layout, generative-design platforms (for example Autodesk's tools) can be
  scripted to explore layouts but need a paid licence and set-up work, and most structural packages (ETABS,
  STAAD) analyse a layout the engineer has already drawn. I did not find a free, browser-based tool that reads
  a plan and returns scored column/beam options for Indian-style RCC. This should be re-checked before launch.
- Everything is computed on the device and no file is uploaded, which matches how cad2bim treats drawings.

**Important honesty rule (also a feature).** The output is a *concept layout for the engineer to confirm*, not
a design. The tool says so on every screen and in every export ("Concept layout, not for construction. Member
sizes are estimates until analysis in ETABS/STAAD"). It gives reasons, never hides a rule it broke, and the
engineer always has the last word. The kg/m2 numbers are from a typical-ratio table and are labelled as estimates.

**Day-to-day use cases.**
- Day 1 of a new job: three options before the first client call, with numbers, instead of one option after two days.
- Architect changes the plan (a bay moves, a shaft grows): re-run, see which columns changed and why.
- Value engineering: "same plan, fewer unique column sizes", drop the count from 14 sizes to 6 and see the steel effect.
- Tender stage: a defensible quantity estimate from the plan alone, before the full design exists.
- Training juniors: they see *why* each column sits where it does, from the rule list, rather than copying a drawing.

### Implementation

Browser only, in a worker; files stay on the device. cad2bim was read for reference; nothing there was changed.

1. **Input adapter (reuse):** the CAD core's reading gives lines, polylines, blocks and layers; the existing
   plan-understanding pipeline classifies walls, openings, cores and bays. Output of the adapter: a small
   `PlanContext` (polygons for walls, openings, cores, bays, boundary, level heights). A manual "fix" step in the
   viewer lets the user reassign any polygon (this absorbs bad drawings).
2. **Rule pack (JSON):** spans, aspect ratios, minimum sizes, cantilever limits, edge-column rules, bay
   clearances, weights per option, and the size/ratio tables. Versioned; the pack name is written into every export.
3. **Solver (Rust to WASM, or TypeScript to start):**
   - candidate grid generation from wall and bay line clustering;
   - greedy column placement at line intersections with openings removed;
   - repair loop: add columns until span/cantilever limits pass;
   - local search (move a column along a wall, merge two close columns) with the weighted score;
   - run 3 weightings, keep distinct results (reject near-duplicates by a layout distance).
   Time budget: aim for under 5 s for a 1,500 m2 floor in the browser (a target to measure, not a claim).
4. **Stacking check:** run the floors in order with the previous result as a soft constraint; report transfer
   columns and shear-wall candidates at cores. Keep the stiffness-balance indicator as a simple centroid
   comparison (centre of rigidity vs centre of mass), labelled as indicative.
5. **Beams, slabs and headroom:** beam grid from the columns, span/depth ratios from the rule pack, soffit levels
   vs opening heads from the plan context; flag clashes and offer "deepen / flatten the beam / move column".
6. **Scorecard:** quantities from the existing BOQ engine on the generated placeholder frame, steel from the
   ratio table; every number carries a tag *measured from geometry* or *estimated from ratio*.
7. **Output:** IFC frame (`IfcColumn`, `IfcBeam`, `IfcSlab`, `IfcGrid`) with Pset marks and a
   `Pset_CadToBim_ConceptLayout` carrying the option name, rule-pack version and "concept" status; overlay DXF
   and a two-page PDF rationale sheet; an Excel scorecard. Optional ETABS grid export as a text/CSV of coordinates.
8. **UI (React, design system tokens only):** a dock panel with three option cards, a slider, a rule list with
   switches, and a "Why here?" popover on any column that shows the rules that placed it. A Python/MCP tool
   `propose_layout(plan, rules)` lets the AI agent run it from chat ("give me three layouts for this plan").
9. **Tests:** hand-made plans with known good answers (a rectangular block, an L-shape, a plan with a core and
   a basement), checks that no column sits in an opening, every span is within the limit, and that results are
   repeatable. Compare steel/concrete scorecards against a few real past projects the team already has
   (outside git) and report the error honestly. Keep drawings out of git.

### Future update

- **Close the loop with analysis:** send the chosen frame to a lightweight in-browser frame check (drift and
  period indicators) and later to ETABS, then bring back column sizes; Column Compass then feeds *Detail Drop*.
- **Steel and composite options:** the same solver with steel-grid rules for *Bolt Up*.
- **Learn from the team's own projects:** read past models, learn the team's preferred spans and column sizes
  as default rule weights ("our office likes 5.4 m").
- **Joint solver with Park Pilot:** one run that co-optimises bays and columns, and with *Plot Pilot* so a massing
  option arrives with a structure already attached.
- **Voice/chat:** "keep the lobby column-free and push everything to the east wall" turned into rule changes by
  the AI agent; the solver still does the geometry.
- **Instagram/LinkedIn kit:** a built-in "Make reel" button (as in *Model Reel*) that records the pop-in sequence
  and scorecard as a short video.
- **Retrofit mode:** read an existing building's columns (from *Scan Check* or *Plan Tracer*) and suggest where
  new openings or a new core can go with the least strengthening.
