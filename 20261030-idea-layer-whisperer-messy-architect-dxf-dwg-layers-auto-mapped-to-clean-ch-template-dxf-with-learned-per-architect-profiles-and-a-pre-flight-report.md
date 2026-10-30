# 20261030 Idea: Layer Whisperer — drop any messy architect DXF/DWG; it works out which junk layer is walls, columns, beams, doors, grids and text, fixes units, xrefs and duplicates, learns each architect's habits, and hands back a clean CH-template DXF that converts to 3D first time

## Entry 1

```
date-time:   2026-10-30 15:30 UTC (scheduled run)
idea name:   Layer Whisperer — "The architect mails a DWG. Layers are called A-WALL-FULL, WALL-NEW, 0,
             Defpoints, LAYER 1, 'wall thk 230', 'COL (RCC)', and the walls are loose LINEs, not
             polylines. Today the modeller spends 1-3 hours renaming layers, exploding blocks, purging,
             closing gaps and re-checking units before 2D-to-3D even starts. With Layer Whisperer you
             drop the file in the browser (it never leaves the device). In about 10 seconds it shows a
             pre-flight card: 'Units: mm (drawing says inches, but lengths say mm) - 1 xref missing -
             3 duplicate wall lines - 14 gaps under 5 mm closed - 2 layers I am unsure about'. Each
             layer gets a guess with a confidence: 'WALL-NEW -> Walls (98%) - thick parallel lines,
             230 apart', 'COL (RCC) -> Columns (99%) - closed rectangles with a hatch', 'LAYER 1 ->
             Beams? (61%)'. The modeller confirms only the 2 unsure ones, by clicking on the drawing.
             The tool writes a clean DXF in the cad2bim CH template (layers, units, closed polylines,
             block names) and the 3D model builds first time. The mapping is saved as a profile for
             that architect, so the next drawing from the same office needs zero clicks."
             Parts:
               (1) Pre-flight scan: units, scale, insertion point far from origin, missing xrefs and
                   fonts, frozen/off layers, duplicate and zero-length entities, overlapping lines,
                   proxy objects, 3D-solid leftovers.
               (2) Layer guesser: name matching (English, Hindi-transliterated and common firm codes)
                   plus geometry clues: parallel line pairs at 115/230/150/100 mm = walls, closed
                   300x600-type rectangles = columns, dashed lines = beams above, arcs near wall gaps
                   = door swings, hatch = RCC, text with 'D1/W2/C1' = door/window/column marks.
               (3) Geometry repair: loose LINE/ARC chains to closed polylines, gap closing, exploded
                   blocks regrouped, wall centrelines from double lines.
               (4) Learned profiles: per architect or per project, stored locally (.c2b / JSON),
                   shareable inside the office. Corrections the modeller makes are remembered.
               (5) Output: clean CH-template DXF, a before/after layer table, a one-page cleanup
                   report (what was changed, what was guessed) and an undo-able change log.
             Front ends:
               (a) cad2bim CAD workspace: a 'Prepare' step before the DXF-to-3D pipeline, built on the
                   CAD engine (CADCraft Rust core in WebAssembly) and the Rust pipeline's v2 reader.
               (b) CLI / pyRevit button: batch-clean a whole folder of sheets overnight.
               (c) Output feeds Plan Tracer (PDF scan -> DXF) and Column Compass, so every entry
                   route ends in the same clean template.
             Outputs: clean DXF, cleanup report (PDF/Excel), saved architect profile, issue list that
             can go to the architect as a BCF-style note ('layer X has gaps at these 6 points').
potential:   High and daily. Every 2D-to-3D job starts with someone else's drawing, and no two offices
             name layers alike. This is the step before the step the team already automates, so it
             raises the hit rate of everything after it. Time today per drawing set (an estimate from
             how such cleanup is usually done by hand, not timed in this office):
               - renaming and merging layers, purge, audit: 20-40 min per sheet,
               - exploding blocks and joining lines into polylines: 30-60 min,
               - checking units and scale, fixing the origin: 10-20 min,
               - fixing the 3D result after a bad guess and re-running: 30-90 min.
             Total about 1.5-3.5 hours per floor plan, repeated for every revision.
             With Layer Whisperer: about 5 minutes the first time for an unknown architect (confirm
             the unsure layers), under 1 minute for a known one. Expected saving: about 1-3 hours per
             plan (estimate). For a team that opens 10-20 plans a week that is 10-60 hours a week.
             Revisions are cheaper still: the saved profile makes re-clean of Rev C a drop-and-go.
             It is also the cheapest way to make the Rust pipeline look reliable, because most failed
             conversions come from input, not from the converter.
             Wow effect: the 'before/after' reveal. A grey spaghetti of 60 junk layers turns into
             five clean coloured layers while a slider moves, and each guess shows WHY ('230 mm
             parallel pair, 41 m long'). Then one click and the 3D model stands up. Short screen
             recordings of 'messy drawing -> clean model in 10 seconds' do well on LinkedIn and
             Instagram, and 'AI cleans CAD' fits the AI-for-AEC posts that are trending now.
             Market: AutoCAD's Layer Translator, Purge and CAD Standards checker exist but are manual
             mappings inside paid AutoCAD, and they know nothing about what a layer contains. Paid
             tools such as Scan2CAD, Lisp routines sold on forums and Revit import-and-fix add-ins
             help with parts, none learns a firm's habits or writes a BIM-ready template. Free DXF
             viewers do not repair. As far as we know no free, browser-based, on-device tool does
             'guess the layer meaning from geometry + names, repair, remember the architect, and
             output a BIM-ready file'.
implementation:
             Phase 1 (1-2 weeks) - Pre-flight scan only: units, origin, duplicates, zero-length,
               missing xrefs; report card UI in the CAD workspace. Reuse the CAD core's reader and the
               v2 reader's checks (tools/ch-v2-check.py is the checklist to port to Rust).
             Phase 2 (2-3 weeks) - Name-based layer guesser with a rules table (JSON, editable) plus
               the confirm-by-click UI and the saved profile. Test set: 15-20 real drawings from
               different architects kept in a scratch folder, never in git.
             Phase 3 (3-4 weeks) - Geometry clues and repair in the Rust core: parallel-pair wall
               detection, gap closing with a tolerance in drawing units, loose lines to closed
               polylines, block regrouping. Each step reports counts so the result can be audited.
             Phase 4 (1-2 weeks) - Clean DXF writer in CH-template v2 layout, before/after slider,
               cleanup report PDF, CLI batch mode.
             Phase 5 - Optional small model for ambiguous layers, run locally; no drawing is
               uploaded. Pure rules first; add a model only if the rules fail on real files.
             Success measures: share of plans that convert to 3D with no manual edit (target 80%+
             on the test set), minutes from drop to clean file, number of guesses a person had to fix.
             Risks: wrong guesses silently change a design - so every change is listed and undo-able,
             low-confidence layers are never auto-applied, and the original file is untouched.
future update:
             - Revision-aware: compare Rev B and Rev C after cleaning and feed Change Radar.
             - Architect scorecard: how clean each office's drawings are, to send as feedback.
             - Shared office rule library across projects and teams.
             - Same engine for MEP and structural drawings (duct, pipe, rebar layers).
             - Layer standards output beyond CH: AIA, ISO 13567, a client's own.
             - DWG straight in via the acadrust path, with the same report.
             - Check against the architect's PDF so a missing layer is caught before 3D.
```
