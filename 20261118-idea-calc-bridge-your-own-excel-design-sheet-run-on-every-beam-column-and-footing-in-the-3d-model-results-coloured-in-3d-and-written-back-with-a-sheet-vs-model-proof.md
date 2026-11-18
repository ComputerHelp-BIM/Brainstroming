# 20261118 Idea: Calc Bridge — Point the office's own Excel design sheet (beam, column, footing, slab, lintel) at the 3D model once. It then runs for every element, shows the result in colour on the model, writes it back, and proves each row matches the element it came from.

## Entry 1

```
date-time:   2026-11-18 (scheduled run, UTC; fourth idea of the day)
idea name:   Calc Bridge (the engineer's own .xlsx calc sheets, run per element on the open model, in the browser, nothing uploaded)
potential:   High for daily use, and a strong demo. Almost every structural office in India (and many
             elsewhere) has its own trusted Excel sheets: beam design to IS 456, column check, isolated
             footing, one-way slab, lintel, development length. Today a person copies b, D, span, load,
             fck and fy from the model into the sheet, one element at a time, and copies Ast and the bar
             choice back. For 300 beams that is a day of typing, and a typo is invisible. Time saved: NOT
             measured. First test: take one real office sheet and one finished model (kept out of git),
             run 50 beams by hand and with the tool, and measure (a) minutes per 50, (b) rows where the
             tool and the hand run disagree. Do not quote hours until that is done.
             Market check (2026-11-18, one search; not exhaustive, so "I did not find one", not "none"):
               - Ideate BIMLink moves parameters between Revit and Excel (round trip). The sheet edits
                 data; it does not run a calculation per element. Paid.
               - ENERCALC for Revit links calculation files and can bulk-recalculate, but uses ENERCALC's
                 own calcs, not an office's Excel sheet. Paid.
               - Structure Booster (Schedule Master) builds structural schedules from Excel. It is a
                 schedule maker, not a per-element calculation runner. Paid, to my reading.
               - SOFiSTiK exports to Excel with a Revit element id per row. One way, analysis to Excel.
               - Dynamo with Excel nodes can do it, but only if someone builds and maintains the graph
                 for every sheet. This is the usual workaround.
               - I did not find a packaged tool that takes an office's own sheet, learns its input and
                 output cells by clicking, and runs it for every element. I did not check Revit add-in
                 stores one by one. I could not measure LinkedIn or Instagram counts; "Excel to Revit
                 automation" is a steady topic with BIM trainers, but that is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Sheet Sync: bulk edits of model parameters through Excel. Calc Bridge runs a calculation
                 sheet per element and keeps the sheet's own logic and look.
               - Pad Packer, Slab Sketch, Tank Smith, Core Cage: they bring their OWN design rules into
                 the app. Calc Bridge is for the offices that must keep THEIR sheet (it is the signed-off
                 method, or the client names it).
               - Load Loader and Frame Ferry: they feed loads and members to ETABS. Calc Bridge feeds the
                 office's Excel, for the quick checks that never go to ETABS.
               - Basis Book / Detail Deck: they write the report and the notes. Calc Bridge is the
                 per-element number behind them.
               - Mark Keeper / Mark Merge: marks stay stable, so a sheet row always means the same element.
               - Guess Gauge: an input the model does not hold (span, load) is flagged as assumed.
             The wow: drop in the office's "BeamDesign.xlsx", click three cells once ("this is b",
             "this is Ast required", "this is the bar set"). Press Run. 212 beams turn green, amber or
             red on the 3D model in under a minute. Click a red beam: the sheet row opens beside it, the
             failing check highlighted ("shear: tv 3.1 > tc max 2.8"). One button writes the bar set into
             the model and the BBS. Change the grade, press Run again, and only the rows that changed
             move, with the steel difference shown.
```

### Description

**The pain.**
- The design sheet is trusted and signed off, but it works on one member at a time. Re-entering inputs by
  hand is slow and the mistakes are quiet (a 230 typed as 320, a span read from the wrong grid).
- Results do not come back to the model. The bar size from the sheet lives in a drawing note or a PDF, and
  the modeller types it again.
- When the architect shifts a column, nobody knows which sheet rows are now wrong.
- A new junior copies an old sheet, edits a formula, and the office has five versions of "BeamDesign".

**What it does.**
1. **Teach once.** The user opens a sheet in the browser and clicks the input cells and the output cells.
   Each is given a model field from a short list: width, depth, clear span, level, grade, cover, load
   from a typed value or from a Load Loader result, Ast required, bar set, utilisation. The mapping is
   saved as a small recipe file with the sheet's hash.
2. **Pick the elements.** All beams on L4, every footing, the whole model, or a selection. Sheets with
   several tabs (one per type) are routed by the element's type.
3. **Run.** For each element, the inputs are put in the sheet's cells, the sheet is calculated, and the
   outputs are read. Each run is one row in a results table: element, mark, inputs, outputs, checks.
4. **Show.** The model is coloured by utilisation (or by pass/fail). Click an element to see its row and
   the sheet's own check cells. A summary gives the count of pass, near-limit and fail, and the groups
   that share a result, so 40 identical beams are reviewed once.
5. **Write back (on request).** The bar set, the required area and the sheet's name and version go into
   the element's properties (and the BBS or rebar sets if those exist). Nothing is written without a
   button press, and one undo reverts the whole batch.
6. **Prove.** Each row carries a proof: "sheet row 12 = beam B-114, b 230, D 450, read from the model at
   rev C". When the model changes, only the rows whose inputs changed are marked stale, and Run can redo
   just those.

**Smart parts.**
- **Sheet fingerprint.** The recipe is tied to the sheet's hash. If someone edits a formula, the tool says
  "this is a new version" and asks the lead to approve it before it is used for issue.
- **Round-trip proof.** On first use, the tool runs 3 elements through BOTH the in-browser calculation and
  the user's own Excel result (pasted from a manual run) and shows a side-by-side. It only trusts the
  engine for that sheet when they match to a set tolerance.
- **Worst case, not average.** For a continuous beam the user can choose "support or span" per element, or
  run both and keep the worse.
- **Unit guard.** Metre against millimetre, kN against kg, is checked from the sheet's cell labels and
  flagged on the first run.
- **Offline, private.** The sheet and the model both stay on the device.

**What it is not.** It is not a design engine and it does not judge a design. The office's sheet makes
the calculation, the engineer signs it, and Calc Bridge only carries values to and from the model. It
does not hide a failing check; a red row stays red until the inputs or the design change.

### Implementation

- **Where it lives.** A new "Calc" panel in the web app, beside the quantities and QA tools, working on the
  model already open. Recipes and results are saved in the project file (`docs/design/project-file.md`).
- **Reading the sheet.** Read `.xlsx` in the browser with a library whose licence fits the rules (SheetJS
  Community is Apache-2.0, as I recall; check the exact package and version before use). Read formulas,
  cached values and named ranges, and the cell labels next to inputs (to help the click-to-map step).
- **Calculating.** This is the hard part, and it needs a decision before the build:
  1. **Formula engine in the browser.** Use an MIT, Apache-2.0 or BSD engine and test it on the office's
     real sheets. HyperFormula is, as I recall, GPLv3 or commercial; that must be checked and, if it is
     GPL, it cannot be used here. Where a sheet uses functions the engine lacks (VLOOKUP over big
     tables is common; macros and data tables are not), the tool lists them before running.
  2. **Fallback: the user's own Excel.** On Windows, the Revit bridge can drive the installed Excel for
     sheets the browser engine cannot run. The data still stays on the device.
  Pick per sheet at the first run; the round-trip proof decides.
- **Mapping to the model.** The IFC properties and quantities the engine already reads give width, depth,
  length, level and grade. Spans and loads that are not in the model come from a typed table, a Load
  Loader result or a flagged assumption (Guess Gauge).
- **Batch.** Run elements in a worker, in chunks, with a progress bar. Cache results by the hash of the
  inputs, so a re-run only calculates what changed. Hundreds of elements should take seconds; that is an
  estimate until measured with a real sheet.
- **Write-back.** Write a property set `Calc_<recipe>` (inputs used, outputs, status, sheet hash, run
  time) and, where rebar sets exist, offer the bar set to them. Revit side: a bridge call that sets the
  same parameters on the elements by GUID.
- **Results table.** A sortable table with export to Excel (the same layout as the office's calc register),
  plus a one-page PDF summary per run for the file.
- **Test first.**
  1. Three real sheets from one office (a beam, a column, a footing), kept out of git.
  2. For each, 50 elements by hand and by tool: minutes, and every row where the numbers differ.
  3. Edit a formula on purpose and check the tool notices. Shift a column in the model and check only the
     right rows go stale.
  4. A sheet with a macro and a sheet with a data table: confirm the tool refuses clearly.
  If the engine cannot reproduce at least two of the three sheets exactly, build the Excel-driving route
  first and keep the browser engine for later.
- **Risks.** A wrong mapping giving a plausible wrong number (mitigate: the round-trip proof and the
  click-to-confirm step); sheets with macros or external links (mitigate: refuse and say why); someone
  treating green as "approved" (mitigate: the sheet's name, version and the signing engineer's name stay
  on every result, and the panel says "carried from your sheet"); formula-engine licence (mitigate:
  decide before any code).

### Future update

- **Sheet library.** The office keeps its approved sheets and recipes in one place; a new project picks
  them and the version is recorded in the project.
- **Sheet to a button.** A sheet that is run often becomes a one-click tool, in the same way as
  Toolsmith turns an AI recipe into a team button.
- **Change alerts.** When a revision changes a member, the affected rows go amber and the engineer sees
  the old and the new result (Change Radar's feed).
- **Report pages.** The per-element sheet pages are printed in the order of the drawing index, for
  the calculation volume that goes with the submission.
- **Edit in 3D.** Drag a beam depth in the model and watch the sheet's utilisation change live.
- **Sheet from a PDF.** For an old calculation that exists only as a PDF, offer to rebuild it as a
  simple sheet that can be checked against the original.
- **Shared with the checker.** A proof-checker gets the same recipe and results, and re-runs them on
  their own copy of the model without touching the office's files.
