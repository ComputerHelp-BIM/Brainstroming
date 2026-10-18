# 20261018 Idea: Area Statement Pro — live FSI / carpet / built-up / RERA area statement from the model, in the municipal approval format

## Entry 1

```
date-time:   2026-10-18 (scheduled run, fourth entry of the day)
idea name:   Area Statement Pro — "Move a wall, watch the FSI meter move."
             Reads the IFC / cad2bim / Structura model (spaces, slabs, walls, balconies, shafts, stair/lift cores,
             parking) and builds the complete area statement an architect or liaison engineer submits to the
             municipality: plot area, permissible vs achieved FSI/FAR, floor-wise built-up, carpet, balcony,
             terrace, staircase/lift/refuge areas (free-of-FSI vs counted), parking count and area, ground coverage,
             open-space/setback check, and RERA carpet-area per flat. Output: XLSX in a rule-pack format, DXF/PDF
             "area table + hatched area plan" ready to paste on the drawing sheet, and a live FSI gauge in the viewer.
potential:   High. Every project, every revision, every approval round repeats the same hand work: tracing polygons
             in AutoCAD, "list" area, copy to Excel, argue about which balcony counts. Typical cost is 3-6 hours per
             revision for one tower, and one wrong deduction means an objection letter and a week lost. A tool that
             recomputes in seconds and explains every number saves modeller + architect time on each of the 5-15
             revisions a project sees, and removes the highest-risk number in the whole approval file.
             Trending angle: "FSI in 10 seconds", "RERA carpet area auto-check" and "approval drawings without
             Excel" are strong LinkedIn/Instagram hooks (before/after screen recording: drag a balcony, the
             gauge turns amber then red at 100% FSI). Fits the daily-productivity posts the BIM community engages with.
             Market: Snaptrude shows an FSI chart in its (paid, subscription) design platform; Hong Kong BD publishes a
             Revit/ArchiCAD area-check plug-in for its own rules only; Revit "Area Schemes" need manual boundary
             work. No free, open, rule-pack-driven tool for Indian-style FSI/RERA statements working on IFC + DXF.
```

### Description

**The pain (daily life of a BIM modeller / architect / liaison engineer).**
- The area statement is rebuilt from scratch every time the plan changes: polygons traced on the DXF, `AREA`/`LIST`,
  paste to Excel, hand-subtract shafts, add balconies at 50%, check against FSI. 3-6 hours per revision.
- Rules differ by city and change by circular: what is "free of FSI" (stairs, lifts, fire refuge, parking, services,
  podium) differs between authorities. People remember the rules instead of the sheet encoding them.
- Carpet area under RERA (net usable area excluding external walls, shafts, balconies) is calculated differently
  from built-up area. Mismatches between the sale agreement and the plan are a legal risk.
- Sales and design teams ask "if we enlarge this unit by 0.6 m, are we still inside FSI?" and get an answer next day.

**The idea.** An **Area Statement Pro** panel with three parts:

1. **Rule packs (JSON/YAML, editable, versioned).** Each authority is a small file: FSI base + premium, which
   element classes are counted / exempt / counted-at-percent (balcony 50%, canopy, ramp), setback table by height,
   parking ratio, ground-coverage cap, RERA carpet definition. Packs ship for a few examples (generic, Mumbai-style,
   Pune-style, RERA); users copy and edit. Every output prints the pack name and version.
2. **Area engine.** Walks the model: space/room boundaries from IfcSpace (or auto-generated from walls when the
   model has none), net vs gross by wall thickness rule, classification by name/property (`Pset`, layer, or a
   mapping table), then applies the rule pack. Every number is *traceable*: click "Floor 7 carpet = 1,842.30 m2" to see
   the list of spaces and deductions that produced it, highlighted in 3D.
3. **Outputs.**
   - XLSX statement (floor-wise table, summary, compliance flags) and PDF.
   - DXF "area table + colour-hatched area plan" per floor for the drawing sheet, using the layer/standards of the
     existing Drawing Autopilot sheet set.
   - Per-flat RERA carpet-area schedule (flat no., carpet, balcony, terrace, exclusive areas).
   - Live gauge in the web viewer: achieved FSI / permissible, remaining area in m2, with "what-if" drag handles.

**Why it is unique / extends the repo.** Reuses what already exists: IFC reading (cad2bim, Structura), DXF writing and
sheet layouts (Drawing Autopilot), Excel round-trip (Sheet Sync), Change Radar (area delta between revisions), Plan
Tracer (spaces from a scanned PDF plan). It adds the one deliverable none of them produces — the statutory area number.

### Implementation

1. **Phase 1 (MVP, 1-2 weeks).** Python + `ifcopenshell`: read IfcSpace/IfcSlab/IfcWall, compute gross/net per storey
   using shapely on 2D footprints, classification via a mapping CSV, one generic rule pack, XLSX via `openpyxl`.
2. **Phase 2.** DXF input path (closed polylines on named layers, using `ezdxf`) so it works for 2D-only projects;
   area-plan hatch + table DXF output; trace panel (element GUID list per number).
3. **Phase 3.** RERA per-flat schedule (flat grouping by IfcSpace property / zone), what-if via a small web viewer
   (`web-ifc` + `three.js`) with a draggable boundary and live recomputation of the gauge.
4. **Validation.** Golden-file tests: 3 hand-checked buildings (Excel statement vs tool output within 0.01 m2);
   rule-pack schema validation; warnings for overlapping or unclosed spaces.
5. **Safeguard.** Output always stamped "Computed from model <hash>, rule pack <name/version> — to be verified by the
   licensed architect"; the tool does not replace statutory sign-off.

### Future update

- Change Radar link: area/FSI delta report between revisions ("Rev C added 42 m2 FSI, 18 m2 free of FSI").
- Zoning import (plot polygon from DXF/GeoJSON) with automatic setback and height-envelope check in 3D.
- Sales-sheet export: unit-wise carpet/saleable area with loading factor.
- Rule-pack library maintained by the community; AI helper that reads a DCPR/UDCPR clause PDF and drafts a rule pack
  for human review.
- Green-building add-on: open-space, permeable area and OSR percentage statements.
- Instagram/LinkedIn "Model Reel" preset that records the gauge moving as a unit is resized.
