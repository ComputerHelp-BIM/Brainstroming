# 20261023 Idea: Hanger Hero — one-click MEP hangers and supports: span rules per service, trapeze grouping of parallel runs, seismic sway bracing, load per anchor, an "anchor-before-pour" insert layout for the slab soffit, and a support BOQ / cut list

## Entry 1

```
date-time:   2026-10-23 15:23 UTC (scheduled run, fourth idea of the day)
idea name:   Hanger Hero — "Select a floor of coordinated ducts, pipes, cable trays and sprinkler
             mains. Press Hang. Every run gets supports at the right spacing for its size and service.
             Parallel runs share one trapeze instead of five rods. Hangers move off beams, PT tendons,
             joints and rebar-dense zones on their own. Each anchor shows its load and passes or fails.
             Seismic braces drop in where the code asks for them. Then one sheet goes to site BEFORE the
             slab is poured: where every cast-in insert or anchor channel goes, tied to the grid, so
             nobody drills 4,000 holes into a fresh slab later."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new MEP stack next to the StrucBIM stack): pick views /
                   systems, Hang, review, apply; places real hanger / trapeze / brace families,
               (2) cad2bim web viewer: "support view" (MEP translucent, rods and channels solid, anchors
                   coloured by load ratio) that the MEP contractor opens in a browser,
               (3) Pour Planner / Site QR Pass: the insert layout per pour zone printed on the pour card,
                   QR on each zone opens the 3D anchor map on the phone,
               (4) Sleeve Punch + Ceiling Composer: shares the same beam / slab / ceiling-height model,
                   so sleeves, ceiling hangers and MEP hangers are checked against each other.
             Outputs: placed hanger families with parameters (rod dia, length, anchor type, load, support
             ID), trapeze assemblies, seismic braces, anchor load table, insert / anchor-channel setting-
             out plan (DXF + PDF) per pour zone, support BOQ (rods, nuts, channels, clamps, anchors,
             braces) with cut list, and BCF for anything it could not support.
potential:   Very high, every day, on every project with a coordinated MEP model (commercial, hospital,
             data centre, IT park, mall, high-rise residential with basements).
             Time today (one 10,000 m2 office floor plate with HVAC, plumbing, fire and electrical):
               - placing hangers by hand in Revit at 1.5-3 m spacing: 1,500-3,000 supports per floor,
                 2-4 days per floor for one modeller, usually skipped and left to site,
               - deciding where parallel runs can share a trapeze and sizing the channel: done on site
                 by the fitter, by eye,
               - moving hangers that land on a beam, a PT tendon band, an expansion joint or a column
                 head: hours of fiddling, or found when the drill hits a tendon,
               - seismic bracing (IS 1893 Part 1 cl. 7.13 non-structural elements, NFPA 13 for sprinkler
                 mains, ASCE 7 ch. 13 / SMACNA for export projects): usually a separate paid vendor
                 drawing,
               - support BOQ: taken off as "lump sum" or a % of pipe cost, always wrong,
               - total: 3-6 days per floor, plus rework on site.
             With Hanger Hero: one floor supported, checked and scheduled in 5-10 minutes; the BOQ
             falls out of the model; the insert layout is ready before the shuttering is closed.
             Site angle (the wow for contractors): post-installed anchors in a 10,000 m2 floor cost
             drilling time, dust, noise permits in occupied buildings, and the risk of cutting
             rebar or a PT tendon (a cut tendon is a structural incident). Cast-in inserts placed from
             the model before the pour cost a fraction and take zero drilling. Today nobody knows the
             insert positions early enough; this tool prints them one pour ahead.
             Money angle: supports are 10-20 % of MEP installation cost and a big share of MEP labour
             hours. Trapeze grouping alone typically cuts rod and anchor count by 25-40 % on corridors
             and plant rooms. A typical take-off swing between "lump sum" and the real BOQ on an office
             tower is many lakhs of rupees of steel and anchors.
             Trending angle: "MEP prefabrication", "modular racks", "DfMA" and "construction-ready
             models (LOD 400)" are constant topics on LinkedIn; Hilti, Victaulic, Unistrut and Mupro
             post support-modelling content weekly; Instagram BIM pages love fast-forward clips of
             models filling up with detail. The reel: bare floating ducts and pipes -> press Hang ->
             thousands of rods rain down from the slab in a wave -> corridor runs snap into trapezes ->
             3 red anchors flip green after auto-shift off a tendon band -> braces appear -> the slab
             goes translucent and shows the insert grid -> "3,140 supports, 812 inserts, BOQ ready,
             6 min 12 s".
             Market: Hilti PROFIS / BIM tools, MagiCAD Supports, eVolve Mechanical, Victaulic Tools for
             Revit, Mupro, Sikla, Unistrut and Walraven plugins, Trimble/SysQue and Revit Fabrication
             (all paid or vendor-locked: they place only that vendor's catalogue, need fabrication
             parts / ITM content, and none of them knows about Indian PT slabs, IS 1893 bracing for
             Indian zones, or a pour-zone insert sheet). Revit has no built-in automatic hanger placement
             for normal (non-fabrication) ducts and pipes. To our knowledge no free, vendor-neutral tool
             does spacing + trapeze grouping + structure-aware anchor placement + load check + insert
             layout per pour + BOQ together.
```

### Description

**The pain (daily life of a BIM modeller doing MEP coordination).**
- Clash-free MEP models are delivered without supports, because placing them by hand is slow and boring.
  The model says "coordinated", but the space for rods, channels and braces was never reserved; on site
  the trapeze for a duct bank hits the sprinkler main that was "clear" in Navisworks.
- Spacing rules depend on service, material and size (GI pipe 15 mm vs. 150 mm, CPVC sprinkler, UPVC
  drainage with slope, rectangular duct by width, cable tray by load). Modellers keep them in a PDF and
  guess.
- Hangers land on beams, slab drops, expansion joints, PT tendon bands, column capitals or right under a
  joint in the false-ceiling grid. Each one needs a manual nudge.
- Load per anchor is never checked in the model; anchor type is chosen on site.
- Inserts that should be cast in the slab are decided after the slab is poured, so everything is drilled.
- The support BOQ is a guess, and the MEP contractor's claim for "extra supports" is hard to verify.

**The idea, in seven parts.**
1. **Read the services and the structure.** MEP: ducts, pipes, cable trays, conduits banks, sprinkler
   mains and branches with system, material, size, insulation thickness, fluid weight (filled pipe), slope.
   Structure: slabs (thickness, PT or RCC, drops), beams (soffit, width), columns, expansion joints, PT
   tendon layout if modelled or a "no-drill band" drawn as a region; from the Revit model, linked
   structural model, cad2bim / Structura IFC, or etabs-mcp.
2. **Spacing rule pack (YAML, office editable).** Per service + material + size: max spacing, extra
   support within x mm of each bend, tee, valve, equipment connection and riser; rod diameter by load;
   shipped presets for common Indian practice and for MSS SP-58 / SMACNA / NFPA 13 / BS EN 12845 style
   export projects, each one clearly labelled so the office picks the one its specification calls for.
3. **Trapeze grouping.** Runs that are parallel (angle < 5°), within a band and at compatible heights are
   grouped; one trapeze per station at the tightest spacing of the group; channel length and section
   picked from a load table (UDL from all services on it); stacked racks get multi-tier trapezes.
   Grouping is shown as a preview so the modeller can split or merge groups.
4. **Structure-aware anchor placement.** Each support point is projected up to the structure above; if it
   lands on a forbidden zone (beam side too close to the edge, PT band, joint, column head, slab opening,
   edge distance less than the anchor rule) it slides along the run within the allowed spacing tolerance;
   if no legal point exists it is flagged (BCF) instead of being placed wrong. Rod length = slab soffit
   (or beam soffit / steel member) to hanger point; long rods get a brace or a drop member flagged.
5. **Load check.** Load per support = dead weight of pipe + fluid + insulation + tray contents (rule
   values per tray type) over the tributary length, with an impact factor; anchor tension and shear vs. a
   capacity table for the chosen anchor type and slab grade; each anchor coloured by utilisation. It is a
   modelling check with standard tables, not a substitute for the anchor vendor's design or the engineer's
   sign-off, and the report says so.
6. **Seismic sway bracing.** For the project zone (IS 1893 zone II-V, or the export code), braces
   (transverse / longitudinal) at the spacing the selected preset gives, on mains above the size threshold
   and on all sprinkler mains; brace length and angle checked against nearby services.
7. **Anchor-before-pour sheet.** All anchor points under one slab are clustered by pour zone (from Pour
   Planner or slab regions), snapped to a buildable grid, and exported as a setting-out plan with grid
   dimensions, insert type and count, and anchor-channel lengths for dense zones (plant rooms, corridors);
   the same points as a CSV for the total station (Set-Out Pack). After the pour the as-cast inserts can be
   ticked off on the phone (Site QR Pass), and the hangers re-snap to the as-built positions.

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)**: the bridge, Bulk Tool and Filter Parameter stacks, AutoLevel and AutoDIM
  already place and annotate elements in batches; Hanger Hero is a new MEP stack using the same transaction
  and UI patterns, and its BOQ uses the existing openpyxl / Excel writer from BBS Generator.
- **cad2bim**: IFC viewer and actionable QA; supports are written as `IfcMechanicalFastener` /
  `IfcDiscreteAccessory` with property sets so the support view works for non-Revit users too.
- **etabs-mcp / Structura**: slabs, beams and joints from the structural model, so the anchor map can be
  checked against the analysis model and PT layout.
- **C2B**: when the MEP consultant only sends DXF layouts, the routes come from the DXF extraction with a
  default height per service layer.
- Idea links: **Sleeve Punch** (openings in beams and walls; Hanger Hero is the same idea for the slab
  soffit), **Ceiling Composer** (ceiling hangers and MEP hangers share the void and must not collide),
  **Pour Planner** (pour zones and pour cards), **Set-Out Pack** (insert points to total station), **Site QR
  Pass** (as-cast check), **Kit Cutter** (precast slabs need inserts cast in the yard, same output),
  **Live Wire** and **Drain Line** (their routes get supported automatically), **IDS QA** (an IDS rule
  "every pipe > 50 mm has a support within 3 m" checks the result), **Bill Painter** (supports counted in
  RA bills by installed status).

**Wow moments.**
- Thousands of rods drop into a bare MEP model in under ten minutes, all at the right spacing.
- Five rods become one trapeze; the counter shows anchors saved and rupees saved.
- A hanger on a PT tendon band slides 300 mm along the duct by itself and turns green.
- The site engineer gets an insert sheet one pour ahead and the slab is never drilled.
- The MEP contractor's "extra supports" claim is checked against a model BOQ in one click.

### Implementation

**Phase 1 — Spacing, placement, BOQ (3-4 weeks).**
- Revit reader (Python 3 via the existing bridge): `Duct`, `Pipe`, `CableTray`, `Conduit` and fitting
  connectors; system classification, size, material, insulation; curve geometry in model coordinates.
- Structure reader: slab bottom faces (via `Floor` geometry / `ReferenceIntersector` ray up from each
  support point, which also handles linked models), beams, joints, "no-drill" regions (filled regions or a
  generic model category the office draws).
- Spacing engine: station points along each curve at max spacing, extra points near fittings and
  equipment; YAML rule pack with presets; unit tests with known runs.
- Generic, vendor-neutral hanger families (rod hanger, clevis, strap, trapeze with channel, wall bracket)
  with shared parameters: `HH_SupportID`, `HH_RodDia`, `HH_RodLength`, `HH_AnchorType`, `HH_Load_kN`,
  `HH_Utilisation`, `HH_Group`; placed with `NewFamilyInstance` in one `TransactionGroup`.
- Excel BOQ: rods by dia and cut length, nuts / washers, channels by section and cut length, clamps by
  size, anchors by type; per level and per system.

**Phase 2 — Trapeze grouping, structure avoidance, loads (3 weeks).**
- Grouping: parallel-run detection by direction vector and offset band (STRtree on run segments),
  height compatibility, merged station spacing; channel and rod sizing from load tables.
- Avoidance: slide along run within tolerance, then alternative trapeze width; unresolved points to BCF.
- Load calculation per support and anchor with configurable unit weights; colour overrides by utilisation.

**Phase 3 — Seismic, insert sheet, viewer (3 weeks).**
- Brace placement per selected preset and zone; clearance check of brace envelope against other services.
- Pour-zone clustering, snapping to a grid, DXF via ezdxf (already bundled in `lib/py3`) and PDF sheet with
  grid dimensions and insert schedule; CSV for Set-Out Pack; QR per zone.
- IFC export of supports and the cad2bim support view; as-cast tick-off read back from the viewer.

**Tech notes / risks.**
- Engineering responsibility: presets are starting points; the office must load the spacing and bracing
  values from the project specification; anchor capacities come from the selected manufacturer's data;
  every report carries "for coordination and quantity, to be approved by the MEP / structural engineer".
- Performance: a 3,000-support floor should place in one transaction group in < 2 min; families are loaded
  once and instances created in batches; previews use DirectShape.
- Sloped drainage: rod lengths follow the slope; supports at each change of direction.
- Fabrication parts (ITM) projects: detect and offer to place native fabrication hangers instead of the
  generic families, so it works for both workflows.
- Re-run safety: supports carry `HH_SupportID` and a hash of the host run; re-running after a reroute
  updates, moves or deletes only the affected supports.

### Future update
- Prefab racks: group trapezes on corridors into rack modules with a rack drawing, lifting points and
  delivery sequence (DfMA), linked with Kit Cutter for precast work.
- Vendor catalogue switch: map generic supports to Hilti / Walraven / Sikla / Unistrut / local fabricator
  parts for a quote, while the model stays vendor-neutral.
- Thermal expansion: anchor and guide points on hot water / steam lines with expansion loop suggestions.
- AI prompt through the cad2bim MCP agent: "support level 5, group corridor runs, avoid PT bands, export the
  insert sheet for pour zone B".
- Progress: hanger installed status from site photos or QR scans feeding Bill Painter and Model Reel.
- Space reservation early: run in "envelope mode" on the coordination model to reserve the support zone
  before final routing, so clashes with supports are found in the same Navisworks / cad2bim clash run.
