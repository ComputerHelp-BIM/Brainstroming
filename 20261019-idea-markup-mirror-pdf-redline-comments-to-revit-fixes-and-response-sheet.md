# 20261019 Idea: Markup Mirror — consultant's red-cloud PDF comments become pinned Revit tasks, one-click fixes and a signed-off response PDF

## Entry 1

```
date-time:   2026-10-19 15:34 UTC (scheduled run, fourth entry of the day)
idea name:   Markup Mirror — "Drop the checked PDF. Every red cloud lands on the right beam, in 3D, with the fix ready."
             A new AnonGee pyRevit tool (Essential panel > Action group, next to AutoDIM) plus a small headless
             Python module that reads the marked-up PDF a consultant, checker, PMC or client sends back (Bluebeam,
             Adobe, Foxit, Drawboard, iPad pen, even a phone scan of a red-pen print), finds which Revit sheet and
             viewport each cloud sits on, maps the cloud back into model space, picks the elements inside it,
             reads the comment ("B12 to be 300x600", "add C7", "check lap", "TOS +3.450 not +3.500"), and turns
             every markup into a task card with: snapshot, highlighted elements, a proposed fix with preview,
             a status, and a Revit revision cloud placed at the same spot on the sheet. When the round is done it
             writes the reply straight back into the consultant's own PDF (green tick + reply text next to each
             red cloud) and a Comment Response Sheet in Excel.
potential:   Very high, used every week on every live project. Drawing review is the loop every BIM team lives in:
             issue sheets -> consultant marks up PDF -> modeller hunts each cloud in Revit -> fixes -> re-issues ->
             types a comment-response sheet by hand. On a typical structural GFC set (40-80 sheets) one review
             round returns 50-300 markups. Realistic manual cost per round:
               - locating each cloud in the model (open sheet, zoom, guess the element): 1-3 min x 150 = 3-7 h
               - typing the response sheet / CRS: 1-2 h
               - missed or half-closed comments found in the next round: rework + embarrassment
             Markup Mirror target: 150 markups located and grouped in under a minute; 40-60 % of them (size,
             mark, level, text/tag, missing-element comments) arrive with a ready one-click fix preview; the
             response PDF and CRS are generated, not typed. 1 working day per round becomes ~1-2 hours of review.
             Trending angle: "PDF comments to model in seconds" and "closing the review loop" are recurring
             high-engagement LinkedIn/Instagram BIM posts (AI review, Autodesk Docs markups, Bluebeam Studio
             sessions). The wow shot is built in: drop a red PDF, the 3D view fills with numbered red pins;
             click "Apply safe fixes", pins turn green, and the consultant's PDF comes back with green ticks.
             Market: Bluebeam Revu (paid subscription) is the markup standard but never touches the Revit model;
             Autodesk Docs/ACC markups and issues are paid and pinned to sheets, not mapped to elements with a
             fix; Revizto / BIMcollab / Dalux issue trackers are paid per seat and need everyone to work inside
             their platform. To my knowledge no free tool reads a third party's ordinary PDF annotations, maps
             them to Revit elements through the sheet/viewport transform, proposes the model change and writes
             the reply back into the same PDF.
```

### Description

**The pain (daily life of a BIM modeller and BIM lead).**
- Consultants do not work in your BIM platform. They print to PDF, open Bluebeam or Adobe, draw red clouds and
  type short notes. That PDF arrives by email or WhatsApp and is the real review record.
- The modeller has to read each cloud on paper, find the same sheet in Revit, find the viewport, zoom to the
  spot, guess which element the cloud means, fix it, and remember to add a revision cloud. It is manual
  geometry work done 150 times.
- Comment-response sheets (the CRS / "comment resolution" table common on Indian, Gulf and UK projects) are
  typed by hand: comment no., sheet, comment text, action taken, status. Copy-paste errors and missed comments
  are normal.
- Nothing links round 2 to round 1: "was comment 37 actually fixed in the model?" is answered from memory.
- This toolkit already generates the sheets being reviewed (BIM Generation, cad2bim, C2B importer, AutoDIM,
  Export Schedule, BBS Generator); review comments are the input that closes that loop, and today nothing
  reads them.

**The idea, as the modeller sees it.** One modeless window (same pattern as Auto Level Manager / Modeless
Window reference), Revit stays usable.

1. **Drop.** Drag in one or many returned PDFs (or a folder per review round). Each page is matched to a
   Revit sheet: first by the sheet number in the PDF page label / bookmark Revit writes on export, then by the
   title-block text on the page (sheet number, drawing number), then by file name. Unmatched pages are listed,
   never guessed silently.
2. **Read markups.** Two readers, same output:
   - *Vector markups* (Bluebeam, Adobe, Foxit, Drawboard): PDF annotation objects are read directly: Polygon
     with cloud border effect (`/BE /S /C`) = revision cloud, Square / Circle / Polygon / PolyLine = region,
     Line / arrow = callout pointing at a spot, FreeText / Text (sticky note) / callout text = the comment,
     Ink = hand-drawn pen strokes. Author, date, colour, subject and replies are kept.
   - *Flattened / scanned markups* (printed and photographed, flattened PDF): render the page, subtract the
     original issued sheet (Revit re-prints it), isolate red/coloured strokes, close them into regions, OCR the
     handwriting/text near each region. Lower confidence, clearly badged.
3. **Map to the model.** For each region on page P matched to sheet S:
   page coords -> sheet coords (scale page box to the title-block extents) -> find the viewport whose outline
   contains it -> `Viewport.GetProjectionToSheetTransform()` inverse and `View.GetModelToProjectionTransforms()`
   give model coordinates -> build a prism over the view range (plan) or view depth (section/elevation) ->
   `FilteredElementCollector(doc, view.Id)` + `BoundingBoxIntersectsFilter` / `ElementIntersectsSolidFilter`
   pick the elements actually visible inside the cloud. Tags, dimensions and text notes inside the cloud are
   resolved to the element they annotate. Arrows/leaders point to the nearest element at the arrow tip.
4. **Understand the comment.** An offline rule grammar handles the common structural comments first:
   - size: `B12 300x600`, `make 230 thk`, `col to be 450x450` -> type change / duplicate type
   - mark / name: `mark C3 not C5`, `rename` -> parameter edit
   - level / offset: `TOS +3.450`, `drop 150` -> offset change
   - missing / extra: `add beam`, `delete`, `C7 missing` -> create-from-neighbour or delete preview
   - annotation: `dim missing`, `tag`, `spelling` -> AutoDIM / tag / text fix
   - rebar: `lap`, `spacing 150 c/c`, `add 2-16#` -> link to BBS Generator / RC Automation workbook row
   - question / note only -> task with no auto-fix
   Free text that the grammar cannot place is optionally sent to an LLM (Claude) with the element list and
   the comment, returning one structured intent; this is opt-in per project and logged, and the result is still
   only a proposal.
5. **Markup Board.** One card per markup: PDF snippet image, consultant text, matched elements (click to
   select / zoom / isolate), intent, proposed fix with a before/after preview table (the One Filter Parameter /
   Sheet Sync preview pattern), confidence, and status `Open / Fixed / Rejected / Query / Not applicable`.
   Filters by sheet, author, discipline, intent, confidence. "Apply safe fixes" applies only high-confidence
   fixes the user ticked, one undo step per card.
6. **Pins and revision clouds.** A `Markup Mirror` 3D view gets numbered pins (generic annotation/model-in-place
   marker or DirectShape) coloured by status. On each sheet a native Revit revision cloud is drawn at the
   mapped region and tagged with the current Revision, so the re-issued sheet already carries the clouds the
   consultant expects.
7. **Reply.** Two outputs from the same data:
   - *Response PDF*: the consultant's own PDF with, next to every red cloud, a green stamp (tick / cross /
     question) and a reply annotation ("Revised to 300x600, see Rev B") added as normal PDF annotations, so it
     opens in their Bluebeam/Adobe with their markups untouched.
   - *Comment Response Sheet (Excel)*: No. | Sheet | Zone | Comment | Author | Elements | Action taken | Status |
     Revision, ready to send. Optional BCF 2.1/3.0 export for teams that use an issue tracker.
8. **Round memory.** A small JSON sidecar per project stores every markup with its element ids. When round 2
   arrives, repeated comments are flagged ("same as R1-37, marked Fixed on 12 Oct"), and a "verify" pass checks
   that every Fixed card's element still has the value that was applied.

**Wow moments (for the team and for posts).**
- 150 red clouds become 150 numbered pins in 3D in under a minute.
- "Apply 63 safe fixes" -> pins go green, revision clouds appear on the sheets.
- The consultant gets back their own PDF, every cloud answered in green, without anyone typing a reply.
- Before/after Health-style card: `Markups 152 | Auto-located 147 | Auto-fixed 63 | Queries 9`.

### Implementation

**Where it lives (no new platform, extends what exists).**
- Revit side: a new pushbutton `MarkupMirror.pushbutton` in the AnonGee extension, CPython 3 engine, built on the
  existing Modeless Window reference (`IExternalEventHandler` for every Revit call), Brand Guidelines components
  for the board, and the existing transaction helpers (one transaction group per Apply, one undo step per card).
- Headless core (pure Python, testable without Revit): `markup_mirror/` package with
  - `pdf_read.py`: annotations via `pypdf` (BSD) — `/Annots`, `/Subtype`, `/Rect`, `/Vertices`, `/InkList`,
    `/BE`, `/Contents`, `/T`, `/IRT` replies; page labels and bookmarks for sheet matching.
  - `pdf_raster.py`: page render via `pypdfium2` (Apache/BSD) for snippets and for the scanned-markup path; colour
    mask + contour close with `numpy`/`opencv-python-headless`; OCR via Tesseract when present (reuse whatever the
    Plan Tracer idea settles on). Avoid AGPL PDF libraries in the shipped extension because of the commercial
    license model in the README.
  - `intent.py`: the rule grammar above (regex + unit/size normaliser shared with cad2bim/C2B mark parsing,
    e.g. `C12-300X900`), optional LLM adapter behind a flag.
  - `reply_pdf.py`: writes stamps and FreeText reply annotations back with `pypdf` (incremental update, original
    annotations untouched).
  - `crs_xlsx.py`: CRS workbook with `openpyxl`, styled like Export Schedule output.
  - `bcf.py`: optional BCF zip (shared with the IDS Gatekeeper idea's BCF writer).
- Revit mapping module `markup_mirror/revit_map.py`:
  - sheet match: `ViewSheet.SheetNumber`, title-block `BoundingBox` on sheet for page scaling.
  - viewport: `Viewport.GetBoxOutline()`, `Viewport.GetProjectionToSheetTransform()` (Revit 2022+; target is
    Revit 2025 per README), `View.GetModelToProjectionTransforms()` for views with split crop regions.
  - pick: `BoundingBoxIntersectsFilter` from an `Outline` for speed, then `ElementIntersectsSolidFilter` on a
    `GeometryCreationUtilities.CreateExtrusionGeometry` prism for accuracy; annotations resolved via
    `IndependentTag.GetTaggedLocalElementIds()` and `Dimension.References`.
  - write: `RevisionCloud.Create(doc, sheet, revisionId, curves)`; fixes go through the same setters the Advance
    panel already uses.
- Tests: fixture PDFs exported from the sample project, marked up in two editors (one vector, one flattened);
  golden JSON of expected sheet/element/intent per markup; unit tests for page->sheet->model transforms with a
  rotated viewport and a section view.

**MVP (2-3 weeks of focused work).**
1. Vector annotations only, plan views only, sheet match by sheet number.
2. Element pick + Markup Board + zoom/select + status.
3. Rule intents: size, mark, level offset, delete. Revision clouds.
4. Response PDF + CRS Excel.

**Phase 2.** Sections/elevations, callouts and arrows, flattened/scanned path, LLM fallback, round memory and
verify pass, BCF export, rebar intents linked to BBS Generator and RC Automation.

**Risks and how they are handled.**
- Page not exported from Revit (consultant re-saved, scaled, rotated): fall back to title-block text + 2-point
  manual registration (click two grid intersections on the snippet and in the view) — still faster than manual.
- Clouds spanning several elements: card shows all, user unticks; fix proposals only when the comment names a
  mark that matches one of them.
- Wrong auto-fix: nothing is applied without a preview row and a tick; every card is its own undo step; the CRS
  records who applied what.

### Future update

- **Web twin:** the same headless core runs in the browser beside a web IFC viewer (structura already renders
  3D in-browser) so a consultant can drop their PDF and see pins without Revit; status syncs through the
  AnonGee Bridge.
- **Reverse direction:** model -> PDF "pre-review": run Warning Doctor / IDS checks and drop the findings as
  native PDF annotations on the issued set, so the checker sees known issues before they comment.
- **Bluebeam Studio / Autodesk Docs connectors** to pull markups without emailing PDFs.
- **ETABS link (etabs-mcp):** comments like "check deflection B12" open the same member in ETABS with its design
  result shown on the card.
- **Team analytics:** comments per sheet / per modeller / per consultant, average close time, top repeated
  comment types -> feeds the firm's checklist and the next project's template (C2B normalise rules).
- **Voice reply on site:** read out open queries and dictate replies on the phone, synced to the CRS.

### Improvement suggestions for existing tools (spotted while scoping this)

- **AutoDIM / Export Schedule / BBS Generator:** write a small `issued_from` stamp (model GUID, view id, revision)
  into exported PDFs/XLSX metadata so Markup Mirror and Change Radar can match returned files exactly instead of
  by sheet number text.
- **cad2bim / C2B:** the mark-parsing rules (`C12-300X900`, `450x450`) would be worth extracting into one shared
  size/mark normaliser module, reused here for comment parsing.
- **Revision workflow:** a "Prepare issue" button that sets the next Revision, exports the set with page labels =
  sheet numbers, and archives it as the baseline Markup Mirror compares returned PDFs against.
