# 20261015 Idea: Twin Check — does the BIM model and the rebar actually match the structural design?

## Entry 1

```
date-time:   2026-10-15 (scheduled run)
idea name:   Twin Check — ETABS / PlanWin design vs IFC model vs rebar (BBS): size mismatch finder and
             "Ast required vs Ast provided" 3D heatmap
potential:   High. The structural design (ETABS .e2k, PlanWin .pwai) and the BIM/drawing model drift apart on
             every project: a column is 400x600 in the analysis but 350x600 in the model, a beam was upsized
             after a drift failure but never updated, a bar schedule was made from an old design run. Finding
             this today means a senior engineer and a modeller comparing two screens for half a day, and it is
             often found on site. Twin Check finds it in under a minute, shows it in 3D, and produces the
             correction list. It joins the three tools the team already owns (Structura, PlanWin AI Pro, cad2bim)
             into one design-to-BIM digital thread that no free tool offers.
```

### Description

**The pain (daily life of a structural BIM modeller).**
- The analysis engineer changes sizes/grades and sends a new ETABS file. The modeller is told "some beams
  changed", not which. They re-check by eye, or rebuild.
- Reviewers ask "is the steel in the drawings what the design needs?" Nobody can answer quickly, because design
  output (Ast required per member) lives in ETABS/PlanWin and provided steel lives in the BBS/model.
- Errors found late (at site, or at the proof-consultant stage) cost rework. This is the classic
  "analysis model is not the BIM model" problem, and it is a hot topic on LinkedIn/Instagram under
  *digital thread*, *analytical-to-physical*, *single source of truth*, *design-to-fabrication* and
  *AI-checked drawings*.

**The idea.** A **Twin Check** mode, in cad2bim (browser, files stay on the PC), that loads two things side by
side: the **design twin** (ETABS `.e2k` parsed by the Structura parser, or a PlanWin AI Pro `.pwai`/results JSON)
and the **BIM twin** (IFC already open in cad2bim, or DXF-to-3D output), plus optionally the BBS Excel. It then:

1. **Auto-aligns the two models.** Matches grids and storey levels first (ETABS stories to IFC storeys by
   elevation), then members by position and type (column at grid B-3 on L4 to the IFC column nearest it).
   Shows a match table and a confidence flag; the user can fix any pairing by clicking two members.
2. **Mismatch report** with a 3D colour layer:
   - **Section size** differs (b x d, diameter, wall/slab thickness) with the delta.
   - **Grade** differs (M25 vs M30, Fe500 vs Fe500D).
   - **Missing in model / missing in design** (ghosted in 3D: red outline = designed but not modelled,
     grey = modelled but not in the analysis).
   - **Position / offset / level** differs beyond a tolerance (e.g. >25 mm).
   - **Releases / cantilever / support** hints that disagree (e.g. a beam modelled continuous but analysed pinned).
3. **Ast required vs Ast provided heatmap** (the wow screen). For each beam/column/slab/footing, pull required
   steel from the design run (PlanWin design results; ETABS concrete-design export CSV when supplied) and the
   provided steel from the BBS or from the IFC rebar/property data. Colour the 3D model:
   green >= 100 %, amber 90-100 %, red < 90 % provided, blue > 150 % (over-reinforced / wasteful).
   Click a red beam: "Required 1,420 mm2 top at support, provided 3-16 phi = 603 mm2... see BBS mark B12-T1".
   Add a total: "Steel in drawings is 6.2 % above design requirement: 4.1 tonnes of possible saving" which is
   exactly the number a project head loves.
4. **Utilisation view.** Same 3D, coloured by demand/capacity ratio (axial, flexure, shear, drift) from the
   design run, so the model doubles as a design status board: "which members are at 0.95+ and why".
5. **Fix list for the modeller.** One sorted table: member, what to change, new value, source (design run
   id). One-click *Copy as Excel*, *Export BCF* (so the issue reaches Revit) and, where cad2bim can edit the
   property, *Apply to model* (resize, regrade) with undo; with the existing Revit Bridge the change flows into
   the Revit element.
6. **Sign-off sheet (PDF).** "Design-to-model reconciliation report": project, design run id/date, model
   revision, % members matched, open mismatches, steel summary. Gives engineers a clean document to issue
   with drawings.

**Why it is a wow.** Drag in an ETABS file onto an IFC and the building lights up: red columns that are the
wrong size, a few grey phantom beams, and a heatmap of every beam's steel adequacy, with one big number,
"3 sections wrong, 12 members under-reinforced, 4.1 t over-steel". It is the first time the analysis engineer
and the BIM modeller look at the same picture and agree.

### Implementation

1. **Design-twin readers** (`packages/engine/src/twin/design/`):
   - ETABS: reuse Structura's `parser.js` / `parse-worker.js` logic (joints, frames, areas, sections, stories,
     materials, grids) ported to TypeScript, or imported as a shared package. Output: neutral
     `DesignMember {id, type, story, p1, p2, section, material, ratios?, astReq?}`.
   - PlanWin AI Pro: add `cli export-design model.pwai --out design.json` (member list, sections, grades, D/C
     ratios, Ast required) built from existing `design/runner.py` + `quantities.py`; cad2bim reads the JSON.
   - Optional ETABS concrete-design CSV table import (columns/beams Ast) for ratios and Ast.
2. **Aligner** (`.../twin/align.ts`): storey match by elevation (tolerance), grid match by nearest line,
   member match by type + midpoint within tolerance + orientation (KD-tree over midpoints). Deterministic, with
   a manual override list stored in the `.shkp` project file as a new optional section (specified in
   `docs/format/`).
3. **Comparator** (`.../twin/compare.ts`): pure functions that emit typed findings
   `{kind: size|grade|missing|extra|offset, memberA, memberB, delta, severity}`; rule thresholds are
   settings. Unit-tested with the existing sample/50k IFC fixtures plus a new matching `.e2k` fixture made by
   `tools/fixtures/`.
4. **Ast engine** (`.../twin/ast.ts`): provided steel per member from the BBS (mark to element id mapping that
   already exists for the BBS-to-Excel export) with a fallback to IFC rebar quantities; required steel from the
   design twin. Provided/required ratio per section (support / mid-span for beams, per face for columns).
5. **3D layers**: reuse the instanced colour attribute approach (no geometry rebuild) from the viewer for
   heatmaps; ghost meshes for missing members; legend + tooltip in `packages/ui`. New bottom-panel tab
   "Twin" next to QA and BOQ; drop zone for `.e2k` / `.json`.
6. **Outputs**: Excel (existing writer: sheets Matches, Mismatches, Ast, Fix list), PDF sign-off (existing PDF
   path), BCF zip (shared with the IDS Gatekeeper idea), and *Apply to model* via the existing property edit
   commands (undo/redo through the command stack) and Revit Bridge parameter editing.
7. **Perf and privacy**: parse in a worker; target < 3 s match + compare for 20k members; everything runs in the
   browser, files never uploaded (same promise as Structura and cad2bim).
8. **Docs**: `docs/design/twin-check.md`, tolerance defaults and the ETABS table names in `docs/research/`,
   roadmap and homepage update.

MVP (2 weeks): ETABS `.e2k` + IFC, storey/grid/member matching, size/grade/missing mismatch table and 3D
colouring, Excel export. Then PlanWin JSON, Ast required-vs-provided heatmap, PDF sign-off, BCF and Apply to model.

### Future update

- **Live loop with PlanWin AI Pro**: *Optimise sizes* changes a column, Twin Check shows the model delta
  instantly, and one click pushes the new sizes into cad2bim / Revit ("analysis changed, model updated in 10 s").
- **Reverse direction (model to analysis)**: build the ETABS `.e2k` / STAAD file *from the IFC* (analytical
  model extraction), aligning with PlanWin AI Pro's roadmap item "IFC/Revit export" in the opposite direction.
- **Combine with Change Radar and IDS Gatekeeper** (see `20261014-idea-change-radar-revision-diff-boq-delta.md`
  and `20261014-idea-ids-qa-autofix-bcf.md`): a design revision is a revision too, so its model delta flows into
  the cost delta; "design twin matches" can be an IDS-style readiness rule before issue.
- **Pour Planner link** (see `20261014-idea-pour-planner-4d-rmc-formwork-cycle.md`): refuse to release a pour
  card for a pour that has open Twin mismatches ("L4-P2 has 2 under-reinforced beams").
- **More analysis sources**: STAAD `.std`, SAP2000 `.s2k`, RISA / Robot via neutral JSON; multi-code
  (ACI 318 / Eurocode) Ast rules.
- **Drawing check**: compare the DXF reinforcement detail drawings (PlanWin `detail_dxf`) with the BBS and the
  design, closing the third corner (design, model, drawing).
- **Steel optimisation hint**: for over-reinforced members, suggest the bar arrangement that meets Ast with
  fewer tonnes and fewer cut lengths, with a site-friendly bar-size rationalisation.
- **Design-status dashboard** for the project head: % members matched, % designed, % detailed, per level.

### Market check

- Autodesk's Revit-ETABS/Robot link and CSiXRevit exist, but they need Revit and the vendor tools, mostly push
  geometry one way, and stop at "synchronised": they do not give a side-by-side mismatch report, an Ast
  required-vs-provided heatmap or a steel-saving number. Tekla Structural Designer, Solibri and
  Autodesk Platform Services checks are paid and generic.
- Free options are Python scripts (IfcOpenShell + custom parsers) that individuals write per project; nothing
  in-browser for an ordinary modeller.
- A free, in-browser, privacy-first **design vs BIM vs rebar reconciliation** is not a standard tool today. The
  parts already exist in this toolkit: Structura's ETABS parser, PlanWin AI Pro's design results and BBS, and
  cad2bim's IFC viewer, colour layers, BBS-to-Excel and Revit Bridge. (Competitor claims here are from general
  market knowledge; verify before external marketing.)
