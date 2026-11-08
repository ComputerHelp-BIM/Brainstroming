# 20261108 Idea: Fire Line — hydrants, wet risers, pump room and fire tank laid out in the 3D model by the rules of NBC Part 4, with a pressure check at the farthest hose, the Fire Service NOC sheet and BOQ filled in, and a "water reaches the top floor" replay

## Entry 1

```
date-time:   2026-11-08 (scheduled run, second entry of the day)
idea name:   Fire Line (auto fire-fighting WATER system in 3D: landing valves and hose reels per floor, wet
             riser and down-comer, yard hydrants, underground and terrace tanks, pump room with main, jockey
             and standby pumps; pressure and flow check at the worst-case outlet; Fire Service NOC drawing
             and calculation sheet; pipe, valve and pump BOQ)
potential:   High. Every building above the small-building limit needs a fire-fighting layout for the
             Fire Service NOC. The sprinkler and detector heads on the ceiling are in this repo already
             (Ceiling Composer), and so are exits and egress (Exit Map). The water system behind them,
             which is half the NOC drawing set, is in none of the 100 ideas here. Today it is drawn by hand
             in 2D, floor by floor, and the tank and pump sizes are done in a separate Excel. A change in
             the floor count or the plot means redoing both.
             The wow: draw nothing. The riser appears in the lift or stair lobby on every floor, the tank and
             pump room find a place on the basement plan, and a blue "water" column rises through the 3D
             model to the top-floor outlet while a gauge shows the pressure left there (live, as you drag
             the pump size or add a floor).
```

### Description

**The pain.**
- Fire-fighting is one of the last services drawn in 2D on many Indian projects. The MEP modeller copies the
  riser and hose-reel symbols floor to floor and counts them by eye.
- The checks that decide the NOC are not in the drawing: tank volume for the building height and use, pump
  flow and head, pressure at the topmost and farthest outlet, hose-reach coverage, and the distance of
  hydrants from the building. These live in a spreadsheet, and it goes stale when the architect adds a floor.
- Riser, tank and pump room compete with the structure and the lift core for space. They are found late,
  in coordination, when a shear wall or a beam is already in the way.
- Fire Service queries come back as "hose reach not shown", "tank volume not matching height". Each costs a
  re-issue and a re-submission, and a few days of waiting each time (the delay is an estimate from how NOC
  rounds usually go, not measured here).

**What exists today (checked 2026-11-08, not exhaustive).**
- Revit MEP and AutoCAD MEP draw pipes, but the rules (what goes where, how many, how big) are left to the
  person. Hydraulic calculation tools for sprinklers (such as AutoSPRINK and HydraCALC) are paid and
  desktop-only. They work mostly to NFPA 13 and 14, and few of them take an Indian building and write a
  Fire Service NOC sheet.
- Several general "fire sprinkler layout" plug-ins exist for Revit, paid per seat and aimed at the sprinkler
  heads, not at the hydrant, riser, tank and pump chain that a typical Indian high-rise needs.
- In this repo: Ceiling Composer (heads and detectors on the ceiling), Exit Map (egress and NOC checker for
  exits), Hanger Hero (supports for MEP pipes), Drain Line (drainage). Nothing lays out or checks the fire
  **water** system. This idea fills that gap and re-uses their output.
- I did not check each product's current price list, so "paid" here means "sold as a commercial licence, as
  far as the vendors' public pages show".

**The idea.** A **Fire Line** panel in the viewer that works on any IFC or `.c2b` and, later, on the Revit
side. It does five jobs in one pass.

1. **Read the building.** Height, use class (residential, assembly, business, mercantile, hospital and so on),
   floor areas, stair and lift lobbies, the basement plan, the plot line and the road side. All come from the
   IFC and from the walls, slabs and spaces cad2bim already knows.
2. **Lay out the system by rule.** A rule table (NBC 2016 Part 4 as the first pack; state fire rules as
   editable overrides) decides what is needed. Then it places:
   - **Landing valves and hose cabinets** in the stair lobby on every floor, with a **hose-reach circle**
     (coverage by the real walking path, not a straight circle) so uncovered rooms show red.
   - **Wet riser and down-comer**, routed in the lift or stair lobby shaft, clear of the structure, and
     marked as a vertical opening for the structural model.
   - **Yard hydrants** around the building at the allowed spacing and distance from the walls, on the road
     side, with the reach of the fire tender shown.
   - **Underground and terrace tanks** sized to the building (volume from the rule table), placed in the
     basement or the podium level, with a clear plan area and depth.
   - **Pump room**: main, jockey and standby pumps, the inlet and outlet headers, valves and a pump-room size
     that fits the floor. It can be moved by hand, and the pipes follow.
3. **Check the water.** A simple, transparent hydraulic pass (pipe friction by a stated formula, the static
   head of the building height, loss at the fittings) gives flow and residual pressure at the topmost and
   farthest outlet. It shows a pass/fail with the numbers, and says what to change (larger riser, a higher
   pump head, a booster stage for tall buildings). It does **not** replace a licensed fire consultant's
   sign-off; it states its formulas and limits on the sheet.
4. **Collision-check against the model.** Pipes cross beams and shear walls. Fire Line finds each crossing,
   marks it, and offers a sleeve or a re-route. It writes the **sleeve list** in the form Sleeve Punch uses,
   so the structural team gets the openings before the pour.
5. **Write the pack.** The NOC drawing set (plan per floor, riser diagram, pump-room layout, tank layout) as
   DXF/PDF, the **calculation sheet** in the layout the Fire Service reviewer expects (tank volume, pump
   duty, residual pressure, hose coverage), and the **BOQ** (pipe by diameter and length, valves, hose
   cabinets, hydrants, pumps, sleeves) as Excel.

**The "water reaches the top floor" replay.** A blue column fills the riser floor by floor in 3D while a
gauge on the top outlet shows the pressure. Drag the pump head up and down and the gauge moves. Switch the
case to "two hydrants open at once" and the number changes. A client or a fire officer understands this in ten
seconds, and it makes a good short video (the Model Reel idea can record it).

**Time saved (estimate, to be measured).** The layout of risers, hydrants and hose cabinets for a typical
15-floor residential tower: from about one to two days of 2D work to about an hour of review. The tank and
pump check goes from a spreadsheet that is redone after every floor change to a live number. These are
estimates from how teams usually spend time on NOC sets; the first real project will give the measured figure.

**Why it is a "wow".**
- Nobody draws the riser. It is simply there on every floor, and the water shows that it works.
- The pressure gauge moves while you drag a pump size. It feels like a game, but the number is real.
- The NOC calculation sheet appears already filled in. It is a deliverable that a client asks for on every
  building, and today it is rebuilt from scratch each time.
- It runs in the browser, on the device, with no upload. That matters for client drawings that are not meant
  to leave the office (cad2bim's own rule: nothing leaves the device unasked).

**Why this fits cad2bim.** It is the fire-water twin of Ceiling Composer and Exit Map, so it plugs into the
"services on the structure" story. It needs only what cad2bim already has: IFC reading, spaces, a path graph
(Exit Map and Access Pass already build one), the QA rule engine for checks, the BOQ and PDF/DXF writers, and
the sleeve and opening coordinator.

### Implementation

1. **Rule pack as data, not code.** A versioned JSON rule table (`firewater-nbc2016.json`): by use class and
   height, the number of risers, hose-cabinet spacing, tank volume, pump duty, hydrant spacing, and the clauses
   they come from. State overrides are a second JSON that wins over the first. The user can edit it, and every
   number on a sheet quotes its clause. *I have not re-read the clauses in this pass; before building, the
   owner's fire consultant should confirm the first pack against the current code text.*
2. **Placement engine** (in `packages/engine`, a new `fire` folder behind the public entry point):
   - Stair and lobby detection from spaces, using the same space graph as Exit Map.
   - Riser position: score candidate cells in the lobby by clearance from beams and walls and by distance to
     the farthest room; pick the best; carry the same x/y through all floors.
   - Hose-cabinet placement along walking paths, with a reach test on the path graph (walls block, doors
     count as open).
   - Hydrants: sample points along the plot line on the road side at the rule spacing, keep those inside the
     allowed distance band from the building.
   - Tank and pump room: find the largest free rectangle on the lowest level that meets the area and height,
     as a first guess the user can move.
3. **Pipe routing.** Orthogonal routing with obstacles (the beams and walls from the IFC), a small A* on a
   grid, then clean-up to straight runs. Keep it simple at first: risers vertical, mains horizontal, branches
   to cabinets short. Output as `IfcPipeSegment` / `IfcPipeFitting` with the proper `IfcDistributionSystem`
   ("FIRE_FIGHTING"), so the result opens in Revit and any IFC viewer.
4. **Hydraulic pass.** Hazen-Williams for friction (formula and constants shown on the sheet), static head from
   level difference, fittings as equivalent length. A pure TypeScript module with unit tests against hand
   worked examples. Results: flow, pressure at each outlet, the controlling (worst) outlet.
5. **Checks** as rules in the existing QA engine, so they show in the same panel and can be exported as BCF
   issues: "cabinet missing on level 7", "room 7.04 not covered by hose reach", "riser crosses beam B-241",
   "tank volume 18 m³ short for this height".
6. **Outputs.** DXF/PDF through the existing writers; Excel through the existing BOQ writer; sleeve list in
   the Sleeve Punch format; a 3D "water" animation as a shader on the riser mesh driven by the computed
   pressure (a simple colour ramp).
7. **UI.** A Fire Line panel: building facts at the top (editable, with the source shown), a switch per
   system (riser, hydrants, tank and pump), the result list, and a pump-duty slider that updates the gauge
   live. Everything is previewed first and applied only on "Place".
8. **Testing.** Three golden buildings (a 4-storey block, a 15-floor tower, a 30-floor tower with a booster
   stage), checked by hand and then by a fire consultant. Test files stay out of git.
9. **Build order, the smallest useful slice first** (about two to three weeks of one developer, an estimate):
   (a) rule pack + hose cabinets and hose-reach check on one level; (b) riser and the tank/pump sizing sheet
   in Excel; (c) pipe routing and the sleeve list; (d) the hydraulic pass and the replay; (e) the NOC pack.

### Future update

- **Sprinkler hydraulics.** Take the heads from Ceiling Composer and run a branch and main hydraulic
  calculation per hazard class, with the remote-area method.
- **Revit side.** Send the placed system to Revit as native pipes and fittings through the Revit bridge, with the
  families the user already owns.
- **Fire-pump curve library.** Real pump curves from vendor data sheets, so the duty point is read from a
  curve, not typed.
- **Tender and vendor quotes.** Send the BOQ to the Rate Rocket pricing and the Bid Mirror comparison.
- **As-built and handover.** Hydrant and valve tags as QR codes through Handover Hub, with the test dates for
  the annual fire audit.
- **Smoke and pressurisation.** Stair pressurisation fans and smoke-extract ducts from the same panel.
- **Other codes.** NFPA and the UK/Gulf rules as extra rule packs, for teams that work for overseas clients.

### Sources

- cad2bim roadmap and design notes (`docs/roadmap.md`, `docs/design/quick-wins.md`, `docs/design/bbs.md`):
  read on 2026-11-08 to confirm the viewer, QA engine, BOQ and export writers this idea re-uses.
- The other ideas in this repository (Ceiling Composer, Exit Map, Sleeve Punch, Hanger Hero, Drain Line, Handover
  Hub, Rate Rocket, Bid Mirror, Model Reel): checked on 2026-11-08 for overlap. None covers the fire-fighting
  water system.
- The National Building Code of India 2016, Part 4 (Fire and Life Safety), and the Hazen-Williams friction
  formula: named from general knowledge, **not re-read in this pass**. Clause numbers and numeric limits are
  deliberately left out until a person checks the current text.
- Market note: tool prices and features are from general knowledge of vendors' public pages, **not checked live
  in this pass** (no web search was run); treat the "paid" claim as likely, not confirmed.
