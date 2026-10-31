# 20261031 Idea: Gap Guard — expansion joints and seismic gaps planned, checked and detailed in the model: where the joint must go, how wide the gap must be, and whether the neighbour (or the plot boundary) will hit your building in an earthquake

## Entry 3

```
date-time:   2026-10-31 (scheduled run, UTC)
idea name:   Gap Guard (Expansion Joint + Seismic Gap Planner and Pounding Checker)
potential:   High. Every long RCC building, every podium with towers, every L/U/T-shaped plan and every plot
             with a neighbour at the boundary has the same question: "where does the joint go, how wide is it,
             and what happens to the columns and beams on either side?". Today the answer lives in a rule of
             thumb, a clause in IS 456 / IS 1893, a call to the structural consultant and a hand-drawn line
             on the plan. It is decided late, it changes the column grid, the beam layout, the waterproofing,
             the lift/stair cores and the boundary setback, and when it is missed the fix is expensive (a
             "double column" retrofit on a built frame, or a cracked parapet at the first summer).
             The checks are small and rule-based, the data is already in the model (lengths, levels, grids,
             storey drifts from ETABS), and I found no tool that does the whole loop: find the joint lines,
             size the gap, frame both sides, draw the cover detail, count it in the BOQ.
             Estimated saving (an estimate, not measured): 2-4 hours per building at concept stage and, more
             importantly, one avoided late redesign per project.
```

### Description

**The pain (daily life of a modeller / structural engineer).**
The architect's plan is a 78 m long slab-and-beam block with a podium. Somebody says "joint at the middle".
The modeller draws a line on the plan, then finds that the line cuts through a lift shaft, a stair, a swimming
pool and a column row at the same time. The structural engineer says "double columns each side, 40 mm gap";
the modeller adds 14 columns by hand, moves 9 beams, and the slab edge now needs a cover plate that nobody
has detailed. At the boundary, the neighbour's building is 150 mm from the shared wall and the consultant
asks "is that enough in an earthquake?" — and nobody can answer in the meeting. The work repeats on every
floor and again in the drawings, schedules and BOQ.

**Why now (trending).** The 2026 LinkedIn feed is full of "talk to your model" and AI-for-compliance posts
(Claude + Revit through MCP, natural-language add-ins, automated code checks; read these as individual posts
and vendor blogs, not as measured results:
[Novatr: BIM trends 2026](https://blog.novatr.com/blog/top-bim-modeling-trends),
[QECAD: AI reshaping BIM workflows](https://www.qecad.com/cadblog/?p=6476)). Earthquake news, and the
steady stream of "pounding" and "building collapse next door" videos on Instagram, make the topic visible to
clients as well, not only to engineers. Gap Guard turns that attention into a practical check that runs on
every project.

**What exists today (checked, thinly, with two searches).**
- Joint **products** exist as Revit families on manufacturer pages: seismic and expansion joint covers for
  floors, walls and ceilings
  ([BIMobject: Emseal seismic joint](https://www.bimobject.com/da/sika/product/emseal-sjs),
  [BIMobject: Balco / inpro joint covers](https://www.bimobject.com/da/inpro/product/2397-024)).
  These are objects to place. They do not decide where a joint is needed or check a gap.
- I did **not** find a Revit or IFC tool that (a) proposes joint lines from length and temperature rules,
  (b) checks the seismic separation between two buildings from their drifts, and (c) reframes the structure
  on both sides. This is a search result, not proof. Structural analysis packages report drift, and
  spreadsheets do the gap sum, but the joint is not tied to the model. Check again before writing "nothing
  like it exists" in any marketing.

**The idea.** In cad2bim, select a building (or two, or a building and the plot boundary) and press
**Gap Guard**. It runs three linked checks and one fix, all visible in 3D:

1. **Where does a joint have to go? (Expansion / movement joints).** The tool measures the longest
   continuous plan dimension of each structural block on every level and compares it with the project's rule
   (the default is IS 456's guidance of a joint about every 45 m of building length; the rule is an editable
   number, because designers also use temperature and shrinkage calculations). Where a block is too long, it
   proposes **joint lines**: ranked by how little they hurt. A line scores badly if it cuts through a lift
   or stair core, a shear wall, a water tank or a pool, and scores well if it runs on a grid line between
   bays, at a change of height, or where the plan already has a re-entrant corner (L, T, U shapes: the
   classic place to split). The modeller sees three options as dashed red lines through the 3D, each with a
   score and the list of what it crosses.
2. **Where does the building meet another one? (Seismic separation).** Set the neighbour: another block of the
   same project, a second model (IFC), or just "a building on the other side of the boundary, height H, at
   distance d". The tool takes the storey displacements of each side from the user's ETABS tables (the planned
   ETABS import) or from a quick hand-entered drift, and computes the required gap by the code rule for
   adjacent structures: in IS 1893 (Part 1) the separation is R times the sum of the two calculated storey
   displacements, with R replaced by R/2 where the floor levels coincide. **Check the clause and the edition
   against your copy of the code; the tool prints the rule it used, with its number, and lets the engineer
   override it.** The result is shown as a **gap ribbon** along the facade: green where the existing gap is
   enough, amber where it is within 20 %, red where the two slabs would hit. Storey by storey, so the typical
   "podium floor level vs tower floor level" mismatch (different floor heights = hits the column, not the
   slab) is flagged separately.
3. **Does the joint land on the structure sensibly? (Framing check).** For each joint line the tool lists
   the columns, beams, slabs, walls and stairs on the line and offers the usual fixes: **double columns** (a
   second row at the gap with the right spacing), **cantilever slab with a drop-in** (slab on a corbel), or
   **shifting the line to the nearest grid**. It also checks the small things that get missed: parapets,
   screed and tile finishes, plinth beams, the water line (basement and terrace), and that rebar does not run
   across the gap.
4. **Fix and draw.** One click applies the chosen fix to the model: splits the slabs and beams at the line,
   adds the second column row, sets the gap width, and places a **joint cover** family (floor, wall, roof,
   basement waterstop) from a small library. The section through the gap is drawn automatically as a detail,
   and the joint length by type goes into the BOQ (joint filler, sealant, cover plate, waterstop).

**The wow moment (a 30-second reel).** A long building with a pool of colours on the floor plan. A red line
cuts the plan: "78 m — joint needed." Three candidate lines fade in, each with a score. The modeller picks
one; the slab splits, a second column row grows out of the ground, and a gap of 60 mm opens in a smooth
animation. Then the neighbour's block is added across the boundary. Both buildings are shaken in an
animation of their drifts, exaggerated 20x, and the gap ribbon on the facade turns from green to red on the
floor where the two slabs collide. A button, **widen setback**, moves the structure back by the shortfall and
the ribbon returns to green. Everyone in the room understands the problem without a single clause number.
(Label the animation "exaggerated, schematic, not a time-history analysis".)

**Why this is unique / extends what is already here.**
- It uses things the toolkit already plans: IFC building, levels and grids (Levels and Datums design), the
  ETABS import with storey drifts (ETABS design note), the Create/Modify tools for splitting and adding
  elements (Native Editing design), and the BOQ/BBS and drawing generation. The new part is the **joint logic**
  and the **gap check**.
- It sits beside, and does not repeat, existing ideas: *Column Compass* proposes the frame layout, *Twin
  Check* compares design against BIM, *Load Path X-Ray* shows load flow, *Basement Box* handles basement
  walls and waterproofing. Gap Guard is the one that handles **separation between structures**, which none of
  them touches (the repository search found no earlier idea on expansion joints, seismic gaps or pounding).
- Free alternatives are rules of thumb and spreadsheets; paid ones are structural analysis suites that
  report drift but do not model the joint, and joint-cover families from manufacturers that only place the
  product. Gap Guard works in the browser on the file in front of you and nothing leaves the PC.

**Day-to-day use cases.**
- **Concept stage:** in two minutes, how many joints does this plan need, and which option disturbs the
  layout least? Useful in the first meeting with the architect.
- **Podium + tower:** compare floor levels and drifts and decide the gap before the column grid is frozen.
- **Plot boundary:** "can I build up to the boundary?" for a neighbour at distance d, height H.
- **Brownfield extension:** a new block next to an existing one, using the existing building's heights and a
  drift estimate; the gap needed is the headline number for the client.
- **Drawing set:** the gap detail and the joint schedule appear in the sheets without drawing them by hand.
- **Review:** before issue, the checker lists every joint line, the gap, and the code rule used, as a one-page
  report for the structural consultant's sign-off.

### Implementation

Browser only; the work runs in a worker and nothing is uploaded. cad2bim was read for reference; nothing there
was changed.

1. **Inputs (existing or planned):** IFC structure and levels (`packages/engine`); grids and levels as datums
   (`docs/design/datums-and-constraints.md`); storey displacements and drifts from the ETABS tables
   (`docs/design/etabs.md`) or hand-entered; the plot boundary from the site plan (DXF → 3D or a drawn
   polyline); a neighbour as another IFC or a simple box (height, distance, floor levels).
2. **Block finder (new, small):** union the structural slabs per level into a polygon, take the oriented
   bounding rectangle for the longest dimension, and group connected slabs into blocks. Pure geometry; unit
   tests with an L, a T, a U and a rectangle with known answers.
3. **Joint-line proposer:** candidate lines at grid lines and at re-entrant corners, cut through the block; a
   cost function counts crossed cores, walls, tanks and columns and the imbalance of the two halves (a 50/50
   split scores best, a 90/10 split worst). Returns the best three. Deterministic; no AI needed.
4. **Gap engine:** the required gap per storey from the two displacement lists and the rule table (code
   edition, R factor, coincident-floor rule, minimum practical width). Rules live in a JSON pack with the
   clause, the edition and a date, so a team can add a state or city rule. Results per storey; flags for
   mismatched floor levels.
5. **Fixer:** uses the planned Create/Modify tools: split slab and beam at the line, add the second column
   row, set the gap, place the cover family. All as one undoable step.
6. **Outputs:** the joint schedule and the gap report as PDF and Excel; a section detail through the gap as
   a DXF in the CH template (the existing CH DXF writer); BOQ lines by joint type and length. An agent tool
   `joint_check()` for the MCP helper.
7. **UI (React, the design system):** a dock panel "Gap Guard" with Joints, Gaps and Report tabs; the gap
   ribbon and the joint lines are overlays in the 3D viewer (colours from tokens only); the animation uses the
   same camera/animation layer as the planned *Camera moves and video* feature.
8. **Tests and honesty:** hand-computed cases for the gap rule (two blocks with known displacements, with and
   without coincident floors); geometry cases for the block finder; a test that the applied fix keeps the
   element count and total concrete volume unchanged except for the added columns. Report what was measured
   and what was estimated.

**Phases.**
- *Phase 1 (about 2 weeks):* block finder, joint-line proposer with scoring, 3D overlay and a PDF/Excel joint
  schedule. Already useful at concept stage and easy to demo.
- *Phase 2:* gap engine with ETABS drifts or hand-entered ones, gap ribbon on the facade, neighbour and
  plot-boundary mode, the printed rule and clause.
- *Phase 3:* one-click fix (split, double columns, cover family), the section detail in CH DXF, BOQ lines and
  the reel-ready animation.

**Honest risks.**
- This is a **screening and coordination tool, not a design.** The structural engineer of record decides the
  joint locations and the gap; the tool shows its numbers, the rule and the edition, and says so on every
  report.
- Gap size depends on code edition and on drifts that come from the user's analysis; wrong drifts give a
  wrong gap. The tool marks hand-entered drifts as "unverified" in the report.
- Clause numbers and limits must be checked against the user's copy of the code before the rule pack is
  trusted. I quoted the IS 456 and IS 1893 rules from memory in this note, and they have to be verified.
- Real pounding is dynamic (the two buildings do not always sway in opposite directions); the code gap is a
  conservative sum. The animation is schematic.
- Retrofit on a built frame (adding a joint to an existing building) is a different, much harder problem and
  stays out of scope.

### Future update

- **Other codes:** ASCE 7 / Eurocode 8 / NBC rule packs, so the same tool serves overseas projects.
- **Temperature calculation:** replace the 45 m rule with a simple thermal movement estimate from the climate
  zone, restraint and the slab's length, shown beside the rule of thumb.
- **Joint cover catalogue:** manufacturer-neutral families (floor, wall, roof, basement) with a sizing table,
  and an optional import of real manufacturer families when the team already has them.
- **Waterproofing hook:** feed the joint lines into *Basement Box*, *Rain Run* and *Tank Smith* so the
  waterstop and terrace joint sealant are placed and priced in the same run.
- **Construction joints and pour strips:** the same line logic for post-pour strips and construction joints,
  linked to *Pour Planner*.
- **Retaining and compound walls:** movement joints at the right spacing along long walls, with the BOQ.
- **Reel export:** the "joint opens, building shakes, gap turns red" scene as a short video for LinkedIn and
  Instagram, like the Model Reel idea.
- **Review mode:** a read-only link for the structural consultant with the joint lines, the gap ribbon and the
  rule used, so sign-off happens on the model and not in an email thread.

Sources used for the market check: [Novatr](https://blog.novatr.com/blog/top-bim-modeling-trends),
[QECAD](https://www.qecad.com/cadblog/?p=6476),
[BIMobject / Emseal](https://www.bimobject.com/da/sika/product/emseal-sjs),
[BIMobject / inpro](https://www.bimobject.com/da/inpro/product/2397-024).
