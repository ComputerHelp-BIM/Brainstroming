# 20261107 Idea: Drop Zone — drop the consultant's whole drawing folder (DWG, DXF, PDF, XLSX, IFC, ZIP) and get a drawing register in a minute: latest revision picked, superseded files flagged, a per-level modelling-readiness board, a missing-sheet list, and the right converter queued, all on your device

## Entry 1

```
date-time:   2026-11-07 (scheduled run, first entry of the day)
idea name:   Drop Zone (Project Intake: Folder / ZIP Sorter, Auto Drawing Register, Revision & Superseded
             Resolver, Per-Level Modelling Readiness Board, Missing-Sheet Chaser, One-Click "Start Modelling" Queue)
potential:   High, and it is the first hour of EVERY project and EVERY revision. The architect or structural
             consultant sends a ZIP or a Drive link with 80 to 300 files named "plan_final2_REV(1).dwg",
             "FOUNDATION LAYOUT R3 (old).pdf", "col schedule new.xlsx". Before one wall is modelled someone
             spends half a day to two days (an estimate from how this usually runs, not measured here) opening
             each file, working out what it is, which revision is current, which level it belongs to, and what is
             still missing. Drop Zone does that triage in about a minute and ends with a board that says, per
             level: "Plan: ready. Column schedule: ready. Section: MISSING. Beam schedule: R2 superseded by R4,
             use R4." It is the front door to the whole toolkit (CH-template pipeline, BBS Weaver, Layer
             Whisperer, RFI Forge) and the wow is a folder going in and a to-do board coming out, with nothing
             uploaded.
```

### Description

**The pain.**
- Consultants send folders, not packages. Names carry no rule, several revisions sit side by side, and "final",
  "final2", "latest" and "new" all appear in one folder.
- The modeller's first job is detective work: Which DWG is the typical floor plan? Is the schedule in this XLSX the
  same revision as that plan? Is anything for the basement at all?
- Working from a superseded sheet is one of the most expensive mistakes in a model: it is found at the pour card
  or at the site, not on the screen. Nobody notices because the wrong file looks perfectly valid.
- Missing sheets are found late ("where is the stair section?"), after the model is half built, so the RFI goes
  out late and the model waits.
- The register is built by hand in Excel, once per project, and is stale by the next mail.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: RFI Forge (2026-10-29) audits **inside** the drawings (plan vs schedule vs section
  conflicts). Layer Whisperer (2026-10-30) fixes the **layers** of one DXF. Rev Merge (2026-11-02) merges a new DXF
  revision into the hand-fixed model. Issue Desk (2026-10-31) is the **outgoing** register and transmittal. Scope
  Scout (2026-11-01) counts a drawing set into hours and a quote. Plan Tracer (2026-10-16) reads a scanned plan.
  **No entry takes the incoming folder as a whole and decides what each file is** (a search for "intake" in this
  folder returns nothing; "register" appears only for outgoing issue). Drop Zone sits **before** all of them and
  feeds them.
- In cad2bim: the CAD engine opens DXF and DWG on the device; the documents core reads PDF (PDF.js), Excel
  (calamine) and Word on the device (`docs/design/documents.md`); a folder or a zip can already be picked and read
  without upload for add-ons (`docs/design/addons.md`, fflate, paths that climb out dropped); the CH pipeline
  already knows plans, levels and the schedule (`docs/design/cad2bim-pipeline.md`). The pieces exist; the
  **sorting brain on top** is the new part.
- In the market (not exhaustively checked): common data environments and document-control systems (construction
  cloud platforms, Aconex-, Procore- and Bluebeam-style tools) offer auto-register, revision control and metadata
  by file-name rules, but they are paid, per-seat or per-project, need an upload to the cloud, and see a file as a
  **name and a stamp**, not as a **drawing they have read**. A free tool that opens the DWG, reads the title block
  and the geometry and says "this is the second-floor slab plan, R3, and the beam schedule is R2" on the
  modeller's own laptop is not something I found. Treat that as unverified until a proper search is done.

**The idea.** A **Drop Zone** on the home screen, and a command anywhere. Everything runs in the browser; no file is
uploaded; the original folder is never changed.

1. **Drop the folder or ZIP.** Nested folders are fine. Archives are opened in memory. Each file gets a type from
   its **content**, not its name: DWG, DXF, PDF (vector or scanned), XLSX / XLS / CSV, DOCX, IFC, RVT (listed, not
   opened), images.
2. **Read what each file says about itself.** Cheap and local, in this order:
   - **Drawings (DXF / DWG):** title-block text (sheet number, title, scale, revision table, date, drawn / checked),
     level names from text and from the CH template's own level cues, the layer set (CH template v2 or not), a
     bounding box and a content fingerprint.
   - **PDF:** the text layer (page 1 title block) and page size; scanned PDFs are marked "image, needs Plan Tracer".
   - **Excel:** sheet names, header row and first rows, so "column schedule", "beam schedule", "BBS", "door and
     window schedule", "bar bending" are recognised from the columns present (mark, size, b x d, bars), not from the
     file name.
   - **IFC:** schema, storeys and element counts.
3. **Classify and place.** Each file lands in a small grid of **discipline** (architecture, structure, MEP, site)
   by **document kind** (plan, section, elevation, schedule, detail, spec, BBS, report) by **level** (basement 1,
   ground, typical, terrace, ...). Confidence shown; low confidence goes to a "check these" lane with one-click
   corrections that are remembered per consultant.
4. **Pick the latest, flag the rest.** Files that are the same sheet (same number or title and level) are grouped
   as a **chain**. Revision order comes, in this order, from the revision table inside the drawing, then the title
   block date, then the file's own date, then the name ("R3", "rev 4", "final2"). The chain shows **why** it picked
   one: "R4 (revision table, 12 Oct) beats R3." A **content diff** (same bounding box and fingerprint = a byte-
   identical re-save; many lines different = a real revision) stops a re-saved copy from looking like a new
   revision.
5. **The readiness board.** A level-by-kind table, like a stage-gate sheet: green ready (current, readable, on the
   CH template), amber ready with notes (older template, scanned PDF only, plan is R4 but its schedule is R2), red
   missing (nothing found for that cell). "Not expected" cells can be switched off (no basement section needed).
   A **mismatch rule** catches the quiet killer: the plan and the schedule it depends on are not the same
   revision.
6. **Chase list.** The red and mismatch cells become a short, ready-to-send email or WhatsApp note to the right
   consultant ("Please send: stair section, beam schedule R4 for levels 3 to 12"), in plain language, one click to
   copy. This is the first RFI, written before modelling starts and based on what was actually found.
7. **Start modelling, queued.** For each ready cell, a button hands the file to the right tool already in the
   toolkit: a CH-template DXF to the DXF -> 3D pipeline; a messy architect DXF to Layer Whisperer first; a BBS
   Excel to BBS Weaver; a scanned PDF to Plan Tracer; an IFC to the viewer. "Start all ready" runs the queue in
   workers and the 3D model builds level by level as files finish. Nothing is modelled from a file marked
   superseded unless the person overrides it.
8. **Re-drop is the update.** Next week's mail is dropped on the same project; Drop Zone compares with last time and
   says "4 new, 2 revised, 1 superseded, 1 still missing", and hands the revised ones to Rev Merge. The register
   is the project's living **intake log**, saved in the project file, exportable to Excel and PDF.

**Why it gets attention (a LinkedIn / Instagram-style hook, not a measured trend).**
- A 20-second screen recording: drag a 300-file ZIP into the browser, a board fills in, red cells appear, the 3D
  model starts building on its own. The caption writes itself: "Messy consultant folder to modelling-ready in
  60 seconds. No upload. Free."
- AI document-intake and "agent that reads your drawing set" posts are everywhere; this one is the version that
  keeps the drawings on the person's own device, which is the objection every contractor and developer raises.

**Practical wins.**
- Hours to minutes for the start of each project and each revision (target, to be measured on three real folders).
- Fewer superseded-sheet mistakes, which are the costly ones.
- The first RFI goes out on day one, with the model's start date no longer a guess.
- A junior can run it; the senior reviews the board, not the folder.

### Implementation

Build on what cad2bim already has. The new work is the **file sniffer, the classifier, the chain resolver and the
board**.

1. **Intake (reuse + small).** Folder and ZIP reading from the add-on source loader (fflate, paths that climb out
   dropped); a worker per type; file handles only, bytes read on demand. Output: a flat `IntakeFile` list
   (`path`, `kind`, `size`, `mtime`, `hash`).
2. **Sniffers (mostly reuse).** One small function per type returning a `Facts` record:
   - DXF / DWG: reuse the CAD engine's reading; add `titleBlock(texts, bbox)` that finds the text cluster in the
     lower-right corner or the largest bordered rectangle and picks sheet no., title, scale, revision rows by
     keyword + position; `levels(texts)` from the CH pipeline's level cues; `layerProfile` (is it CH v2? use
     `tools/ch-v2-check.py` logic as a library call); `fingerprint` (hash of normalised entity geometry, ignoring
     handle ids and timestamps).
   - PDF: PDF.js text layer of page 1; the same title-block finder on text positions.
   - Excel: calamine first 40 rows; `scheduleKind(headers)` by column-set scoring (reuse the BBS / BOQ header
     scorer the Bid Mirror idea also needs).
   - IFC: header and storey list from web-ifc without loading geometry.
3. **Classifier (new, pure functions, testable).** Rules first (title words, sheet-number prefix like S-101 / A-201,
   column sets), a small weighted score for each discipline / kind / level, confidence 0 to 1, and a `reason`
   string kept for the "why" popover. Optional AI second opinion through the existing "AI chat with your own key"
   path, sending **only the title-block text**, never geometry, and only after the person agrees. Corrections are
   stored per sender in the project file.
4. **Chain resolver (new, the core).** Group by normalised (sheet no. | title, level); order by revision source
   priority (revision table > title-block date > mtime > name); mark `current`, `superseded`, `duplicate`
   (same fingerprint) and `conflict` (two "current" with different content). Always keep the reason.
5. **Readiness board (new).** A pure function `readiness(chains, expectation)` -> grid of cells with `state`,
   `files`, `notes`; the `expectation` is a small template per building type (residential tower, basement + podium,
   warehouse) that the person can edit and save. Mismatch rules: plan R vs schedule R; level count in plan vs
   levels in schedule; CH template version.
6. **Chase list (new, small).** Templates per state, filled from the board; copies to clipboard; optional export as
   a one-page PDF with the documents writer.
7. **Queue to the toolkit (reuse).** A job list that calls the existing entry points (`dxf -> ifc` pipeline,
   Layer Whisperer, BBS Weaver, Plan Tracer, the IFC loader); jobs run in workers, show progress, and write the
   result into the project as a new revision with the intake file as its source, so "where did this element come
   from" (Trace Lens) can point back to the register row.
8. **Register output (reuse).** Excel through the documents writer (ExcelJS), PDF through krilla; columns: no.,
   title, discipline, kind, level, revision, date, status, source path, note.
9. **Tests.** A synthetic folder generator with planted faults: a re-saved identical file, "final2" newer than
   "final" by name but older by revision table, a schedule one revision behind its plan, a missing section, a
   scanned PDF, a ZIP with a path that climbs out. A test that each fault is reported and a clean folder
   produces an all-green board. Then three real folders (kept outside git, as for drawings) to measure how many
   files are classified right and how many land in "check these".
10. **Phasing.**
    - Phase 1: folder / ZIP in, type sniffing, title-block read, register table, Excel export. (1 to 2 weeks,
      estimate.)
    - Phase 2: classifier, chains, latest-revision picker, superseded flags, "why" popover.
    - Phase 3: readiness board, mismatch rules, chase list.
    - Phase 4: queue to the pipeline, BBS Weaver, Layer Whisperer; re-drop diff and Rev Merge hand-off.
    - Phase 5: optional AI second opinion, per-consultant learned corrections.

**Risks.** Title blocks vary wildly -> start with CH-template drawings and a correction step, and show
confidence and the "check these" lane always; revision order is sometimes not knowable -> say "cannot tell", never
guess silently; scanned PDFs -> flagged for Plan Tracer, not guessed; a wrong "superseded" flag is dangerous ->
nothing is ever deleted or hidden, only labelled with its reason, and the override is one click; privacy -> device
only, no file name or content leaves the machine unless the person turns on the AI second opinion.

### Future update
- **Model-side reverse check:** after modelling, mark each cell "modelled from R4"; when a newer file arrives,
  say which elements depend on the old revision (feeds Change Radar and Rev Merge).
- **Mail and Drive watch:** a folder watcher (or a shared drop link) that re-runs the intake whenever a consultant
  adds a file and posts the "4 new, 2 revised" line to the team.
- **Consultant scorecard:** how often each consultant's package arrived complete, on a stable template, with
  revision tables filled in; a polite quality note to send back (a quiet nudge toward the CH template).
- **Project-start kit:** from the board, open the BEP Bolt scorecard, the Scope Scout estimate and the first RFI
  pack from one screen.
- **Offline phone drop:** share a site photo set or a scanned drawing from the phone straight into the intake.
- **Learned house style:** the second project from the same architect needs zero corrections.
