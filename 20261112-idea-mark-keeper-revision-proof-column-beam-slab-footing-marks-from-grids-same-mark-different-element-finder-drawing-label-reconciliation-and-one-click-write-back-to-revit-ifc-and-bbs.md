# 20261112 Idea: Mark Keeper — revision-proof marks (C1, B12, S3, F2) for every column, beam, slab and footing: numbered by grid and level, kept when the architect moves things, "same mark, different element" found, drawing labels checked against the model, and written back to Revit, IFC and the BBS

## Entry 1

```
date-time:   2026-11-12 (scheduled run)
idea name:   Mark Keeper (the one place where marks are made, kept and checked: rule-based numbering from
             grids, levels and section/rebar, memory of each element's mark across revisions by GUID and
             position, a conflict finder for one mark on two different elements and two marks on identical
             ones, a check of the marks written on the DXF/PDF drawing against the 3D model, and write-back to
             Revit's Mark parameter, the IFC Tag, and the BBS and schedules)
potential:   High for daily use, medium-high for the "wow". Every modeller and BBS engineer touches marks every
             day, and they go wrong in the same quiet ways: the architect shifts a column and half the marks
             renumber; "C7" on level 3 is not the C7 on level 2; the BBS says B14 and the drawing says B15; the
             site asks about "C4" and three people find three columns. Marks are the keys that tie the
             drawing, the model, the BBS, the BOQ and the site together. When they drift, every other tool in
             this toolkit (BBS Weaver, Mark Merge, Pour Planner, Cage Manual, Steel Ledger, Site QR Pass)
             inherits the error.
             Trending now: AI agents and MCP in Revit are the 2026 talking point (Autodesk's Revit 2027 and
             community MCP servers), and the loudest worry under those posts is "AI output looks right and is
             quietly wrong" (miscounted elements, wrong references). Marks are the check that keeps a
             prompt-built or agent-built model honest. A "no silent mark errors" story fits that mood.
             The wow: press Number. Every column lights up with its mark in a clean grid-based order, level by
             level. Load the architect's new revision: a "mark ladder" shows 212 marks kept, 6 new, 4 gone, 1
             moved, and not one old mark changed. Click a red tag: two different beams share B12 (one 300x600
             with 16 mm bars, one 300x650 with 20 mm). That is a conflict nobody saw, found in a second.
```

### Description

**The pain.**
- Marks are typed by hand or numbered with a one-off script, then break at the first revision. Revit has no
  built-in way to keep a mark attached to "the same column" when the column moves a little or is recreated by
  the pipeline (a new DXF becomes a new set of elements, with new GUIDs).
- The same mark is used for two different things. The usual cause is copy-paste: a beam is copied, its size is
  changed, its mark is not. It reaches the BBS, and then the site, as one bar list for two beams.
- Two marks are used for one thing, so the BBS has two lines and the steel yard cuts two batches
  (this is the opposite problem to Mark Merge, which merges on purpose; Mark Keeper only finds and reports it).
- The drawing and the model disagree. The structural drawing says C14; the model says C16. Nobody checks until
  a bar arrives on site.
- Naming is a house habit (C1, C-A1, C3-L2, 3C14). A team that joins a new project rewrites it by hand.

**What exists today (checked 2026-11-12 with a web search; not exhaustive, and the tools' current Revit
versions were not checked).**
- Numbering tools for Revit exist and some are cheap: AGACAD Sort Mark (paid, has a grid-location method),
  Ideate ReNumber (paid, covers structural columns), NumberingEz on the Autodesk App Store (about $10), and
  Revit's own Numbering Options with Type Comments prefixes. Dynamo scripts are common for "name a column by its
  grid intersection". Source pages: [AGACAD Sort Mark](https://agacad.com/products/tools4revit/sort-mark),
  [Ideate ReNumber](https://ideatesoftware.com/support/help/ideate-apps/ideate-renumber/using-ideate-renumber/numbering-methods),
  [Autodesk help: numbering columns and framing](https://help.autodesk.com/cloudhelp/2027/ENU/Revit-DocumentPresent/files/GUID-4444BC91-9580-455F-94F0-CFD496A6A46A.htm),
  [forum thread on grid-intersection names](https://forums.autodesk.com/t5/revit-structure-forum/naming-structural-columns-based-on-grid-intersection/m-p/12178274).
- What these did not appear to do (from the pages above, not tested): keep a mark stable across a *new revision
  of the model*, find "same mark, different content", compare marks with the labels on a DXF/PDF, or work on an
  IFC without Revit open. They number what is in the open Revit file, once.
- In this repo: Mark Merge (type diet) merges marks on purpose with an IS-safe upsize. Rev Merge keeps hand
  edits across a new DXF. Trace Lens shows an element's DXF source. Grid Lock aligns models by grids. The
  cad2bim app already has "paste marks to select" (0.38.0, quick-wins B2). **No idea here makes the marks
  themselves, remembers them across revisions, or checks them against the drawing.**

**The idea in one line.** Number by a rule you choose, remember the number through every revision, and prove the
drawing, the model and the BBS all say the same mark.

### How it works (user view)

1. **Pick a naming recipe** from a short list, or write one in a line: `{type}{seq}`, `{type}-{grid}`,
   `{type}{level}{seq}`, `{level}-{type}-{x}{y}`. Presets for common Indian site habits (C1, B12, S3, F2) and
   for ISO 19650-style names. The recipe is saved per project and per client, so the next project starts right.
2. **Number.** Order is by grid, left to right then bottom to top (or by your choice), per level. Columns that
   stack get the same base mark on every floor, with a level suffix if the recipe asks. Beams are numbered along
   the grid line they lie on. Slabs are numbered by panel, footings by column.
3. **Memory across revisions.** Mark Keeper stores, for each element, a small fingerprint: its GUID if still
   present, else the nearest element of the same type within a tolerance (the default 150 mm, editable), else
   the same grid cell and level. When a new revision loads, each element gets its old mark back. New elements
   get the next free mark in the recipe. Removed ones are listed, and their marks are *not reused* unless the
   user asks (so "C14" never means two different columns in the project history).
4. **The conflict finder.** Two lists:
   - *Same mark, different element* (section, grade, rebar, length, or level-set differ). Each row shows the two
     in 3D, side by side, with the differences. Choices: give the second a new mark, or accept as a typed
     variant (C7, C7a).
   - *Different marks, identical element* (and the same rebar). Listed with the number of bars and the steel
     tonnes that a merge would save, as an estimate. It does not merge; that is Mark Merge's job.
5. **Drawing reconciliation.** Drop the structural drawing (DXF or DWG; PDF text if present). The text labels
   near each column, beam and footing are matched to the model by position. The result is a three-way table:
   *drawing says* / *model says* / *BBS says*, with a single colour for each row. Mismatches pin to the
   drawing line and to the 3D element. This reuses the pipeline's reading of text and tables (cad2bim's CAD core
   already reads text and `*T` table blocks).
6. **Write back.** One click writes the marks to Revit's *Mark* parameter (through the Revit add-in), to the
   IFC `Tag` and a `Pset_cad2bim_Mark` property, and refreshes the BBS, BOQ and schedules that group by mark.
   A change log lists what changed, per element, so the BIM lead can see it.
7. **Ask in words** (through the existing English commands or the MCP helper): "renumber beams on level 4 by
   gridline B", "which marks are used twice?", "what was C14 in revision A?". The agent calls the same
   operations, so a prompt-built model gets the same checks.

### What the user gets

- A mark that does not change when the drawing is revised, so the BBS, the pour card and the site list stay
  valid.
- A list of conflicts with the elements in 3D, instead of finding them on site.
- A one-page *Mark Report* (PDF/Excel): marks used, new, gone, moved, conflicts, drawing mismatches.
- A saved recipe for each client.
- The BIM lead sees which marks changed in a revision and why.

### Time saved (estimates, to be measured)

- A 10-floor frame has roughly 600 to 1,000 marked members. Numbering by script or by hand and checking against
  the drawing is commonly half a day to a day per revision (an estimate from the way the work is usually done;
  not measured here). Mark Keeper aims at minutes, because the recipe and the memory are reused.
- The larger saving is avoided rework: one wrong mark found on site costs more than the numbering ever did.
- To measure: pick three past projects with two revisions each; count marks changed by a baseline script and by
  Mark Keeper; count conflicts found that were not known.

### Implementation

**Reuse in the repo.**
- The CAD engine's reading of DXF text, tables and entities for the drawing side (`packages/cad-engine`).
- The IFC model, the live model and the one cad2bim API (0.84.0) to read elements and write marks, with the
  edit as one undo step.
- The BBS (`packages/engine/src/bbs/`) and BOQ already group by member; they would read the mark from the
  `Pset_cad2bim_Mark` property.
- Mark search (0.38.0) and plain-English commands (0.81.0) for "go to C14".
- The Revit add-in (`bridge/revit`) for the Mark parameter write-back, and Update .c2b for round trips.
- The MCP helper (`packages/mcp`) for the agent operations.

**New parts.**
1. `marks/recipe.ts`: parse the recipe, grid-order the elements, produce the mark.
2. `marks/memory.ts`: the fingerprint store, kept in the `.c2b` project file (no server). Matching by GUID,
   then by nearest within tolerance, then by grid cell. A confidence is kept for each match, and the weakest
   ones are shown for a person to confirm. Never silently guess.
3. `marks/conflicts.ts`: group by mark; compare section, grade, rebar set and length; also group by content to
   find identical elements with different marks.
4. `marks/drawing.ts`: match DXF/PDF text to elements by position; three-way table.
5. A "Marks" panel in the app with the ladder view, the conflict list and the report. UI from the design
   system; colours from tokens.
6. Write-back: IFC property set; Revit parameter through the add-in; a change log.

**Rules to keep (from the repo's own).**
- Files stay on the device; nothing is uploaded. The AI part, if used, is the user's own agent over MCP.
- No GPL code. Everything above is the team's own code on MIT/Apache libraries already in use.
- Cut and test with the user's own drawings in a scratch folder; no drawings in git.
- A version bump, changelog entry and decision row when it ships (the repo's rules).

**Build order (small first).**
1. Week 1: the recipe, the numbering by grid and the conflict finder on an open IFC. Value already, no Revit
   needed.
2. Week 2: memory across two revisions, with the confidence list.
3. Week 3: drawing reconciliation from DXF text.
4. Week 4: write-back (IFC first, then the Revit add-in), the report, the agent operations.

**Checks before it is called done.**
- Number three real projects twice with a revision between; the share of marks kept (target: all marks of
  unmoved elements kept; moved elements listed, not silently renamed).
- Seed known errors (copy-pasted beam, swapped label) and confirm all are found; count false alarms.
- Round trip to Revit and back: the Mark parameter matches the IFC.

### Risks and honest limits

- Matching by position can pair the wrong two elements when a column moves a long way. That is why every weak
  match is shown to a person with a confidence; the tool never guesses quietly.
- DXF text near an element can belong to another element (crowded plans). The three-way table shows the
  distance and offers "pick the label" by hand.
- A PDF without a text layer needs OCR; this is left to Plan Tracer's work, not repeated here.
- Market claims are from one quick search and the tools' pages. Check current versions and prices before the
  idea is used in a pitch.

### Future update

- **Mark Memory for the firm**: a recipe library per client, with the habits learned from past projects.
- **Mark → site**: the Site QR Pass and Cage Manual read the same marks, so a scan always opens the right
  element, even after a revision.
- **Mark history in the Change Radar**: "C14 moved 200 mm and its steel changed by 18 kg" in one line.
- **IDS rule**: export "every column has a unique, recipe-valid mark" as an IDS file, so any openBIM checker can
  test it.
- **Cross-discipline marks**: the same recipe for architecture and MEP tags, tied to Grid Lock.
- **Auto-fix suggestions** for conflicts (new mark or typed variant) learned from what the team accepts.
