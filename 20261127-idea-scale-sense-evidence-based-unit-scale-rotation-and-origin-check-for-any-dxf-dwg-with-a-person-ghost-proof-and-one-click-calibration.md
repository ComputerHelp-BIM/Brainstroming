# 20261127 Idea: Scale Sense — drop any DXF or DWG and know in two seconds if it is really mm, really 1:1, straight and in one place; a person-and-car ghost proves it; one click calibrates it for every tool

## Entry 1

```
date-time:   2026-11-27 (scheduled run)
idea name:   Scale Sense — "Is this drawing really in mm and 1:1? Don't measure it. Ask it."
             The step before every 2D to 3D job. A received drawing is read for evidence of its true unit, scale,
             rotation and origin: the declared units, the dimension text against the measured geometry, grid
             spacing, door arcs, parking bays, stair treads, column sizes, the "1:100" in the title block and the
             viewport scale. The evidence votes, and the result is a short card: "mm (96 %), 1:1, rotated 12.4°,
             3 clusters, 14 stray objects 3 km away". A ghost of a 1.7 m person, a 0.9 m door and a 2.5 x 5 m car
             is dropped on the plan so the answer is visible, not claimed. One click writes a Scale Card that
             Plan Tracer, Bar Reader, Layer Whisperer, Takeoff Lens, MEP Tracer and the pipeline all read, so the
             drawing is calibrated once, never re-measured by hand, and the original file is never changed.
potential:   High, and cheap to build, because it sits in front of ideas already proposed. Every 2D to 3D tool
             in this repository assumes the drawing's units and scale are right. When they are not, the damage is
             silent: a column traced at 4,500 instead of 450 looks fine on screen until the coordination meeting.
             The daily pain: consultants send DXFs in mm, m, cm or feet and inches; a plan exported "to scale"
             from PDF software at 1:100 paper units; a detail pasted at 1:20 inside a 1:100 sheet; a file turned
             to fit the sheet; a stray point at 1e9 that makes Zoom Extents show nothing. Today the modeller types
             DIST on a known dimension, UNITS, SCALE, ROTATE and ALIGN, purges the strays and moves it to 0,0.
             My estimate (not measured, to be tested on 20 real drawings): 10 to 30 minutes per drawing, about
             a drawing set of 8 to 15 sheets per project, and several wrong-scale surprises a year that cost
             hours each. For a team handling 20 sets a month that is one to three person-days a month.
             Market: to my knowledge (not verified for this run) AutoCAD and Revit read the declared units
             ($INSUNITS) and, when it is missing or wrong, leave the person to measure. Revit's import dialog
             offers a unit choice and an auto-detect that, as far as I know, relies on the header. CAD cleaning
             add-ins are paid and tidy layers and duplicates, not units by evidence. A free in-browser tool that
             gives a confidence, shows its evidence and proves it with a human figure is not one I know of.
             Trending: "AI checks your drawing before you waste a day" and "stop using the DIST command"
             suit short LinkedIn and Instagram clips. The hook is visual: a 1.7 m person shown as 40 m tall on
             a wrongly scaled plan, then shrinking to the right size at one click. Treat this as a signal of
             interest, not a measured trend.
```

### Description

**The pain (a BIM modeller's day).**
- A drawing arrives from an architect, a structural consultant or a client. Before any tracing, the modeller has
  to answer four questions by hand: what unit is this, is it 1:1, is it straight, where is it.
- The header says one thing and the geometry another. `$INSUNITS` is often 0 (not said) or left at the template's
  value; a drawing made in metres in a millimetre template looks right to its author and is 1,000 times off to
  everyone else.
- PDF-to-DXF converters and "plot to DXF" exports often give paper millimetres: the plan is 1:100, so a
  3,000 mm bay is 30 units long. The dimension text still says 3000. Nothing is "wrong" in the file.
- One sheet holds a plan at 1:100, a section at 1:50 and a node detail at 1:10. Measured as one drawing, two of
  the three are wrong.
- Floor plans are turned on the sheet to fit the paper, so walls are at 12.4°, not square to the axes.
- A stray entity (a point, a hatch, an old block) a long way off pushes the extents out, so the plan is a dot.
- Several floors share one sheet ("typical floor", "terrace", "stilt") with no sign of where one ends.
- The cost is in the silence. A wrong scale does not stop the job; it moves every later number.

**What Scale Sense does.**

1. **Read the evidence.** Each check is small, independent and returns *a vote for a unit and scale, a weight and
   the objects that support it*. A check that finds nothing says nothing; it does not vote.
   - *Declared:* `$INSUNITS`, `$MEASUREMENT`, the paper-space viewport scales, the scale in the title block text
     ("SCALE 1:100", "NTS"). Weak on their own; they are priors.
   - *Dimensions:* every DIMENSION entity has a measured value and a displayed text. If the text is not
     overridden, the file says what the drawing thinks its unit is. If it is overridden, text against measured
     length gives the scale. Loose numbers ("3000", "4500", "3.0", "10'-6\"") next to a line and parallel to it
     give the same ratio. Strong evidence: many dimensions agreeing.
   - *Grids:* grid-line spacing is usually 3 to 9 m in building plans, and the bubbles carry labels (A, B, 1, 2).
     A set of equal parallel long lines with bubbles is a grid; its spacing under each unit is plausible or not.
   - *Fixtures of known size:* door swing arcs (0.75 to 1.2 m leaf), parking bays (2.5 x 5 m), stair flights with
     equal treads (250 to 300 mm) and risers (150 to 190 mm), standard wall thicknesses (115, 150, 230 mm),
     column and lift-shaft sizes, toilet fixtures. Each one is a ruler of known length in the drawing.
   - *Text height:* annotation is plotted 2 to 5 mm high on paper. Text height times the sheet's plot scale
     says the drawing scale.
2. **Vote.** For each candidate (mm, cm, m, ft, in) times each scale step (1, 10, 20, 50, 100, 200) keep a
   score: the sum of weights of the checks whose measurement fits, minus those that contradict. Weights are a
   fixed table written in the open, not learned; the figures in the table are assumptions from common
   practice and are checked on real drawings before they are trusted (see the test plan). The card shows the best
   candidate, the runner-up and the margin. Below a set confidence the card says "not sure" and shows both.
3. **Straighten.** A histogram of line and polyline segment angles, weighted by length, modulo 90°. A single
   clear peak away from 0° is the rotation; two peaks (a plan with a skewed wing) are reported as two zones, not
   one rotation.
4. **Find the drawing.** Group entities into clusters by their bounding boxes (a spatial hash, linear in the number
   of entities). Name each cluster from what is in it: the plan (many walls, a grid), a section (levels, hatches),
   a title block (a frame, text rows), a legend, a key plan, a north arrow, a detail. Entities more than a few
   cluster-widths from every cluster are "stray" and listed. Several plans on one sheet are split and titled from
   the nearest big text ("FIRST FLOOR PLAN").
5. **Prove it.** The viewer puts a flat ghost of a 1.7 m person, a 0.9 m door leaf and a 2.5 x 5 m car on the
   plan, at the best candidate's scale. A person who is a speck, or 40 m tall, shows the problem before any
   number is read. The measured test is also written out in one line: "Grid A to B: 4,500 mm measured, 4500 in the
   dimension text".
6. **Calibrate.** One click writes the Scale Card: `{ unit, scale, rotation, origin, clusters, strays, confidence,
   evidence[] }`, stored with the drawing in the project. It does not change the file. Everything downstream reads
   the card: the pipeline's input applies the transform when it reads the entities, the viewer shows the drawing
   in the card's frame, and every number a tool prints is in real millimetres.
7. **Optional export.** "Save a calibrated DXF" applies the transform to a copy: units set, strays removed,
   rotated, moved to the grid origin and, if wanted, one file per floor. Off by default; the card is enough for
   cad2bim's own tools.

**Why it is a wow.**
- The person-and-car ghost is the demo: drag any wrong drawing in, see an absurdly sized human, press one key,
  and the plan snaps to size with the evidence listed beneath it ("11 dimensions agree; 6 door arcs; 14 bays").
- The card is shown as *evidence you can click*: each line flies to the objects behind it. It never asks to be
  trusted.
- It makes the other ideas safer. Plan Tracer, Bar Reader, MEP Tracer and Takeoff Lens all start from "the
  drawing is calibrated"; one check serves them all, and they stop each doing their own unit guesses.
- It teaches juniors what to look at when checking a drawing, which they otherwise learn by being burnt.

**Day in the life.**
1. 9:10. The architect's plan arrives. Drop it in. Card: "metres, not mm (94 %); 1:1; rotated 12.4°; 2 clusters
   (plan, title block); 14 strays, 3 km away." The ghost car is 5,000 m long, which settles it.
2. One click. The plan snaps to mm, turns square, the strays are set aside and the grid corner sits at 0,0.
   Time: under a minute, instead of a DIST, a SCALE, an ALIGN and a purge.
3. 9:15. The structural consultant's sheet arrives: three drawings on one sheet at three scales. The card splits
   it into "plan 1:100", "section 1:50" and "node 1:10", each titled from its own text, each calibrated alone.
4. 9:20. A PDF-converted plan arrives with "NTS" and no dimensions. The card says "not sure: 1:1 mm (52 %),
   1:100 mm (41 %)". It asks for one known length ("click two column centres and type the distance"), then
   calibrates by that and remembers it for the file.
5. Afternoon. The Plan Tracer run starts from the card. The 3D model's grids land on the architect's grids.

### Implementation

**Reuse what exists.** (The repository's docs and the headers of the CAD core were read for this idea; the code
was not built or run.)
- `packages/cad-engine/core/src/checks.rs` already runs AUDIT and OVERKILL on a copy of the drawing, report only,
  with findings that select their objects. Scale Sense is the same shape: a new `sanity.rs` beside it, a read-only
  report whose findings select their objects, shown in the same Drawing check window as one more tab.
- The CAD engine's `info(id)` already returns units, extents, counts and what was skipped. The card extends it.
- `packages/pipeline/core/src/v2/input.rs` already carries `insunits`, and v2's input is built from the CAD core's
  `pipeline::v2_input`. The transform from the card is applied in that one place, so v1's `Entity` and `analyze`
  are not touched (the rule in CLAUDE.md).
- The viewer already has overlays and colour overrides for the ghost figures and for flying to evidence.
- *Layer Whisperer*, *Drop Zone*, *Grid Lock* and *Plan Tracer* are proposed ideas in this repository that
  would read the card; none needs it to start.

**New pieces.**
1. **Evidence checks** (`packages/cad-engine/core/src/sanity/`): one small function each (declared, dimensions,
   loose numbers, grids, doors, bays, stairs, wall thickness, text height). Each returns `Vec<Vote>`:
   `{ unit, scale, weight, handles[] }`. Adding a check never changes another.
2. **Vote and card:** a pure function from votes to the card, with the margin and the "not sure" rule. Pure and
   deterministic, so it is easy to test.
3. **Angle histogram and clusters:** about 150 lines each, no new dependencies. Run in WebAssembly beside the
   other checks; the numbers go to the card.
4. **Ghost figures:** a few filled outlines as an overlay layer, scaled by the card. No new rendering path.
5. **Scale Card** in the project file (`.c2b`), with the evidence handles, so it travels with the drawing. Shared
   by the team once the project is shared; per person until then.
6. **Pipeline input transform:** one function applying `{ scale, rotation, origin }` to the entities as the
   input is built, with the legacy path (`hosting` off) byte for byte unchanged when no card exists. The existing
   `tools/pipeline-compare.py` check proves it.
7. **Agent tool:** `scale_sense(drawing)` returns the card; an agent can calibrate before it traces, and must
   stop and ask when the card says "not sure". The agent helper (MCP) already exposes drawing tools.
8. **Calibrated DXF export (later):** a transform applied to a copy through the CAD core's writer. It is the one
   step that rewrites entities, so it comes last. Blocks stay as they are and only their inserts move; text
   heights, dimension scales, hatch patterns and linetype scales are handled as AutoCAD's SCALE does, and tested.

**Privacy.** All of it runs in the browser on the device. Nothing is uploaded to read, check or convert the
drawing.

**Test plan (what must be true before it is trusted).**
- Take 20 or more of the team's own real drawings that are known good (kept in a scratch folder, never in git).
  Mutate each: scale by 1000, 0.001, 25.4, 100, 0.01; rotate by 7° and 37°; move 1e6 away; add a stray at 1e9; set
  `$INSUNITS` wrongly. Run Scale Sense and count: right answer, "not sure", wrong. Aim: at least 95 % right,
  under 1 % confidently wrong. These are targets, not results.
- A fixed list of drawings with no dimensions (hand-drawn PDF conversions, details) to see that "not sure" is
  said, not guessed.
- Property tests: the card is unchanged by entity order, by adding duplicates and by the file's layer names.

**Build order (my estimate, not measured).**
- Step 1: declared units and dimension checks, the vote, the card in the Drawing check window. These two are
  the strongest evidence and need no guessing. About a week.
- Step 2: the strays and clusters with the ghost figures, the "not sure" flow with a two-click manual length. A
  week.
- Step 3: the fixtures (grid, doors, bays, stairs, wall thickness) and the angle histogram. About a week, mostly
  tuning on real drawings.
- Step 4: the Scale Card in the project, the pipeline transform and the agent tool. A few days.
- Step 5: multi-drawing sheets split and titled; calibrated DXF export. A week or two.

**Risks and honest limits.**
- A confident wrong answer is worse than no answer. Calibrate only above a high confidence; below it show both
  candidates and ask. Keep the "wrong but confident" count as the one number the test plan must drive down.
- Drawings with no dimensions, no grid and no fixtures (a bare site outline, a detail) give little evidence.
  The tool must say so and fall back to one known length.
- Indian and Gulf drawings often use mixed practice: metres on site plans, mm on floor plans, feet and inches on
  architects' sketches. The unit is per drawing and sometimes per cluster, not per project.
- Fixture sizes differ by country and by building type. The table is editable, and a firm can add its own
  standard door and bay sizes.
- 3D DXFs, drawings with heavy xrefs and drawings whose geometry is inside blocks are checked by their
  exploded extents; the CAD core's reading of blocks and xrefs is assumed to be right.
- This idea tidies the input; it does not fix a badly drawn plan. AUDIT, OVERKILL and Layer Whisperer remain
  the tools for duplicates, errors and layers.

### Future update

- **PDF and scans:** the same card for Plan Tracer, using the printed scale bar, the "1:100" text and the paper
  size to calibrate a vector or raster plan.
- **IFC sibling:** the same voting for IFC files whose length unit does not match their geometry (a model that is
  in metres with a millimetre unit declared), shown as one more card in the IFC viewer and in the QA panel.
- **Consultant memory:** "this consultant always sends metres" and "this one always turns the plan 90°" are
  remembered per sender and offered as the first guess, with the evidence still shown (links to *Layer
  Whisperer* and *Repeat Offender*).
- **Folder mode:** drop a whole drawing folder and get one card per drawing in the register (links to *Drop
  Zone*), with the odd one out flagged ("11 of 12 plans are mm; this one is m").
- **Night run:** *Night Shift* calibrates every new drawing overnight, so the morning's queue shows only the
  ones that said "not sure" (links to *Fix Queue*).
- **Revit:** the bridge reads the card and sets Revit's import units and position, so a linked DWG is right on
  the first link, not corrected after.
- **Revision check:** on a new revision, compare its card with the last: "the architect changed the unit or turned
  the plan 90° in this revision" before anyone traces a single line (links to *Change Radar* and *Rev Merge*).
- **Teaching mode:** the evidence list doubles as a lesson ("here is how to tell the unit from a door arc"),
  for *Dojo Drill* exercises.
