# 20261126 Idea: Plumb Line — the surveyor's level book and total-station readings become a column-lean and slab-level heatmap in 3D, checked against tolerance, with a lift-by-lift correction plan before the next pour

## Entry 1

```
date-time:   2026-11-26 (scheduled run)
idea name:   Plumb Line — "Paste the surveyor's readings. See the building lean."
             The site surveyor already writes numbers after every pour: column centre offsets (X, Y) at the top
             of each lift, slab and beam soffit levels from the auto-level, sometimes a total-station CSV.
             Today these live in a notebook, a WhatsApp photo or an Excel sheet nobody opens again. Plumb Line
             reads that sheet (or CSV, or a photo of the sheet), matches every reading to the right element in the
             cad2bim / IFC model, and shows the as-built deviation on the 3D model: a heatmap, an exaggerated
             "lean" view, a pass / watch / rework status per element against the tolerance in the spec, and a
             correction plan for the next lift ("column C14 is 11 mm east at L7: shift the next form 3 mm west").
potential:   High. Verticality and level errors are the most common reason for hidden rework in RC frames:
             lift cores that do not line up, beams that no longer meet the column face, finishes that eat into
             headroom, lift guide rails that need a second survey. The readings exist; the checking is done by eye
             on a notebook, or not at all until a finishing trade complains. My estimate (not measured): an engineer
             spends 2 to 4 hours per floor copying readings into a sheet and comparing to the drawing, and a
             missed 15 mm drift at floor 6 costs days of chipping or packing at floor 20. A tool that turns the
             sheet into a coloured 3D model in under a minute, and predicts where the drift will be 5 floors up,
             saves the hours and catches the drift while it is still cheap to steer back.
             Trending angle: "the leaning tower view" is an instant visual hook. A 15-second reel shows the
             surveyor's Excel pasted in, the tower tilting at 20x exaggeration with two red columns, and the
             correction arrows appearing. It fits the LinkedIn "digital QA on site" and "construction
             quality 4.0" posts, and the Instagram site-engineer audience that shares survey tricks.
             Market (from my own knowledge; I did not re-check prices today): Trimble, Leica and Topcon field
             software can compare points to a design, but they are paid, tied to their own instruments, and
             stop at a table of deltas. Dalux, Procore and Autodesk Build hold QA forms but do not read the
             survey sheet back onto the 3D elements or predict the next lift. Scan-to-BIM tools need a laser
             scanner. Nothing free and open turns the surveyor's own Excel into a per-element 3D tolerance
             report.
```

### Description

**The pain (daily life of a site BIM engineer / quality engineer).**
- After each pour the surveyor shoots column tops and levels. The numbers go into a notebook. Someone later types
  them into Excel and compares to the drawing by hand, usually only for the "important" columns.
- The grid line is a wire in the drawing, but the building is a set of concrete elements with real offsets. Nobody
  sees the pattern: a whole line of columns drifting 2 mm a floor in the same direction.
- Spec tolerances exist (for example, column plumb and slab level limits in IS 456 / IS 14687 practice and ACI 117
  style tables; each project writes its own in the specification) but are not applied element by element.
- When a lift core or a facade bracket does not fit, the first question is "when did this start?" and the readings
  to answer it are scattered across weeks of notebooks.
- Sub-contractors (lift, glazing, MEP risers) ask for "as-built centreline of the shaft" and get it by a manual
  survey again, though the data already exists.

**The idea.** A **Plumb Line** panel with four parts.

1. **Read the readings, any shape.** Drop an XLSX, CSV or a clear photo of the survey page. The reader finds the
   columns for mark / grid, level, X, Y, reading type (top of column, soffit, slab), and date, with a preview where
   the person confirms the mapping once; it remembers the mapping per project. Units are checked (mm vs m, a very
   common error) and sign convention is asked once ("+X = east or toward grid B?").
2. **Match to the model.** Each reading is matched to the element by mark, grid intersection, or nearest
   element within a search distance; unmatched readings go to a short list with a "pick on 3D" button, never a
   silent drop. Design position comes from the model (IFC placement); deviation = reading minus design.
3. **See it.**
   - Heatmap on the 3D model by deviation (green inside tolerance, amber past 70% of it, red out).
   - **Lean view**: exaggerate deviations 5x to 50x so a 9 mm drift reads like a leaning tower; slider on screen.
   - Per-grid-line chart: deviation by level for one column line, so a trend is obvious.
   - Per-slab level contour: a coloured surface of ±mm against the design level, high and low spots circled.
4. **Act on it.**
   - **Correction plan for the next lift**: for each column, how much to shift the form (a small, signed number in
     mm, with a cap per lift so the correction itself stays inside what the spec allows). Shown as arrows in 3D
     and as a one-page sheet for the foreman.
   - **Drift forecast**: a simple, explainable fit (straight-line per column over the last lifts) says where the
     column will be at the top, and flags lines that will leave tolerance in N floors if nothing changes.
   - **Impact check**: slab-level deviations are turned into extra or missing concrete (m³) and finish
     thickness (packing or chipping) per room; column offsets are checked against beam-column face
     alignment and lift-shaft clear width.
   - **Pass / rework sheet** (PDF + XLSX): per element and per pour, reading, tolerance, status, who accepted;
     ready for the pour-card, the ITP file and the client's quality review.

**Why it fits cad2bim.** The model already has the exact design position and the element identity for every column
and slab; the web-ifc reading, the 3D viewer, the QA panel and the XLSX reader exist. The new parts are the reading
mapper, the match, the deviation view and the correction maths. Everything stays on the device: the survey sheet is
read in the browser and is never uploaded.

### Implementation

Build on what exists; no new heavy dependency. Estimates, not measured.

1. **Reader (1 week).** XLSX/CSV through the existing sheet reader; a column mapper with remembered profiles
   (one JSON per project in the project file). Photo input is phase 2 and goes through the same mapper after OCR
   on the device (or manual confirm).
2. **Matcher (1 week).** By mark, then by grid intersection (the grid is already in the model for the CH
   template), then nearest element within a distance. Output: matches, ambiguous, unmatched. Unit tests with a
   synthetic 10-storey frame: 100% matched, injected typos found.
3. **Deviation engine (1 week).** Per reading: vector deviation in model axes (the tilt of the grid is respected),
   resultant, and a status from a tolerance table (JSON, per project: plumb per lift, plumb overall, level per
   bay, level overall). Ship two example tables labelled "example only; use your spec".
4. **3D view (1 to 2 weeks).** Heatmap through the existing colour-by-property path; lean view as a vertex offset
   shader with a slider; per-grid-line and contour charts with the design system's chart tokens.
5. **Correction + forecast (1 week).** Capped per-lift correction; least-squares line per column over the last k
   lifts with a visible "why": the three numbers it used. No black box.
6. **Impact + sheet (1 week).** Slab-level to m³ by area × mean deviation; beam-column face check from the
   model; PDF/XLSX through the existing report path.
7. **Docs and tests.** `docs/design/plumb-line.md` in the app repo; a sample survey sheet in `docs/examples/`.

Rule from the app repo: no file leaves the device; no GPL code; tolerance values are the project's, not ours, and
the tool says so on the sheet.

### Future update

- **Total-station live link**: read the instrument's CSV from a watched folder (pairs with Night Shift) so the
  heatmap updates when the surveyor saves the file.
- **Photo of the notebook page** with on-device OCR, confirm-then-import.
- **Set-out loop**: feed the correction plan into Set-Out Pack so the surveyor stakes out the corrected form
  position for the next lift.
- **Scan Check bridge**: when a point cloud exists, show surveyor readings and scan deviations on the same model
  and flag disagreements between the two.
- **Facade and lift hand-off**: export the as-built centreline of the lift shaft and of facade brackets as DXF/IFC
  for the lift and glazing sub-contractors.
- **Cross-project learning**: learn typical drift per contractor and per formwork system, and warn at the start
  of a new tower ("this system drifts 1.5 mm a floor to the east: pre-offset the forms").
- **Steel and PT**: add base-plate level and anchor-bolt offset reading for steel frames (pairs with Bolt-Up) and
  tendon anchorage position for PT slabs (pairs with Tendon Tracer).
- **Quality score**: a project-level number per pour and per contractor for the monthly review, with the proof one
  click away in 3D.

### Related ideas in this repo

Complements Set-Out Pack (before the pour), Scan Check (laser, after the pour), Stage Shield, Cover Cop, Cage Ghost
and Pour Proof. None of them reads the surveyor's own readings back onto the elements.
