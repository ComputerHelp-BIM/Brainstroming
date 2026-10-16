# 20261016 Idea: Plan Tracer — drop in any PDF or phone photo of a structural plan, get a clean layered DXF and a 3D IFC model

## Entry 1

```
date-time:   2026-10-16 (scheduled run)
idea name:   Plan Tracer — "PDF / scan / site photo of a plan -> clean Computer-Help-format DXF -> 3D IFC",
             with a human "confirm the guesses" step and a trace-quality score
potential:   Very high. cad2bim's 2D -> 3D only starts from a DXF in the Computer Help layer format. In real life
             the architect, client or old project sends a PDF, a scanned print or a WhatsApp photo, and the modeller
             spends 2-6 hours re-drawing the grid, columns, beams and walls before modelling can even begin. Plan
             Tracer removes that step: it is the missing "front door" of the toolkit, and it feeds the existing
             DXF -> 3D -> BBS/BOQ -> Revit chain. Paid equivalents (Scan2CAD, Vectorworks/AutoCAD raster-to-vector,
             Autodesk PDF import, Bluebeam/Kreo-style takeoff) either only vectorise lines without understanding
             "this rectangle is a column", or are per-seat cloud products. A free, in-browser, privacy-first tool
             that outputs structurally-aware layers does not exist for Indian / Asian small and mid-size firms.
             LinkedIn/Instagram trend fit: "AI turns a drawing into a 3D model", "PDF to BIM in 60 seconds" - the
             before/after split-screen reel is the demo.
```

### Description

**The pain (daily life of a BIM modeller).**
- 70% of incoming drawings are PDFs (plotted from CAD, so vector) or scans/photos (raster). Nobody sends clean DXF.
- Re-drawing a 30 x 40 m floor with 60 columns and 90 beams is a half-day of boring work, and errors made here travel
  into the model, the BBS and the cost.
- Autodesk/Zwcad "PDF import" gives thousands of loose lines on one layer with text exploded into polylines;
  cleaning that costs almost as long as redrawing.

**The idea.** A **Plan Tracer** step in cad2bim (browser, the file never leaves the PC):

1. **Drop a PDF page, PNG or JPG.** Vector PDF: read the drawing operators directly (exact geometry, no guessing).
   Raster: de-skew, threshold and vectorise (OpenCV.js in a worker).
2. **Calibrate scale in one click**: pick two points and type the distance, or pick a grid dimension. OCR of the
   dimension text suggests the value ("4500") so the click is a confirmation only.
3. **Structure-aware recognition, rule-based first, ML second**:
   - *Grid*: long thin dash-dot lines + bubbles with A/B/C and 1/2/3 text -> `GRID` layer, names kept.
   - *Columns*: small closed rectangles / circles, hatched or solid filled, sitting on grid intersections -> `COLUMN`
     with size read from the nearest tag ("C1 300x600" via OCR/vector text) or from the geometry.
   - *Beams*: parallel line pairs between columns with a beam tag ("B12 230x450") -> `BEAM` centreline + width.
   - *Walls*: thick double-line runs and filled bands -> `WALL`; openings from gaps and door swing arcs.
   - *Slabs*: closed regions bounded by beams, with thickness from the "Slab Thk 150" note.
   - *Text and dimensions* are kept as text, not exploded polylines.
4. **"Confirm the guesses" review screen**: every recognised element is overlaid on the source image, coloured by
   confidence (green sure, amber check, red unknown). Click to accept, change type, merge, or draw the missing one.
   Accepting a whole amber group ("these 14 rectangles are all C1") is one click. This keeps the human in charge and
   builds trust, which is why it can be used on real jobs.
5. **Export**: a clean layered DXF in the Computer Help format (opens in AutoCAD/ZWCAD, passes the DXF cleaner) *and*
   direct hand-off into cad2bim's DXF -> 3D to produce IFC4 in one go. A **Trace Report** (PDF/Excel) lists counts,
   unrecognised items, and a quality score, so the modeller can state how reliable the model is.

**Why it is unique / extensible.** It reuses what exists:
- `docs/dxf-format.md` in cad2bim already defines the layer/format contract and DXF -> 3D pipeline; Plan Tracer only
  has to *produce that format*, so every downstream feature (IFC, BBS, BOQ, Revit export, QA) works unchanged.
- The Anthropic-skills DXF structural generator and DXF cleaner in this toolkit set already know the target layer
  conventions and the "prove nothing moved" check; the cleaner can be the last step of Plan Tracer's export.
- The cad2bim MCP helper (`packages/mcp`) lets an AI agent drive it: "trace this PDF, confirm all green, build 3D".
- Structura and PlanWin AI can consume the same trace (column/beam layout) as the starting point for a design or
  reinforcement run instead of re-entering geometry.

**What makes it a "wow".** 20-second reel: a crumpled phone photo of a plan on a site table -> straightened -> grid
and columns light up in colours -> "Build 3D" -> a navigable building appears, then BBS and quantity table. Clients
immediately understand the saving.

### Implementation

Ship in stages, each useful alone:

1. **Vector PDF -> DXF (2 weeks).** `pdfjs-dist` (Apache-2.0) `getOperatorList()` gives paths, fills, transforms and
   text with positions; flatten transforms, merge collinear segments, snap endpoints, map text items to nearby
   geometry. Output through the existing DXF writer. Test on 10 real plotted structural PDFs.
2. **Scale and units (1 week).** Two-point calibration, dimension-text suggestion, unit picker (mm/m/ft-in), paper-size
   fallback (A1/A3 at 1:100).
3. **Rule-based recognisers (3-4 weeks).** Pure functions over a normalised segment/polygon list: grid detector,
   column detector (closed small polygons + grid snap + tag association), beam pairing (parallel offset search with
   width tolerance), wall bands, slab regions (planar face finding). Each returns elements with a confidence and the
   evidence it used, which feeds the review screen. Unit tests on synthetic plans from `tools/fixtures`.
4. **Raster path (3 weeks).** OpenCV.js in a Web Worker: perspective correction for photos (4-corner pick or auto
   rectangle), adaptive threshold, skeleton/Hough lines, then feed the same recognisers. Tesseract.js for tags with a
   whitelist (digits, C/B/W/S, x). Hard cap on image size with a clear "too small, retake" message.
5. **Review UI (2-3 weeks).** Overlay canvas (reuse the view and picking code in `packages/engine`), confidence colours,
   group accept, undo, keyboard-first. State is a plain JSON "trace project" saved next to the PDF.
6. **Hand-off and report (1-2 weeks).** DXF export, one-click "Build 3D" into the DXF -> 3D pipeline, trace report with
   quality score, and optional DXF-cleaner pass.
7. **Optional learning (later).** Collect user corrections (locally, opt-in export) to train a small on-device model
   for symbols that rules miss (hatch styles, firm-specific tag formats).

Risks and mitigations: scans are messy (always show confidence and require confirm; never auto-build silently), tag
formats vary by firm (editable regex templates, saved per project), huge A0 PDFs (tile and stream, process per layer),
copyright of client drawings (nothing uploaded; processing is local and the report says so).

### Future update

- **Multi-sheet projects**: trace plan, sections and schedule PDFs together; read the beam/column schedule table into
  sizes and grades so every tag becomes a fully typed member (feeds Twin Check and Bar Optimizer).
- **Elevation/section reading** to set storey heights and beam depths automatically.
- **Detailing sheets to bar data**: recognise standard reinforcement detail callouts ("8-16 dia + 2-12 dia") into BBS
  inputs.
- **Architectural plans**: rooms, doors, windows and walls for quantity takeoff and carpet area.
- **Differences between PDF revisions**: trace Rev B and Rev D and hand both to Change Radar for a revision diff
  with BOQ delta, even when the consultant only issued PDFs.
- **Mobile capture mode** with live edge guide, glare warning and multi-shot stitching for large plans.
- **Agent mode** via the MCP helper: the AI traces, flags only the amber items, asks the modeller 3 questions, and
  builds the model.

### Market check

- Raster-to-vector tools (Scan2CAD, Vectorworks, AutoCAD Raster Design) are paid and produce geometry, not
  structural members. Autodesk PDF import gives raw lines. AI takeoff products (Kreo, Togal.AI, Bluebeam Revu with
  tools) are subscription, cloud-upload, and focused on quantities, not an IFC/BBS-ready model.
- Free browser tool that goes PDF/photo -> layered structural DXF -> IFC with human confirmation and a trace-quality
  report: not found. Overlaps are only partial, and all of them need a licence or an upload of the client drawing.
- Trend note: the 2026 BIM conversation is about practical AI that removes repetitive prep work and false effort
  (clash AI, model validation, auto openings). Reading existing drawings into models is the front end of the same
  trend, and is a good fit for the cad2bim "no sign-up, files stay on your device" promise.
