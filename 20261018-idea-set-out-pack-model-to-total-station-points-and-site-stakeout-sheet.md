# 20261018 Idea: Set-Out Pack — from the 3D model to total-station points, offset lines and a phone stakeout sheet in one click

## Entry 1

```
date-time:   2026-10-18 (scheduled run, third entry of the day)
idea name:   Set-Out Pack — "The model goes to the field as coordinates, not as a drawing."
             Pick a floor / pour / element set (columns, footings, walls, shafts, sleeves, embeds) in the IFC /
             cad2bim / Structura model. The tool generates the full setting-out pack: (1) point file for the total
             station / GNSS rover (CSV in Trimble, Leica, Topcon, Emlid and generic P,N,E,Z formats + DXF + LandXML),
             (2) a grid-referenced table "column C14: 3.250 m right of grid B, 1.100 m above grid 4" for the
             tape-and-theodolite crew, (3) offset (control) lines 500 mm / 1 m outside the work, (4) a one-page
             PDF/phone sheet per pour with a mini-plan, point IDs and tolerances, and (5) the "as-staked" import:
             the surveyor's measured points come back as a CSV and are compared with the model (deviation table,
             red if outside tolerance), feeding the next floor automatically.
potential:   High, and it saves real hours every day on site. Setting-out is the first thing that happens before
             every pour and the place where a wrong drawing read becomes a 25 mm error that costs a column. Today
             the surveyor re-reads the DXF/PDF and keys or traces points by hand (1-3 hours per floor), or the BIM
             modeller exports points from Revit/ArchiCAD through a paid add-in. Errors in reading grid offsets, in
             rotated buildings and in the project base point are the usual cause of "column is 40 mm out" disputes.
             Time saved: ~2 h per floor for the surveyor, ~30 min for the modeller who answers questions; fewer
             set-out errors (the most expensive kind of RCC rework). It also lets small contractors without a robotic
             total station use a simple tape-from-offset-line sheet derived from the same model.
             Business value: model-to-field loop is closed, and an as-built deviation log exists for every floor,
             which is exactly what clients now ask for in handover (tolerance records, "as-built vs design").
             Trending angle: "BIM-to-field", "layout robots", "digital construction" and surveyor+BIM reels get
             high engagement on LinkedIn/Instagram; the visual of a model whose points turn from red to green as
             a surveyor walks the slab is highly shareable. A one-click CSV that loads in the controller is a clear
             "I did this in 2 minutes" demo.
             Market: see Market check.
```

### Description

**The pain (daily life of a BIM modeller / site engineer / surveyor).**
- The surveyor asks: "Send me the column coordinates for the 4th floor." The modeller exports a DXF, the surveyor
  explodes it, snaps points, renumbers them and types them into the controller. A 20-column floor takes 1-2 hours
  and any rotated grid, offset or non-zero project base point gives a silent error.
- Many sites have no robotic station. The crew works from tape measures and offset lines; the site engineer
  hand-calculates distances from grid lines on the back of a drawing.
- After the shuttering is erected nobody records where it actually is. When an argument starts three floors later,
  there is no deviation log, only memory.
- The model and the field use different origins (model internal origin vs site grid vs national grid). One wrong
  origin note and the whole floor is shifted.
- The same coordinates are needed for sleeves/embeds/anchor bolts (after Sleeve Punch), for column starter bars, and
  for pile/footing centres. Each time the process is repeated by hand.

**The idea.** A **Set-Out Pack** panel that knows what to stake and how the field wants it:

1. **Select what to stake.** Presets: *Footing centres + corners*, *Column centres + 2 face points*, *Wall ends and
   corners*, *Slab edge/opening corners*, *Lift/shaft corners*, *Sleeve/embed centres*, *Starter-bar group*, or any
   selection. A "Stake level" toggle chooses plan (N, E) or 3D (N, E, Z) with level offsets (e.g. +1.000 m benchmark line).
2. **Coordinate system wizard.** Three clear questions: where is the model origin, which grid line is "A/1", what is
   the rotation to the site/survey grid? Presets: project base point, survey point, two-point calibration
   (pick two known model points and type two surveyed coordinates, the tool solves the 2D similarity transform and
   shows the residual). Saved per project, shown on every output ("Datum: SG-2026, rotation 17.43 deg").
3. **Point naming.** Stable, human IDs: `L04-C14-A` (level, mark, corner/face), with the element GUID kept in a hidden
   column so the point returns to the same element after re-export. Codes follow survey practice (`COL`, `FTG`, `WALL`,
   `EMB`, `CTRL`).
4. **Offset lines.** For columns and walls the tool creates control points 500 mm / 1 m outside the formwork on
   the grid lines, so the crew re-establishes the position after shuttering without losing the line. Each offset has
   its own ID and distance to the work.
5. **Output formats.**
   - Controller files: **CSV** in the common layouts (P,N,E,Z,Code; P,E,N,Z,Code; with/without header), plus Leica
     GSI-style text and DXF points; **LandXML** points/alignment for GNSS rovers. Format is picked from a dropdown;
     an "example row" preview shows exactly what the controller will receive.
   - **Grid-offset table** for tape crews: each point as "distance from grid X, distance from grid Y" with check
     diagonals between neighbouring columns (so the crew can verify with two tape measures, no instrument).
   - **PDF/phone sheet** per pour: mini-plan with point IDs, tolerance box (e.g. +-5 mm columns, +-10 mm wall ends),
     datum note, QR code (Site QR Pass) to the live model.
6. **As-staked import and deviation report.** Drop the controller's export (CSV). The tool matches points by ID,
   transforms back to model space and gives: dX, dY, dZ, radial error, pass/fail by tolerance, an overlay in 3D
   (green/amber/red dots on each element), and a signed PDF per floor ("Level 4 set-out record"). Systematic
   shifts are detected ("the whole floor is 12 mm east: probably instrument setup, not shuttering").
7. **Next-floor carry-forward.** The deviation of the lower floor is shown when staking the upper floor, with an
   optional "correct to design" or "follow as-built" decision recorded, which is the real daily problem in tall RCC.

**Why it creates the wow effect.** One click on a floor gives a CSV that loads straight into the surveyor's
controller. The same model then shows the surveyor's returned points as green/red dots on the columns, with a
ready-to-sign deviation PDF. The modeller looks like they have a robotic layout system in a browser tab.

### Implementation

Browser-first, all local, building on the existing viewer and exporters.

1. **Geometry source**: reuse the IFC / DXF-to-3D model already in cad2bim (`packages/viewer`, `packages/engine`) and
   Structura's `parser.js` / `exporters.js`. Add `stakeable(element)` -> `{ centre, corners[], faces[], z }` for
   columns, footings, walls, slab edges, openings. Corner extraction from the extruded profile (no mesh scanning).
2. **Transform engine** (`packages/engine/src/setout/transform.ts`, pure TypeScript, tests):
   - 2D similarity: `p_site = s * R(theta) * p_model + t`, from the project base point or from two or more
     control point pairs by least squares; return RMS residual and flag > 3 mm.
   - Datum string builder and units guard (mm vs m; metre/feet files are the common mistake).
3. **Point writer** (`formats/`): a small registry `{ id, label, columns, delimiter, header, decimals, order }` for
   Trimble Access CSV/JOB-friendly, Leica, Topcon, Emlid, generic; plus a DXF POINT/TEXT writer and a minimal
   LandXML `CgPoints` writer. Each format has a fixture file in tests, and a "validate file" step that reparses the
   output to catch swapped N/E.
4. **Naming and stability**: `L{level}-{mark}-{corner}`; IDs persisted in `setout.json` (per project) keyed by
   IFC GlobalId, so rerunning after a design change keeps IDs, flags moved/new/removed points, and exports a
   *delta* file for re-staking only what changed (feeds Change Radar).
5. **Offset line generator**: for each stake group compute control points along grid lines at a configured distance,
   skipping positions inside other elements (simple point-in-polygon vs the model's footprints).
6. **Grid-offset table**: needs the grid definition (IfcGrid, or DXF layer `GRID` axes already read by cad2bim);
   compute perpendicular distance to the two nearest axes and the neighbour check diagonals; export to Excel via the
   existing `xlsx.js`.
7. **As-staked import**: parse CSV (delimiter and column auto-detect with a mapping dialog), match by ID, apply the
   inverse transform, compute dX/dY/dZ/radial; tolerance profile editable (default from IS 456 / ACI 117 /
   project spec, shown as an assumption, never hidden). Overlay with instanced spheres coloured by status.
8. **Report**: PDF through the existing `jspdf`; one page per floor with datum, tolerances, table and plan; signature
   box for surveyor/engineer.
9. **UI**: right panel "Set-Out" with 4 steps (Select -> Datum -> Format -> Export), live count and a map preview of
   points on the plan. A "Dry run" mode draws the points on the plan without exporting.
10. **Tests / fixtures**: rotated and translated sample building (e.g. 17.43 deg, base point 500 km away), expected
    coordinates computed independently; golden CSV per format; as-staked fixture with injected 3/8/15 mm errors
    and a 12 mm systematic shift.
11. **Privacy / safety**: runs locally; the PDF states that coordinates are derived from the model and the
    datum entered by the user, and requires the surveyor to verify control before use.

MVP (1-2 weeks): column/footing centres + corners, base-point datum, generic CSV (+ one controller format), DXF
points, grid-offset table. Then two-point calibration (week 3), more controller formats, offsets and phone/PDF
sheet (week 3-4), as-staked import with deviation report (week 4-5), delta re-staking and floor carry-forward.

### Future update

- **Phone stakeout without an instrument**: use GNSS/RTK rover via Web Serial/Bluetooth NMEA (Emlid-class) to
  show "go 12 cm north-east" arrows to each point on a phone; a browser-based stakeout for small sites.
- **Sleeve / embed / anchor bolt mode**: take Sleeve Punch openings and Shutter Plan inserts and stake them
  with their orientation and level, printing a one-page template for each repeated detail.
- **Rebar starter and verticality**: Scan Check point cloud of the column forms compared with the stakeout to flag
  plumb/lean before the pour (extends the existing as-built check).
- **Auto control network**: suggest 3-4 well-placed benchmark points around the building from the footprint and
  show intervisibility lines.
- **Revit / pyRevit side**: a pyRevit button exports the same points from a Revit selection through the shared
  JSON so Revit-only teams get the same formats.
- **Pour Planner integration**: the set-out record becomes a mandatory "pre-pour clearance" item with a green tick
  before the RMC order is released.
- **Tolerance statistics**: project dashboard of average deviation per crew/floor to improve means and methods.
- **Voice command**: "stake column C14" on a phone/tablet to jump to the point sheet (pairs with Chat-to-Model).

### Market check

- Layout and stakeout from BIM exists but is generally paid or tied to specific ecosystems: web platforms such as
  STRATUS generate total-station CSV point files and DXF graphics from scoped model elements, and controller
  vendors (Trimble, Leica) provide their own field software that assumes the points already exist in a file.
- Open-source IFC tooling (IfcOpenShell, IfcCSV, ifc5d) can export properties or quantities to CSV/XLSX, but there
  is no ready browser workflow that writes controller-ready coordinate files with a datum wizard.
- Search on 2026-10-18 did not find a free, browser-based tool that combines IFC / DXF-to-3D selection, datum
  calibration, controller CSV formats, grid-offset tape tables, offset lines and an as-staked deviation report.
  (Verify again before launch. Confirm the exact column order and delimiter of each controller format with its
  manual; this search did not verify Leica import formats.)
- It extends the toolkit: Sleeve Punch, Shutter Plan, Scan Check, Pour Planner and Change Radar each gain a
  field-ready output or a field-return input.

Sources:
- [Package Point List (STRATUS: total station CSV and DXF from model)](https://gtpservices.atlassian.net/wiki/spaces/SK/pages/166821968/Package+Point+List)
- [NRCS: Stakeout .csv points and key-in lines (Trimble Access file requirements)](https://www.nrcs.usda.gov/sites/default/files/2023-03/Stakeout%20.csv%20Points%20and%20Key-In%20Lines%20-%20Survey%202023.pdf)
- [ifc5d (IFC to CSV, quantity takeoff library)](https://pypi.org/project/ifc5d/0.7.9)
- [Emlid community: CSV export with stakeout errors](https://community.emlid.com/t/csv-export-format-one-point-per-line-project-stakeout-errors/44358)
