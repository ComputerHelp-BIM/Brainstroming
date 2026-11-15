# 20261115 Idea: Frame Ferry — the CH drawing or the 3D model becomes an ETABS-ready analysis model (.e2k) in one click, cleaned and checked, with labels that never drift — and ETABS's results ferry back onto the 3D elements

## Entry 1

```
date-time:   2026-11-15 (scheduled run, UTC)
idea name:   Frame Ferry (model -> ETABS analysis model, and results back, in the browser, nothing uploaded)
potential:   Very high for the structural team and for any BIM office that models for a structural consultant.
             Today the same building is drawn up to three times from the same drawing: once in CAD, once in
             Revit/cad2bim, and once more by hand in ETABS. Every drawing revision means redoing the ETABS
             part. That is the biggest repeated, low-value effort left in a structural design workflow, and
             it is where wrong-by-one-grid mistakes get in.
             Time saved: from how this work usually runs, NOT measured here. First test: take one real
             20-storey frame, have an engineer build it in ETABS by hand with a stopwatch, run Frame Ferry on
             the same drawing, and compare time AND the two models (weight, first periods, reactions).
             Do not quote hours before that test.
             Market check (2026-11-15, two searches, not exhaustive). It is NOT empty, so I do not claim unique:
               - CSiXRevit (CSI's own Revit link): needs Revit and ETABS on one Windows machine and works only
                 as well as the Revit analytical model is prepared; users report partial imports and manual
                 fixing (forum reports, not a controlled test).
               - ETABS's own IFC import: supported by CSI for IFC2x3 and IFC4 (members, slabs, structural
                 members); one user reported slabs coming in tilted. Output still needs manual clean-up.
               - ETABS's DXF/DWG import: plan geometry (architectural plans and grids), no full analysis
                 model that I could find.
               - A third-party plugin ("Axiom", vendor page on a domain I do not know, unverified) says it reads
                 a structural PDF with AI and creates ETABS elements; it needs the user's own LLM key and the
                 vendor says to review the result carefully.
               - Freelance "drawing to ETABS model" services, and LinkedIn posts of people driving ETABS through
                 its API from an LLM prompt (an anecdote of a model built "from one prompt").
             What I did not find: a free, deterministic tool that runs in the browser, starts from the same
             CH drawing the modeller already built the 3D model from, cleans the geometry the way an analyst
             would (snap, split, offsets, diaphragms), keeps every ETABS label tied to the IFC element, and
             brings the ETABS results back onto the 3D elements. That is the gap. Only a claim from two
             searches; verify before marketing.
             Extends tools already in this repo (it does not repeat them):
               - docs/design/etabs.md (cad2bim): plans to READ an .e2k and show ETABS results, and notes that an
                 .e2k can be WRITTEN for geometry and sections. Frame Ferry is that write side, plus the clean-up
                 and the label map. It shares the same .e2k reader/writer pair (build once).
               - Detail Drop: ETABS design -> rebar. Frame Ferry is what feeds ETABS in the first place.
               - Load Loader: wall, parapet and finish loads as ETABS line loads. Frame Ferry puts them in the
                 same file, on the right members.
               - Quake Preview: a screen BEFORE the analysis model exists. Frame Ferry builds that model.
               - Twin Check, Pad Packer, Load Path X-ray, Basis Book: all want ETABS numbers on 3D elements.
                 Frame Ferry is the one door through which those numbers arrive.
               - Mark Keeper and Rev Merge: marks and revisions that survive; Frame Ferry gives ETABS the same
                 treatment (labels do not change when the drawing changes).
             Trend: one LinkedIn post this month shows ETABS built from a text prompt through an LLM, and the
             press talks about AI agents inside engineering tools with an engineer in the loop. Frame Ferry is
             the safer version: no guessing, every step shown, and the engineer decides. I could not measure
             LinkedIn or Instagram counts; the sources are search results.
             The wow: a CH drawing becomes a 3D model; click "Send to ETABS"; a 20-storey building turns into
             lines and shaded panels (the analysis view), a "readiness 96%" ring appears, three red dots mark
             two floating beams and a column that does not reach the footing. Fix them, click again, an .e2k
             downloads. Open it in ETABS: it is already there, grids, storeys, sections, diaphragms. After the
             run, drop ETABS's tables back: the 3D model colours by column axial ratio and drift.
```

### Description

**The pain.**
- The structural engineer needs a line model: columns and beams as lines on centrelines, slabs and walls as shell
  areas, supports at the bottom, a diaphragm per floor. The architect's drawing and the BIM model hold the same
  building as solids, in different places and with different ends.
- So someone re-enters it: grids, storeys, sections, then every column, beam, slab, wall. For a tall building this
  is days. Every revision, they do it again or patch it, and the ETABS model and the drawing quietly drift apart.
- Both mistakes people make: a beam that ends 40 mm short of its column (a floating joint, a mechanism), and a
  label that now means a different member after a revision. Both are found late, in the results.
- The results then live only inside ETABS. Nobody sees them on the 3D model that the drawings come from.

**The idea in one line.** The model the team already built is the single source; Frame Ferry carries it across to
ETABS as a clean analysis model, remembers who is who, and carries the results back.

### How it works (user view)

1. **Open the project.** A cad2bim project built from a CH drawing, or any IFC (Revit, ArchiCAD, Tekla). Press
   *Send to ETABS*.
2. **It builds the analysis view.** Storeys from the levels, grids from the grids. Columns become vertical lines,
   beams become lines at their top, slabs and walls become panels. Footings become supports; the engineer picks
   fixed, pinned or springs (a plain choice, with the default shown).
3. **It cleans like an analyst would.** All of it listed, none silent:
   - beam ends within a tolerance snap to the column or supporting beam;
   - a beam crossing another is split at the crossing, so it is connected;
   - a beam off-centre to its column gets rigid end offsets from the section sizes;
   - joints within a tolerance merge; a column that crosses a floor is split at the floor;
   - slabs follow the beam bays; a bay with no beam edge and no wall is reported, not filled.
4. **It sets what ETABS needs to run.** A rigid diaphragm for each floor (the engineer can switch to semi-rigid),
   a mass source, materials from the concrete grades in the model (the elastic modulus from the grade, with the
   formula shown), section property modifiers as proposed defaults the engineer can change, and load patterns with
   the names the office uses. Wall and finish loads come from Load Loader when it is present.
5. **A readiness check, before ETABS.** A list and a 3D heatmap: floating members, columns that stop short,
   unsupported beams, slab holes, members shorter than a tolerance, storeys not aligned, units that look wrong.
   A score, and a click on any item flies to the element. The file is allowed to be written with open items; the
   open items are written into its header.
6. **Output.** An `.e2k` text file (units in kN-m or the office's choice), plus an Excel list of sections and a label map.
7. **Bring the results back.** After the run the engineer exports ETABS's tables (reactions, member forces,
   drifts, periods). Drop them into cad2bim: each value is attached to its element by the label map, and
   the 3D model can be coloured by axial ratio, drift or reaction. Pad Packer, Detail Drop and the design basis
   read from there.
8. **Revision.** A new drawing revision: Frame Ferry compares the new analysis view with the last one it sent
   and lists what changed (added, moved, resized, deleted) with the same labels. It writes a new full `.e2k` and
   the change list, so the engineer sees what to re-check. (Whether ETABS can take a partial update is not
   assumed; see the limits.)

### Why a BIM team would use it

- The BIM modeller and the structural engineer stop drawing the same building twice.
- Labels (C12, B104) are tied to the element, not to a position, so a revision does not scramble them.
- Mistakes that kill an analysis (floating joints, missing supports) are found in a minute, before ETABS opens.
- ETABS numbers appear on the model that the drawings and the BBS come from, so the design loop is closed.
- It runs on the device; a client's drawings are not uploaded.

### Implementation

Runs in the browser. Reuses what cad2bim already has: the Rust pipeline's elements with their marks, levels and
stable IDs; the IFC reader for other tools' models; the IFC writer; the issue list and 3D pins; the AI helper and
the MCP agent protocol for an optional natural-language front end.

1. **Shared `.e2k` library (one package, both directions).** Structura (same owner) already has a 1,572-line `.e2k`
   parser; docs/design/etabs.md plans to reuse it. Build the reader and the writer as a pair with a round-trip
   test: an `.e2k` read, written, read again must give the same model. Pick one ETABS version as the first
   target (the one the team uses) and test the others after.
2. **Analysis view builder.** A pure function from "elements, levels, grids" to a list of lines, panels and
   supports, each with the element's GlobalId and mark. Rules are data (insertion point for beams, which face a
   wall's centreline takes, how a core wall is split into piers), so an office can set its own.
3. **Clean-up engine.** Small separate steps (snap, split, offsets, merge) each with a tolerance, a count and an
   undo, in the order an analyst works. A step never moves a member by more than its tolerance; anything bigger is
   a finding for a person.
4. **Label map.** A table: ETABS label, GlobalId, mark, storey, revision first seen. Labels are made once and
   kept; a deleted element retires its label rather than letting it be reused. Stored in the project file.
5. **Readiness rules.** Same pattern as the other checkers: each rule has an id, a sentence, a tolerance, a 3D pin.
6. **Writer.** Stories, grids, materials, sections, joints, frames, areas, diaphragms, mass source, load patterns
   and supports, in the exact section order ETABS expects. Anything the model does not hold (load combinations,
   response spectrum, design preferences) is left out on purpose or written from the office's template file, never
   invented.
7. **Result import.** Read ETABS's table exports (Excel or text) into a result store keyed by label; then by the
   map to GlobalId. Show as colour-by, a table, and a pin.
8. **Test plan.** (a) Round trip: an existing ETABS model -> `.e2k` -> cad2bim -> `.e2k` -> ETABS; compare total
   weight, first periods, base reactions. (b) Three real drawings, built by hand and by Frame Ferry, compare as
   above and report the differences and why. (c) Plant faults (a floating beam, a missing support, a
   misaligned storey) and report how many the readiness check finds and how many false alarms it raises. The team
   sets the tolerances for "same"; do not claim a number before the test.

Effort guess, not measured: the shared library and the writer for frames and slabs first (about 3 to 4 weeks for
one developer, ETABS available to test), the clean-up and the readiness check next (about 2 weeks), the result
import and colours last (about 1 to 2 weeks).

### Future update

- **Push to ETABS directly.** On a machine with ETABS, use its API to open the model and run it, so "Send to ETABS"
  can also press Run and bring the tables back with no export step. Needs a small local helper, in the spirit
  of the existing Revit bridge; always ask before it changes anything.
- **SAP2000 and SAFE.** The same analysis view is written for them (SAFE for the raft and flat slabs).
- **Steel and composite.** Link with the steel idea (Bolt Up): members as steel sections with the connection
  flags.
- **Tolerance profiles.** Per-office profiles saved as rules, shared with the team.
- **What-if.** Move a column in cad2bim, write the changed file, compare the two ETABS runs side by side.
- **Plain-language front end.** "Make the ground floor columns 600 x 600 and rerun" through the user's own AI key,
  with a preview of the change list before anything is written.
- **Authority pack.** Feed the results into the design basis report (Basis Book).

### Improvements for tools already in the repo

- **`Frames in one call` / `Create through the API`:** let every generated member carry its analysis line (start, end,
  insertion point) as an IFC property. It is almost free at creation and saves recomputing it later.
- **The pipeline's checks (CH v2):** add a small set of "analysis-ready" notes (beam end not on a support, column
  stopping short of the footing). They are cheap, they already know the geometry, and they help even without ETABS.
- **docs/design/etabs.md:** change the "write an `.e2k`" line from "partly" to a named phase, and build the reader and
  the writer as one package so Load Loader, Detail Drop and Frame Ferry share it.

### Reel idea for LinkedIn / Instagram

15 seconds: a CH drawing (DXF) on the left; it builds into a 3D tower; one click on *Send to ETABS*; the tower
dissolves into lines and panels; three red dots, a counter "3 problems, fixed", a green "Ready"; cut to the same file
open in ETABS; then the 3D model coloured by column load. End card: "Draw it once." Caption: "How many times do you
draw the same building?" (Show a real run with ETABS on screen and say what was real. Do not post invented numbers.)

### Honest limits

- I could not run ETABS here. The `.e2k` format and the exact section order differ between ETABS versions and must be
  tested on the team's version before any claim.
- An analysis model is a judgement, not geometry. Rigid or semi-rigid diaphragms, section modifiers, how a core is
  meshed, and what is a support are the engineer's calls. Frame Ferry proposes defaults and shows them; it does not
  decide silently, and the engineer remains the author of the analysis.
- Snap and split are tolerance-based. A bad tolerance can join things that should not join; that is why each step
  counts, shows, and can be undone.
- A partial update of an open ETABS model may not be possible from a text file; the first version writes a whole file
  and a change list.
- Results import only helps if the ETABS table exports are in the format the reader expects; the first version
  supports one named export.
- The market check is two searches; the vendor page for the AI plugin is unverified, and paid tools I did not try may
  do parts of this.
- Time saved is an estimate to be measured with a real building.

### Sources (checked 2026-11-15)

- CSiXRevit and its known limits, user reports: [Eng-Tips thread](https://eng-tips.com/threads/exporting-revit-model-to-etabs.484060),
  [CSi BIM interoperability](https://www.csiamerica.com/bim), [CSiXRevit manual (Scribd copy)](https://www.scribd.com/document/688680124/CSiXRevit-Manual).
- ETABS IFC import: [CSi wiki: Import IFC into ETABS](https://web.wiki.csiamerica.com/wiki/x/coAW),
  user report on slab orientation: [Graphisoft community](https://community.graphisoft.com/t5/Collaboration-with-other/Export-IFC-and-DXF-file-to-ETABS/m-p/331753).
- ETABS DXF/DWG import (plans and grids): [CSi ETABS help, File > Import](https://docs.csiamerica.com/help-files/etabs/Menus/File/Import/Import.htm).
- AI and LLM routes to ETABS (vendor and personal pages, claims unverified): [Axiom plugin page](https://eagleseye.pages.dev/axiom),
  [Viktor: automate ETABS workflows with AI](https://www.viktor.ai/blog/254/automate-etabs-workflows-ai),
  [a LinkedIn profile post on Claude with ETABS](https://tr.linkedin.com/in/izzet-samet-can-2266a923b),
  [cognitivefuture.ai review, mid-2026](https://cognitivefuture.ai/ai-structural-analysis-software/).
- Inside this repo: cad2bim `docs/design/etabs.md` and `docs/design/structura-lessons.md`; ideas Detail Drop (20261021), Load Loader
  (20261113), Quake Preview (20261105), Basis Book (20261114), Pad Packer (20261104).
- These are search results and vendor pages. I could not measure LinkedIn or Instagram trends directly.
