# 20261017 Idea: BBS Weaver — drop any Excel bar bending schedule, get the 3D rebar cages in the model (IFC / Revit), with weight proof and mark-by-mark check

## Entry 1

```
date-time:   2026-10-17 (scheduled run, third idea of the day)
idea name:   BBS Weaver — drag in a bar bending schedule Excel (from a site engineer, a detailer, a subcontractor or
             any software), and the bars are woven into 3D: every bar mark becomes a correctly shaped, correctly
             placed rebar in its host beam / column / slab / footing, exported as IFC rebar and as native Revit rebar
potential:   High. Two directions exist today and the middle is missing. (1) Model -> BBS is solved in this toolkit
             (cad2bim BBS to Excel, pyRevit BBS Generator, PlanWin AI Pro BBS). (2) Real life runs the other way:
             detailers, consultants and bar-bending yards still produce the schedule in Excel first, and the
             modeller is asked "can you show this in 3D / check it against the model / give me a clash-free
             cage?". Today that means redrawing every bar by hand (days per floor), or buying Tekla Structures,
             Allplan Precast / Reinforcement, Revit rebar extensions (Advance Steel is steel only), StruSoft
             or ReboCAD-class tools - all paid, all need their own input format, none reads "whatever Excel the
             site sent". A free, browser-based, format-tolerant Excel-BBS -> 3D rebar converter does not exist.
             It also finally closes the loop in the toolkit: Joint Congestion Doctor and Twin Check need bars
             in 3D; Bar Optimizer and Site QR Pass need bar marks that point to real bars. BBS Weaver is the
             on-ramp that gets any legacy or external schedule into that chain. Strong LinkedIn / Instagram
             fit ("Excel BBS to 3D rebar in 30 seconds - no Tekla licence"): a screen-recording of a plain Excel
             sheet turning into a glowing rebar cage is instantly understood by every modeller and site engineer.
```

### Description

**The pain (daily life of a BIM modeller / detailer / site engineer).**
- A consultant or subcontractor sends `BBS_L4_Beams.xlsx` with 300 rows: bar mark, member, dia, shape, A/B/C/D/E
  dimensions, no. of bars, cut length, weight. Each office has its own column names and layout.
- To see it in 3D (for clash, congestion, coordination with MEP sleeves, client presentation) the modeller
  re-enters every bar manually in Revit/Tekla. A floor of beams takes days; nobody does it, so the 3D model has
  *no* rebar and the schedule is never checked against geometry.
- Errors hide in the sheet: a shape dimension that does not add up to the cut length, a hook missing, a bar
  longer than the member, a mark used for two different shapes, lap lengths shorter than code, a member
  that exists in the BBS but not in the model. They are found at the bending yard or when the cage will not fit.
- Revisions arrive as a new Excel with "some rows changed": nobody knows which bars to redo.

**The idea.** A **BBS Weaver** mode in cad2bim (browser, files stay on the PC) that:

1. **Reads any BBS Excel/CSV, tolerant of layout.** Auto-detects the header row and maps columns by name and by
   content ("Dia", "Ø", "Bar size", "No.", "Nos", "Qty", "Shape", "Code", "Cut length", "CL", "Wt"). A mapping screen
   shows the guess with confidence; the user fixes any column once and the mapping is saved as a named profile
   ("Consultant A format"). Optional AI assist (the cad2bim MCP agent) for really messy sheets: merged cells,
   multi-line headers, totals rows, bars grouped under member titles. Units (mm / cm / m / ft-in), bar grades
   (Fe415 / Fe500 / Fe500D / B500B) and shape-code standards (IS 2502 / SP 34, BS 8666, ACI 318 / ASTM
   hooks) are detected or chosen.
2. **Understands shapes.** Shape code + dimension letters A-E become real bent-bar geometry, using the same shape
   library as the existing BBS exporters (00 straight, 11 L, 21 U, 23, 41, 46, 51 stirrup, 51A crank, 99 free
   shape...), with bend radii and hooks from the chosen standard. Cut length is **recomputed from the geometry**
   (deducting bend allowances) and compared with the sheet: "Mark 12: sheet 4,350 mm vs. geometry 4,290 mm
   (-60 mm)". Free-text shapes the tool cannot parse are listed, not guessed.
3. **Matches bars to host elements.** Each BBS row names a member ("B12", "C5 L3", "S4", "F2" or a grid
   reference). The tool matches that label to the IFC element by mark/tag/name, or by the Plan-Tracer / DXF layer
   label; unmatched rows go to a one-screen "assign" table where the user clicks the element in 3D. A
   tolerant fuzzy matcher handles "B-12", "B12 (L4)", "Beam 12".
4. **Weaves the bars in.** Rule-based placement inside the host's real geometry using cover from the project
   or element (editable): main bars along the member, top/bottom layers by row hint or dia order, stirrups at the
   stated spacing with end-zone densification, column ties, slab mesh by direction, footing mesh, starter bars
   and laps. Each bar is created as an **IfcReinforcingBar** (with bar mark, shape code, nominal diameter,
   steel grade, weight per metre) so IFC viewers and the existing cad2bim BBS export see it. A *sweep* animation
   (bars "weave in" per member) gives the wow moment.
5. **Proves the weight.** Total weight per diameter, per member, per floor from geometry vs. the sheet, as a
   side-by-side table with a tolerance (default 1 %). Discrepancies are the report that site and consultant both
   want: "sheet says 18.4 t of 16 mm, geometry gives 17.9 t: 9 rows differ".
6. **Checks like a reviewer.** Built-in rule checks per bar: cover and clear spacing (IS 456 cl 26.3 / BS / ACI),
   development and lap length, minimum/maximum steel %, max spacing, stirrup hook angle (135 degrees for seismic),
   duplicate mark with different shape, bar longer than the element, bar outside the member. Every finding is
   clickable (zooms to the bar) and exportable as **BCF** issues for the IDS Gatekeeper pipeline.
7. **Round-trip.** The weaving is not one-way: edit a bar in 3D (change dia or number) and export the **updated
   BBS Excel in the original column layout** (using the saved mapping profile), with a diff against the old one.
   When a revised Excel arrives, the **BBS diff** shows added / removed / changed marks and only re-weaves
   those (feeds Change Radar for the cost delta).
8. **Exports.** IFC (IfcReinforcingBar, IFC4 Reference View), **native Revit rebar through the cad2bim Bridge for
   Revit**, bar shapes as DXF (bending-yard sketches per mark, per the Drawing Autopilot sheet style), cage
   groups with "lift weight" and "bundle" tags for the Site QR Pass and Bar Optimizer cut plans.

**Why it is a wow.** An ordinary Excel file that a contractor sent in the morning turns into a lit-up 3D
reinforcement cage in under a minute, with a "weight proof" table and a list of real errors found - things people
normally need a Tekla seat, a detailer and a week to get. For the BIM team it opens a service line: "send us
your BBS, we'll model, check and certify it" - a day-one revenue generator with a tiny learning curve.

### Implementation

1. **Use what exists.** cad2bim already has the BBS module (shape codes, BS 8666 / IS 2502, `bbsExcel.ts`), the IFC
   writer with property fixtures (`make_props_ifc.py`), SheetJS-style Excel parsing (Structura already ships
   `xlsx` lazily), the element tree and picking in `@cad2bim/engine`, and the Revit Bridge for native elements.
   BBS Weaver is mostly a new importer plus a placement engine plus a comparison table. Put the shape/geometry
   code in a framework-free package (`packages/rebar`, TypeScript) so Structura and PlanWin AI Pro (Python port
   of the same tables, or a shared JSON shape library) can reuse it.
2. **Phase 1 - Import + proof (2-3 weeks, no 3D yet).**
   - Excel/CSV reader with header detection, column mapping UI and saved profiles; unit and standard selection.
   - Shape library as JSON (`shapes.json`: code, dimension letters, formula for cut length, bend rules per
     standard). Recompute cut length and weight for every row; compare with the sheet; show the diff table.
   - Output: "Mismatch report" Excel + on-screen table. This alone is useful and shippable.
3. **Phase 2 - Match + place (3-4 weeks).**
   - Element matcher (label normalisation, fuzzy match, manual assign table).
   - Placement engine per member type (beam, column, slab, footing, wall) with cover, layer order, stirrup
     spacing, end zones; start with straight bars + stirrups + ties + L/U shapes; add cranks and free shapes next.
   - Create `IfcReinforcingBar` with sweep geometry (`IfcSweptDiskSolid` along a poly-line with bend arcs), mark,
     shape, grade and weight properties; stream into three.js as instanced tubes (a floor has 10-50 k bars, so
     use instancing and a worker, as the 50k-element fixture already does).
4. **Phase 3 - Rules, round-trip, Revit (3 weeks).**
   - Rule checks (cover, spacing, laps, hooks, steel %), BCF export via the planned IDS Gatekeeper pipeline.
   - Round-trip: edit in 3D, re-export Excel in the user's original layout; BBS revision diff and partial re-weave.
   - Revit Bridge: create native Revit `Rebar` from the same placement data (RebarBarType per dia, shape per code).
5. **Data safety.** All parsing and weaving runs locally in the browser (worker thread); nothing is uploaded.
   Mapping profiles and shape overrides are stored in the browser (and exported as JSON for the team).
6. **Testing.** Golden files: five real BBS sheets from different consultants (anonymised) with known totals;
   unit tests for every shape formula against SP 34 / BS 8666 worked examples; round-trip test (Excel -> 3D ->
   Excel gives the same marks and weights); a clash-free cage test on the sample towers in `tools/fixtures`.
7. **Honesty.** Placement is rule-based and for coordination, quantity proof and communication; the engineer of
   record still owns the detailing. The report states every assumption (cover, layer order, lap position).

### Future update

- **PDF / scanned BBS in:** run it through the Plan Tracer OCR so a photographed schedule works too.
- **Cage assembly and lifting:** group bars into prefab cages (column cages, beam cages), with lift weight,
  centre of gravity, rigging points and a build-sequence animation for the bar-bending yard.
- **Congestion link:** feed the woven bars into Joint Congestion Doctor automatically; failing joints suggest
  bar re-arrangement that goes back into the Excel as a proposed revision.
- **Bar Optimizer link:** one click turns the verified BBS into the cutting-stock plan and the carbon meter.
- **Machine files:** export BVBS / bending-machine files (Schnell, Progress, MEP) from the verified schedule.
- **Other standards:** Eurocode 2 / EN 10080 detailing rules, ACI 318 development lengths, AS/NZS 4671 grades.
- **AI reviewer:** the MCP agent explains each finding in plain language, and drafts the reply to the consultant.
- **Precast and steel:** same engine for precast mesh/cage shops and for plate-and-bolt (connection) schedules.

### Market check

- Tekla Structures, Allplan Reinforcement, Revit with Naviate / Rebar extensions and Cadwork / Advance Concrete:
  paid, model-first (you draw bars in their model, they make the BBS). None accepts an arbitrary Excel BBS as
  the *source of truth* and builds the bars from it.
- Free tools: generic BBS calculators/spreadsheets (weights only, no 3D, no host matching) and Dynamo scripts
  that need Revit, a specific template and a custom sheet format.
- Current gap: a free, browser-based, format-tolerant, standards-aware "Excel BBS -> verified 3D rebar" tool
  that also feeds an open IFC pipeline. (Competitor feature claims here are from general knowledge and should
  be re-checked before any marketing copy.)
