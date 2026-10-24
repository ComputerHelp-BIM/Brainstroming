# 20261024 Idea: Edge Guard — find every open edge, shaft, lift pit and slab opening at each construction stage from the 4D model, place cuplock scaffolding, guardrails, safety nets and opening covers automatically, price the rental, and give the site a daily safety pack with a red/green 3D hazard replay

## Entry 1

```
date-time:   2026-10-24 (scheduled run)
idea name:   Edge Guard — "Press Play on the 4D model and watch every place a worker can fall turn red,
             then press Protect and watch it turn green: cuplock scaffold along the façade, guardrails
             on every slab edge, covers on every shaft and cut-out, safety nets under the deck, with a
             rental bill for the scaffolding and a one-page safety card per floor per day."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Site Safety" stack next to StrucBIM): run the
                   hazard scan, pick protection rules, place native scaffold / guardrail / net / cover
                   families on a "Temporary Works" workset and phase,
               (2) cad2bim web viewer: 4D stage slider; hazards in red, protected edges in green,
                   rental clock per scaffold bay; share link for the safety officer and client,
               (3) etabs-mcp bridge: storey heights, slab edge geometry and slab capacity for
                   scaffold base loads, props and material stacking zones,
               (4) earlier engines reused: Pour Planner (stage dates), Crane Canvas (lift zones and
                   exclusion areas), Site QR Pass (QR sticker per zone that opens today's safety card).
             Outputs: hazard register (Excel + BCF), scaffold layout drawings (plan + elevation per face),
             guardrail / toe-board / net / cover schedules, cuplock component BOQ (verticals, ledgers,
             transoms, base jacks, U-heads, planks, ties) per stage, rental-days bill per bay, scaffold
             tag register (inspected / not inspected), and a daily toolbox-talk PDF per floor.
potential:   Very high, and it touches people, not only drawings. Falls from height are the single largest
             cause of construction deaths in India and the Gulf; every big contractor (L&T, Shapoorji,
             Tata Projects, Afcons) and every developer audit now asks for a "fall protection plan" and a
             "scaffold plan" before a floor starts. Today both are hand sketches or a generic PDF.
             Time today (one G+15 residential tower, 4 façades, 2 lift cores, ~40 shafts and cut-outs):
               - safety officer walks each floor and marks open edges / openings on a printed plan:
                 half a day per floor per week, always behind the actual work,
               - scaffold layout and component count by the scaffold vendor: 3-5 days, re-done when the
                 façade or the floor-to-floor height changes; quantity disputes at every bill,
               - scaffold rental is billed per component per day; nobody can prove which bays were idle,
                 typical over-billing 10-20 % on a ₹40-80 lakh rental package,
               - hazard register and toolbox talk prepared by hand in Word every morning: 1 hour/day.
             With Edge Guard: first full hazard scan and protection layout in 20-40 minutes from the
             existing Revit/IFC model; re-run per revision or per new schedule in 2-5 minutes.
             Expected saving: 8-15 working days of planning per tower, 10-20 % on scaffold rental through
             right-sizing and idle-bay tracking, and a defensible, audit-ready safety plan.
             Wow factor: the 4D replay. Floors rise, edges flash red the day the slab is cast, then go green
             as guardrails and scaffold "grow" on them. A 30-second clip of this is exactly the kind of
             post that goes viral on LinkedIn under #ConstructionSafety #BIM #DigitalTwin, and it is the
             slide that wins a safety-audit or client meeting.
             Market check: Avontus Scaffold Designer, Layher LayPLAN / PLUS, Smart Scaffolder and
             PERI CAD are paid or tied to one scaffold brand, are European-system-only (no Indian cuplock /
             H-frame catalogues), and none of them run a 4D hazard scan from the structural model.
             Navisworks/Synchro can show 4D but do not find hazards or place protection.
             Free, IS/NBC-aware, cuplock-native and linked to the team's own 4D/pour engine: not available.
```

## Description

**The problem.** The BIM model knows exactly where every slab edge, shaft, lift pit, stair void, balcony and
duct cut-out is, and (with Pour Planner) on which day each slab is cast. But the safety plan is still made by a
person with a printout. Hazards are found after they exist, scaffolding is counted by the vendor who bills it,
and nothing links the two.

**What Edge Guard does.**

1. **Hazard scan (per 4D stage).** For each stage date, takes the elements built so far and finds:
   - free slab edges with a drop > 2 m (no wall / parapet / façade yet built on it),
   - openings in slabs: shafts, lift wells, stair voids, cut-outs > 300 mm (IS 3696 / NBC Part 7 limits),
   - lift pits and excavation edges,
   - roof and terrace edges before the parapet is cast,
   - work-at-height zones under cast-in-place work (beam bottoms, chajjas, façade).
   Each hazard gets an ID, type, length/area, floor, stage date, risk level (drop height × exposure) and a
   suggested protection.

2. **Auto protection layout (rule-based, editable).**
   - **Guardrails:** top rail 1.0 m, mid rail 0.5 m, toe-board 150 mm on every free edge; posts spaced to the
     rule; corners and gaps closed; tie-in to columns where available.
   - **Opening covers:** plywood / chequered-plate covers sized to the opening + bearing, or guardrails if the
     opening is > the cover limit; "Do not remove" tag text.
   - **Cuplock / H-frame façade scaffold:** bays laid along each façade face from the setting-out, standard
     verticals 1.0/1.5/2.0/2.5/3.0 m, ledgers 1.3/1.8/2.5 m, lift height matched to floor-to-floor, working
     platforms at each floor, ties to the slab edge every 2 lifts × 3 bays (editable), base jacks on the
     actual ground/podium level, corner returns and gaps for the hoist and crane loading bays.
   - **Safety nets / catch fans:** horizontal nets or catch platforms every n floors as per rule.
   - **Material stacking zones** checked against slab capacity from etabs-mcp.
   Everything is placed as native Revit families on a "Temporary Works" workset and phase, and exported to IFC
   (IfcBuildingElementProxy / IfcElementAssembly with a Pset_EdgeGuard) so the cad2bim viewer shows it too.

3. **Quantities and rental.** Per stage, counts every cuplock component, plank, base jack, tie, guardrail
   metre, net m² and cover. Scaffold bays get erect / dismantle dates from the 4D schedule, so the tool prints a
   **rental-days bill** per component and flags bays that stand idle (façade finished but scaffold still up).

4. **Daily safety pack.** One page per floor per day: plan with red (open) / green (protected) / grey (not yet
   built) zones, the hazards to watch, the protection that must be in place, permit-to-work needed (height,
   hot work near nets), a 5-line toolbox talk in English + Hindi/Marathi/Gujarati, and a QR (Site QR Pass) that
   opens the live 3D view for that zone.

5. **4D replay.** In cad2bim, the stage slider replays construction with hazards and protection appearing day
   by day; export to MP4 (Model Reel engine) for LinkedIn, client and audit.

**Who uses it every day.** BIM modeler (runs scan, places protection, issues drawings), planning engineer
(links stages), safety officer (daily pack, inspections), QS (rental bill), project manager (audit pack).

## Implementation

**Data model (shared JSON, same pattern as other engines).**
```
hazard     { id, type: EDGE|OPENING|PIT|ROOF|ZONE, floor, stage, polyline/polygon, drop_m, length_m,
             area_m2, risk, status: OPEN|PROTECTED|CLOSED, protection_ids[] }
protection { id, kind: GUARDRAIL|COVER|SCAFFOLD_BAY|NET|FAN, geometry, components[{code, qty}],
             erect_date, dismantle_date, inspected_on, tag: GREEN|YELLOW|RED }
catalogue  { system: CUPLOCK|HFRAME|RINGLOCK, items[{code, name, length_m, weight_kg, rate_per_day}] }
rules      { min_drop_m: 2.0, max_cover_opening_m: 1.2, rail_heights: [1.0, 0.5], toe_mm: 150,
             tie_every: {lifts: 2, bays: 3}, net_every_floors: 3, languages: ["en","hi"] }
```

**Hazard scan algorithm.**
- Read slabs/floors per level (Revit API in pyRevit, or IFC via web-ifc in cad2bim). Union slab footprints per
  level and stage → polygon with holes.
- Outer boundary segments = candidate edges; inner rings = openings. Subtract segments covered by walls,
  parapets, curtain walls or railings that are built **at or before** that stage (stage from the Pour Planner /
  Phase / a "Stage" parameter / MS Project or Primavera XML import).
- Drop height = level elevation − elevation of the next built surface below the edge (ray down to slabs/ground).
- Opening size from the inner ring's bounding box; classify cover vs guardrail by rule.
- Lift pits / excavation from foundation & topography; roof from top level.
- Output the hazard list; BCF topics per hazard (reuse the IDS QA/BCF exporter).

**Placement.**
- Guardrails: offset edge polyline inward 150 mm, split into post spacing, place a guardrail family with
  instance length; corners joined.
- Scaffold: offset each façade face outward (wall face + clearance 200-300 mm); run a 1D cutting-stock fit
  (reuse Bar Optimizer) to pick ledger lengths that fill the face with the fewest bay types; stack lifts to
  match floor heights with standard vertical lengths; place a "Cuplock Bay" adaptive / nested family per bay
  with parameters (width, height, lifts, platform levels). Ties at slab edges where a beam/slab exists.
- Covers: rectangle family sized to the opening + 150 mm bearing each side.
- All on workset "Temporary Works", phase "Construction – Temporary", with Stage, Erect, Dismantle params.

**Stack (fits the repo).**
- `pyRevit-addons`: new `Site Safety.stack` under `Essential.panel` with buttons *Scan Hazards*, *Protect*,
  *Scaffold Plan*, *Safety Pack*; IronPython/CPython engine in `lib/py3/edgeguard/`; WPF UI using the AnonGee
  theme XAML; families shipped in the extension (guardrail, cover, cuplock bay, net).
- `cad2bim`: an "Edge Guard" panel in the web app using `@cad2bim/engine` for geometry and the 4D slider;
  hazard/protection overlay layers; MCP tools `edgeguard.scan`, `edgeguard.protect`, `edgeguard.pack` so an AI
  agent can run it ("show me all open edges on L7 next Monday").
- `etabs-mcp`: read storey data and slab capacity for stacking/base load checks.
- Output: openpyxl (Excel BOQ/rental/register), ezdxf (scaffold plan/elevation DXF), reportlab/HTML-to-PDF
  (daily pack), BCF zip.

**MVP (2-3 weeks).**
1. Hazard scan for slab edges + openings per level using Phase or a Stage parameter.
2. Guardrail + cover auto placement in Revit.
3. Excel hazard register + guardrail/cover schedule.
4. Red/green colouring in a 3D view (view filters) and in cad2bim.

**V1 (next 4-6 weeks).** Cuplock façade scaffold with component BOQ, rental-days bill, daily safety pack PDF with
QR, MP4 4D replay, Primavera/MS Project stage import.

## Future update

- **Photo check from site:** safety officer takes a phone photo of a floor; AI vision compares it with the
  expected protection for today and flags missing guardrails ("L7 north edge: no toe-board").
- **Scaffold tag register:** weekly inspection per bay with green/yellow/red tag, expiry alerts on WhatsApp
  (Chat-to-Model engine).
- **Scaffold structural check:** leg loads from platform loading + wind per storey (etabs-mcp wind data),
  base plate bearing on podium slab, tie forces; flag overloaded bays.
- **Vendor catalogues:** add ringlock, H-frame, mobile towers and brand catalogues (with rates) so the same plan
  can be priced by 3 vendors for a comparison sheet.
- **Formwork + scaffold together:** merge with Shutter Plan so props, staging and access scaffold share one
  temporary-works model and one rental bill.
- **Crane + hoist logistics:** keep loading bays and hoist gaps in the scaffold, check crane slew over nets
  (Crane Canvas).
- **Safety KPIs dashboard:** open hazards per floor over time, days-with-zero-open-edges, idle scaffold ₹ per
  week — a one-screen slide for the weekly site meeting.
- **Compliance packs:** IS 3696 (Part 1 & 2), IS 4014, NBC 2016 Part 7, BOCW Act checklist, and Gulf
  (OSHAD / Dubai Municipality) rule sets switchable in the rules JSON.
- **Improvement to existing tools:** add a "Stage" parameter writer to the existing AutoLevel/Bulk Tool buttons
  so any model gets 4D stages in one click, which Edge Guard, Pour Planner and Model Reel all reuse.
