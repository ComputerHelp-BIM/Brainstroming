# 20261128 Idea: Sag Sense — every beam, slab and cantilever checked for span/depth and long-term sag the moment it is drawn or modelled; the building "sags" in 3D (exaggerated) so the weak members and the walls that will crack are obvious; one click proposes the smallest size fix before ETABS or the first RFI

## Entry 1

```
date-time:   2026-11-28 (scheduled run)
idea name:   Sag Sense — "Will this beam droop and crack the wall under it? Know it at modelling time, not at handover."
             A fast serviceability screen, not a design. For every beam, slab panel and cantilever it reads span,
             depth, support type and the wall or finish it carries, applies the code's span/depth rules (IS 456
             cl. 23.2 with the steel-stress and flange modifiers; the same rule for other codes as a setting) plus
             a long-term deflection estimate (creep and shrinkage), and colours the 3D model green / amber / red.
             A "Sag replay" bends the real model 50 to 200 times exaggerated, so the 6 m balcony, the 7.5 m
             transfer beam under a brick wall and the 150 mm slab over a 5.5 m panel visibly droop. Where a sagging
             member sits under a masonry partition, the partition is marked "crack risk" (deflection against
             span/500 and the wall's tolerance). One click offers the smallest fix that passes (depth +25 mm steps,
             compression steel, a drop beam, a column, a camber note) with its steel and concrete cost, and
             writes the chosen size to the model, the schedule and an RFI note for the structural consultant.
potential:   High, because it moves a late, expensive surprise to the first hour of modelling.
             The daily pain: drawings reach the BIM team with member sizes that were set at concept stage and
             never rechecked. The modeller builds them faithfully. The sag shows up as cracked partitions, doors
             that jam, ponding on a terrace, and tiles that lift, months after handover. The checks that would have
             caught it (span/depth, cantilever limits, long-term deflection) are done in ETABS/SAFE or by hand, late,
             if at all, by someone who is not the modeller. The modeller, who touches every member, has no quick way
             to ask "does this look sane?".
             My estimate (not measured, to be tested on 10 real projects): a first screen of a 20-storey tower takes
             under a minute and finds a handful of members worth an RFI; the same review by hand takes half a day.
             A project that gets one avoided "re-cast the transfer beam" or "re-do the cracked partitions" case
             pays for the tool many times over, but that is a story, not a number: log real catches to measure it.
             Market: to my knowledge (not verified for this run) ETABS, SAFE, STAAD and Robot compute deflections
             after a full analysis model exists, and they are paid. Revit and Navisworks have no span/depth check.
             Some paid Revit add-ins check minimum sizes. A free, in-browser screen that runs on the drawing or
             the IFC, shows the sag in 3D and ties it to the partitions above and below is not one I know of.
             Trending: "this beam is why your wall cracks" and "watch the building sag at 100x" are short,
             visual, shareable clips for LinkedIn and Instagram, and structural engineers and site teams react to
             them. Treat this as a signal of interest, not a measured trend.
```

### Description

**The pain (a BIM modeller's day).**
- The architect's plan and the structural drawings give sizes: 230 x 450 beams, a 150 slab, a 6 m balcony with a
  125 mm tip. The modeller models them. Nothing in Revit, IFC or CAD says "this is too slim for its span".
- The people who would check are busy: the consultant sees the member only in the analysis model, and the
  modeller is told "model as drawn".
- What a modeller can see that nobody else does: the span of every member, the depth, what stands on it (a
  230 brick wall, a parapet, a planter, a water tank) and what hangs under it (a partition, a glass door, a false
  ceiling). All of it is in the model and none of it is looked at together.
- The failures are slow: a cantilever that drops 25 mm at the tip, a transfer beam that cracks the wall under
  it, a 5.5 m slab that ponds on the terrace. They are expensive to repair and easy to prevent early.
- A modeller who raises the doubt needs numbers, a picture and a proposed fix, or it is "just an opinion".

**What Sag Sense does.**

1. **Collect each member's facts** from the model (IFC) or from the 2D drawing that the pipeline already reads:
   - *Beams:* clear span and effective span between supports, width, overall depth, support type (simply
     supported, continuous at one or both ends, cantilever), from what it rests on. Beams ending on another
     beam are flagged "secondary beam, span to the girder face".
   - *Slabs:* each panel's short span and long span, thickness, edge continuity, one-way or two-way by ratio,
     openings.
   - *Cantilevers:* projection length, root depth, tip depth, what is on the tip (a balcony wall, a chajja, a
     planter).
   - *Loads that matter for sag:* the wall height and thickness on the member (counted from the model, as in
     the proposed *Load Loader*), a parapet, a finish thickness, a tank.
   - *What is under it:* partitions, glass doors, tiles, plaster ceilings. Their tolerance to deflection is a
     setting (brick: span/500 total after the finish; glass: tight; false ceiling: loose).
2. **Check in three layers, cheapest first:**
   - *Span/depth by the code's rule* (IS 456: 7 cantilever, 20 simply supported, 26 continuous, times the
     tension-steel and compression-steel modifiers; 10 m span cap and the 8.5 m reduction for brittle finishes
     where the code says so). With the steel ratio unknown at modelling time, the check uses a stated default
     (and says so) and the range from "lightly reinforced" to "heavily reinforced", so the answer is "passes in
     the whole range", "passes only if steel is low" or "fails in the whole range".
   - *Long-term deflection estimate* using the code's creep factor and shrinkage curvature, from the span, the
     depth and the dead and sustained live load found in the model, with a plainly stated accuracy ("an estimate
     within a factor of about 1.5, enough to rank members, not to design them").
   - *Consequence check:* the estimated deflection against what is above and below (partition cracking limit,
     tile limit, glass limit, ponding on a slab with a slope of less than 1:100).
3. **See it.** Colours on the 3D model (green, amber, red) and a **Sag replay**: one press bends every member by
   its estimated deflection at 50x, 100x or 200x, the worst ones first, with the partitions under them drawn
   cracked where the check says so. A slider scrubs from "as drawn" to "after ten years". It is a picture of an
   estimate, labelled as one.
4. **Fix it.** For each failing member, the smallest changes that pass, ranked by cost:
   deeper by 25 mm steps, a wider beam, compression steel, a drop beam or column added at a stated position, a
   camber note, or a lighter wall (AAC block in place of brick) with its load change. Each option shows the
   steel and concrete delta and the effect on headroom (a link to the proposed *Flush Fit* check) and on the
   floor-to-floor height.
5. **Close the loop.** The chosen fix is written to the model as a proposal (a ghost beside the original), the
   schedule and an RFI note with the numbers. It is applied only when the person accepts it. When the
   consultant answers "keep as drawn, checked in ETABS", the member is marked "accepted by consultant, date" and
   stops being flagged.

**What it is not.** It is a screen. It does not replace the analysis or the engineer's signature. Its job is to
find the few members worth a question, early and with proof, and to say clearly when it cannot tell.

### Implementation

**Reuse what exists.** (Docs and folder layout in this repository were read for this idea; the code was not built
or run.)
- `packages/engine` already reads IFC with web-ifc, has `model`, `qa` and `params`, and a BOQ and BBS that give
  concrete and steel quantities. Member facts (span, section, material, level) come from the same loaded model
  that the QA checks read. The checks join the existing QA list (`docs/design/actionable-qa.md`), so findings
  already select and fly to their elements.
- The pipeline (`packages/pipeline`) already knows each member's drawn length and section for DXF models, so the
  same screen runs on a drawing before any IFC exists.
- The viewer already colours elements by a rule (the QA and phasing colours); the Sag replay is a vertex
  displacement on the same meshes, done in the vertex shader from a per-element value, so a 100,000-element model
  stays at a usable frame rate. No mesh is rebuilt.
- *Quake Preview*, *Load Path X-ray* and *Load Loader* (proposed in this repository) share the same inputs
  (spans, tributary loads, walls on members). Sag Sense reads their results when they exist and computes its
  own simple version when they do not, so it can ship first.
- The agent helper (MCP) can expose `sag_screen(model)`, which returns the list and the proposed fixes, and an
  agent can draft the RFI text.

**New pieces.**
1. **Member facts extractor:** supports (from what each beam end touches), effective span, continuity, wall and
   finish loads on the member, and what is under it. The hard part is supports; the first version asks the
   person to confirm uncertain ones in one panel ("this beam ends on a beam, not a column: confirm").
2. **Rules as data:** span/depth tables, modifiers, creep and shrinkage factors and partition tolerances in a
   JSON file per code (IS 456 first; the setting names the code, so a team can fork its own). Each result cites the
   clause so the number can be traced to the code, never to the tool.
3. **Estimator:** a small, pure, tested module (no I/O), with a table of hand-worked examples from the code's
   own worked cases as the test set. Every output has a range and a "confidence" word.
4. **Sag replay shader:** per-element displacement as a function of position along the span (a parabola or a
   cantilever curve), exaggeration and time sliders.
5. **Fix proposer:** a search over a short, ordered list of changes; it re-runs the estimator after each one and
   returns the cheapest that passes in the whole range. It never changes the model without a click.
6. **RFI note and member status** in the project file (`.c2b`) so "accepted by consultant" survives revisions.

**Privacy.** Everything runs in the browser on the device. Nothing is uploaded to read or screen a model.

**Test plan (what must be true before it is trusted).**
- Hand-check 30 members, chosen by the team's own senior engineer, across beams, slabs and cantilevers on three
  past projects. The tool's pass/fail verdict must match the engineer's on at least 27, with every miss a
  "cannot tell" rather than a wrong "pass". Record the result in the repository; measured, not estimated.
- Take two past projects with known site problems (cracked partitions, a sagging balcony). Does the screen flag
  those members? Count both the catches and the false alarms.
- A model with 50,000 members must screen in under 10 seconds on a normal laptop, and the replay must hold
  30 frames per second; measure and record both.
- Show the result to two consultants and ask only: "would you act on this?". Keep their answers, including the
  negative ones.

### Future update

- **Other codes** (ACI 318, Eurocode 2, BS 8110): the rules file is data, so a team adds its own with a clause
  reference for each number.
- **Read deflections from ETABS or SAFE results** (the E2K bridge is proposed as *Frame Ferry*): replace the
  estimate with the analysed value, keep the same colours, replay and partition check. The screen then becomes a
  viewer for the analysis.
- **Site check:** the surveyor's levels (*Plumb Line*, proposed) compared with the predicted sag at a pour or a
  year later, so "predicted 14 mm, measured 17 mm" tunes the estimator for the team's own materials and
  workmanship.
- **Stage-aware sag:** early-age loading and back-propping (the proposed *Stage Shield*) give the real deflection
  history of a slab through construction; the replay plays the construction months, not only the final state.
- **Pre-camber and levelling:** write the camber value to the shuttering (*Shutter Plan*, proposed) and the
  finish-level schedule, so the floor is flat after it settles.
- **Vibration comfort** for long slabs and cantilevers (a footfall frequency estimate) as a second screen in
  the same panel.
- **Client card:** a one-page "why this beam is deeper" for the developer, with the sag picture and the cost of
  the fix against the cost of the failure, so value engineering does not quietly thin the members again.

*References read for this run: the repository's `docs/roadmap.md` and `docs/design/` folder listing (cad2bim),
and the existing idea files in this folder, to avoid repeating them. Nothing in the source repository was
changed.*
