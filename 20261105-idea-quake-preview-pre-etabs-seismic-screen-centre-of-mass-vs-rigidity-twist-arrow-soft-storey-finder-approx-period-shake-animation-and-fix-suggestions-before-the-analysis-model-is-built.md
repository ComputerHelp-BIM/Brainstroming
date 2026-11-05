# 20261105 Idea: Quake Preview — a pre-ETABS seismic screen on the 3D model: centre of mass vs centre of rigidity as a twist arrow, soft-storey finder, approximate period and base shear, a shake animation, and "move this wall / grow this column" fix suggestions before the analysis model is built

## Entry 3

```
date-time:   2026-11-05 (scheduled run, third entry of the day)
idea name:   Quake Preview (Pre-Analysis Seismic Screen on the Architect-Stage or Structure-Stage 3D Model:
             Mass and Stiffness Proxies per Storey, Centre of Mass vs Centre of Rigidity Twist Arrow,
             Soft / Weak Storey Finder, Approximate Period and Base Shear, Shake Animation, Ranked Fix
             Suggestions, Hand-Off Sheet for the Analysis Engineer)
potential:   High for any team doing RC buildings in zones III to V. The expensive loop today is:
             model in Revit, hand over to the analysis engineer, wait a day or two, ETABS shows torsional
             irregularity or a soft storey or a drift failure, the engineer asks for "a wall here, a bigger
             column there", the modeller changes the architecture-constrained layout, repeats. Each loop costs
             1 to 3 days (an estimate from how this usually runs, not measured here) and often 2 to 4 loops
             per tower. A screen that tells the modeller, in seconds, "this layout will twist: shift the lift
             core wall 3 m east or add a wall at grid 7" removes loops before they start. It also gives the
             BIM modeller something no one else on the team has: a feel for lateral behaviour while the plan
             can still move.
```

### Description

**The pain.**
- Plans are fixed by architecture and parking first; lateral behaviour is looked at last, by someone else, in
  another program. By then, changing a wall means redoing doors, MEP and the area statement.
- Typical findings that come back late: the lift core and stair walls are all on one side (the building twists);
  the stilt / podium storey has no walls above the car park (soft storey); a column stops at a transfer slab
  (weak storey); the corner columns are tiny compared with the core (drift and torsion fail).
- The modeller can see the plan but has no number for "where is the stiffness". The engineer has the numbers
  but not until the analysis model exists.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Load Path X-Ray (2026-10-26) is about **gravity** load flow and footing size.
  Gap Guard (2026-10-31) is about seismic gaps between blocks. Detail Drop (2026-10-21) detailed rebar **after**
  ETABS design. Column Compass (2026-10-30) makes column layouts for cost and fit, not for lateral behaviour.
  No entry screens lateral stiffness, torsion or soft storeys on the model itself.
- In cad2bim (`docs/design/actionable-qa.md`): an **"IS 1893 screening (geometry only)"** group is designed
  (setback ratio, discontinuous columns, floor-to-floor variation, labelled "Screening, not compliance"), and the
  QA types already carry a `clause`, a `measured` line and a `limits` line. It is listed as phase 3 and not built.
  `docs/design/etabs.md` plans an ETABS reader, so results can be compared later. **Quake Preview is the extension:
  from geometry-only ratios to a stiffness- and mass-based screen with pictures.**
- In the market (not exhaustively checked): ETABS, STAAD, Robot and SAFE analyse well but only once an analysis
  model exists, and they are paid. Revit's structural analytical model is a separate step and needs setup.
  Some research and hobby tools estimate period from height only. A free, in-browser tool that reads the
  architectural-stage 3D model and shows centre of mass, centre of rigidity and a soft storey in seconds, with
  fix suggestions, is not something I found; treat that as unverified.

**The idea.** A **Quake** panel in the cad2bim viewer, on top of the QA panel, using the same finding cards. It
runs on the model in the browser; nothing is uploaded. It is a **screen** with the label "Screening, not
analysis", never a design result.

1. **Read the model as mass and stiffness.** Per storey, from the IFC the viewer already has:
   - *Mass:* slab, beam, column, wall and (typed-in) finishes and live-load fraction per IS 1893 Table 10
     (the person sets the numbers; defaults are shown and marked as defaults). Centre of mass (CM) per storey.
   - *Stiffness proxy:* for columns, `12EI/h³` of the section, with a cracked-section factor the person can set;
     for shear walls, a flexural plus shear stiffness of the wall section; infill ignored unless switched on.
     Centre of rigidity (CR) per storey from these. Beams are treated as rigid links to start; a limit that is
     written on the card.
2. **Show it, don't tabulate it.** In the 3D view:
   - a **twist arrow** at each storey from CM to CR, scaled by the eccentricity (`e / plan dimension`), coloured
     green / amber / red against the 0.15 b trigger commonly used for torsional concern (the code's wording to
     be checked by the engineer);
   - **soft-storey colouring**: storeys whose lateral stiffness is below 70 % of the one above, or below 80 % of
     the average of the three above (the IS 1893 Table 6 tests, to be re-read against the edition in use);
   - **weak storey**: strength proxy (sum of column and wall shear capacity estimates) below 80 % of the one above;
   - a one-line **"Why"** under each flag in plain words: "Storey 1 has 14 columns and no walls; Storey 2 has the
     lift core. Storey 1 is 38 % as stiff as Storey 2."
3. **Approximate period and base shear.** `T = 0.075 h^0.75` (moment frame) or `0.09 h / √d` (with walls) as the
   code's empirical periods, plus a Rayleigh estimate from the stiffness proxy; base shear from zone, importance,
   response reduction and soil type typed in the panel. Both numbers are shown with the label "estimate" and the
   formula. The point is the **ratio** to the ETABS result later, not the number.
4. **Shake animation (the wow).** Press **Shake**: the building sways in its estimated first mode, exaggerated
   and clearly marked as "illustration", with the twist visible if CM and CR do not coincide. The soft storey
   visibly folds. The recording is a short clip for a client or a team post (the Model Reel pipeline planned in the
   2026-10-18 idea can be reused).
5. **Fix suggestions that respect the plan.** A small search (not a solver) tries a short list of moves and
   re-scores each: add a wall on a chosen grid line, move a lift-core wall, turn a column into a wall segment,
   enlarge the corner columns by one size step. Candidates are limited to places the **architectural layer allows**
   (no doors, no windows, not on a parking bay line: the checks other ideas in this repo build). Output is a ranked
   list: "Wall at grid 7, between B and C: CM-CR offset 3.1 m → 0.9 m; adds 4.2 m³ concrete; fits behind the
   stair." One click previews it in 3D as a ghost; **Apply** is a normal undoable edit.
6. **Hand-off sheet for the analysis engineer.** One page (and a CSV): per-storey mass, CM, CR, stiffness
   ratios, flags, the assumptions typed in, and the model's GUID list. After the ETABS run, import the engineer's
   storey results and Quake Preview shows **screen vs analysis side by side**, so the team learns how far the
   screen was off on this kind of building.

**Why it is practical.** It moves the first lateral look from week 3 to hour 1 of the structural model. The
modeller can bring a flagged layout to the architect with a picture and a candidate fix, instead of a message that
says "engineer says it twists". It uses what cad2bim already has: element geometry and materials, the QA card and
its `clause` / `limits` fields, selection and ghost preview, the ETABS reader that is planned.

**Wow factor (what the person sees and can post).**
- A live **twist arrow** that swings across the plan as a wall is dragged: drag the lift core and watch the arrow
  shrink.
- The **Shake** clip: a tower folding at the open stilt floor, then the same tower with one added wall, standing.
- "Before / after in 40 seconds" for a layout review meeting.

**Trend link (a judgement, to be checked against live feeds before posting).** Building collapses in recent
earthquakes (Turkey 2023, Myanmar and Bangkok 2025) were widely discussed on LinkedIn with soft-storey and
construction-quality themes, and India's seismic zoning and IS 1893 are under revision (news of a new zone map was
seen; I have not verified its status). A free "will it twist?" check is share-friendly. Verify before using in a post.

### Implementation

**Where it lives (cad2bim is not modified by this idea; this is the plan for later).**
- A new QA group `is1893-screen` beside `is1893` in `packages/engine/src/qa/` (pure functions, same `Finding` shape),
  plus a `quake/` module for the per-storey numbers so the panel, the MCP agent and the Python console share it.
- Section properties come from IFC profile definitions (`IfcRectangleProfileDef`, `IfcCircleProfileDef`, wall
  thickness and length from the quantity sets or the geometry the engine already holds).
- UI: Quake panel (inputs, flags, ghost fixes), a twist-arrow overlay in the render package, a sway shader that
  displaces vertices by a storey-wise mode shape (a vertex-shader job, no solver).

**Build order.**
1. *Week 1:* storey grouping, mass, CM, and the geometry-only IS 1893 checks already designed (so this ships on top
   of phase 3, not in place of it). Test on 3 real frames with a hand calculation.
2. *Week 2:* stiffness proxy, CR, twist arrow, soft-storey ratios, plain-words "Why". Compare CM / CR against
   ETABS on 3 buildings the team already analysed; record the error, publish it in the panel's About text.
3. *Week 3:* approximate period and base shear; hand-off sheet; shake animation.
4. *Week 4:* fix search with architecture constraints and the ghost preview; storey-results import for the
   side-by-side.

**Data and licences.** Formulas are standard mechanics and the code's published tests, written in the team's own
words and constants, not copied tables (IS 1893 text is copyrighted: store only the numbers the person confirms
and cite the clause). No GPL code; no network use.

**Limits and honesty (written on the cards).**
- A stiffness proxy is not an analysis: beams as rigid links, no slab flexibility, no P-delta, no soil, no
  infill by default. It will over- or under-state by a margin we must **measure on real buildings** before saying
  more than "screen".
- It does not replace the engineer's judgement or code compliance; every card says "Screening, not compliance".
- Irregular, tall (> 90 m) or base-isolated buildings are out of scope and the panel says so instead of guessing.
- Results depend on the IFC carrying real sections; proxies and bad profiles are listed as "cannot screen".

**Success measures.** Hours from first model to a layout the engineer accepts; analysis loops per project;
screen-vs-ETABS error on CM-CR offset and storey stiffness ratio (the table is the honest scoreboard).

### Future update

- **Live while editing:** the twist arrow and flags update as a wall or column is moved or created
  (Create tools in the roadmap).
- **Wind:** the same panel for IS 875-3 along-wind drift and torsion screening.
- **Link to Pad Packer / Load Path X-Ray:** footing and load-path checks start from the same storey masses.
- **Link to Gap Guard:** pounding and joint checks use the period estimate.
- **Link to Mark-Merge and Column Compass:** a type-diet or layout option shows its lateral score as a third axis
  beside steel and cost.
- **Learned correction:** keep each team's screen-vs-ETABS pairs on the device and fit a small correction factor
  per building type, shown as "calibrated on 12 of your buildings".
- **Detail hand-off:** the engineer's final ETABS design feeds Detail Drop (2026-10-21) so the loop closes into
  rebar and BBS.
