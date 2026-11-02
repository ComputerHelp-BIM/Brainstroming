# 20261102 Idea: Grid Lock — drop in the architect's, MEP's and steel consultant's models and they snap into one place by matching grids and columns: wrong origin, rotation and units fixed in one click, with a proof score and a clean federated file

## Entry 1

```
date-time:   2026-11-02 (scheduled run)
idea name:   Grid Lock — "Four consultants, four models, four different origins. One floats 2 km away, one is
             rotated 17 degrees, one is in millimetres where the others are in metres. The modeller drops all
             four on the project. In about five seconds each model flies into place, found by matching its
             grid lines and column centres to the structure. A card per model says: 'Moved 1,842.3 m east,
             rotated -17.0 deg, scale x0.001. 31 of 32 columns land within 5 mm. Confidence 98%.' One column
             does not match and is marked 'check: 85 mm off'. Press Lock and the aligned set is saved as
             one federated project, with the correction written into each file (IfcMapConversion / site
             placement) so the next person never repeats it. Next revision of any file: dropped on top, it
             re-aligns itself with the same fix."
potential:   High. Every coordination meeting starts with 'your model is in the wrong place'. Every office
             loses time to it, and it comes back with every new revision of every consultant's file. Cheap to
             build because cad2bim already reads grids, levels and columns from DXF and IFC. Strong
             short-video value: models scattered across the screen, then one click and they all snap
             together with a green score. A common LinkedIn subject (BIM coordination, shared coordinates,
             'single source of truth', federated models).
             Time today (estimated, not timed): finding why a model is off, trying Origin-to-Origin vs
             Shared Coordinates, exporting IFC again from the consultant, 1-3 hours per model per
             project start, plus 15-30 minutes again whenever a revision arrives re-offset. With Grid Lock:
             under a minute per model, plus a re-align on each revision that takes seconds. Estimate:
             3-10 hours saved per project start, 20-60 hours over a project's revisions.
             Market: Revit Shared Coordinates and Acquire/Publish, BIMcollab Zoom's georeferencing
             option, Navisworks append/transform, Autodesk's APS 'aps-model-alignment' sample and
             Blender/Bonsai OffsetObjectPlacement all depend on a shared survey point agreed earlier, or on
             the person picking points by hand. They fail when the file arrives wrong. None that we found
             matches grids and columns automatically to fix origin, rotation AND units together, scores
             the result, and writes the correction back. Paid tools do parts of it behind a licence.
```

### Description

**The pain.** Consultants model in their own origin. The architect uses a project base point, the MEP firm
imports a DWG at its own corner, the steel detailer works in millimetres, someone exports IFC2x3 with no map
conversion. Loaded together, the models do not line up. The usual fixes are manual: agree a survey point, pick
points in a viewer, ask for a re-export. Then Rev C arrives and the offset has changed again.

**The idea.** The structure's grid and columns are a fingerprint every discipline shares. Grid Lock finds the
move, turn and scale that make each model's grid and columns fall onto the reference model's.

1. **Choose the reference** (usually the structure, or whatever has the grids).
2. **Drop other models** (IFC, Revit export, `.c2b`, DXF/DWG). Grids, columns, walls corners and level heights
   are extracted from each.
3. **Find the transform.** Units first (compare grid spacing ratios: x1, x0.001, x0.0254, x0.3048), then
   rotation (compare grid directions), then translation (match grid intersections and column centres).
   Several candidates are scored, and the best is kept.
4. **Show proof.** A card per model: move, rotation, scale; how many columns and grid crossings land within
   5 mm / 25 mm; vertical level offset; confidence. Misfits are marked on screen.
5. **Lock.** Nothing moves until the person presses Lock. The correction is saved with the project, and can be
   written back into the exported IFC (`IfcMapConversion`, site placement) so Navisworks, BIMcollab and Revit
   see the same alignment.

**What makes it feel like magic.** Models arrive scattered and wrongly scaled, then fly together. Ghost
overlays show where each file was before. A 'wobble test' toggle blinks old and new positions. A final
'cube test' places the agreed 1 m cube that BIM teams use to prove alignment.

**Re-align on revision.** The fix is remembered per file name and author. A new revision is checked against
the saved fix first: 'same offset as before, 100% match' is the usual answer. If the consultant moved the
origin again, the fix is recomputed and the change is reported ('origin shifted 12.4 m since Rev B').

**Outputs.**
- Aligned federated project (`.c2b`) and an aligned IFC per model.
- One-page alignment report (PDF/Excel): the transform per file, the match scores, misfit columns, who
  should fix what ('MEP model is 85 mm off at C14: please check').
- A short message for the consultant, ready to paste. Nothing is sent by the tool.

### Implementation

Built from what cad2bim has. This note only describes the idea; nothing in the cad2bim source is changed by it.

| Piece | Uses | New work |
|---|---|---|
| Reading grids, columns, levels | IFC reading (`packages/engine`, web-ifc); the CAD core and Rust pipeline already find grids, columns and levels in DXF/DWG | one 'fingerprint' extractor with the same output for every format: grid lines, grid crossings, column centres, level heights |
| Matching | none | unit guess from spacing ratios; rotation by direction histogram; translation by voting on crossing and column pairs (RANSAC-style); score by matched-within-tolerance count |
| Shared coordinates | roadmap 'Shared coordinates' (project base, survey point, true north) | write the found transform as a shared-coordinate placement per model |
| Many disciplines in one project | roadmap 'Every discipline in one project' | apply the transform at load, show the card, save in the project |
| IFC write-back | IFC writer | add or fix `IfcMapConversion` and site placement in the exported file |
| Revision memory | project file and revision design (see Rev Merge) | per file: author, name, last transform |
| Viewer | three.js viewer, ghost and diff colours | 'before / after' ghost, wobble toggle, misfit markers |

**Phases.**
- *Phase 1 (about 1-2 weeks of work, my estimate):* IFC and `.c2b` models, translation and rotation, units,
  score card, Lock. Structure as reference.
- *Phase 2:* DXF/DWG input, level offsets in Z, write back `IfcMapConversion`, the PDF report and the
  consultant message.
- *Phase 3:* the Revit add-in button (apply the found transform as Shared Coordinates in the live model),
  per-office rules, fallback of picking points by hand when a model has no grid.

**Licences and rules.** Fits cad2bim's rules: files stay on the device, no GPL code, no upload to read or
convert, colours from tokens.

**Honest risks.**
- A model with no grids and no repeating columns (a pure MEP model) gives weak fingerprints. It then falls back
  to walls, shafts and lift cores, and says the confidence is lower. Manual two-point picking remains.
- Symmetric buildings can give two equally good answers (rotated by 180 deg). Show both and ask; never choose
  silently.
- True geographic georeferencing needs a surveyed point; this tool aligns models to each other, not to the
  Earth. It says so, and can import a survey point once to anchor the set.
- The result is only as good as the reference. Different grid revisions between models show up as misfits;
  that is a finding, not an error.

### Future update

- **Drift watch:** on every revision, report if a model's alignment moved since last time, and who changed it.
- **Wrong-level finder:** detect a model whose floors are one level off (a basement counted as ground).
- **Auto-clash after lock:** run clash and flush-fit checks right after alignment (links to Flush Fit, Sleeve
  Punch, Hanger Hero).
- **Layer and unit sanity:** warn on mm/m mix-ups and mismatched level names before alignment (links to Layer
  Whisperer).
- **Site context:** anchor the aligned set to terrain and neighbouring buildings (roadmap 'Site context').
- **Team profile:** remember each consultant's habits ('this firm always exports at its corner, 1,842 m east').
- **Share the proof:** a one-image before/after of scattered vs aligned models for a coordination meeting or
  LinkedIn, with no model data leaving the device unless asked.

Sources: [BIMcollab: coordinating IFC models with world coordinate information](https://helpcenter.bimcollab.com/en/articles/326917-coordinating-ifc-models-with-world-coordinate-system-information),
[Autodesk: georeferencing BIM using Revit](https://forums.autodesk.com/t5/community-blog-aec-english/georeferencing-bim-using-revit/ba-p/14073927),
[NovEdge: align linked Revit models using Shared Coordinates](https://novedge.com/blogs/design-news/revit-tip-align-linked-revit-models-using-shared-coordinates),
[OSArch community: shifting IFC coordinates](https://community.osarch.org/discussion/comment/19430/);
in cad2bim: `docs/design/next-milestones.md` (Shared coordinates; Every discipline in one project).
