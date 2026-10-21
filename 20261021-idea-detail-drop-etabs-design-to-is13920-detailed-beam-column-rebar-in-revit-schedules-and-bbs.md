# 20261021 Idea: Detail Drop — ETABS design results in, ductile-detailed beam and column rebar out (native Revit rebar, beam/column schedule sheets and BBS), in one click

## Entry 1

```
date-time:   2026-10-21 (scheduled run, fourth idea of the day)
idea name:   Detail Drop — "Design is done in ETABS. Click once. Every beam and column in the Revit model gets
             its bars: the bar count and diameter worked out from ETABS's required steel, IS 456 / IS 13920
             rules applied (hoops close together at the ends, laps only in the middle half of the span, hooks
             anchored into the joint, curtailment), clashes at the joints sorted, beam and column schedule
             sheets drawn, and the BBS made."
             One rule engine (TypeScript, in etabs-mcp/packages/client, next to designRebarKg) with three
             front ends:
               (1) etabs-mcp web app: new "Detailing" tab. Reads ConcreteDesignRow (top/bottom Ast and shear
                   Av/s at every station) through the bridge it already has, proposes the bars, shows them in
                   the 3D view and lets the engineer change the bar choice,
               (2) pyRevit button (AnonGee tab, Structure panel, next to BBS Generator): takes the approved
                   detailing JSON through the AnonGee Bridge and places native DB.Rebar in the matching Revit
                   beams and columns (matched by grid + level + section, the Twin Check matcher),
               (3) cad2bim / structura: the same JSON written as IfcReinforcingBar, so the browser viewer,
                   BOQ and Joint Congestion Doctor see the same bars.
             Outputs: 3D rebar in Revit + IFC, beam schedule (longitudinal elevation per beam line, the usual
             Indian "beam detail" sheet) and column schedule (section per lift) as DXF/PDF, BBS (Excel,
             IS 2502 shape codes, through the existing BBS Generator), a steel-provided vs steel-required
             report per member, and a steel kg/m3 summary per floor.
potential:   Very high, every week. In an RC building job the step between "ETABS design passed" and "rebar
             drawings issued" is the slowest step, and it is done by hand twice: once as 2D drawings, once
             again as 3D rebar if the client wants BIM.
             Time today (G+10 residential tower, ~180 beams and ~60 columns per floor plate, typical + 3 odd
             floors):
               - reading Ast from ETABS (Display -> Design info, or Excel export) and choosing bars: 1-2 days,
               - beam schedule + column schedule drawings in AutoCAD: 4-7 days,
               - 3D rebar in Revit (when asked): 2-4 weeks for one modeller, often skipped because of this,
               - BBS from the drawings: 3-5 days,
               - every design revision (a beam upsized after a drift check): repeat a big part of it.
             With Detail Drop: about 1-2 hours for the whole building including the engineer's review of the
             proposed bars; a revision is a re-run plus a "what changed" list. Across a typical project with
             3-4 design revisions this is 4-8 weeks of detailer/modeller time saved.
             Quality saved: wrong lap positions (laps at the column face, which IS 13920 does not allow),
             hoop spacing not reduced near joints, bars that cannot fit through the column cage, and BBS made
             from an old design run are the routine site and proof-check objections. Detail Drop applies the
             rules every time and keeps the design run ID on every bar.
             Market: CSI Detail, Bentley RCDC, SOFiSTiK Reinforcement Detailing, AGACAD Smart Rebar, Naviate
             Rebar and Tekla are all paid, per-seat annual licences, and
             most give 2D drawings, not native rebar in the Revit model the team already has. There is no free
             tool that goes ETABS design -> IS 13920 detailing -> native Revit rebar -> BBS in one chain.
             Trending angle: "design-to-fabrication", "analytical to physical", "automated rebar detailing"
             and "AI in structural engineering" are among the most shared structural posts on LinkedIn in
             2025-26. The reel: ETABS with the design check green, one click, and the Revit frame fills up with
             bars storey by storey, hoops packing tight at every joint, then the beam schedule sheet slides in.
```

### Description

**The pain (daily life of a structural BIM modeller and a detailer).**
- The engineer finishes the ETABS design and sends "the design is OK, prepare the drawings". The detailer opens
  ETABS, clicks each beam, reads "Top 1245 mm2 at the left end, 680 mid, 1310 right end", converts that to
  "3-16 + 2-12 extra", and types it into an AutoCAD table. 180 beams per floor, many floors.
- IS 13920 (ductile detailing) adds rules nobody remembers in full: hoops at d/4 or 6x bar diameter within 2d of
  the support, at least 2 bars top and bottom through the whole span, bottom steel at the face at least half the
  top steel, laps only in the middle half with hoops at 150 mm, special confining reinforcement in columns over
  the larger of D, L/6 and 450 mm. Each one is a site objection when it is missed.
- The modeller is then asked for the same bars in 3D (client BIM, rebar clash at congested joints, quantity
  proof). With Revit's own rebar tools that is weeks, so it is usually skipped, and the 3D model and the drawings
  never agree.
- When the engineer revises the design (it always happens after the drift or torsion check), all of it is redone
  and someone has to find which beams changed.

**The idea.** Detail Drop turns ETABS design output straight into detailed bars:

1. **Read the design.** The ETABS bridge already returns `ConcreteDesignRow` (from
   `DesignConcrete.GetSummaryResultsBeam`, plus the column equivalent): required top/bottom longitudinal steel and
   shear steel at each station along every frame. For PlanWin AI Pro users the same rows come from PlanWin's beam
   design output, so the engine is not ETABS-only.
2. **Pick the bars.** For each beam line (continuous beams across columns are grouped by grid line, so the
   schedule reads like a human made it) the engine chooses:
   - through bars (top and bottom) from the minimum of the line, from the office's preferred diameters
     (e.g. 12/16/20/25),
   - extra bars at supports and mid-span only where the required steel exceeds the through bars, with
     curtailment lengths from IS 456 cl 26.2.3,
   - stirrups: diameter and spacing from Av/s, then tightened in the 2d end zones as IS 13920 asks,
   - columns: bar arrangement for the section (4, 6, 8, 12 bars...), ties, confining zones, lap in the middle
     half of the clear height, and bar-size continuity checks storey to storey.
   It keeps "bar families" small (the office rule "no more than 3 diameters per floor", "same bars on similar
   beams") so the site can actually build it. Every choice gets a reason line ("3-16 T + 2-16 extra at C4:
   required 1310, provided 1407").
3. **Check the joint.** Before placing anything, the engine checks that beam bars pass between the column bars
   (clear spacing, bar-to-bar from both directions), that hooks fit in the column depth (anchorage 4.2 / Ld), and
   flags joints that need a bar change. This is the light version of Joint Congestion Doctor running on the
   proposed bars, before they reach the model.
4. **Engineer reviews in the web app.** The 3D view colours each member: green (provided / required 1.00-1.15),
   amber (over 1.15, steel being wasted), red (rule or fit problem). The engineer can lock a member, change a bar
   choice, or apply "same as B12 on all similar beams". Total steel kg and kg/m3 update live, so the cost of a
   choice is visible.
5. **Drop into Revit.** The approved JSON goes through the AnonGee Bridge; the pyRevit listener places native
   `DB.Rebar` with shapes from the office rebar shape library, sets the partition / schedule mark / bar mark, and
   tags the design run ID. The existing BBS Generator then produces the BBS from real Revit rebar, unchanged.
6. **Draw the sheets.** Beam schedule elevations and column schedule tables are generated as DXF in the Drawing
   Autopilot sheet style (title block from the C2B template), and as Revit drafting views if wanted.
7. **Revision mode.** On a new design run, Detail Drop compares the new proposal against what is in the model and
   shows only the beams and columns that change ("B-4/C-D level 7: 3-16 -> 4-16 top; stirrup spacing 150 -> 125"),
   with steel kg delta. One click updates only those.

**What makes it a "wow".** One click, and a bare concrete frame becomes a full rebar model with the hoops visibly
packed at every joint: the clip people share. For the team the bigger win is quieter: drawings, 3D bars and BBS
all come from one design run, so they agree with each other by construction.

### Implementation

**Where it lives (no new product, it joins tools that exist).**
- `etabs-mcp/packages/client/src/detailing/` (new): the rule engine in TypeScript, unit-tested like
  `results.ts`. Pure functions: `designRows -> beamLines -> barProposal -> checks`. Runs in the browser (web app)
  and in Node (MCP server tool `propose_detailing`, so an AI assistant can be asked "detail level 5 and tell me
  what is red").
- `etabs-mcp/apps/web`: Detailing tab with the 3D colour map, the member table and the "lock / same as" actions.
- `pyRevit-addons/AnonGee.extension`: new pushbutton **Detail Drop** using the bridge listener pattern
  (modeless window, `IExternalEventHandler`), the Twin Check member matcher, and `Rebar.CreateFromCurves` /
  `Rebar.CreateFromRebarShape`. All placement in one transaction group, so one undo removes it all.
- cad2bim: writer for `IfcReinforcingBar` (the IFC writer already handles properties; bars are swept disk solids).

**Data contract (`detailing.json`, versioned like the other schemas).**
```
{ "schema": "detail-drop/0.1.0", "designRun": "ETABS 2026-10-21 14:05 / combo envelope",
  "code": "IS456+IS13920", "members": [
    { "id": "B-4/C-D@L7", "etabsFrame": "B123", "section": "300x600", "fck": 30, "fy": 500,
      "longitudinal": [ { "face": "top", "type": "through", "n": 3, "dia": 16, "from": 0, "to": 1 },
                        { "face": "top", "type": "extra",   "n": 2, "dia": 16, "from": 0, "to": 0.30 } ],
      "stirrups": [ { "dia": 8, "legs": 2, "zones": [ [0, 1.2, 100], [1.2, 4.8, 150], [4.8, 6.0, 100] ] } ],
      "laps": [ { "face": "bottom", "at": 0.5, "length": 960 } ],
      "check": { "AstReqTop": [1245, 680, 1310], "AstProvTop": [1005, 603, 1005], "ratio": 1.07,
                 "status": "ok", "reasons": ["..."] } } ] }
```

**Rule engine details.**
- Rules as data (`rules/is13920-2016.yaml`, `rules/is456-2000.yaml`), not code, so ACI 318 / EC2 rule files
  can be added later and an office can set its own preferences (preferred diameters, min 2 bars, rounding of
  spacing to 25 mm).
- Bar selection is a small search: for each station, smallest steel area from allowed combinations that meets
  required area and spacing rules, then a smoothing pass across the beam line and across similar beams
  (group by section + span band + steel band), so the schedule has few unique types.
- Development length, lap length and hook length from the code formulas with fck/fy from the ETABS material.
- Unit handling: design rows already come in model units (see `designRebarKg`); convert once at the edge.

**Phases.**
1. **Phase 1 (2-3 weeks): beams, read-only.** Rule engine for beams, Detailing tab with the colour map and
   member table, Excel export of the proposed beam schedule. Value from day one even without Revit.
2. **Phase 2 (2-3 weeks): columns + joint check.** Column bar arrangement, confining zones, storey continuity,
   beam-through-column fit check.
3. **Phase 3 (3 weeks): Revit placement.** pyRevit Detail Drop button through the bridge; shape library mapping;
   BBS Generator runs on the result; one-undo placement.
4. **Phase 4 (2 weeks): sheets + revision mode.** Beam elevation / column table DXF in the C2B template; design
   run diff with "update only changed members".
5. **Phase 5: slabs and footings** (bottom/top mesh from area design results, footing bars from the RC
   Automation workbook format).

**Testing.** Golden tests: 10 hand-detailed beams and 5 columns from a past project, checked by a senior
engineer, must come out the same (or better, with a reason). Mock bridge rows for CI, like the existing
`results.test.ts`. A fit test that fails if any proposed bar overlaps another in the joint.

**Guard rails.** Detail Drop proposes, the engineer approves: nothing reaches Revit until the member is marked
approved, every bar carries the design run ID, and the report states the code clauses used. It does not replace
the engineer's signature; it removes the typing.

### Future update

- **Slab and raft detailing** from ETABS/SAFE strip or area design (top/bottom mesh, extra bars over supports),
  drawn as plan rebar sheets.
- **Steel-saving advisor:** "switching B-series beams on levels 3-9 from 3-16+2-12 to 3-20 saves 1.8 t and cuts
  12 lap couplers"; feeds Bar Optimizer for cutting stock and couplers.
- **Site link:** each beam mark gets a Site QR Pass card with its 3D cage, so the bar bender and the site
  engineer see the same bars the drawing shows.
- **Other codes:** ACI 318-19 and EC2 rule files for export projects; BS 8666 shapes are already in BBS Generator.
- **AI review chat over MCP:** "why is B-12 red?", "make all level 7 beams use 16 and 20 only and show the steel
  change" answered from the rule engine, using the etabs-mcp server that exists.
- **Twin Check loop closed:** Twin Check compares design and model; Detail Drop fixes the mismatch it finds with
  one click, so the toolkit covers find + fix.
- **Model Reel preset:** "rebar build-up" video (frame -> bars storey by storey -> BBS total) for the client
  presentation and LinkedIn.
