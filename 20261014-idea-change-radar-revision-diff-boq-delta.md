# 20261014 Idea: Change Radar — "What changed since Rev B, and what does it cost?"

## Entry 1

```
date-time:   2026-10-14 (scheduled run)
idea name:   Change Radar — 3D revision diff with automatic quantity (BOQ/BBS) and cost delta
potential:   High. Every project gets 10–30 revisions; each one triggers hours of "what moved?" hunting,
             manual re-take-off and a rushed variation/claim note. Change Radar answers in under a minute:
             which elements were added / removed / resized / moved, how many m3 of concrete and kg of steel
             that is, what it costs in Rs, and which drawings must be reissued. Paid tools (Navisworks Compare,
             Solibri model comparison, Autodesk Construction Cloud) do part of it, are desktop/paid, and stop at
             geometry — they do not give structural quantities, BBS or cost delta.
```

### Description

**The pain.** The architect shifts a wall, the client adds a floor, the engineer bumps a column from 300x600 to
300x750. The modeller gets "Rev C attached, please update". Today they flip between two models, eyeball
differences, re-run quantities in Excel, and the QS/PM still asks "how much extra steel is this?". Missed changes
reach site as RFIs and variations. Revision control and change tracking is a recurring BIM/AEC topic on LinkedIn
(model versioning, "single source of truth", variation management, openBIM diffing).

**The idea.** A **Change Radar** mode in cad2bim: open two revisions (IFC vs IFC, or DXF-to-3D vs DXF-to-3D, or
a PlanWin AI Pro `.pwai` export) and get, entirely in the browser (files never leave the PC):

1. **3D diff view** with a before/after slider: green = added, red = removed, amber = modified (section, grade,
   level, position), grey = unchanged ghosted. Click any item for old-vs-new values.
2. **Smart matching** so the diff is not noise: match by GUID first, then by mark + level, then by geometry
   proximity/overlap (CAD-exported models usually regenerate GUIDs). Tolerance slider (e.g. ignore < 5 mm).
3. **Quantity delta** per element class and per level: concrete m3, formwork m2, steel kg (from BBS data),
   and net change as a table plus a waterfall chart ("+18.4 m3 concrete, +1.2 t steel, +Rs 2.9 lakh").
4. **Cost delta** using the rate table already used for BOQ (editable rates, same as PlanWin BOQ & cost sheet).
5. **"Reissue list"**: which levels/grids/sheets are touched, so the drafter updates only those drawings.
6. **One-click outputs**:
   - Excel "Revision Impact Sheet" (summary + element-level change log) for the QS/PM,
   - BCF with one topic per change group (viewpoint + GUIDs) so Revit users see it via the Bridge,
   - a short plain-text/WhatsApp summary ("Rev C vs Rev B: 4 columns resized, 1 beam added, +Rs 2.9 L").
7. **Revision timeline**: each comparison is saved as a revision node, giving the cumulative change history of
   the project (matches the existing revision-timeline design doc).

**Why it is a wow.** Drag two files in, a coloured 3D diff plus a rupee figure appear in seconds. It turns
a vague "please check what changed" into evidence the PM can forward to the client for a variation order.

### Implementation

1. **Diff engine** (`packages/engine/src/diff/`): runs in the existing IFC worker on the element model already
   produced by the loader. Inputs: two element lists (type, mark, level, section, material, placement, bbox,
   property sets). Steps: (a) bucket by IFC class and level, (b) match GUID -> mark+level -> nearest centroid with
   bbox IoU above threshold (Hungarian / greedy with spatial grid), (c) classify added / removed / modified /
   moved, (d) emit `{id, status, changes[{field, old, new}]}`. Pure functions, unit-tested with fixture pairs.
2. **Fixtures**: extend `tools/fixtures/make_props_ifc.py` to generate Rev A / Rev B pairs with known edits
   (resized column, moved beam, deleted slab, GUIDs regenerated) and golden diff results.
3. **Quantities**: reuse the existing quantity and BBS-to-Excel code; compute totals for A and B and subtract.
   Rates come from the same editable rate table; keep currency and unit configurable.
4. **Viewer**: two layers in three.js with per-status materials; before/after slider via clipping plane or
   opacity blend; selection sync reuses picking. Cap draw calls with instancing for 50k elements.
5. **UI** (`packages/ui`): new bottom-panel tab "Changes" (summary cards -> grouped list -> element detail) and a
   ribbon button "Compare revision".
6. **Exports**: Excel via existing writer; BCF writer shared with the IDS Gatekeeper idea (build it once);
   text summary is a template string.
7. **PlanWin AI Pro hook**: a CLI/exported JSON of the PlanWin model lets the same engine compare design
   revisions *before* modelling (`PlanWinAIPro.exe cli boq` already supports revisions; reuse its rate sheet).
8. **Perf target**: <= 5 s for two 50k-element models, off the main thread and cancellable.
9. **Docs**: `docs/design/change-radar.md`; update roadmap and `docs/ifc-compatibility.md` (matching rules).

MVP (2 weeks): IFC vs IFC match + colour diff + element list + quantity delta to Excel. Then cost delta, BCF,
timeline, DXF-to-3D comparison.

### Future update

- **4D link**: attach each change to a pour/activity and show schedule impact (e.g. "column resize on L3 adds
  half a day to the L3 pour"); import a simple Excel/MS Project schedule.
- **Structura link**: compare ETABS `.e2k` revisions (sections, storey heights, loads) in the same UI and flag
  which design changes alter the BOQ.
- **Auto variation note**: AI-drafted (Claude/Ollama pattern from PlanWin) cover note from the diff table; the
  numbers stay deterministic, AI only writes the wording.
- **Revit Bridge**: "Show change in Revit" and "apply Rev C parameters to native elements".
- **Drawing diff**: overlay two DXF sheets and highlight changed linework (revision clouds generated automatically).
- **Clash-on-change**: run the IDS Gatekeeper rule pack only on changed elements for instant re-check.
- **Team dashboard**: change count per revision and per discipline to expose churn and late design changes.

### Market check

- Navisworks Compare / Solibri model comparison / BIMcollab Zoom / ACC model compare: paid, geometry- or
  issue-oriented, no structural quantity, BBS or Rs cost delta, and not browser-based with local-only files.
- Free IFC diff scripts (IfcOpenShell / ifcdiff, IfcDiff) are developer tools: JSON output, no 3D, no costing.
- A free, in-browser, structural-aware **diff + quantity + cost + BCF + reissue list** flow is not offered today.
  Complements, and shares the BCF writer with, the IDS Gatekeeper idea (same date, separate file).
