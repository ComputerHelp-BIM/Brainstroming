# 20261106 Idea: Core Cage — shear walls and lift / stair cores: ETABS wall design (pier and spandrel tables) becomes boundary elements, curtain mesh, coupling-beam diagonals and opening trim bars as real 3D rebar in the browser, with a congestion preview, BBS and section sheets

## Entry 1

```
date-time:   2026-11-06 (scheduled run, second entry of the day)
idea name:   Core Cage (RC Shear Wall / Core Wall Rebar Generator: Pier + Spandrel Table Import, Boundary Element
             Zones by IS 13920, Curtain Mesh with Ties, Coupling Beam Diagonal Cages, Opening Trim Bars,
             Congestion Preview Before Anyone Draws a Section, Wall BBS + Section Sheet Set)
potential:   High for any structural BIM team working on towers and mid-rise blocks, which in India is most of the
             residential and commercial pipeline. Shear walls and cores are where rebar modelling hurts most:
             boundary zones with 20 to 40 bars, ties on every layer, 2 curtains, lap and anchorage into the slab and
             the footing, openings for doors and lifts, coupling beams with diagonal cages that collide with
             everything. A modeller typically spends one to three days on a single typical-floor core
             (an estimate from how this work usually runs, not measured here). Core Cage turns that into
             minutes per floor and, because the cages are real 3D bars in the same model, it shows the
             congestion problem before the drawing is issued instead of at the site on pour day.
```

### Description

**The pain.**
- ETABS (or STAAD / the designer's sheet) gives **required** steel per wall pier and spandrel: area per leg, a
  boundary element flag, a ratio. Someone must turn that into **bars you can draw**: which diameter, how many, which
  spacing, where the boundary zone starts and ends, where the ties go.
- IS 13920 (ductile detailing) asks for boundary elements when the compressive stress at the edge passes a limit,
  closer ties inside them, two curtains in thick walls, and special detailing of coupling beams (diagonal cages when
  the span-to-depth is short and the shear is high). It is a rule set that fits a program, and today it sits in the
  designer's head and in a hand-drawn detail.
- Modelling it in Revit rebar is slow and fragile: every wall is its own bar set, every opening breaks the sets,
  and any change in thickness or opening moves nothing automatically.
- The place everything collides is the wall-beam-slab junction: coupling beam diagonals, wall verticals, slab top bars
  and the lift opening trims. It shows up on site as "bars do not fit, cut them", which is a structural decision made
  by a bar-bender.
- Result: the typical drawing set shows boundary elements as a stock section, and the 3D model either has no wall rebar
  or has a rough one nobody trusts.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Detail Drop (2026-10-21) turns ETABS design into **beam and column** detailing.
  Slab Sketch (2026-10-28) does slabs. Pad Packer (2026-11-04) does footings. Lift Lord (2026-10-28) sizes the lift
  shaft and mentions wall thickness around it. Joint Congestion Doctor (2026-10-15) checks congestion on the model
  that exists, and Quake Preview (2026-11-05) looks at the shear-wall layout as a seismic screen. Cover Cop
  (2026-11-04) sets cover. **None of them designs or models the rebar of a shear wall, a boundary element or a
  coupling beam** (a search for "boundary element" in the folder only finds passing mentions).
- In cad2bim: the rebar viewer reads and draws bar sets from the IFC (`docs/design/rebar.md`), the BBS reads rebar and
  has a bar-by-bar schedule (`docs/design/bbs.md`), and the ETABS design (`docs/design/etabs.md`) plans to read
  ETABS's own design tables from an export and to design RCC members to IS 456 / IS 13920. The pieces that are
  missing are the **wall detailing rules** and the **bar generator** for them.
- In the market (not exhaustively checked): Revit's rebar tools can model wall bars by hand, structural-analysis
  suites such as ETABS and SAFE show required steel, and paid detailing add-ons (for example the rebar detailing
  extensions for Revit and standalone RC detailing software) can generate wall reinforcement, usually per seat and
  often tuned to Eurocode or ACI rather than IS 13920 with the Indian bar-bending practice. A free, in-browser tool
  that goes from the ETABS wall tables to IS 13920 boundary zones to 3D bars with a clash and congestion preview and
  a BBS is not something I found. Treat this as unverified until a proper search is done.

**The idea.** A **Wall rebar** workspace in the structural app. Input is the model (walls, openings, slabs, coupling
beams, all in IFC) plus the designer's wall results. Output is real rebar in the same model, a congestion report, a
BBS and the section sheets. Everything runs on the device.

1. **Read the walls and the results.**
   - Geometry: every wall, pier and spandrel from the IFC (or from the CH template DXF -> 3D pipeline), with
     openings, thickness steps and the storeys it passes through. Piers and spandrels are found from the openings
     (a pier is wall between openings, a spandrel the wall above and below an opening).
   - Results: ETABS wall pier / spandrel design tables (Excel or XML export, per the ETABS design note), or a simple
     template for a designer who works in a spreadsheet: pier name, storey, Ast required per leg, boundary flag,
     shear Vu. Missing values are asked for, not guessed.
2. **Apply the rules (visible and editable).** A rule card per wall type, with the clause beside each number:
   - minimum vertical and horizontal steel ratios, spacing limits, two curtains above a thickness;
   - **boundary element** length and steel by IS 13920 (and the stress check that decides it), confinement ties at the
     closer spacing, cross-ties on alternate bars, 135 degree hooks;
   - **coupling beams**: conventional or **diagonal cage** by span-to-depth and shear, with the cage cross-section,
     the confining ties and the embedment length into the wall;
   - **opening trim**: extra bars around every opening, with the corner diagonals;
   - anchorage and lap into the slab above, the footing below, and the starter bars of the next lift.
   The default rule set is **IS 456 + IS 13920**, with other codes as selectable sets later. Every number is shown with
   its clause; the engineer can override any of them. **This is a detailing aid from the designer's required steel,
   not a design; the designer signs it.**
3. **Generate the bars.** Boundary elements, curtain mesh, ties, coupling beam cages and trim bars become
   `IfcReinforcingBar` sets in the model, with marks, diameters and shapes (one shape code per bar, the same as the
   BBS). A **typical floor** is generated once and applied to the repeating floors, with a **different-floor
   exceptions list** ("level 12 has a thicker pier") so the repetition does not hide a real change.
4. **See the congestion before drawing a section.**
   - Run the same checks the rebar viewer can do on the cages: minimum clear spacing between bars (aggregate size +
     5 mm, or bar diameter, whichever governs), tremie / vibrator access gaps in boundary zones, space for the
     slab bars to pass, couplers or lap overlaps at the same level.
   - **Heat map on the wall**: green fine, amber tight, red does not fit. Click a red zone to get the cause
     ("boundary zone P3, level 9: 24 T25 in a 200 mm wall at 50 mm cover leaves 38 mm between bars"), and three
     fixes ranked by steel change: bigger bar fewer in number, thicker wall locally, or grade of concrete (the
     engineer decides; the tool only lists).
   - Coupling beam diagonals are checked against the wall verticals and slab top bars in 3D, and the walk-through
     shows each pour from the outside in.
5. **Quantities and drawings.**
   - **BBS** for the walls by pier, storey and bar mark, the same schedule the app already reads, with weight and
     laps, and the steel in kg per m3 of wall as a sanity number.
   - **Sections and elevations**: a standard set: wall elevation with boundary zones shaded, the plan section of each
     boundary element with bar marks, the coupling beam detail, the opening trim detail. The sets come from the
     generated bars, so a change in the bars changes the drawing.
   - **Pour card** per wall pour, listing the cages in the order they go in and the tie spacing in the boundary zone.
6. **Change loop.** When the architect moves an opening or the designer changes a pier's steel, the cages for that
   pier are regenerated, changed bars are highlighted, and the BBS shows the weight difference. Hand-edited bars are
   locked and listed as conflicts, never overwritten (the same rule as Rev Merge).

**What the person sees.** Click a wall -> a card with its piers and spandrels, each coloured by status (done, needs a
result, congested). A **Generate floor** button, a **Congestion** toggle that paints the wall, and a **Drawings**
button. An **Explode** slider that pulls the boundary cages out of the concrete is the demo moment.

**Why it can wow people (and is honest about it).**
- LinkedIn-friendly demo: drag in an ETABS wall table, press one button, and a 3D lift core fills with
  boundary cages, curtain mesh and diagonal coupling beam cages; press Congestion and a red patch appears where the
  diagonals meet the slab bars; click it, change the wall to 250 mm, and it turns green. That is a real structural
  decision shown in ten seconds.
- It is one of the few places where BIM rebar modelling is hard enough that people stop at "we show it in a section".
- On trend with "reinforcement detailing automation", "AI in structural BIM" and "IS 13920 ductile detailing"
  conversations in structural engineering feeds. I cannot browse LinkedIn or Instagram from here, so this is a
  judgement, not a count.

**Limits and honesty (written on the cards).**
- It is a **detailing aid**: the engineer's required steel and the engineer's decisions come first. It does not
  design the wall, check the load path or replace the analysis.
- Rules are the common cases of IS 13920 / IS 456. Unusual shapes (flanged, L / T / C-shaped cores with eccentric
  openings, staggered openings, transfer walls) start as "needs a person" instead of a guess.
- Congestion is a geometric test on the generated bars. It does not model concrete flow, vibrator reach or the
  contractor's cage assembly method.
- Only as good as the wall geometry and the result tables it was given; missing data is asked for.

**Success measures.** Hours from the wall table to a clash-checked typical-floor wall cage (target: under 30 minutes
for a tower core, against the 1 to 3 days estimated above); share of bars needing a hand edit after generation;
red zones found that were not seen on the 2D detail (checked on 3 past projects where the site had trouble);
BBS weight within 3 % of the designer's hand count on 3 sample walls.

### Implementation

Build on what cad2bim already has; the wall rules, the generator and the congestion checks are new.

1. **Wall model (new, small).** From the IFC: for each `IfcWall`, openings (`IfcRelVoidsElement`), thickness steps,
   the storey, and connected slabs / beams from adjacency (the native-editing relationships). Derive piers and
   spandrels from the openings; give each a stable id that survives a regeneration.
2. **Results reader (reuse + new).** The Excel reading already planned in `docs/design/documents.md` and the ETABS
   table import in `docs/design/etabs.md`; add a `WallResult` (`pier`, `storey`, `AstReq`, `boundaryFlag`, `Vu`) and a
   plain Excel template for non-ETABS users.
3. **Rule engine (new, the core, pure functions).** `boundaryElement(wall, result, rules)`, `curtain(wall, rules)`,
   `couplingBeam(span, depth, Vu, rules)`, `opening trim(opening, rules)`. Each returns bar groups plus the
   **clause text** used, so the UI can show why. Rules live in data (JSON) so a second code or a company standard
   is a file, not a release.
4. **Bar generator (new, reuse the rebar writer).** Convert the groups into bar centrelines and shape codes, and
   write `IfcReinforcingBar` with `IfcSweptDiskSolid` along an `IfcIndexedPolyCurve` the way Revit does (and the
   0.70 bar-path reader expects), with host links to the wall so the BBS finds them. Marks from the BBS numbering.
5. **Congestion and clash checks (new, reuse the viewer's spatial index).** Bar-to-bar clear distance in a
   neighbourhood query; flagged results use the QA card shape (`measured`, `limits`, `elementIds`), so the "select
   elements" action, the colouring and BCF export come for free.
6. **Drawings (reuse).** The section views come from the existing section tools; add wall-specific templates (bar
   marks, tie spacing, hooks) and the Drawing Autopilot sheet set when it exists.
7. **Change loop (reuse).** The element-id hashing and diff from Change Radar / Rev Merge; locked hand-edits kept in a
   per-pier "overrides" table in the project file.
8. **Where it lives.** A tab in the structural workspace; the engine side is TypeScript in `packages/engine`
   (reader output -> rules -> bars -> checks) so the Python console and the agent's tools can call it too
   (`wall.generate`, `wall.checks`), in line with "one model, every view".
9. **Tests.** Three synthetic cores with known answers: (a) a rectangular wall with a flag set -> boundary zone length
   and bar count equal the hand calculation; (b) a coupled wall with a short, deep spandrel -> a diagonal cage and the
   embedment length; (c) a wall with a planted congestion fault (50 mm cover and 28 mm bars at 90 mm spacing) -> the
   red zone is found and nothing else is. Unit tests per rule with the clause text. Then 3 real past cores (kept
   outside git, as for drawings) for BBS weight against the hand count.
10. **Phasing.**
    - Phase 1: rectangular walls, curtain mesh, boundary elements by the flag, BBS. (2 to 3 weeks, estimate.)
    - Phase 2: openings and trim bars, piers / spandrels, congestion heat map.
    - Phase 3: coupling beams with diagonal cages, slab / footing anchorage, typical-floor repeat with exceptions.
    - Phase 4: change loop, section sheets, flanged / L / C cores.

**Risks.** Responsibility for the design (shown on every card and every drawing as "from the designer's required
steel"); rule coverage (start with the common cases and say so); the correct reading of ETABS pier labels and
orientation (test with the real exports before building the generator); bar generation volume on a tower (generate per
pier and per storey, and page the result in as the "very large models" design does).

### Future update

- **Load-path link:** with Load Path X-Ray, the wall that carries the transfer load or the largest axial ratio is
  generated first and shown with its slack.
- **Cover and grade link:** Cover Cop's cover by exposure feeds the cage geometry, and Mark Merge consolidates the wall
  marks across floors with the same "safe upsizing" idea.
- **Shuttering and pour link:** Shutter Plan and Pour Planner read the cage heights, the starter lengths and the
  pour break from Joint Judge, so the cage, the form and the pour card agree.
- **Cage prefabrication:** each cage unrolled as a cut-and-bend list for the rebar yard, with a QR tag per cage
  that opens the 3D (Site QR Pass).
- **Detailing from the drawing:** read a designer's wall schedule drawing (Plan Tracer) and check the generated bars
  against it, line by line.
- **Other codes and company standards:** ACI 318, Eurocode 8, BS 8110 as rule sets next to IS; a firm's own
  details as a rule file.
- **Steel-saving what-if:** show the kg per m3 for 2 or 3 alternative layouts (bigger bar, fewer bars; thicker wall,
  less steel) with the congestion result beside each, for the designer to pick.
