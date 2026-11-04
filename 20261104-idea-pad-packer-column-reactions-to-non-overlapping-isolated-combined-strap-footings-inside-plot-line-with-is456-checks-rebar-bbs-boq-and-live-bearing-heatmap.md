# 20261104 Idea: Pad Packer — column reactions plus safe bearing capacity become non-overlapping isolated, combined and strap footings that stay inside the plot line, with IS 456 checks, tie beams, rebar, BBS, PCC / excavation quantities and a live bearing-pressure heatmap

## Entry 4

```
date-time:   2026-11-04 (scheduled run, fourth entry of the day)
idea name:   Pad Packer (Foundation Layout From Column Reactions: Isolated / Combined / Strap Footing
             Auto-Fit, Plot-Line and Neighbour Clash Handling, IS 456 Checks, Tie Beams, Footing Rebar,
             BBS, PCC and Excavation BOQ, Bearing-Pressure Heatmap)
potential:   High. Every RC project starts below ground, and the footing plan is the first drawing the
             site needs and the first one to change. Today the modeller gets a list of column reactions
             (ETABS / STAAD / SAFE / Excel) and a soil report, sizes pads on a calculator or sheet, then
             draws them one by one, then discovers that two pads touch, that an edge pad crosses the plot
             boundary, that a lift pit or a neighbour's wall is in the way, and starts again. Modeller
             time saved (an estimate, not measured): 4 to 8 hours for a typical G+7 footing plan, plus
             about 1 day per redo when the architect moves a column or the soil report changes the SBC.
             The money is in the redo: a footing plan that is final in one hour instead of three
             revisions changes the date the excavation can start. Fits the toolkit's rebar line (BBS
             exists), the ETABS plan (docs/design/etabs.md, E3 "footings from reactions"), the planned
             Create tools (footings) and the earlier Dig Deep / Set-Out Pack ideas, which take footings
             as an input but do not make them.
```

### Description

**The pain (what the modeller and the structural checker actually do today).**
- Inputs arrive in three pieces: a table of column service and factored loads (from the analysis model), a
  soil report with safe bearing capacity (SBC) and founding depth, and the architect's plan with the plot
  line, neighbours' walls and the basement or lift pit. Nothing joins them.
- Sizing is simple and repetitive: area = service load plus about 10 % for footing self-weight and backfill,
  divided by SBC; then depth from shear and bending. A person does this 30 to 120 times, per load case.
  (Worked example: 1,200 kN service load, SBC 200 kPa gives 1.1 x 1,200 / 200 = 6.6 m², about a 2.6 m square.)
- The hard part is **geometry**, not arithmetic: pads of neighbouring columns overlap (so they must become a
  combined footing), a column close to the plot line cannot have a centred pad (so it needs a strap or an
  eccentric footing and a tie beam), a pad swallows a lift pit or crosses an expansion-joint line, and two
  footings at different levels need a 45-degree influence-line check. Revit does not know any of this; it
  draws what it is told.
- When one column moves 300 mm, or the SBC changes from 200 to 150 kPa after the second boring, the whole
  plan is redrawn by hand, and the schedule, the BBS and the excavation volume are redone.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Dig Deep (2026-10-19) takes footings *already modelled* and gives pit, shoring
  and trucks. Set-Out Pack (2026-10-18) gives staking points for them. Load Path X-Ray (2026-10-26) checks
  a footing size against a quick takedown, and Pile Pilot (2026-10-27) lays out **piles** and pile caps.
  Basement Box (2026-10-27) makes a raft for a basement. Drawing Autopilot (2026-10-15) draws a footing
  plan once footings exist. Joint Congestion Doctor and Mark-Merge look at what is already modelled. **None
  of them creates the spread-footing layout from reactions and bearing capacity, resolves overlaps and
  plot-line conflicts, or keeps the layout live when a load or the SBC changes.**
- In cad2bim (from `CLAUDE.md` and `docs/`): `docs/design/etabs.md` lists E3 "RC design from ETABS forces:
  beams, columns, slabs, footings from reactions" and E5 outputs (footing schedule); the first release is
  open to beams and columns only, so footings are a natural next step. `docs/roadmap.md` lists Create tools
  (columns, beams, walls, slabs and footings drawn in cad2bim). The BBS engine (`packages/engine/src/bbs/`)
  already cuts bars from shapes, and the QA module (`packages/engine/src/qa/`) already turns findings into a
  selectable, zoomable list. So the receivers exist; the layout engine does not.
- In the market (not exhaustively checked): CSI SAFE and similar design programs size and design footings,
  but they are paid, they do not deliver a Revit / IFC model with rebar, and the geometry fix-ups (plot line,
  neighbours, strap) are done by the engineer on screen. Revit add-ins that place footings usually place one
  per column with a fixed size. A free, in-browser tool that goes from a load table to a clash-free,
  IFC-ready footing layout with bar schedule is not something I found; treat that as unverified.

**The idea.** A **Foundation** panel in the cad2bim web viewer (and the same through the MCP agent and the
Python console). Nothing leaves the device; the load table and the soil figures stay in the browser.

1. **Bring the three inputs.**
   - *Loads:* paste from Excel or import a CSV (column mark or grid point, service P, Mx, My, factored Pu).
     Later: read the reactions straight from an ETABS / STAAD export once the ETABS reader exists.
   - *Soil:* SBC (kPa), founding level, optional allowable increase for wind / seismic (IS 1893 / IS 6403
     let SBC rise for short-term loads), water table level. The person types these; the tool never
     guesses a soil value.
   - *Site:* the plot line and neighbours' walls from the DXF / CH drawing the pipeline already reads
     (a polyline on a named layer), the lift pit and shafts from the model, and the columns already in
     the 3D model.
2. **Auto-fit in one click, with the reason shown.** For each column: pad size from load and SBC, rounded to
   the team's module (for example 50 mm), centred under the column. Then the packer runs a small set of
   explicit rules, in order, and logs each decision:
   - *Overlap:* two pads closer than a set clearance become **one combined footing**, sized so the resultant
     of the loads sits at the footing centroid (so the pressure is even), rectangular or trapezoidal.
   - *Plot line:* a pad that would cross the boundary is pulled back to the line and flagged **eccentric**;
     the packer proposes a **strap beam** to the nearest interior column (or a combined footing, or a
     cantilever strap) and recomputes both pads.
   - *Obstacles:* a pad that touches a lift pit, a shaft or an expansion-joint line is shifted, notched or
     flagged. It never moves a footing silently.
   - *Levels:* where neighbouring pads sit at different founding levels, the 1:1.5 / 45-degree influence
     line is checked and the shallower pad is dropped or the gap flagged.
   Result: a footing plan with a mark on every footing (F1, F2, CF1, SF1) and the **number of types**,
   reported so the person can trade a few extra concrete cubic metres for fewer unique sizes.
3. **IS 456 checks per footing, with clauses.** Bearing pressure (including eccentricity, with the no-tension
   check), one-way shear, two-way (punching) shear at d/2 from the column face, bending moment at the column
   face, minimum thickness at the edge, and development length of bars. Each result lists clause, demand,
   capacity, utilisation and pass / fail. The tool presents these as a **cross-check and a starting layout**:
   the structural engineer owns the design and checks it in the design software (see Limits).
4. **Live bearing-pressure heatmap in 3D.** Footings are coloured by utilisation (green under 80 %, amber to
   100 %, red above). Drag the SBC slider from 200 to 150 kPa and watch pads grow, merge and the strap
   appear in front of you. Change a column load and only that footing and its neighbours re-pack.
5. **Reinforcement and the paperwork.** Mesh bars both ways, with the right spacing for the bending demand
   and the minimum steel; bent-up or straight bars as the team prefers; column starter bars with lap length
   (the BBS engine already has IS 2502 shapes). Strap and tie beams get stirrups and top / bottom bars.
   Outputs: 3D footings and rebar as IFC (with the usual stable ids), the **footing schedule**, the **BBS**,
   **PCC and excavation quantities** (the pit follows the pad plus working space, which Dig Deep can then
   take over), and a one-page summary with the type count and total concrete and steel.
6. **Change radar for the foundation.** When a new column reaction file arrives, the tool shows which
   footings are now over-stressed or under-used, proposes the new sizes, and lists the changed marks and the
   concrete and steel difference — the way Change Radar (2026-10-14) does for drawings.

**Why it is practical.** The modeller stops redrawing the footing plan each time a load, a column or the
soil number moves, and the checker reads a pass / fail list instead of re-sizing by hand. The layout
decisions that used to live in someone's head (why is this one a strap?) are written down per footing.

**Wow factor (what the person sees and can post).**
- Paste 80 column loads, press one button, and a footing plan **grows out of the columns in 3D**; pads swell,
  touch, snap together into combined footings and a strap beam draws itself to the plot line.
- The **SBC slider**: "200 kPa → 150 kPa" and the building's footings re-pack live, with total concrete in
  m³ and steel in kg changing beside it. A 20-second screen recording is a ready reel: "foundation plan in
  3 minutes, redone in 3 seconds when the soil report changed".
- The **heatmap** that turns a footing red on screen before the site finds out on the day of excavation.
- One click: *Combine similar footings* — "42 sizes → 9 types, +3.1 m³ concrete" — which delights both the
  checker and the contractor.

**Trend link (a judgement, to be checked against live feeds before posting).** "Foundation failed because
nobody checked the soil report" clips, plus short-video posts on tilting buildings after monsoon, come back
every year; AI-in-structural-design and "from Excel to model" posts are common in AEC feeds. The hook:
*"Your soil report changed. Your footing plan just redrew itself."*

### Implementation

Keep it inside cad2bim's own code (no GPL; see `CLAUDE.md`). Phases, smallest useful first:

1. **Isolated pads from a table (1 to 2 weeks).** A load-table importer (paste / CSV) in `apps/web`; a pure
   function in `packages/engine` that sizes a square or rectangular pad from service load, SBC and column
   size, with a unit test against hand calculations. The 3D footing is created as an `IfcFooting` through
   the existing IFC writer, one per column, centred. Shows the heatmap by utilisation.
2. **Geometry rules.** Add overlap merge (two-pad and chain cases), plot-line pull-back with the eccentric
   flag, and obstacle avoidance on 2D rectangles first. Keep the rule engine as a list of small functions
   that each return "changed / unchanged / flagged" with a reason string, so the decision log is free and
   rules can be switched off per project. Start with plain rectangle logic; avoid anything heavy.
3. **Combined and strap footings.** Resultant-at-centroid sizing for combined footings (rectangular first,
   trapezoidal later); strap footing sizing with the strap beam as a rigid link; both reported with their
   checks. Use well-known textbook methods and cite the clause or the method name in each result.
4. **IS 456 check module.** Bearing with eccentricity, one-way and two-way shear, bending, development
   length, minimum thickness. Reuse the clause-and-capacity layout planned for ETABS E3 so one results panel
   serves both. Validate against a small set of worked examples from published textbooks and an independent
   spreadsheet; ship the examples as tests.
5. **Rebar and BBS.** Footing mesh, strap / tie beam cages and starter bars on the existing bar shapes;
   quantities into the BOQ; PCC and excavation volumes with working space as a setting.
6. **Live re-pack.** Dependencies between loads, SBC, columns and footings so that only the touched footings
   are re-solved; undo as one step. Link to the Dig Deep and Set-Out Pack ideas through their inputs.
7. **Revit side.** Footing families and rebar through the Revit 2025 add-in (`bridge/revit`) so the layout
   lands as native footings, using the same marks.

**Data and licences.** Code clauses (IS 456, IS 1893, IS 6403) are written into the rules as the team's own
wording with the clause number; no copyrighted tables are copied in bulk. SBC and all soil numbers stay user
input. Standard libraries only (MIT / Apache-2.0 / BSD / ISC).

**Limits and honesty.**
- A first-pass layout and cross-check, **not a replacement for the geotechnical report or the design
  engineer's sign-off.** The tool does not model soil-structure interaction, settlement, differential
  settlement, liquefaction, expansive soil or groundwater uplift; it should say so on the summary page and
  refuse to size a footing when the SBC or founding level is missing.
- Pads on slopes, footings near existing neighbours (needing underpinning) and raft / pile cases are out of
  scope here; they go to Basement Box and Pile Pilot. When the pads cover more than about half the plan
  area, the tool should suggest a raft instead.
- All time-saving figures above are estimates. Validate on three real, finished projects (loads, soil report,
  issued footing drawings) and report the match: sizes within a stated tolerance, types, concrete and steel.

### Future update

- Read reactions straight from ETABS / SAFE / STAAD exports and write the footing back as a support
  change request for the analysis model (human decides).
- Raft and pile-cap suggestion when pads cover most of the plan, handing over to Basement Box / Pile Pilot.
- Combined footing under shear walls and core walls, with wall-footing strips and boundary element starters.
- Settlement estimate from the borehole data once Pile Pilot's bore-log reading exists.
- Cost and carbon view of alternatives (fewer types vs less concrete, deeper vs wider), with Carbon Lens.
- Site mode: scan a footing QR on the pour card, tick PCC / rebar / pour, and compare the as-built level
  with the model (Site QR Pass, Scan Check).
- Team library of "how we lay out footings" settings per client, learned from accepted layouts.
