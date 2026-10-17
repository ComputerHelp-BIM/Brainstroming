# 20261017 Idea: Sheet Sync — a live Excel <-> 3D model grid: edit 300 elements in a spreadsheet, see the change ghosted in 3D with steel/concrete/cost delta, then apply as one undo step

## Entry 1

```
date-time:   2026-10-17 (scheduled run)
idea name:   Sheet Sync — "Excel as the control panel of the model": export any selection or filter (columns, beams,
             slabs, footings, rebar hosts) to a clean sheet, edit sizes / marks / grades / levels / cover / custom
             parameters in bulk in Excel (or in the in-app grid), and get a ghosted 3D preview + quantity and cost
             delta before anything is written. Same flow in cad2bim (IFC) and in Revit (pyRevit tool).
potential:   High. Bulk property editing is the most repeated chore of every BIM modeller after drawings change:
             "C1 to C7 sizes changed in the new ETABS run", "re-mark all beams by grid", "set grade M30 above L10",
             "fill the missing Fire Rating / Zone / Pour No. for 800 elements", "rename types to the office standard".
             Today that means selecting element by element, 20-100 clicks per change, or learning Dynamo / scripts.
             A typical revision touches 100-500 elements; at 15-30 s each it is 1-3 hours of pure clicking, and
             typos ("C-12" vs "C12") break schedules later. With Sheet Sync it is: export, edit in Excel with
             filters and fill-down, import, review, apply = 10-15 minutes.
             Paid equivalents are Revit-only and per-seat (Autodesk BIMLink / Ideate BIMLink, DiRoots OneFilter and
             similar add-ins); Dynamo + Excel nodes need scripting skill; BlenderBIM spreadsheet export is free but
             not user-friendly and has no preview or cost delta. A free, in-browser, IFC-native, preview-before-apply
             version that also reports the quantity / cost impact of the edit does not exist as far as I know
             (to be re-verified before the build starts). Good LinkedIn / Instagram fit ("edit 500 elements from
             Excel in 10 minutes", "stop clicking, start syncing", "BIMLink alternative, free"): a 15-second reel
             of a spreadsheet cell being changed while the 3D columns grow with a ghost outline and a "+3.2 t steel,
             +Rs 1.9 lakh" badge is easy to understand and share.
```

### Description

**The pain (daily life of a BIM modeller).**
- Design revision arrives (new ETABS / PlanWin run, new architectural drawing). 150-400 element properties change:
  section sizes, grades, top-of-concrete levels, marks. The modeller edits them one by one.
- Marks and custom parameters (Zone, Pour No., Fire Rating, Package, Subcontractor, Drawing No.) are missing or
  inconsistent; schedules, BOQ and the IDS / readiness check then fail.
- The engineer sends an Excel with the "right" sizes. The modeller retypes it. Nobody can prove the model now
  equals the sheet.
- After a bulk change nobody knows what it did to the quantities until the BOQ is re-run next day.
- Existing bulk tools either do not preview (change is already applied) or only exist for Revit, as a paid add-in.

**The idea.** A **Sheet Sync** mode with six steps:

1. **Pick the scope.** Whole model, current selection, or a filter (level, type, category, grid range, pour,
   "elements missing parameter X"). The scope is shown as a count: "312 elements selected".
2. **Export a smart sheet.** One row per element, stable key column (IFC GlobalId / Revit UniqueId, hidden but
   locked), plus visible columns the user chooses from a preset: `Mark | Level | Grid | Type | b | D | Length |
   Grade | Cover | Top level | Zone | Pour No. | custom...`. Read-only columns (computed volume, area, weight) are
   greyed and formula-protected; editable columns have data-validation drop-downs (grades, levels, standard
   section list). Presets per discipline: Columns, Beams, Slabs, Footings, Walls, "Data fill".
3. **Edit anywhere.** In Excel / Google Sheets / LibreOffice, or in the built-in grid (virtualised table with
   sort, filter, fill-down, find-and-replace, paste from clipboard). A colour legend marks changed cells.
4. **Import and diff.** Drop the sheet back. The tool matches rows by key, validates each cell (unknown grade,
   negative size, level that does not exist, duplicate mark, size smaller than the design minimum) and shows a
   **change list** with old -> new per cell. Errors are listed with row and cell address; valid rows can still be
   applied while invalid ones are held back.
5. **Ghost preview in 3D.** Changed elements show a translucent *new* shape on top of the old one (green bigger,
   red smaller), plus a live **impact badge**: delta concrete m3, delta steel kg (if rebar exists), delta
   shuttering m2, delta cost (via the existing BOQ / costing rates). Click a row -> camera flies to the element;
   click an element -> row highlighted. Clash hint: "C12 grows to 450 x 600: now overlaps beam B34 by 50 mm."
6. **Apply as one undo step + audit.** One transaction in Revit (single undo), one command in cad2bim (undo
   stack, saved to IFC / `.shkp`). A `sheet-sync-log.csv` (who, when, element, field, old, new) is written next
   to the model so the change can be audited or rolled back (a "reverse sheet" is generated automatically: import
   it to undo the whole sync).

**Why it is a wow.** The user changes a number in a cell and sees the building respond, with a rupee / dollar
value on it, before committing. It feels like a 3D BOQ editor. For a BIM team lead it is also a control tool:
the sheet becomes the single place where engineer, modeller and QS agree numbers.

**Daily-use examples.**
- "Apply the new column schedule from the design office": paste the engineer's table, review ghost, apply.
- "Re-mark all beams by grid and level": sheet formula builds `B-L3-A1-A2`, import, schedules become stable.
- "Fill Zone / Pour No. / Package for the whole tower" by fill-down on filtered rows.
- "Change grade above L10 to M30 and see how the concrete cost moves" before the client call.
- "Rename 40 sloppy types to the office standard" with find-and-replace and a preview.
- Pre-handover: export every element missing a required IDS property (IDS Gatekeeper link), fill it, import.

### Implementation

1. **Core module** (`packages/engine/src/sheetsync/` in cad2bim; shared pure-TypeScript logic with unit tests):
   `Scope`, `SheetSchema { key, columns[{name, path, type, editable, validation}] }`, `exportRows()`,
   `parseSheet()`, `diff(rowsOld, rowsNew)`, `validate(row, rules)`, `impact(diff)` (delta volume / area / weight /
   cost via existing quantity and BOQ code), `applyPlan(diff)`. No UI dependencies, so it also runs in the MCP agent
   helper (`@cad2bim/mcp`) for "edit 300 elements from a CSV" through an AI agent.
2. **Excel I/O**: reuse the XLSX writer already used for quantities and BBS; add a second lightweight reader
   (SheetJS community build or the existing `xlsx` vendor in Structura) in a worker. Hidden key column, sheet
   protection on read-only columns, data-validation lists, a `_meta` sheet with model name / revision / hash so a
   sheet exported from Rev B cannot be silently imported into Rev C (a warning with "import anyway" instead).
3. **cad2bim (IFC) write path**: map columns to IFC properties (`Pset_*Common`, `Qto_*BaseQuantities`, type
   profile, custom `Pset_ComputerHelp`), reuse the existing IFC writer and its parameter-editing code, with
   property fixtures from `tools/fixtures/make_props_ifc.py` as test models. Geometry-driving edits (b, D, height)
   regenerate the extrusion profile for the preview and on save.
4. **Revit write path (pyRevit, Advance panel)**: new `Sheet Sync.pushbutton` next to One Filter / Multi Filter /
   Parameter Combination, reusing their preview-before-write pattern and the modeless window + `IExternalEventHandler`
   reference tool; write inside one `TransactionGroup` so Ctrl+Z reverts it all. Keep the matching, validation and
   diff in plain Python modules (Revit-free) so they are testable in `tests/`.
5. **Bridge sync**: the cad2bim Bridge for Revit already pairs, edits parameters and pushes live updates; use it to
   let the browser grid edit a Revit model directly ("Revit mode"), with the same preview and log.
6. **Ghost preview**: reuse the viewer's overlay layer (`overlays.js` in Structura, three.js overlay in cad2bim); one
   instanced translucent mesh per changed element, coloured by sign of delta. Do not rebuild model geometry until
   Apply.
7. **UI**: a bottom-panel tab "Sheet" (cad2bim) with scope chips, preset menu, grid, change list, impact badge,
   Apply / Revert buttons; full keyboard flow (arrows, Enter, Ctrl+Z).
8. **Tests / docs**: `docs/design/sheet-sync.md`; fixtures for duplicate marks, wrong key, stale revision hash,
   locale numbers ("1,5" vs "1.5"), units (mm / m / ft-in), formulas that return text.

MVP (2 weeks): export / import for Columns and Beams in cad2bim with change list and validation, no preview. Then
ghost preview and impact badge (week 3), Revit-side pyRevit tool (week 4), presets, reverse sheet, audit log,
bridge "Revit mode".

### Future update

- **Live link mode**: keep the sheet open and re-sync on save (file watcher in the desktop bridge), so Excel formulas
  such as "b = span/12 rounded to 25 mm" drive the model live.
- **Rule columns**: `If span > 6 m then D = 600` expressed as small rules next to the sheet; show which rows the
  rule changed, one click to accept.
- **Design hand-off**: import the ETABS / PlanWin section results directly as the sheet (Twin Check link) so
  "model equals design" becomes a single import.
- **Change Radar link**: every Sheet Sync becomes a named revision with the standard BOQ / BBS delta report.
- **Bill Painter / Pour Planner link**: bulk-set Pour No., Package and Subcontractor for painting and 4D sequencing.
- **Smart Mark generator**: auto-numbering of marks by grouping identical size + grade + reinforcement, so "C1..C9"
  is produced consistently and a legend sheet feeds Drawing Autopilot.
- **Google Sheets / SharePoint connector** for teams that cannot exchange files by email (optional, off by default;
  files stay local unless the user connects it).
- **AI helper**: "make all L5-L8 columns 450 x 600, grade M35" through the MCP helper builds the sheet change,
  shows the ghost preview and waits for confirmation; numbers stay deterministic.
- **Templates marketplace**: share presets (e.g. "CPWD column schedule", "BS 8666 mark standard") as small JSON files.

### Market check

- Paid / Revit-only: Autodesk BIMLink, Ideate BIMLink, DiRoots OneFilter and similar add-ins; per-seat licences, no
  3D ghost preview of the change, no quantity / cost impact, no IFC.
- Scripting: Dynamo + Excel nodes, pyRevit scripts, IfcOpenShell / BlenderBIM spreadsheets: free but need skill, no
  safe preview or audit trail, rarely used by the daily modeller.
- A free, in-browser, IFC-native and Revit-capable **"export -> edit in Excel -> diff -> ghost preview + cost impact
  -> apply as one undo step + reverse sheet"** workflow is not offered as far as I could tell (verify once more
  before launch). It extends existing pieces in the toolkit: parameter editing and IFC writer in cad2bim,
  Export Schedule and Parameter Combination in pyRevit, BOQ / costing in Structura and PlanWin AI Pro.
