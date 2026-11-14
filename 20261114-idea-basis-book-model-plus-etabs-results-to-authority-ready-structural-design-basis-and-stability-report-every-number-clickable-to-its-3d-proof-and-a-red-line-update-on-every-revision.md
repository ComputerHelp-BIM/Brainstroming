# 20261114 Idea: Basis Book — the model plus the ETABS results become the structural design basis and stability report the authority asks for, every number clickable to its 3D proof, and a red-line update on every revision

## Entry 1

```
date-time:   2026-11-14 (scheduled run)
idea name:   Basis Book (drop in the model — IFC or .c2b — the ETABS export or .e2k, and an Excel or form of the
             project's inputs; get the structural design basis report and stability summary as a Word + PDF file:
             codes and parameters, materials by member, loads built from the model's real finishes and walls,
             seismic and wind data, ETABS period / drift / base-shear results with pass/fail, member and
             foundation summary, quantity ratios, assumptions, drawing list. Each number is a link that opens
             the 3D elements behind it. On the next revision it re-runs and returns the same report with the
             changed lines in red, plus a one-page "what changed since last issue")
potential:   High, every project. Every job needs this document: the municipal / PMC / client / peer-review
             asks for a "structural design basis report" or "structural stability report", and the engineer
             writes it from an old Word file, copying numbers from ETABS, the drawings and the BOQ by hand.
             From the team's experience this takes 1 to 3 days per submission and is redone at each revision
             (not measured here; to be timed on 3 past projects before it is quoted anywhere).
             Checked the 104 ideas already in this repo (searched design basis, DBR, stability certificate,
             structural report, design report): none builds it. Twin Check, Load Path X-Ray, Load Loader,
             Quake Preview and Ratio Radar each produce one *number set*; none puts them in the one document
             people actually sign. Basis Book is where they land.
             Honest on "news": I cannot read LinkedIn or Instagram from here, so I cannot say it is trending.
             The nearest hot topic is "AI writes the boring documents" and "report automation"; the case here
             is the daily pain. The wow: press one button, a 40-page report assembles page by page on the
             screen in about a minute, and clicking "Base shear 4,820 kN" flies the 3D view to the storeys it
             came from. Then a new revision arrives and the report comes back with red lines.
```

### Description

**The pain.**
- The report is a Word file carried from the last project. Parameters (zone, soil type, importance factor,
  grades, cover, loads) are retyped; some are stale from the old job. Wrong zone or grade in a signed
  report is a real risk, and nobody checks the Word file against the model.
- Numbers come from four places: ETABS (period, drift, base shear), the model (members, grades, sizes),
  the BOQ (concrete, steel) and the architect (finishes, wall types, occupancy). They are copied by hand.
- At every revision some numbers change and some are forgotten. The report and the drawings disagree by the
  time they reach the authority; the reviewer's first comment is often "report does not match drawings".
- The engineer's time goes to formatting and copying, not to judging.

**What exists today (checked 2026-11-14; not exhaustive).**
- ETABS has its own report generator (analysis tables, story and grid data) and Python-editable templates;
  CSI's Revit link exchanges the model. That covers the analysis printout, not a design basis report that
  joins model, loads, grades, quantities and assumptions. IDEA StatiCa gives a printable report per member
  or connection check, not the whole-building basis. A web search found no packaged tool that builds this
  document from a BIM model. Sources: [CSIxRevit manual](https://www.csiamerica.com/sites/default/files/CSIxRevit_2016_Manual.pdf),
  [CSI ETABS](https://csiamerica.com/products/etabs), [IDEA StatiCa ETABS link](https://www.ideastatica.com/fr/bim/etabsconcrete).
- Many firms have in-house Excel / Word templates. That is the real competitor: free, familiar, and manual.
  Do not claim the market has nothing; claim that no browser tool joins model + results + inputs and keeps
  it in step with revisions.
- In this repo: Twin Check (design vs BIM), Load Loader (wall loads), Load Path X-Ray, Quake Preview,
  Ratio Radar, Cover Cop, Carbon Lens and BEP Bolt each give a table. Spec Sage reads specs into rules. None
  assembles the signed document. Basis Book reuses their outputs as chapters.

**The idea in one line.** Write the design basis report from the model, so the engineer reviews and signs it
instead of typing it.

**What it does not do.** It does not design, does not certify, and does not replace the engineer's judgement or
signature. It never invents a number: every value is read from the model, the ETABS file or the engineer's
input form, and anything missing is printed as a visible `[INPUT NEEDED]` box, not filled with a guess. If
the user's own AI key is set (roadmap: AI chat with your own key), AI may only reword the narrative
paragraphs; numbers and tables come from code and are marked with their source.

### How it works (user view)

1. **Start a report.** Pick a template: *Municipal submission*, *PMC / client*, *Peer review*, or *Own firm*
   (the firm's headings, logo and cover page are set once and kept on the device).
2. **Drop the sources.** The open model (already in cad2bim), the ETABS results (tables exported to Excel
   or CSV, or the .e2k plus its result tables), and optionally the architect's finish schedule.
3. **Check the input form.** One page, pre-filled from what the files say, each field showing where it came
   from: location and zone, soil type, importance factor, response reduction, wind speed, exposure,
   occupancy live loads, grades by member type, exposure condition and cover, fire rating. Fields no file
   answers are empty and orange. Nothing leaves the device.
4. **Build.** The report assembles in a preview, chapter by chapter:
   1. Project and scope, drawing list with revisions (from the drawing register if Drop Zone ran).
   2. Codes used and their editions (IS 456, IS 875 parts, IS 1893, IS 13920, IS 2502, NBC).
   3. Materials: grades of concrete and steel found in the model, by member type and storey, with a flag
      when the model holds a grade the form does not list.
   4. Loads: dead (self weight, floor finish from the model's layers, masonry from wall types and openings),
      live (by room use), wind, seismic parameters. The wall-load line uses Load Loader's count.
   5. Structural system: grid, storeys, member sizes found (smallest, largest, most common).
   6. Analysis results from ETABS: period, base shear, storey drift with the code limit and pass / fail,
      torsion and soft-storey flags, modal mass participation.
   7. Foundations: type, sizes, bearing used vs allowed (from Pad Packer, Pile Pilot or Raft Run if they
      ran; otherwise from the model's footings).
   8. Quantities: concrete, steel, formwork, with kg per m3 and kg per m2 against Ratio Radar's norms.
   9. Durability and fire: cover and grade by exposure, from Cover Cop.
   10. Assumptions and exclusions (editable list, with the ones the tool sees as likely from missing data).
   11. Open points: every `[INPUT NEEDED]` and every failed check, listed on one page.
5. **Click any number.** It opens the 3D elements behind it (the storeys of the base shear, the columns of
   the largest size, the slabs with the live load). Hold Alt and it shows the formula and the file / cell it
   came from. This is the "show your proof" a reviewer wants.
6. **Review and edit.** The engineer edits text, adds notes, accepts or rejects each flag. Edits are kept
   across revisions.
7. **Export** Word (to finish and sign) and PDF. A footer line names the model revision, the ETABS file and
   its date, and the report id. The signature block stays blank.
8. **Next revision.** Open the new model; Basis Book re-runs, keeps the engineer's edits, and returns the
   report with the changed values in red, plus a one-page "changed since revision B" list (grade changed
   on 14 columns, drift now 0.0031 vs 0.0027, concrete +38 m3). If a changed number sits in a paragraph the
   engineer wrote, it is flagged "text may now be wrong".

### Why a BIM team would use it

- Saves 1 to 3 days a submission (estimate, to be measured); saves most of it again on every revision.
- One source of truth: the report cannot disagree with the model because it is built from it.
- Catches mismatches before the authority does: grade in the model not in the report, ETABS storey height
  differing from the model's level, a member size in the model outside what the analysis used (shown as a
  list the BIM manager can hand to the engineer).
- Junior engineers get a complete, correctly ordered report to review, not a blank page.

### Implementation

**In this repo (read-only study; nothing here was changed).**
- Model reading, properties and quantities: `packages/engine` (web-ifc) already gives members, grades if
  present in psets, levels, and BOQ / BBS quantities.
- Level-wise and member-wise tables: the schedules work on the roadmap and the BOQ window.
- The report preview and Print / PDF: the CAD viewer's Print / PDF path (0.100) and the app's windows.
- Agent / Python access for the inputs form and tests: the cad2bim API (41 operations, `docs/api.md`).
- `.c2b` project file keeps the engineer's edits, assumptions list and report history beside the model
  (`docs/design/project-file.md`, `project-versioning.md` for revisions).
- ETABS: `docs/design/etabs.md` is the place for the e2k reader; Basis Book needs only the result tables
  at first, so it can ship before the full ETABS import.

**Build plan.**
1. **Data model.** A `ReportSpec` (template, chapters, fields) and a `Fact` record: `{id, label, value, unit,
   source: {file, sheet, cell | element ids | rule}, status: ok | input-needed | check-failed}`. Every
   sentence and table cell in the report is rendered from Facts. This is what makes click-to-proof and
   red-lining possible: a revision is a diff of Facts, not of text.
2. **Fact collectors** (pure functions, unit-tested): from the model (members, grades, sizes, levels,
   quantities), from ETABS tables (period, base shear, drift, modal mass), from the input form, and from
   other ideas' outputs when present (Load Loader, Cover Cop, Ratio Radar, Pad Packer).
3. **Rule checks.** Drift limit, minimum modal mass participation (90 percent), soft-storey ratio, grade vs
   cover vs exposure, kg per m3 band. Each check cites the clause as the engineer wrote it into the
   template, not as the tool remembers it; shipped templates are marked "clause numbers to be verified by
   the engineer against the current edition" (do not hard-code clause text from memory).
4. **Templates.** Chapters are Markdown-with-slots, so the firm can edit headings and wording. Four starter
   templates; a firm template is a file kept on the device (or in the team's repo).
5. **Output.** DOCX in the browser (docx.js, MIT) and PDF through the existing print path. No upload; no
   outside CDN (the repo's rule).
6. **Revision diff.** Fact-by-Fact compare, red text in the report, and the changed-list page. Manual edits
   are anchored to Fact ids so they survive.
7. **Click-to-proof.** A Fact with element ids selects them in the 3D view; one with a file cell opens the
   source table in a side panel.
8. **Optional AI wording** (user's own key): rewrites the narrative around Facts, forbidden from touching
   numbers; a test checks that every number in the output exists in the Facts.

**Effort (a guess, not measured).** First useful version (model facts, input form, 6 chapters, DOCX/PDF) a
few weeks for one developer; ETABS tables, red-line revisions and click-to-proof another few; templates
for other authorities as the team sends real examples.

**Risks and honesty.**
- A signed engineering document: the tool must never imply it certifies anything. Cover note says "prepared
  from the model, to be reviewed and signed by the responsible engineer". Missing inputs stay visible.
- Local authority formats differ by city. Ship one neutral template and learn others from real past reports
  the team supplies (kept on the device, never in git).
- ETABS table exports vary by version and settings; start from the common ones and report "not found" rather
  than guess.
- Clause numbers and code limits must be set by the engineer in the template; the tool does not quote
  standards from memory.

### Future update

- **Learn the firm's report.** Drop 2 or 3 past signed reports; the tool learns headings, wording and
  order, and fills the same layout (like Layer Whisperer's per-architect profiles).
- **Reviewer mode.** Drop a PMC's comment list; each comment is matched to the chapter and Fact it concerns,
  and a response sheet is drafted (shares logic with Markup Mirror).
- **Package with drawings.** Issue Desk bundles the report, drawings and transmittal in one go.
- **Steel and PT chapters** from Bolt Up and Tendon Tracer; a **retrofit chapter** from Scan-to-Frame.
- **Live bindings.** A report that watches the model (like Ask & Watch) and shows a badge, "report is
  out of date: 3 facts changed".
- **Client one-pager.** A plain-language summary page (structure type, grades, safety factors in words),
  with no numbers the client could misread.
- **Cost of the report in hours saved** feeds Model Pulse's ROI meter.

### Reel idea for LinkedIn / Instagram

Split screen. Left: a blank Word file and an ETABS window, a clock at 2 days. Right: drop the model, press
Build, 40 pages appear in a minute, click a drift number and the 3D building lights the storeys. Then
"revision C arrived": the report returns with red lines. Caption: "Design basis report: the part nobody
wants to type." (This is a suggestion; no trend data was read.)
