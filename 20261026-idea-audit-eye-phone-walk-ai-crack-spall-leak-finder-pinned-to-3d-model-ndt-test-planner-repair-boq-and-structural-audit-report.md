# 20261026 Idea: Audit Eye — walk an old building with a phone, AI marks every crack, spall, exposed bar and damp patch and pins it to the element on a 3D model; it plans the NDT tests, suggests a repair per element, prices it from the model and writes the structural audit report with photos, plans and a 3D link for the housing society

## Entry 1

```
date-time:   2026-10-26 (scheduled run, second idea of the day)
idea name:   Audit Eye — "A 35-year-old G+4 housing society needs its structural audit. The engineer and one
             modeller walk the building with a phone. Before the visit, the old plan drawing (a DXF, or a
             PDF scan through Plan Tracer) became a 3D model in cad2bim in a few minutes. On site the
             phone shows the floor plan. Tap column C7 on the 2nd floor, take a photo. A small scale card
             is in the shot. The AI outlines a longitudinal crack along the corner bar and reads its width
             as about 1.2 mm. It tags it 'possible corrosion crack, severe'. C7 turns red on the 3D model.
             By the end of the walk, 186 defects are pinned to 74 elements. The tool proposes 12 rebound
             hammer points, 6 UPV points and 3 core locations. It spreads them over every floor and puts
             them on the worst elements. Back in the office the repair schedule is already filled: epoxy
             injection 38 m of crack, polymer mortar 21 m², column jacketing for 6 columns at 2.4 m³ of
             micro-concrete. A draft repair cost and a draft audit report with marked plans and a photo
             per element are ready by lunch. At the society meeting the committee rotates the building on
             a phone and taps C7 to see the photo."
             One engine with four front ends:
               (1) cad2bim web app: a new "Audit" workspace. It runs as an offline-capable web app on the
                   phone (site capture) and on the desktop (review, NDT, repairs, report). Photos stay on
                   the device, like every other file in cad2bim.
               (2) pyRevit button (AnonGee tab): it brings the audit into a Revit model. A defect marker
                   family goes at each point with a photo link. Shared parameters AE_Defect, AE_Severity,
                   AE_Width_mm, AE_Repair and AE_Qty go on the elements. A repair schedule and
                   colour-by-severity views are added.
               (3) C2B / cad2bim DXF -> 3D: these build the "as-is" model from the society's old drawing.
                   Missing marks are filled from the photos on site.
               (4) etabs-mcp / Structura: the measured concrete strength (from NDT) and the lost bar area
                   (from corrosion) go back into the ETABS model as reduced sections. The tool then shows
                   which elements fail now and which pass after jacketing.
             Outputs: a defect register (Excel), marked-up floor plans (DXF / PDF), a 3D distress heatmap,
             an NDT test-point plan with a QR sheet, an NDT results table, an element-wise repair schedule
             with quantities and a priced repair BOQ, a draft structural audit report (Word / PDF) in the
             office's format, a BCF file and a read-only 3D web link for the society.
potential:   Very high, and new for the team. Structural audits of old buildings are a steady and growing
             business in Indian cities. Mumbai (MCGM) requires periodic audits of buildings older than 30
             years, other municipal bodies ask for them before repairs and redevelopment, and every
             monsoon brings a new round of notices. The work is mostly paperwork and photo sorting, not
             engineering. It is also the one part of the business where the BIM team does nothing today.
             Time today for one G+4 building (estimates from how the office works, not timed):
               - site visit: 1-2 days of photos plus handwritten notes on a printed plan,
               - matching about 300-600 photos to elements and drawing defect marks in AutoCAD: 1-2 days,
               - writing the defect table and the report and pasting photos into Word: 1-2 days,
               - repair quantities and BOQ in Excel, measured by hand from the drawing: 0.5-1 day,
               - drawing the NDT locations and copying lab results into tables: 0.5 day.
               Total back-office time: about 3-6 days per building.
             With Audit Eye: the site visit takes the same time or a little less (one tap per photo
             replaces the notes). Back-office review and edits take about 0.5-1 day, because the photos,
             marks, quantities and report draft already exist. Expected saving: 2-5 days per building
             (estimate). For a team doing 4-10 audits a month, that is 2-8 engineer-weeks a month. It
             also opens a new service the BIM team can sell: "3D audit + digital twin of your society".
             Wow effect: the old building turns into a 3D model that shows its health. Columns and beams
             glow from green to red by severity, and each defect is a pin with a photo. A slider ("2026
             vs 2023 audit") shows a crack growing on the same column. The phone outlines the crack the
             moment the photo is taken and prints "1.2 mm, severe". "AI finds cracks" and "digital twin
             of an old building" are popular on LinkedIn and Instagram: drone and robot crack-detection
             reels from IIT labs, startups and overseas firms get shared widely. This one is made from a
             real society's building by a normal phone in a normal site visit.
             Market: drone and photo inspection tools exist, but are paid and made for bridges, towers and
             façades: DroneDeploy, Pix4Dinspect, Bentley iTwin Experience / OpenTower, Skyspecs-type
             platforms. They report on a photo mesh or map, not on a BIM element with a mark. They do not
             plan NDT, choose a repair, measure repair quantities from the model or write an Indian-format
             audit report. Generic snag and site apps such as Fieldwire, PlanRadar and Procore (all paid)
             pin photos on a plan but have no defect AI, no crack width, no 3D element link and no repair
             BOQ. As far as we know, no free tool goes old drawing -> 3D model -> phone defect capture with
             AI -> NDT plan -> repair schedule and BOQ -> audit report in one flow.
```

### Description

**The pain (daily life of the BIM team and the audit engineer).**
- Audits come in bulk before the monsoon and before redevelopment votes. Each one is a small job with a fixed fee,
  so the time spent on paperwork decides whether it pays.
- On site, photos and notes are taken separately. Back in the office someone spends hours working out "which
  column was IMG_4417?".
- Defect marks are drawn by hand on an AutoCAD plan, the table is typed in Excel, and the photos are pasted into
  Word one by one. Three copies of the same facts drift apart.
- Repair quantities (crack lengths, spalled areas, jacketing volumes) are guessed from photos or measured by hand,
  so BOQs vary from one engineer to the next.
- NDT locations are chosen on site without a plan, so some floors get no tests and others get too many.
- The society committee cannot read a 40-page report full of photos. It wants to see "which columns are bad and
  what it costs".
- Next audit, three years later, nobody can say whether a crack grew, because the photos are not tied to the
  element.

**What Audit Eye does.**
1. **As-is model in minutes.** The society's old drawing is turned into a quick 3D model (columns, beams, slabs,
   walls, levels) by the cad2bim DXF -> 3D pipeline, C2B, or Plan Tracer for a scanned PDF. Without drawings, a
   grid and column sizes measured on site build a "grid model" in the Audit workspace (a simple form: grid
   spacing, floor heights, column sizes).
2. **Phone capture tied to elements.** The phone shows the floor plan of the current level. Tap the element
   (column C7, beam B12 bottom face, slab panel S4, wall W3, staircase, chajja, OHT). Then take a photo or a
   short video. Placement is by tap, on purpose: indoor GPS and phone tracking are not reliable enough to pick a
   200 mm column. Each capture records element, face, height band (bottom / middle / top), time and the
   person's name.
3. **AI defect finder.** An on-device model outlines cracks, spalling, exposed or rusted bars, honeycombing,
   damp patches / leaks and efflorescence. A printed scale card (a black-and-white card with known marks, in the
   photo near the defect) turns pixels into mm, so crack width and spall area are measured. Without the card the
   tool asks for a crack-gauge reading. The AI only proposes; the engineer accepts, edits or rejects with one tap.
4. **Severity and pattern rules (editable table).** Crack width bands, type and position give a severity. For
   example: a crack along the corner bar of a column means "possible corrosion"; a diagonal crack near the
   support of a beam means "possible shear"; a vertical crack mid-span at the bottom of a beam means "flexure";
   map cracking on a slab soffit means "shrinkage / leak". The office's engineers write the rules. Each finding
   shows the rule that fired.
5. **NDT test planner.** It proposes rebound hammer, UPV, core, carbonation and half-cell / cover-meter points.
   They are spread over floors and element types, weighted to the worst elements, with an office minimum per
   floor. It prints a QR test sheet. The lab technician scans the QR at each point and types the reading. The
   results flow back to the element and give an "estimated in-situ strength" band from the office's correlation
   table. The engineer confirms it.
6. **Repair per element.** From the severity and the NDT, a repair library (office data, not code) proposes a
   method per element: crack sealing / epoxy injection, polymer-modified mortar patch, rebar cleaning plus
   anti-corrosion coat, micro-concrete or RC jacketing, FRP wrap, waterproofing / leak repair. Quantities come
   from the model and the AI marks: crack length in m, patch area in m², jacket volume and extra steel from the
   column's real size and height. Rates come from the office rate sheet, which gives the BOQ.
7. **Classification proposal.** For Mumbai jobs, the tool proposes MCGM's category (C1 / C2-A / C2-B / C3) from
   the findings, with the reasons. The engineer decides and signs. The tool never signs.
8. **Report and 3D link.** It drafts the report in the office's Word template: building data, method, NDT
   table, element-wise defect and repair table, marked plans, photos grouped by element, BOQ, category and
   recommendations. A read-only 3D web link (the Handover Hub viewer) lets the committee tap any red element to
   see its photo.
9. **Re-audit.** The next audit opens the old one. The same element shows both photos side by side, crack
   growth in mm and "new since last audit" defects.

**Daily use cases.**
1. *Routine 30-year audit*: the whole flow above; the report is ready a day after the visit.
2. *Leak and seepage complaint*: a quick walk of one flat and the floor above, with the damp patches pinned. The
   tool traces them up to the toilet, terrace or OHT above and reports them with a repair and its cost.
3. *Redevelopment vs repair advice*: the repair BOQ for the whole building next to a rough rebuild cost (from
   Plot Pilot / the BOQ module). Societies ask for exactly this comparison.
4. *Pre-purchase or bank inspection*: a 1-hour, one-flat check with a short PDF.
5. *Post-earthquake or post-fire rapid check*: a fast walk with a "red / amber / green" tag per element and a
   one-page summary for the authorities.
6. *Repair contractor bill check*: during the repair, the site team marks each element "repaired" with an
   after-photo. The finished quantities become the bill (links to the Bill Painter idea).
7. *Selling the service*: the 3D health model and the before / after reel are the office's marketing on
   LinkedIn and Instagram.

### Implementation

**1. As-is model.**
- Reuse the existing paths: cad2bim's Rust DXF -> 3D pipeline (and the CAD viewer for DWG), C2B `extract` +
  `normalize` for messy client DXFs, and Plan Tracer (idea of 2026-10-16) for PDF scans.
- New: a "grid model" form in the Audit workspace for buildings with no drawing. Grid lines, floor heights and
  a column-size table make an IFC with columns, beams and slabs.
- Every element gets a stable audit mark (grid-based, e.g. `C-B3-L2`) so the next audit finds the same element
  even if the model is rebuilt.

**2. Phone capture (cad2bim web, offline-capable).**
- The Audit workspace works as an installable web app with a service worker, so it runs in a basement with no
  signal. Photos are stored in the browser's own storage (IndexedDB / OPFS) and moved to the office laptop by
  cable, local Wi-Fi share, or the person's own cloud folder when they choose. Nothing is uploaded unasked
  (cad2bim rule).
- UI: a large floor plan, tap an element, choose a face, then the camera opens. Thumb-sized buttons, works with
  gloves, Hindi / Marathi labels later.
- Optional 360° camera walk: the person stops at a column, taps it on the plan, and the frame at that moment is
  linked.

**3. Defect AI.**
- An instance-segmentation model for six classes (crack, spall, exposed / rusted bar, honeycomb, damp / leak,
  efflorescence), run on the device with ONNX Runtime Web (MIT) on WebGPU / WASM. No photo leaves the phone.
- **Licence check first:** many popular detection models and their weights are AGPL (for example Ultralytics
  YOLO), which cad2bim does not allow. Use an architecture and weights under MIT / Apache-2.0 / BSD (for example a
  U-Net / DeepLab-style segmenter or an Apache-2.0 detector), and train it ourselves. Each public crack dataset's
  licence is checked and written in `docs/decisions.md` before use. The office's own audit photos (with the
  owners' permission) become the main training set over time.
- Crack width: skeletonise the crack mask, measure the width across the skeleton, and convert to mm using the
  scale card's marks (detected with OpenCV.js-style marker detection — check the licence of the exact build).
  Report the max and the mean width and the length. Say clearly that this is "photo-estimated, ±0.2 mm (to be
  measured against a crack gauge on 50 real cracks before it ships)".
- Everything has a manual path: draw the crack with a finger and type the width.

**4. Rules, NDT and repair libraries (data, not code).**
- `severity-rules.json`: width bands per element type, pattern -> likely cause, severity. Written by the office's
  engineers from the codes and guidelines they use (IS 456 durability clauses, IS 15988 for seismic evaluation
  and strengthening of existing RC buildings, MCGM audit guidelines). Values are entered from the published
  documents, not typed from memory.
- `ndt-plan.json`: minimum points per floor and per element type, the share on the worst elements, and the test
  types. Rebound hammer and UPV follow the current IS 516 (Part 5) / IS 13311 editions; the edition is confirmed
  by the engineer.
- `repair-library.json`: method, trigger (severity + NDT), quantity formula (per m crack, per m² face, per m³
  jacket from section + jacket thickness + height), materials, rate code.
- Rates from the office rate sheet (Excel), the same format the BOQ module already reads.

**5. Report.**
- A Word template with fields and repeating sections (element tables, photo grids), filled by a docx writer in
  the browser (a permissively licensed library such as `docx` (MIT)). PDF from the same layout.
- Marked plans: defect symbols and NDT points written onto the plan as DXF layers (`AE-DEFECT-SEVERE`,
  `AE-NDT-RH`, …) through the CAD engine, then printed to PDF.
- BCF 2.1 / 3.0 with one topic per severe element, for Revit / BIMcollab users (reuse the QA export).

**6. Revit (pyRevit, AnonGee tab).**
- "Audit Import": read the audit JSON, place a defect marker family on each element face, write the shared
  parameters, create views coloured by severity, and create a repair schedule with quantities. Everything lands
  in one undo step, like the Auto Level Manager.
- "Audit Export": after the engineer edits in Revit, write the changes back to the JSON.

**7. ETABS (etabs-mcp).**
- Push the measured strength (per floor or per zone) and the reduced bar area of corroded elements to the ETABS
  model as new materials and section modifiers. Run the analysis and read back the demand/capacity of each
  member. Elements that fail are red in Audit Eye. Then try jacketed sections and show the result after repair.

**Tech stack.** TypeScript in `apps/web` (Audit workspace) and `@cad2bim/engine` (element links, quantities,
report data). An ONNX model in a Web Worker. three.js for the 3D heatmap (already used). A Python module shared
by pyRevit and C2B for import and export. Shared JSON test cases keep them in step. No new outside services. The
model weights and the WASM runtime are served by cad2bim itself and pinned by SHA-256, like Pyodide.

**MVP (4 weeks).**
1. As-is model from DXF (existing pipeline) plus the grid-model form; stable audit marks.
2. Phone capture: tap element -> photo -> manual defect type, width and length. Offline storage. Export to the
   laptop.
3. Desktop review: 3D heatmap, defect register Excel, marked plan DXF / PDF, repair library v1 with quantities
   from the model, BOQ.
4. Report draft in the office's Word template.
5. AI defect outlining for cracks only (the most common and the most time-consuming), with the scale card. The
   other classes come in v2.
6. Test on three past audits from the office (photos re-tagged by hand). Compare the time and the BOQ with the
   original reports.

### Future update

- **More AI classes and auto crack-pattern reading** ("diagonal shear crack near support") with an explanation
  drawn on the photo.
- **Photo-to-mesh of the façade** (photogrammetry or Gaussian splats from a phone video, with permissive tools)
  and defects mapped on the real façade, linked to the BIM elements.
- **Drone mode** for façades and terraces of tall buildings: a flight plan from the 3D model, with each frame
  projected to the element it shows.
- **Crack monitoring:** printed crack-tell-tale tags with a QR; the society's watchman photographs them every
  month and the tool plots crack growth. If it passes a limit, it sends an alert to the engineer.
- **Strengthening designer:** RC / steel / FRP jacket design per IS 15988, with the result drawn as a Revit
  family and a BBS (links to BBS Weaver and Detail Drop).
- **Redevelopment pack:** the audit plus Plot Pilot's massing becomes one "repair or redevelop" deck for the
  society's general meeting.
- **Hindi / Marathi voice notes** on site, turned into defect text.
- **City-scale:** for a ward or a developer with many old buildings, a map of all audits by category and a repair
  budget.

### Improvement suggestions for existing tools in the repos

- **cad2bim:** add a general "attachment" layer to the engine's element model (photos, notes, readings linked to
  an element by a stable mark). Audit Eye, Site QR Pass, Handover Hub and Bill Painter all need the same thing.
  Also a reusable "colour by value" heatmap legend in the viewer.
- **cad2bim (CAD workspace):** a "stamp symbols to layer" export, so any tool can write marks (defects, NDT
  points, sleeves, set-out points) back onto the client's own DXF.
- **C2B:** a "grid-only" quick mode. It takes grid lines and column marks from a poor or partial old drawing and
  builds the column stack even when beams are missing. Old society drawings are often like that.
- **pyRevit-addons (AnonGee):** a generic "attach photo to element" tool (with the photo path in a shared
  parameter and a preview in a modeless window). It is useful for snagging and site queries too.
- **etabs-mcp:** MCP tools for "set material per story/zone", "apply section modifier to frames by label" and
  "member D/C ratios by label", so the reduced-strength check of an existing building takes one call each.
- **Structura / PlanWin AI Pro:** an "existing building" flag that uses measured strength instead of the design
  grade, and reports which members need jacketing.
