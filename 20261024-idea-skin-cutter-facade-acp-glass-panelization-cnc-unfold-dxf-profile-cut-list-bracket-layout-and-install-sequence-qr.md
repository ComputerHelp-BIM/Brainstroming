# 20261024 Idea: Skin Cutter — turn the Revit façade (curtain walls, ACP / HPL cladding, louvres, fins) into fabrication: panel numbering, flat vs. warped check, ACP folded-tray unfolding to CNC DXF, glass cut list, aluminium profile cutting optimisation, bracket layout on the slab edge, wind-load check from ETABS and a QR install sequence

## Entry 1

```
date-time:   2026-10-24 (scheduled run)
idea name:   Skin Cutter — "Select the façade. Press Cut. Get every panel numbered on the elevation, every
             ACP tray unfolded into a CNC-ready DXF with V-grooves and corner notches, every glass pane in
             a cut list with size, make-up and quantity, every aluminium mullion and transom cut from 6 m
             bars with the least waste, every bracket placed on the slab edge and checked for wind, and an
             install sequence the site team scans from a QR sticker."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Façade" stack next to StrucBIM): pick curtain walls /
                   cladding walls / generic façade families, set rules, preview the panel map, export,
               (2) cad2bim web viewer: façade unrolled flat per elevation, panels coloured by type /
                   status (designed, ordered, fabricated, installed), click a panel to see its DXF and data,
               (3) etabs-mcp bridge: storey heights, slab-edge levels and wind pressure per storey for
                   bracket and mullion checks,
               (4) Bar Optimizer + Site QR Pass engines (earlier ideas): cutting stock for profiles and
                   QR stickers per panel for site.
             Outputs: panel schedule (Excel), CNC DXF per ACP tray (layers: CUT, V-GROOVE, DRILL, TEXT),
             nested sheet DXFs (1220 x 2440 / 1500 x 4000 ACP sheets), glass cut list for the glass vendor,
             profile cut list with offcut reuse, bracket set-out on slab edges, façade elevation sheets with
             panel tags, BOQ (m² ACP, m² glass by make-up, kg aluminium, brackets, anchors, sealant m),
             and an install sequence per floor and per crane / gondola drop.
potential:   Very high. Glass + ACP façades are on almost every commercial, hospital, mall and premium
             residential project in India and the Gulf, and the façade package is usually the most
             expensive finishing item (₹4,000-15,000 per m²). Today the BIM model stops at "curtain wall
             with a pattern" and the façade contractor redraws everything in AutoCAD.
             Time today (one G+12 office block, ~4,500 m² façade, ~2,200 ACP trays, ~1,400 glass panes):
               - panel numbering on elevations by hand: 2-3 days, re-done after every revision,
               - unfolding ACP trays in AutoCAD one by one (size + 2 x return + groove + notch): 8-15 days
                 for a draughtsman, and each mistake is a wasted ₹3,000-8,000 sheet,
               - glass schedule from elevations: 2-3 days, with wrong sizes at corners and soffits,
               - mullion / transom cut list in Excel: 2 days, waste typically 8-12 %,
               - bracket layout on the slab edge (clash with beams, edge distance, embeds before pour):
                 2-4 days, usually done late and drilled on site,
               - revisions: every architect change to the grid or a floor height repeats most of it.
             With Skin Cutter: 30-60 minutes to a first full set, 5-10 minutes per revision. Expected
             saving: 15-25 working days per façade package, sheet waste down from 10-15 % to 4-6 %, and
             zero "panel does not fit" surprises from wrong numbering.
             Wow effect: a 3D tower "peels" its façade on screen, panels fly flat onto the elevation and
             then unfold into CNC outlines and nest onto sheets — a 20-second reel that is exactly the
             kind of BIM-to-fabrication content trending on LinkedIn and Instagram (parametric façades,
             "design to fabrication", digital fabrication, robot-cut panels).
             Market: Rhino + Grasshopper + Elefront / Kangaroo (paid, needs a computational designer,
             not Revit-native), Evolute Tools (paid), FacadeCAD, LogiKal, Orgadata (paid, aimed at the
             façade fabricator, not the BIM team, with their own system catalogue), Dynamo scripts
             (free but one-off, no unfolding, no nesting, no QR, nobody maintains them). To our knowledge
             nothing free gives Revit façade -> numbered panels -> unfolded CNC DXF -> nested sheets ->
             profile cut list -> bracket set-out -> site QR in one tool.
```

### Description

**The pain (daily life of a BIM modeller on a façade package).**
- The architect's Revit model has curtain walls with a grid and a few cladding walls. It looks right in a
  render and is useless for fabrication: panels are not numbered, corners and soffits are not resolved,
  and nobody knows how many unique panel sizes there are.
- The façade contractor's draughtsman redraws every elevation in AutoCAD, numbers panels by hand, unfolds
  every ACP tray (face + four returns + V-grooves + corner relief), and sends DXFs to the CNC router.
  When the architect moves a floor line by 150 mm, half of it is done again.
- Glass is ordered from elevation dimensions. A pane at a column corner or a sloped soffit comes back
  20 mm too big; the replacement takes 3-6 weeks.
- Brackets are placed "every 1.2 m" without checking beams, slab edge distance or wind; anchors are
  drilled after the pour, sometimes into rebar.
- Status tracking (which panels are ordered, cut, delivered, installed) lives in WhatsApp photos.

**The idea, in eight parts.**
1. **Panel map.** Read every curtain panel, mullion, cladding wall and façade family on the selected
   elevations. Each panel gets a stable key, its real 3D corners, its plane, its host face, and a
   "flatness" value (max deviation of the 4th corner from the plane of the other 3). Panels within a
   tolerance (e.g. 3 mm) are flat; others are flagged as warped / cold-bent / needs-special.
2. **Smart numbering.** Number panels by rule: `Elevation-Floor-Bay-Row` (e.g. `N-07-B12-R2`) or by
   type family. Identical panels (same size within tolerance, same make-up, same edge conditions) share
   a type code (`ACP-T014`), so the schedule says "T014 x 186" instead of 186 rows. A "uniqueness
   chart" shows how many unique sizes the design has — the single most useful number for cost, and a
   lever for the architect ("move this grid by 50 mm and 140 trays become one type").
3. **ACP / HPL tray unfold.** For each tray type: face size + return depth on each side (rules per edge:
   top/bottom/left/right, stiffener, hook or rivet fixing), V-groove lines at the fold, corner notches
   / relief cuts, fixing holes at pitch. Write a CNC DXF with fixed layers (`CUT`, `V-GROOVE`, `DRILL`,
   `TEXT`, `GRAIN-ARROW`) and the panel code engraved on a hidden face. The same engine handles
   perforated panels (hole pattern from an image or a density map) and louvre blades.
4. **Nesting.** Nest unfolded trays on standard sheets (1220 x 2440, 1500 x 3050, 1500 x 4000 or a custom
   size), respecting grain / colour direction for metallic finishes, with kerf and clamp margins. Output
   one DXF per sheet and a sheet count per colour — that becomes the ACP order.
5. **Glass cut list.** For every glass pane: clear size from the mullion / transom profile bite rules,
   make-up (e.g. 6 HS + 12 Ar + 6 HS low-E DGU, 10.76 laminated for spandrel / overhead), edge work,
   special shapes (trapezium, corner), weight per pane (for gondola / crane), and the vendor sheet format
   (Excel + one DXF outline for each non-rectangular pane).
6. **Profile cutting.** Mullions, transoms and sub-frames become a cut list with lengths, end cuts
   (90° / mitre) and drilling. The Bar Optimizer engine (earlier idea, cutting stock problem) packs
   them into 6 m / 6.5 m bars with offcut reuse — the same engine that already optimises rebar.
7. **Brackets and anchors.** Place brackets at mullion positions on slab edges using the real structure
   (from Revit, or levels from etabs-mcp), check edge distance, clash with beams / rebar zones, and
   compute bracket load from panel weight + wind pressure per storey (wind from etabs-mcp load cases or
   an IS 875-3 calculator). Output a slab-edge set-out per floor so embeds / anchor plates go in
   **before the pour** (same "anchor-before-pour" logic as Hanger Hero).
8. **Install sequence + QR.** Group panels by floor, by elevation and by gondola / crane drop; export a
   per-day install list, print QR stickers per panel (Site QR Pass engine) that open the panel in the
   cad2bim viewer with its location, orientation and the neighbours. Site taps "installed" and the
   façade elevation colours itself — a live progress view for the client and for RA billing (Bill
   Painter idea).

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)** already reads and edits elements in bulk (Bulk Tool, Parameter Combine,
  FilterParameter) and generates geometry (BIM Generation, AutoLevel). Skin Cutter adds a façade stack
  using the same parameter-writing pattern: it writes `SC_PanelCode`, `SC_Type`, `SC_Status` back to
  every panel so schedules and tags work natively in Revit.
- **cad2bim** already parses `IfcCurtainWall` and plates / members in its engine and has a 3D viewer;
  the unrolled-elevation view and the panel side panel are a new viewer mode, and IFC import means the
  tool also works on façade models that come from ArchiCAD or Rhino.
- **etabs-mcp** already exposes storeys and load data — the source for slab edges and wind per storey.
- **structura** quantity / BOQ outputs gain a façade section (m², kg, nos) instead of a lump-sum line.
- **Earlier ideas reuse:** Bar Optimizer (profile cutting), Kit Cutter (nesting + truck loads), Drawing
  Autopilot (elevation sheets), Site QR Pass (stickers), Sun Compass (fin / louvre sizing per
  orientation), Bill Painter (progress to billing), Model Reel (the "peel and unfold" video).

### Implementation

**Phase 1 — MVP (2-3 weeks): flat curtain walls + ACP trays.**
- pyRevit (IronPython / CPython) button `Façade.stack/SkinCutter.pushbutton`:
  - collect `CurtainGrid` panels (`FamilyInstance` of category OST_CurtainWallPanels), mullions
    (OST_CurtainWallMullions) and walls with a cladding parameter,
  - get panel geometry (`get_Geometry` -> largest planar face, or the curtain cell polygon via
    `CurtainGrid.GetCurtainCells()` -> `CurveLoops`), compute corners in the host wall's local UV
    so everything is flat and numerable per elevation,
  - flatness check (4th corner distance to the plane of 3), uniqueness grouping by rounded size + edge
    rules,
  - write parameters, colour the view by type with a temporary view filter, one `TransactionGroup`.
- Python core (shared, testable outside Revit): `unfold.py` (tray rules -> 2D outline + grooves +
  notches + holes), `nest.py` (rectangle / bottom-left-fill nesting first; later a no-fit-polygon
  nester), `dxf_out.py` using `ezdxf` (already the natural choice for DXF in this toolkit), `cutlist.py`
  that calls the Bar Optimizer solver.
- Rules in one JSON / Excel sheet the team edits: return depth, groove depth, notch size, hole pitch,
  sheet sizes, kerf, glass bite per profile system, tolerances.
- Output folder: `Panel_Schedule.xlsx`, `DXF/trays/ACP-T014.dxf`, `DXF/sheets/S001_Silver.dxf`,
  `Glass_CutList.xlsx`, `Profile_CutList.xlsx`, `Facade_BOQ.xlsx`.

**Phase 2 (3-4 weeks): viewer, glass, brackets.**
- cad2bim viewer mode "Façade": unrolled elevation canvas (Three.js orthographic per elevation),
  panels as instanced quads coloured by type / status, click -> tray DXF preview (render DXF with the
  existing 2D pipeline), filter by floor / elevation.
- Glass rules per profile system; special-shape outline DXF; weight per pane.
- Bracket placement on slab-edge curves (floors' edge from `Floor.GetBoundaries` or the slab sketch),
  clash with beams via bounding-box + solid intersection, wind load from etabs-mcp (`storeys`,
  load case / IS 875-3 basic wind speed + terrain factor), simple bracket / anchor capacity check with a
  pass / fail table and a slab-edge set-out sheet.

**Phase 3 (3-4 weeks): sequence, QR, status, reel.**
- Install sequence builder (by floor, by drop, by crane reach from Crane Canvas if present).
- QR sticker PDF (one per panel: code, floor, elevation, small key plan) linking to
  `viewer/?model=…&panel=N-07-B12-R2`; status updates stored in the cad2bim API / db
  (`designed -> ordered -> cut -> delivered -> installed`).
- "Peel and unfold" animation export (Model Reel engine) — the LinkedIn / Instagram moment.

**Data model (per panel).**
`key, code, type, elevation, floor, bay, row, host_id, corners_3d[4], size_w, size_h, flatness_mm,
material (ACP / HPL / glass / louvre), colour / finish, edge_rules{t,b,l,r}, weight_kg,
wind_kpa, bracket_ids[], status, dxf_path, revision`.

**Accuracy and safety rules.**
- Never changes the architect's geometry; only writes `SC_*` parameters (and only on request creates
  bracket families).
- Revision-safe: panel keys are stable across re-runs; a revision diff lists "changed size / new / deleted
  panels" so only those DXFs are re-issued (same idea as Change Radar).
- Warped panels and non-standard corners are flagged, never silently unfolded.
- Wind / bracket checks are marked as "preliminary, for coordination" — final design stays with the
  façade engineer.

### Future update
- Double-curved and twisted façades: cold-bend check for glass, panel rationalisation (snap to fewer
  types) with a "cost vs. design deviation" slider for the architect.
- Parametric perforation from an image (logo / pattern on a façade) straight to CNC hole DXF — a
  guaranteed wow post.
- Unitised curtain wall: unit numbering, stacking joints, lifting points and truck loading (Kit Cutter
  engine).
- Energy / daylight link: glass make-up and fin depth suggested per orientation from Sun Compass, with
  a façade U-value / SHGC summary for green-building compliance.
- Vendor export presets (LogiKal / Orgadata CSV, CNC router post-processors) so the façade contractor
  can import directly.
- AI assistant: "Make all panels on the north elevation 1,200 wide max and tell me how many types we
  save" — previews the change, the uniqueness chart and the cost delta.
- Maintenance / FM: each installed panel keeps its make-up, vendor and warranty; a broken pane five
  years later is re-ordered by scanning its QR.
