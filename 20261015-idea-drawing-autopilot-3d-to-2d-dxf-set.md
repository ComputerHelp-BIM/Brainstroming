# 20261015 Idea: Drawing Autopilot — "3D to 2D" auto-dimensioned DXF/PDF structural drawing set (plans, schedules, sections, bar callouts)

## Entry 1

```
date-time:   2026-10-15 (scheduled run)
idea name:   Drawing Autopilot — one-click, auto-dimensioned, auto-tagged DXF + PDF drawing set generated from the
             model (IFC / Excel workbook / PlanWin / Structura), with a live "draft preview" in the browser
potential:   Very high. cad2bim already does 2D -> 3D; this is the missing reverse leg, 3D -> clean 2D. Every
             modeller spends 30-60 % of project time on drawing production: tagging, dimensioning, column / beam /
             footing schedules, sheet layout, revision clouds. Revit's auto-dimension is weak, Dynamo scripts are
             fragile, and good "drawing automation" tools (Tekla / Allplan / paid plug-ins) cost thousands per
             seat. A free, browser-based, layer-standard DXF set is not available.
             LinkedIn/Instagram angle: "AI generates the full structural drawing set in 60 seconds",
             "generative drafting", "BIM to CAD without Revit sheets" — a split-screen reel (IFC left, drawing
             draws itself line by line right) is highly shareable.
```

### Description

**The pain (daily life of a BIM modeller / draughtsman).**
- Model is "done", but the client wants DWG/DXF + PDF: plan per floor, column layout, beam layout, footing plan,
  schedules, general notes. That is 2-5 days of dimensioning and tagging, redone at every revision.
- Revit view templates / tags need a perfect model and still need manual cleanup; exporting to DWG gives layer
  mess (the dxf-cleaner problem) and tags that float.
- Site wants a *plain* DXF to open in ZWCAD/AutoCAD on a cheap laptop; the BIM file never reaches them.

**The idea.** A **Drawing Autopilot** panel that turns the model into a standards-clean drawing set:
1. **Pick a drawing recipe** (templates, not code): *Column Layout*, *Beam Layout (per level)*, *Footing Plan*,
   *Slab Reinforcement Plan*, *Column / Beam / Footing Schedule sheet*, *Section through a beam*, *GA with grid*.
2. **Auto-generate**: grids with bubbles, centre-line dimensions chain-by-chain (grid-to-grid, column-to-edge,
   cumulative), member tags from the mark/type already in the model (C1, B12, F3), level and elevation marks,
   north arrow, scale bar, title block (project, revision, date, drawn/checked, from `INFO` sheet).
3. **Schedules drawn as tables in the DXF** (and Excel/PDF): size, grade, main bars, stirrups, quantities —
   straight from the same data used for BBS/BOQ, so drawing and schedule can never disagree.
4. **Smart dimension placement**: avoids overlaps (collision-free text placement, leader lines), keeps
   dimension rows outside the plan, merges equal spans ("4 @ 3000 = 12000").
5. **Revision clouds from Change Radar**: elements changed between rev A and rev B are cloud-marked on the new
   sheets automatically, with the revision table filled.
6. **Layer standard** (AIA / ISO 13567 / company) selectable; output is DXF R2010 with clean layers, BYLAYER
   properties, no junk — so it passes the dxf-cleaner audit by construction. PDF generated from the same geometry.

**Why it wows.** The browser shows the sheet *drawing itself* (animated line-by-line), then "Download DXF + PDF".
Change one column size in the workbook, hit regenerate: drawing, schedules and revision cloud update in seconds.

**Why unique / not paid.** Existing options are Revit sheets (manual, licence heavy), Dynamo (needs expertise),
or paid drawing-automation products. Browser-only, standards-driven, IFC/Excel-in and DXF-out is not offered free.

### Implementation

Fits the existing stacks; no new server.
1. **Geometry source** — reuse what exists: cad2bim IFC element model (`packages/engine`), pyRevit-addons Excel
   workbook rows (columns/beams/footings with marks and levels), Structura `.e2k` parse, PlanWin result.
   Normalise to a small `PlanModel { grids, levels, members[{type, mark, outline, section, rebar}] }`.
2. **Plan cutter**: for each level take the member footprints at the cut plane (columns = rectangles/circles,
   beams = centre-line + width, footings = outline) → 2D polylines in model coordinates.
3. **Drawing engine** (`packages/engine/src/drawing/`): pure functions emitting an in-memory entity list
   (LINE, LWPOLYLINE, TEXT/MTEXT, DIMENSION as exploded lines+text for compatibility, HATCH optional).
   Modules: `grid`, `dims` (chain + overall + collision-aware placement), `tags`, `schedule-table`,
   `titleblock`, `revcloud`.
4. **Writers**: DXF writer (the dxf-structural-generator skill is a good reference for entity/layer layout),
   PDF via jsPDF (already vendored in Structura), SVG for the live preview canvas.
5. **Recipes as JSON** (layer map, text height by scale, which tags/dims to draw) so a company can fork its own
   standard without touching code.
6. **pyRevit bridge**: "Draw in Revit instead" option writes the same recipe into a Revit drafting view / sheet
   (detail lines + tags) for teams that must stay in Revit.
7. **Tests**: golden DXF snapshots on fixture models; round-trip check — import generated DXF into cad2bim and
   confirm the 2D->3D result matches the source model; run the dxf-cleaner audit as a CI gate.

MVP (2-3 weeks): grid + column layout + column schedule + title block, DXF + PDF export, one layer standard.
Then beam/footing plans, collision-aware dimensions, revision clouds, Revit drafting-view output.

### Future update

- **Section/elevation recipes**: auto beam longitudinal section with stirrup spacing zones and bar callouts
  from the rebar data; couples naturally with Bar Optimizer cut lists.
- **Rule-based "drawing checker"**: flag missing dimensions, unlabeled members, overlapping text; one-click fix.
- **Template learning**: upload a company DXF as sample; the tool infers layers, text heights and title block.
- **AI helper (optional)**: "draw the roof beam layout at 1:100 for L5" in plain language selects the recipe;
  geometry stays deterministic.
- **Two-way sync**: edit a dimension or move a tag in the browser preview; write the change back to the workbook /
  IFC property.
- **Pair with Pour Planner and IDS Gatekeeper**: pour-sequence sheet and "model ready" certificate printed as
  part of the same drawing set.
