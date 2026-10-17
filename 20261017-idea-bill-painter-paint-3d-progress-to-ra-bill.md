# 20261017 Idea: Bill Painter — paint what is built on the 3D model, get the measurement sheet, subcontractor RA bill, DPR and % complete in one click

## Entry 1

```
date-time:   2026-10-17 (scheduled run)
idea name:   Bill Painter — "paint the work done" on the 3D model (IFC / DXF-to-3D / Structura ETABS model) and get the
             running-account (RA) bill measurement, certified quantity vs BOQ, previous-bill deduction, retention,
             Daily Progress Report (DPR) and a green-grows-over-the-building progress video, with no manual re-measurement
potential:   High. Every month each site engineer / QS re-measures work done (columns cast, slab shuttered, bars tied)
             from site notebooks, WhatsApp photos and Excel, then argues quantities with the subcontractor. A 20-25 day
             cycle of "measure, cross-check, dispute, correct" costs 1-3 days per bill per package, and disputes
             (over-measurement, double billing of the same element in two bills) are a real money leak. The model already
             knows every quantity; nobody uses it as the *measurement book*. Bill Painter turns the 3D model into a
             tamper-evident measurement book: an element can only be billed once, quantities come from the model, and
             every line is traceable to "who painted it, when, with which photo".
             Paid equivalents (Procore progress billing, Autodesk Build/Cost, Dalux, Trimble Connect + Tekla, Bentley
             SYNCHRO Field, Revizto, Oracle Aconex) are enterprise SaaS with per-seat pricing and need a native model +
             admin set-up; none targets the small/mid contractor with "IFC or DXF + Excel BOQ + phone". A free,
             in-browser, files-stay-local version does not exist. Strong LinkedIn/Instagram fit ("progress billing in
             10 minutes", "digital measurement book", "stop double billing", "site progress on a 3D twin"): a 15-second
             reel of a grey building turning green storey by storey while a bill total ticks up is easy to share.
```

### Description

**The pain (daily life of a BIM modeller / QS / site engineer).**
- Month end: the contractor submits an RA bill; the client's QS rejects half the lines. Measurements live in site
  notebooks and Excel with no link to the drawings, so every line is argued from scratch.
- The same slab or column is billed in two bills (once as "shuttering", again as "concrete" with different names), or
  a revision (see Change Radar) changes a quantity after billing and nobody notices.
- "Percent complete" is guessed by eye (e.g. "columns 70%") and rolled up into a progress report no one trusts.
- The BIM model exists but is not used after drawings are issued; the site team never opens it. Progress tracking
  tools (Procore, Dalux, SYNCHRO) are too heavy for a 40-person contractor or a freelance QS.
- Daily Progress Reports are typed by hand from the same notebook every evening.

**The idea.** A **Bill Painter** mode in cad2bim (browser, files stay on the PC; Structura models can use the same data):

1. **Import the BOQ link.** Model elements already carry quantities (concrete m3, steel kg from the BBS, shuttering
   m2) and a BOQ item code via the existing BOQ/costing sheet (`boq.js` / `costing.js` in Structura, BOQ rates in
   PlanWin AI Pro). Each element resolves to one or more *work items* (e.g. `RCC-M25-Col`, `Shuttering-Col`,
   `Rebar-Fe500`) with rate, unit and weight of each *stage* (shuttering 25%, rebar fixing 35%, concreting 35%,
   curing/de-shuttering 5%, editable) so partial progress is billable.
2. **Paint.** Click, box-select, or "select by pour/level/grid" to paint a stage as done on a date. Colours: grey
   not started, amber shuttering, blue rebar, green concrete, dark green certified. A tree filter paints a whole
   storey ("all L3 columns concreted on 12 Oct") in one click. Phone layout works with a big tap target and a
   level picker for the site engineer.
3. **Measurement sheet, automatically.** Painted elements produce an auditable sheet: item, element id, grid location,
   L x B x D or quantity from the model, stage %, date, painter, photo thumbnail. No typing. Duplicate-bill guard:
   an element-stage already in a previous bill is greyed out and cannot be billed again.
4. **RA bill generator.** Cumulative vs previous vs this bill per item, rate x quantity, retention %, mobilisation
   advance recovery, GST/tax lines (template per region), material-on-site line, and deductions. Output as the
   contractor's standard Excel layout (column map configurable) plus a PDF summary and the *measurement abstract*.
5. **Subcontractor mode.** Split by package/vendor (e.g. "steel fixing - Gang A", "shuttering - Vendor B"): each
   package gets its own certified sheet and a QR-linked read-only 3D view so the subcontractor can see exactly
   which elements were certified and which were not. Disputes become "click on the grey element".
6. **Progress dashboard + DPR.** Overall % (value-weighted, not area-weighted), per-level % complete, planned vs actual
   S-curve (links to Pour Planner), and a one-page DPR PDF (today's painted elements, quantities, photos, manpower
   fields) shareable on WhatsApp.
7. **Revision protection.** On loading a new model revision, a Change Radar diff flags *billed* elements that changed
   ("column C14 L3 billed 12 Oct; section changed 300x450 -> 300x500: +0.08 m3 extra / re-measure needed"), so
   post-billing changes are caught at the moment of revision, not at final account.
8. **Time-lapse export.** A 15-second 3D playback by painted date (green grows over the building), exportable as
   WebM/GIF for the client WhatsApp group. This is the wow moment and the LinkedIn/Instagram asset.

**Why it is a wow.** The first time a QS clicks *Generate RA bill* and gets a fully traceable measurement sheet from a
painted 3D model in 10 seconds (instead of 2 days), the model becomes the contractor's money tool, not a drawing
tool. For a BIM modeller, it opens a new paid service line: "progress billing on the model".

### Implementation

1. **Data model** (`packages/engine/src/progress/` or the Structura `js/` equivalent): `WorkItem {code, unit, rate,
   stages[{name, weight}]}`, `ElementProgress {elementId, stage, pct, date, user, photoId, billId?}`,
   `Bill {id, date, package, lines[], retention, deductions[]}`. Pure functions; unit-tested with fixture towers.
2. **Element -> work-item mapping**: reuse the existing BOQ item resolver and BBS-to-element mapping used by the Excel
   BOQ export; unmapped elements show in a "needs mapping" list (like IDS Gatekeeper rules) so nothing is silently
   un-billable.
3. **Painting UI**: reuse Structura's selection/filter/overlay code (`filters.js`, `overlays.js`, `viewer.js`): colour
   per element via an instanced colour attribute (no geometry rebuild); undo/redo through the command stack;
   multi-select by level, grid, pour, or selection box.
4. **Billing engine**: cumulative = sum(painted stage weight x model qty) per work item; this bill = cumulative -
   previous bills; guard: `billId` is written when a bill is *frozen*, frozen rows are immutable (hash chain of
   bill JSON so any edit is detectable). Retention/advance/tax lines are configurable templates per package.
5. **Persistence**: progress stored in a sidecar JSON next to the model (and `.shkp` section in cad2bim); Git-friendly
   so history is auditable; photos stay local (IndexedDB) with thumbnail references. No server.
6. **Exports**: Excel via the existing XLSX writer (sheets: Abstract, Measurement, Previous-vs-Current, Deductions);
   PDF via the existing jsPDF path; DPR one-pager; WebM via `MediaRecorder` on the canvas.
7. **Phone/PWA**: the same module as a "site mode" with level picker + big buttons and offline queue; a QR from
   Site QR Pass opens the right element for painting (`20261016-idea-site-qr-pass-scan-tag-see-3d-pour-card.md`).
8. **Docs/tests**: `docs/design/bill-painter.md`; test cases for double-billing guard, revision-after-billing,
   partial-stage billing, rounding rules.

MVP (2-3 weeks): element-stage painting, measurement sheet, cumulative-vs-previous, Excel RA bill, double-billing
guard. Then packages/subcontractors, retention and taxes, DPR, time-lapse, revision protection.

### Future update

- **Photo-to-progress assist**: attach a site photo to an element/area; an optional offline vision model suggests the
  stage ("rebar tied, no concrete") for the engineer to confirm; the engineer's tap stays the authority.
- **Scan Check link**: a phone/drone point cloud confirms the painted state geometrically
  (`20261016-idea-scan-check-point-cloud-vs-ifc-as-built.md`) before certifying.
- **Pour Planner link**: planned-vs-actual 4D, cash-flow forecast from painted progress, delay alerts by pour.
- **Client portal read-only link**: shareable static progress page (no login) for the client QS and the lender
  inspection reports.
- **Materials-on-site and bar-tag reconciliation**: bars delivered (Bar Optimizer tags) vs bars painted as fixed to
  catch steel wastage per package.
- **Escalation and price variation indices**: automatic variation clause calculations from index tables.
- **Standard formats**: CPWD / MES / FIDIC measurement-book layouts and a Primavera / MS Project progress update
  export (XER / XML) so the planner's schedule gets actual dates.
- **Voice painting**: "paint L4 columns C1 to C12 concreted yesterday" through the action-based assistant pattern,
  with numbers kept deterministic.

### Market check

- Paid/enterprise: Procore Progress Billing, Autodesk Build / Cost, Oracle Aconex, Dalux Field, Trimble Connect,
  Bentley SYNCHRO Field, Revizto. Per-seat pricing, cloud upload, need admin set-up and usually a native model.
- Free/small tools: Excel measurement sheets and generic QS apps without a 3D link; viewer apps (BIMcollab Zoom,
  Dalux Viewer) that colour by status but do not produce the measurement sheet, RA bill or duplicate-billing guard.
- A free, in-browser, offline-capable **"paint 3D progress -> certified measurement sheet -> RA bill + DPR"** flow
  with a double-billing guard and revision-after-billing alert is not offered today. It extends existing
  toolkit pieces (BOQ/costing, BBS-to-element mapping, viewer selection/filters, Excel/PDF exporters), so most of
  the plumbing already exists.
