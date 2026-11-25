# 20261125 Idea: Sheet Delta — the architect sends only a PDF ("Rev D, please update"); overlay it on the last one, cloud every change, map each change to the model elements it touches, and get a to-do list

## Entry 1

```
date-time:   2026-11-25 (scheduled run)
idea name:   Sheet Delta (PDF/Image Revision Overlay + Change-to-Element Map + Model To-Do + Unseen-Change Catcher)
potential:   High. The most common daily event in a BIM modeller's week is "new architect drawings
             arrived". Very often there is no DWG, no revision cloud and no change list, only a PDF
             that says "Rev D". Today the modeller flips between two PDFs, or overlays them by hand,
             and still misses things. A missed change is found at site, as an RFI or a rework. Sheet
             Delta shows every change in one view, says which model elements each change touches, and
             turns them into a checklist. Estimated saving (an estimate, not measured): 1 to 3 hours
             per revision per discipline, and fewer "we modelled Rev C" surprises. Very shareable demo:
             two PDFs dropped, 40 seconds later every change is clouded and the beams it hits are lit
             up in 3D.
```

### Description

**The pain (weekly, for every modeller who gets drawings from an architect or consultant).**
- The email says "Revised plans attached". The files are PDFs. Changes are not clouded, or only some are.
- The modeller compares two PDFs by flicking between tabs, or lays one over the other in a PDF tool.
  Small changes are missed: a door moved 300 mm, a column shifted, a sunk depth edited in a note, a
  level changed in a schedule table.
- Even when found, the modeller then has to work out which elements in the model are affected, and
  keep a private list.
- Nobody can later say "Rev D had 23 changes; we did 21; two were never seen." That is what becomes a claim.

**What exists today (a short search; re-verify before saying "nothing like it").**
- Autodesk Forma Data Management is reported to include Drawing Change Analysis (mentioned in a LinkedIn
  post by an Autodesk Expert Elite member after AU 2026, single anecdotal source); it is a paid
  subscription tool and compares sheets, not model elements.
- Bluebeam Revu (Compare Documents) and Procore, ACC and similar tools overlay two PDFs and cloud the
  differences. They are paid, and they stop at the picture: they do not know your model, so they cannot
  say "this change moves column C14 and beam B22".
- Free overlay tricks exist (Acrobat compare, open-source diff scripts), also picture-only.
- Trade press in 2026 is full of AI inside drawing review and documentation (see
  [BDC Network on BIM trends 2026](https://www.bdcnetwork.com/sponsored/article/55343350/the-bim-trends-that-will-define-aec-project-delivery-in-2026),
  [Allplan on AI in AEC 2026](https://www.allplan.com/blog/from-ai-design-to-autonomous-construction-how-predictive-data-centric-workflows-and-ai-agents-are-reshaping-aec/)).
  I could not read LinkedIn or Instagram from here; this is trade press and one LinkedIn mention, not
  feed data. A search is not proof of absence.
- In this repo: **Rev Merge** (20261102) handles a new architect *DXF* with a three-way merge; it
  only writes PDFs out and does not read PDF input. **Change Radar** diffs two *models*. **Markup
  Mirror** reads *redlines* inside a PDF. **Issue Desk** clouds changes the *modeller* made. **Trace
  Lens** and **Plan Tracer** read one drawing or scan into 3D. None compares two incoming PDF or image
  revisions and maps the changes to model elements. Sheet Delta fills that gap and hands DXF-grade
  revisions to Rev Merge.

**The idea.** A **Sheet Delta** window in the cad2bim web app (and the same through the agent helper).

1. **Drop two sheets.** The old revision and the new one (PDF, PNG, JPG or DXF), or two whole folders;
   sheets are paired by sheet number or title-block text. Unpaired sheets are listed ("new sheet",
   "sheet removed").
2. **Align.** The pages are lined up automatically (scale and offset found from the drawing's own grid
   lines, column marks or title-block corners; the person can confirm with two clicks on matching
   points). Scale is read from the sheet's dimensions or the scale note when present.
3. **Overlay.** Old in red, new in green, unchanged in grey, with a slider and a flicker button. Changes
   are grouped into numbered clouds (nearby differences merge into one cloud).
4. **Say what changed.** Each cloud is typed: *moved*, *added*, *removed*, *text or number changed*
   ("Slab level +3.150 to +3.100", "Beam 230x450 to 230x600"). Text changes are read with the
   PDF's own text layer where it exists, so a one-digit edit in a note or schedule is not missed;
   scanned pages use on-device OCR.
5. **Map to the model.** With a model open (or the DXF-built model), each cloud is placed in model
   coordinates and linked to the elements under it: "Cloud 7: 3 elements, column C14, beams B22 and
   B23, slab S9". Click a cloud and those elements light up in 3D with a section box around them.
6. **To-do and status.** Every cloud becomes a numbered item: *To do*, *Done*, *Not applicable (not
   structural)*, *Ask (RFI)*. Not applicable and Ask need a one-line reason. The list exports to Excel,
   BCF issues or a PDF "Rev D response sheet".
7. **Unseen-change catcher.** When the modeller issues the next model version, Sheet Delta lists clouds
   still *To do* and, for changes marked Done, checks whether the mapped elements really changed
   (a moved column should have moved). "Cloud 7 is marked Done but C14 has not moved."

**The wow 1: two PDFs, 23 clouds.** Drop Rev C and Rev D, and in under a minute every change is
red/green with numbers. The one-digit slab level change in a note is on the list.

**The wow 2: click a cloud, see the beam.** The drawing change lands on the model element, in 3D, with
the old and new size beside it.

**The wow 3: the honest closing line.** "Rev D: 23 changes, 19 done, 3 not applicable, 1 asked." One
shareable screenshot, and the evidence if a claim comes later.

### Implementation

How this fits the repo as it is today (read from `CLAUDE.md` and `docs/`, not from code; check names
before starting). This is a brainstorm; no cad2bim source was changed.

- **Where it lives.** Page reading and comparison belong in the CAD side of the repo
  (`@cad2bim/cad-engine`, with its own version), because the DXF/DWG reader and 2D viewer are there. The
  window is in `apps/web` using the design system. The model-element lookup uses the IFC engine's
  element bounds (`@cad2bim/engine`). Dependencies point down only. Each bump gets a CHANGELOG entry,
  README and `guide.ts` entry per `docs/workflow.md#versions`.
- **PDF reading.** The `roadmap.md` already lists PDF and image underlays under "More to import". Sheet
  Delta can share that reader. Use a permissively licensed library (PDF.js is Apache-2.0; check
  `docs/decisions.md` licence rules, no GPL, not even in a worker). Vector PDFs: read paths and text
  directly, which gives exact geometry. Raster/scans: render to a bitmap.
- **Alignment.** Vector PDFs: match line and text features, then a robust transform fit (RANSAC for
  translation, rotation of 0/90/180/270, uniform scale). Raster: feature matching on a downscaled
  image, then refine. Always show the fit residual; if it is poor, say so and ask for two points.
  Never silently overlay a bad fit.
- **Difference finding.** Vector: compare primitives after alignment within a tolerance (in mm at
  sheet scale, shown and editable); text compared by string at the same location with a fuzzy
  match for moved labels. Raster: pixel difference, blur, threshold, connected regions, merge
  neighbours into clouds. Ignore the title-block revision area by default (it always changes) but list
  its changes in a separate "title block" group.
- **Typing a change.** Matched primitive pair with an offset = moved; unmatched in new = added;
  unmatched in old = removed; text pair with different strings = changed (show old and new text).
  Numbers in text are parsed so "+3.150" and "+3.100" show the delta.
- **Model mapping.** Register the sheet to the model by the grid (the app already draws and names
  grids) or by two picked points; save the transform per sheet in the `.c2b` project file so the next
  revision reuses it. Cloud bounding boxes are tested against element bounds on that level; ties are
  ranked by overlap. If no model is open, the window still works as an overlay and list.
- **To-do and BCF.** Items are stored in the project (so they survive reopening) and exported as BCF
  (Issues BCF is on the roadmap as planned). Status history is kept with a timestamp, not a name,
  until accounts exist for the project.
- **Catcher.** On a new model revision, re-run the mapping: element bounds or type changed since the
  cloud was created = evidence of Done. Report both directions: Done without a model change, and a
  model change with no cloud (hand to Change Radar).
- **Agent helper.** `sheet_delta_compare(old, new)`, `sheet_delta_clouds(sheet)` and
  `sheet_delta_set_status(id, status, reason)` through the MCP helper (see `docs/agent/protocol.md`).
- **First version (smallest useful).** Two vector PDFs of the same size, aligned by two clicks,
  overlay with clouds and text-change list, saved per project. Model mapping, scans/OCR, folders,
  BCF and the catcher come after.
- **Test.** Use three real pairs of architect PDFs kept out of git ("people's drawings never go in
  git"), plus a small synthetic pair made in the repo's tests: a plan where one column moves 300
  mm, one door is removed, one level note changes. Measure and report only what was measured:
  recall of the planted changes, false clouds per sheet, alignment residual, seconds per sheet, and
  minutes a modeller needs for the same check by hand.
- **Rules to respect.** Everything runs on the device; no file is uploaded. No outside CDN for code.
  OCR must be bundled and licence-checked (Apache-2.0 / MIT only). No AI and no key needed. Plain
  short wording in the app and no helper text (the design rule).

### Future update

- **Hand-over to Rev Merge.** When the new revision also arrives as a DXF, send the confirmed clouds
  into Rev Merge's three-way merge so manual model edits are kept.
- **Cost on each cloud.** Show the concrete, steel and rupee effect of each cloud using Change Radar and
  Rate Rocket numbers, so a variation note is drafted while the list is built.
- **Multi-discipline.** Compare architect, MEP and structural revisions in one run and flag a change
  that touches two disciplines (links to Sleeve Punch and Flush Fit).
- **Change trail per element.** Click any element to see which sheet revisions touched it and what
  was decided (links to Basis Book and Trace Lens).
- **Phone capture.** Photograph a printed marked-up sheet at a site meeting and compare it to the
  issued one the same way.
- **Honest scoreboard.** After five projects, publish measured recall and hours saved, not estimates.
