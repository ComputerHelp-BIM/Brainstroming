# 20261025 Idea: Bolt Up — turn an ETABS / Revit steel frame (PEB sheds, mezzanines, steel-composite floors, pipe racks, canopies, staircases) into IS 800 designed connections in 3D, assembly marks, shop drawings, DSTV NC1 files for CNC, plate nesting DXF, bolt / weld / paint lists, tonnage BOQ and a QR erection sequence

## Entry 1

```
date-time:   2026-10-25 (scheduled run)
idea name:   Bolt Up — "Pick the steel frame. Press Bolt Up. Every beam-column, beam-beam, bracing and
             base joint gets an IS 800 connection designed from the ETABS forces and modelled in 3D (plates,
             bolts, welds, stiffeners, cleats, anchor bolts). Every member and plate gets an assembly mark.
             You get one shop drawing per assembly, a DSTV NC1 file per part for the CNC line, nested plate
             DXFs, a bolt list, a weld-length list, paint area, a tonnage BOQ and an erection sequence the
             site team scans from a QR on each member."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Steel" stack beside StrucBIM): pick framing / columns /
                   bracing, choose a connection rule set, preview each joint with a pass / fail colour,
                   place connections, generate sheets and fabrication files,
               (2) etabs-mcp bridge: frame sections, end releases, load combinations and end forces
                   (V, M, P, T) per member end — the design input for every connection,
               (3) Structura web viewer: the same steel model in the browser, joints coloured by
                   utilisation, click a joint to see the calc sheet, members coloured by erection day,
               (4) Bar Optimizer + Site QR Pass engines (earlier ideas): cutting stock for 12 m sections
                   and plates, QR stickers per assembly mark.
             Outputs: 3D connections in Revit (native Steel Connections where possible, else Generic Model
             plate / bolt families), IS 800 calc sheet per connection type (PDF), assembly + part marks,
             GA plans / elevations with marks, shop drawings per assembly (DXF + PDF), DSTV .nc1 per part,
             nested plate DXF per thickness (layers CUT, HOLE, MARK), section cut list from 12 m bars,
             bolt list (grade 8.8 / 10.9 / HSFG, dia, length, qty incl. 5 % spare), weld list (fillet size
             x length), paint / galvanising area (m²), tonnage BOQ by section and by zone, and an erection
             sequence per crane position.
potential:   Very high. Steel is growing fast in India and the Gulf: PEB warehouses and logistics parks,
             data centres, metro and airport roofs, mezzanines in factories, steel-composite high-rise floors
             (Make in India / Gati Shakti push, NHAI and DFC sheds). Every one of these jobs needs connection
             design, shop drawings and NC files, and most small and mid BIM teams do not own Tekla.
             Time today (one 60 m x 120 m PEB / steel warehouse with mezzanine, ~1,400 members, ~2,600
             joints, ~320 t):
               - connection design in Excel / STAAD sheets joint type by joint type: 6-10 days,
               - modelling plates, bolts and stiffeners in Revit or AutoCAD by hand: 10-15 days,
               - assembly marking and GA drawings: 3-4 days,
               - shop drawings (300-600 assemblies): 3-5 weeks of drafting,
               - typing NC files / plate lists for the fabricator: 3-5 days, with hole errors found on site
                 (re-drilling costs ₹300-800 per hole plus crane idle time),
               - bolt, weld, paint and tonnage lists in Excel: 2-3 days, re-done after every revision.
             With Bolt Up: first full set in 1-2 hours including review, 10-20 minutes per revision.
             Expected saving: 5-8 weeks per steel package, plate waste down from 12-18 % to 5-8 %, and
             zero "holes don't match" surprises because the NC files come from the same 3D model.
             Wow effect: the bare stick model "snaps" — plates, bolts and stiffeners grow onto every joint in
             a wave across the building, joints flash green / amber / red by utilisation, then members fly
             off one by one into flat shop drawings and the plates drop onto nested sheets with a CNC tool
             path. A 20-second reel of exactly the "design to fabrication" / steel-erection content that is
             trending on LinkedIn and Instagram (Tekla and Advance Steel reels, PEB erection time-lapses).
             Market: Tekla Structures (very expensive per seat), Advance Steel (paid, AutoCAD-based, weak
             link to Revit families), IDEA StatiCa Connection (paid, design only, no shop drawings),
             SDS2 (paid), Revit native Steel Connections + Autodesk Steel Connections for Revit (design
             needs a paid cloud / Robot add-in, Eurocode / AISC first, no IS 800, no DSTV, weak shop
             drawings), PEB vendors' in-house tools (closed, not for consultants). To our knowledge nothing
             free gives ETABS forces -> IS 800 connections in Revit -> assembly marks -> shop drawings ->
             DSTV NC1 -> plate nesting -> bolt / weld / paint lists -> erection QR in one tool.
```

### Description

**The pain (daily life of a BIM modeller on a steel package).**
- The structural engineer sends an ETABS model with I-sections and a line in the report: "connections by
  fabricator". The BIM team models sticks in Revit. The joints are empty.
- Someone then designs connections in an Excel sheet, one joint type at a time, with forces copied by hand
  from ETABS tables. A wrong combination or a missed release is common.
- Plates, bolts and stiffeners are modelled by hand in Revit (or redrawn in AutoCAD), and then drawn again in
  2D for each assembly. Hole positions are typed again into the fabricator's NC software.
- Every architect or engineer revision (bay change, extra mezzanine, heavier crane) breaks all of it, and the
  lists (bolts, welds, paint, tonnage) are redone in Excel.
- On site, members arrive with mismatched holes, wrong marks or in the wrong order for the crane.

**What Bolt Up does.** It treats a steel joint as data: *which members meet, at which angle, with which forces,
under which rule*. From that one record it designs the connection, models it, marks it, draws it, writes its
NC file, counts its bolts and schedules its erection. Change the record (or the ETABS model) and everything
downstream regenerates.

**Daily use cases.**
1. *PEB / warehouse*: portal knee and apex end-plate moment joints, purlin cleats, X-bracing gussets, base
   plates with anchor bolts and pedestal check, crane bracket — all generated in one pass.
2. *Mezzanine in a factory*: beam-to-column fin plates, beam-to-beam notched cleats, chequered-plate cut list,
   staircase stringer joints.
3. *Steel-composite office floor*: secondary-to-primary shear tabs, shear-stud layout and count, column splices
   per two floors, moment joints at the core.
4. *Pipe rack / canopy / ramp / signage frame*: repetitive frames where 90 % of joints are one or two types —
   the biggest time saving.
5. *Tender stage*: tonnage, bolt count and paint area in minutes from the ETABS model, before anyone models
   connections, for an accurate steel quote.

### Implementation

**1. Read the frame (etabs-mcp + Revit).**
- From ETABS via etabs-mcp: members, sections (ISMB / ISHB / ISMC / ISA / tubes / built-up), end releases,
  orientation, joint coordinates, load combinations and end forces per combination (P, V2, V3, T, M2, M3).
  Envelope per member end, keep the governing combination name for the calc sheet.
- From Revit: Structural Framing / Columns / Bracing with their analytical model, matched to ETABS members by
  geometry (tolerance 10 mm) and the GUID mapping used by Twin Check (earlier idea).
- Build a **joint graph**: each node lists the members that meet, their angles, the main member, the flange or
  web face they land on and the gap to the nearest other element.

**2. Classify and pick a connection.**
- Joint classifier (rules, not AI first): beam-to-column flange, beam-to-column web, beam-to-beam (same / lower
  level), splice, bracing gusset, base, apex, knee, purlin cleat, stair stringer.
- Rule sets in YAML per office / client: "Simple shear: fin plate up to 400 kN, double angle above",
  "Moment: extended end plate, 4 bolts per row, M20 8.8", "Base: 4 or 6 anchor bolts, grout 50 mm".
- Connection library (pure Python, unit tested): fin plate, end plate (flush / extended), double angle cleat,
  web cleat with notch, gusset plate for bracing (Whitmore section), splice cover plates, base plate (pinned /
  fixed with stiffeners), purlin cleat, stiffeners and doubler plates.

**3. Design to IS 800:2007.**
- Bolt shear, bearing and tension (cl. 10.3), HSFG slip (cl. 10.4), prying (cl. 10.4.7), block shear
  (cl. 6.4), fillet weld strength (cl. 10.5.7), plate bending, column web panel shear and stiffener check,
  base plate bearing on concrete (IS 456 grade) and anchor pull-out.
- Iterates: if a joint fails, the engine tries the next option in the rule set (more bolts, thicker plate,
  stiffener, grade up) and records why. Output per joint: utilisation, governing check, combination.
- Calc sheet per connection *type* (not per joint): the 2,600 joints of a warehouse typically collapse into
  25-60 types, each with a one-page PDF a checking engineer can sign.

**4. Model in Revit.**
- Preferred: Revit native Structural Connections (`StructuralConnectionHandler`) with the matched type and
  parameters written from the design.
- Fallback (and for detail the native API does not expose): Generic Model / Plate families placed by the
  AnonGee bridge — plates as extrusions with holes, bolts as a nested family with dia / length / grade,
  welds as model lines with a size parameter. Members are trimmed / notched / coped with void cuts.
- Clash check of each connection against slabs, MEP and other joints (re-uses Sleeve Punch clash engine);
  bolt spanner clearance (minimum socket clearance per dia) flagged in amber.

**5. Mark, draw, export.**
- Assembly marks: identical members + their attached parts get one mark (geometry hash of member + plates +
  holes). Part marks for plates (P1, P2…). Marks written to Revit parameters and to the IFC (IfcElementAssembly).
- Shop drawings: one sheet per assembly from a template (front view, top view, end views, plate details, part
  list, bolt list, weld symbols, dimensions to hole centres), generated by the Drawing Autopilot engine
  (earlier idea) in DXF + PDF.
- DSTV NC1 writer (plain text format: header ST, contour AK, holes BO, marks SI) per part; plate parts also as
  DXF nested per thickness with a simple true-shape nesting (bottom-left fill, rotation 0/90/180/270).
- Section cut list from 12 m / 10 m mill lengths via Bar Optimizer's cutting-stock solver, offcuts tagged.
- Lists: bolts (by grade / dia / length, + 5 % spare), welds (fillet size x length, shop vs site), paint /
  galvanising area per member (perimeter x length − contact faces), tonnage by section, zone and erection day.

**6. Erection sequence (4D-lite).**
- Grid / bay-wise sequence: columns + base → bracing bay first (stability) → rafters / primary beams →
  secondary → purlins / decking. Crane position and reach from Crane Canvas (earlier idea) if available.
- Each assembly mark gets a day and a QR (Site QR Pass engine): scan on site → see the member in 3D, its
  joints, bolt list and torque value, then tick "erected / bolted / torqued".
- Structura web viewer plays the sequence and colours by status — a live progress view for the PMC.

**Tech stack.** Python 3 core (connection library + IS 800 checks + DSTV writer + nesting) shared by pyRevit
(IronPython / CPython bridge already in AnonGee.extension) and a small WASM / JS port for the web viewer;
etabs-mcp for analysis data; Excel template in `bridge/excel` style for rule sets and overrides; Three.js in
Structura for the 3D joint view.

**MVP (4-6 weeks).**
1. Read ETABS end forces through etabs-mcp, build the joint graph, classify 5 joint types (fin plate, end plate,
   gusset, base plate, splice).
2. IS 800 checks for those 5 with calc sheet PDF per type.
3. Place Generic Model plates / bolts in Revit, assembly marks, bolt + weld + tonnage lists in Excel.
4. DSTV NC1 for beams and columns (holes + cuts), plate DXF per thickness.
5. Demo on a sample PEB shed and record the "snap" reel.

### Future update

- **More codes:** AISC 360 / Eurocode 3 rule packs for Gulf and export work; IS 800 limit-state + working-stress
  toggle for older PMC specs.
- **Moment-connection seismic detailing** per IS 800 Section 12 (special moment frames, reduced beam sections).
- **Cold-formed purlins and sheeting:** Z / C purlin lap lengths, sag rods, sheeting cut list and flashings.
- **Fabricator portal:** the fabricator logs in to the web viewer, downloads NC files per batch, marks parts as
  cut / welded / painted / dispatched; dispatch list per truck (Kit Cutter truck-load engine).
- **Bolt torque log:** site engineer scans QR, enters torque wrench reading, photo; stored against the joint for
  QA and handover.
- **AI joint suggester:** learn from the office's past approved joints which type and size they pick for a
  given force range, and pre-fill the rule set.
- **Tekla / Advance Steel exchange:** IFC 4 with IfcElementAssembly + DSTV so a fabricator who already uses
  Tekla can continue in it, and import their shop model back for a Twin Check.
- **Carbon:** embodied carbon per tonne by mill (EPD values) in the tonnage BOQ.

### Improvement suggestions for existing tools in the repos

- **etabs-mcp:** add a tool that returns envelope end forces per frame end with the governing combination
  name — useful for Bolt Up, Detail Drop and any design check, not only steel.
- **Structura:** recognise Indian steel section names (ISMB / ISHB / ISMC / ISA) in `sections.js` and add a
  "steel tonnage by section / storey" table to the BOQ export — a quick win even before connections exist.
- **pyRevit-addons (AnonGee):** a shared "assembly mark" utility (geometry hash → mark) would serve Bolt Up,
  Kit Cutter (precast) and Skin Cutter (façade panels) alike.
- **cad2bim:** add DSTV NC1 to the export list in `docs/design/formats-and-exchange.md`, so steel models built
  on the web can go straight to a CNC line.
