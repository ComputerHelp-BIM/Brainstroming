# 20261028 Idea: Slab Sketch: pick a floor slab (or a whole floor of beams) and cad2bim splits it into panels, classifies each as one-way or two-way, sizes the thickness and the steel to IS 456, lays the bottom mesh, top bars over supports, extra bars at openings and corners, and chairs as real 3D bars, and writes the slab BBS, BOQ and a slab reinforcement drawing. A "pour check" flies bar by bar so the site engineer sees what is laid before the concrete

## Entry 1

```
date-time:   2026-10-28 (scheduled run, UTC)
idea name:   Slab Sketch. "A typical floor: 62 beams, 41 slab panels, 3 shafts, a sunken toilet
             slab, a balcony and a staircase opening. The modeller selects the floor and
             presses Slab Sketch. In seconds cad2bim finds every panel (the area closed by
             beams or walls), reads its lx and ly, edge conditions (continuous or
             discontinuous, from the neighbours) and the load (finish, live load, partition
             allowance from a table the office edits), and marks each panel on the plan:
             'S12 two-way, 4.1 x 4.8 m, case 4 (one short edge discontinuous), D 150,
             Ast 8 mm @ 150 short, 8 mm @ 180 long, top 10 mm @ 200 over the long support'.
             Panels that fail turn red with the reason: 'S27 deflection span/depth 31 against
             26 allowed: use D 170 or add a beam', 'S33 cantilever 1.6 m: 10 mm @ 100 top,
             D 200 at the root'. The modeller accepts, changes one panel, or asks 'make all
             typical slabs 150 mm'. One click later the bars are in the model: bottom mesh
             both ways with the right cover, top bars over supports (with the standard
             0.3 L / 0.15 L extents), extra bars at the four corners of discontinuous
             panels, trimmer and diagonal bars at every opening, chairs and spacers on a
             grid, sunken-slab and balcony steps handled, all as IfcReinforcingBar with
             bar marks. 'Pour check' then walks bar by bar over the slab, like a site
             engineer with the drawing, and shows the cover, the lap positions and the
             chairs, so nothing is found after the concrete."
             One engine, five front ends:
               (1) cad2bim web app: a "Slabs" dock panel (find panels, design, place bars,
                   check, walk), working on the IFC already open; reuses the rebar and BBS
                   code (docs/design/rebar.md, bbs.md),
               (2) cad2bim CH template v2: a slab outline on a SLAB layer with a label such
                   as "S1 D150" (or a thickness note per floor) sets the slab in the
                   pipeline, and panels are found by the same rule,
               (3) Revit (bridge add-in + pyRevit AnonGee button): "Slab rebar" creates the
                   native Revit Rebar for every panel from the same design, with the BBS as a
                   schedule, for offices that must deliver in Revit,
               (4) etabs-mcp: where a floor is already analysed, take the slab moments from
                   ETABS (or the plate results) instead of the code-table method, and show
                   both side by side,
               (5) Structura: the panel table and the code check view show the same results.
             Outputs: IFC (slab, bars, chairs), BBS Excel (cut length, shape code, weight per
             bar mark per floor), BOQ (steel kg per m2 of slab and per floor, concrete m3),
             slab reinforcement drawing (plan with mesh zones, top-bar plan, sections at
             openings and steps; DXF + PDF), a design note PDF with the working for each
             panel, a one-page "steel per m2" comparison across floors, and the pour-check
             video.
potential:   Very high, and every single job has it. In a typical residential or commercial
             building the slabs are the biggest steel tonnage after the footings and columns
             together are counted (often a third of all the steel by weight, check on your
             own jobs) and the most boring to draw: tens of panels per floor, every one with
             bottom bars, top bars, extra bars and openings.
               - Today slab steel is drawn by hand as a 2D plan with callouts, or as a Revit
                 Area Reinforcement that does not give real bars, a real BBS or a real
                 cut length. The BBS is typed into Excel from the drawing.
               - It is where steel is lost: the wrong lap, the top bar too short, the missing
                 corner bar, the opening without trimmers, the chair spacing that lets the
                 mesh sink. These are also what a site engineer finds on the pour day.
               - Doing it once at the slab level and reading the numbers is a common thing
                 to ask of the BIM modeller after the architectural and structural models
                 are done; it should take minutes, not days.
               - A "steel per m2" figure per floor catches mistakes early (one floor at
                 double the others means a design or a modelling error) and lets the
                 project team compare options for the client.
             Market: from a quick search (not a full survey, prices are as listed, check
             before relying on them): Naviate Rebar is listed on the Autodesk marketplace at
             about $190 a year and the listing does not mention slabs; BARBOX is free but its
             FAQ says slab reinforcement is not done; ModPlus has a slab reinforcement tool
             for Revit that places bars or area reinforcement (the page's price and period are
             unclear); SOFiSTiK and Graitec are paid suites. No tool found does the whole
             chain here: IS 456 panel design + real 3D bars + openings + chairs + BBS + BOQ,
             in a browser with no install, working on an IFC from any modeller. The tools
             above are Revit-only and mostly code-neutral.
             Why it makes people say wow: the slab goes from a bare concrete block to a
             full mesh of bars on screen, panel by panel, with the failed panels turning
             red and fixed with one click; and the flight over the laid bars is the first
             time many site teams see the slab before they pour it. This is a good reel for
             LinkedIn or Instagram: "41 slab panels, designed, detailed and scheduled in
             38 seconds", with the steel per m2 shown at the end.
description: Slab Sketch is a slab designer-detailer built on what cad2bim already has: it
             reads the IFC (slabs, beams, walls, openings), finds panels, applies the IS 456
             tables, and writes the bars with the existing rebar and BBS code.
             1. Find panels. Take the slab polygon, cut it by the beam and wall centre lines
                from the model, and keep each closed region as a panel. Slabs with openings,
                steps (sunken or raised) and cantilevers are separate panels with flags.
             2. Classify. lx (short), ly (long); ly/lx above 2 is one-way, else two-way. Edge
                conditions come from the neighbour (another panel at the same level is
                continuous, an edge beam with no slab beyond is discontinuous, a wall may be
                taken as fixed or not, an office setting).
             3. Load. Self weight from the thickness, floor finish and partition allowance by
                room type (toilet, kitchen, living, balcony, terrace, stair landing), live
                load by use (residential, office, assembly), all from a table the office
                can edit, each row showing its source.
             4. Design. Two-way panels by the IS 456 Annex D moment coefficients (cases 1 to
                9), one-way by the standard span moments, cantilevers by the fixed-end
                moment; minimum steel and maximum spacing from the code; deflection by the
                span/depth rule with the modification factor; shear check; cover from the
                exposure (for example toilet and terrace slabs get the higher cover).
                The code clauses used are printed in the design note (the user checks the
                clause numbers against their own copy of the code).
             5. Detail. Bottom bars in two layers (short span below), bent-up bars if the
                office uses them, or straight bars with extra top bars; alternate-bar
                curtailment; top bars at supports with the length rule; corner bars for
                discontinuous corners; trimmer bars (and the cut-bar replacement) at openings
                over a size limit; extra bars at steps, sunken slabs and balcony roots;
                chairs/spacers on a grid sized to the slab depth. Bar marks and shape codes
                from the existing BBS module.
             6. Check and walk. Cover, spacing, clash with sleeves and embeds (Sleeve Punch),
                lap clash with column and beam bars (Joint Congestion Doctor), and a bar
                count per panel. The walk-through camera follows the order a bar-bender would
                lay them: bottom short, bottom long, chairs, top over supports, extra bars.
             7. Compare. 'Steel per m2' for each floor and each option (D 150 against D 170
                plus a lighter mesh) with the concrete and steel cost side by side, so the
                office decides with numbers.
             The designer remains in charge: every panel result is editable and locked
             after editing, and the output says in plain words that the numbers are a
             first design check to be reviewed by the engineer of record, not a stamped
             design.
implementation:
             Mostly assembly of what is in the repo, in four stages.
             Stage 1, panels and table (about 2 weeks): panel finder in the engine (planar
             polygon splitting using the beam and wall footprints, with tolerance), panel
             table UI in the dock, IS 456 coefficient tables as data (not code), unit tests
             against a few textbook worked examples (so the engine can be shown to give the
             same answer as a hand calculation).
             Stage 2, bars (about 3 weeks): bar generator for mesh, top bars, extra bars,
             trimmers and chairs, writing IfcReinforcingBar through the existing writer
             (docs/design/ifc-writer-parameters.md); bar marks and the BBS through bbs.md;
             a bar-count and weight cross-check against the BBS output.
             Stage 3, drawing and compare (about 2 weeks): slab reinforcement plan to DXF
             (using the drawing generation work in Drawing Autopilot), steel per m2 sheet,
             the pour-check camera path (reusing the camera moves item on the roadmap).
             Stage 4, other front ends (about 3 weeks): CH v2 SLAB rule, the Revit button
             (native Rebar through the bridge), and the ETABS moment import.
             Data: panel tables and design results kept in the .c2b project file next to
             the model, so a re-run after a beam moves only changes the panels touched.
             Performance: panels are small and independent, so the design runs in a worker
             and in parallel; a 50-panel floor should be seconds, to be measured on a real
             job, not assumed.
             Licence and privacy: the code tables are facts, formulas are standard
             mechanics; no outside code beyond what cad2bim already uses; the model and the
             design never leave the device, as with the rest of cad2bim.
             Testing: unit tests with worked examples; a comparison of totals against a
             few completed jobs' real BBS (steel kg per m2 within a stated tolerance, to be
             agreed); a golden IFC for the bar writer.
             Risks: edge conditions that are not obvious (the first version asks the
             modeller to confirm the uncertain edges rather than guess); irregular panels
             (non-rectangular panels are flagged for the engineer and treated by an
             equivalent rectangle with a warning); and the legal weight of any design
             output (the note on the output says it is a first check; the user decides).
future update:
- **Flat slabs and drop panels:** column strips and middle strips, punching shear at columns,
  drop panels and shear studs; a "punching shear heat map" as the wow view.
- **Ribbed and waffle slabs:** rib layout, rib bars and the filler, with formwork pans in the
  Shutter Plan.
- **Post-tensioned slabs:** tendon layout and drape from a simple profile, with the stressing
  sequence in 4D (a later and separate project).
- **Other codes:** ACI 318, Eurocode 2 and BS tables in the same engine, since the table is
  data, so the tool also sells outside India.
- **Link to the analysis:** read slab moments and deflections from ETABS or SAFE and show
  "code-table design against analysis" per panel, flagging where they differ more than a limit.
- **Live with the model:** move a beam and only the panels next to it are redesigned and the
  BBS reissued, with a revision cloud on the drawing (Change Radar).
- **Site loop:** the pour card (Site QR Pass) shows the slab's laid-bar checklist; the
  site engineer ticks the bars and photographs the mesh before the pour; the photo is
  stored on the panel.
- **Procurement link:** the BBS grouped into stock lengths with the Bar Optimizer, and the
  slab steel ordered per floor with the delivery date from the 4D schedule.
- **Carbon link:** steel per m2 and concrete per m2 in the Carbon Lens, with a 'thinner slab
  plus a beam' option shown against the cost and the carbon.
- **Agent:** through the cad2bim agent helper (MCP), "make the 3rd to 14th floors typical
  and show me the steel difference" runs the whole thing.

### Improvement suggestions for existing tools in the repos

- **cad2bim (engine):** a panel-finder helper (split a slab by the beams and walls around
  it) as a general function; Lift Lord, Basement Box and Tank Smith each need the same for
  openings and bays.
- **cad2bim (BBS):** add a slab shape-code set (straight, one-end bent, cranked) and a "per
  floor, per panel" grouping to the BBS output; today the grouping is by element.
- **cad2bim (CH template v2):** a SLAB layer rule with a thickness label, in
  `docs/design/ch-template-2-scope.md`, so the pipeline writes slab thickness from the drawing.
- **cad2bim (QA):** a minimum-cover and maximum-spacing rule on bars, usable for slabs,
  walls and footings; also a "steel per m2" summary in the QA report.
- **Joint Congestion Doctor / Sleeve Punch:** read slab bars so the clash check sees the
  mesh, not only beams and columns.
- **Detail Drop:** share the bar-mark and curtailment code, so beams, columns and slabs use
  one set of rules and one BBS.
- **etabs-mcp:** a tool to return slab plate moments by area, to feed the comparison.
- **pyRevit-addons (AnonGee):** a "Slab rebar" button that turns the design JSON into native
  Rebar, for offices that do not use the bridge.
```
