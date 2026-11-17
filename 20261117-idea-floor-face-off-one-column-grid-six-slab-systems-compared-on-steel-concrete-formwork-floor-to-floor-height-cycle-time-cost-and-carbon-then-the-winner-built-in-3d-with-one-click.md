# 20261117 Idea: Floor Face-Off — Take the real column grid of your project, build six slab systems on it (beam-slab, flat slab, flat slab with drop panels, PT flat slab, waffle, precast hollow-core), and see them side by side on concrete, steel, formwork, floor-to-floor height, cycle time, rupees and carbon. Pick one and the 3D model is built with it in one click.

## Entry 1

```
date-time:   2026-11-17 (scheduled run, UTC)
idea name:   Floor Face-Off (floor system comparison on your own grid, in the browser, nothing uploaded)
potential:   High. The floor system is the first big structural decision and it is usually made by habit
             ("we always do beam-slab") or by one person's spreadsheet. It drives about 40-60 % of the
             concrete and steel, the formwork cycle, the floor-to-floor height (and so façade area, lift and
             stair size, and total building height under the height band) and the carbon. Today, comparing
             two systems means two ETABS/SAFE models and two days. So the comparison is skipped, and the
             client never sees what a different choice would have saved.
             Time saved: NOT measured here. First test: take 5 past projects (kept out of git), have the
             design lead pick the system the usual way, run Floor Face-Off, and compare the answer and the
             time taken. Do not quote hours or rupees until that is done.
             Market check (2026-11-17, two searches; not exhaustive, so "I did not find one", not "none"):
               - Free: Concrete Centre "Cost and Carbon: Concept V4" (UK, Eurocode 2, 13 RC forms, ranks by
                 cost, programme or carbon) and CRSI "Reinforced Concrete Concept" (US, flat slab, flat
                 plate, joist, waffle, one-way beam, voided slab, quantities and cost).
                 Both take typed-in bay sizes. They do not read your drawing or model, they do not use
                 IS 456 / IS 13920 or Indian rates, and they do not build the winner in 3D.
               - Sorvimo "Floor Design" compares carbon for (mostly timber) floors. Not RC in India.
               - Research: genetic-algorithm and Pareto studies on floor cost vs carbon (a Chalmers thesis,
                 a 2024 Scientific Reports paper). Papers, not products for a BIM team.
               - A 2023 journal study (SAFE, 4/6/8 m spans) and a thesis abstract: RC flat slab cheaper up
                 to about 6 m, PT flat slab with drops about 10-12 % cheaper from about 7 m. Weak evidence
                 and not Indian; used here only to show the trade-off exists and flips with span.
               - I did not find a tool that reads the project's own grid, compares systems with Indian
                 rules and rates, and writes the chosen system into an IFC model.
             Extends tools already in this repo (does not repeat them):
               - Column Compass: gives three COLUMN layouts from an architect plan. Floor Face-Off takes ONE
                 chosen column layout and compares the FLOOR systems on it. They run one after the other.
               - Carbon Lens: swaps materials and scores carbon on a finished model. Floor Face-Off decides
                 the system BEFORE the model exists (or for a what-if on an existing one).
               - Slab Sketch: designs a beam-slab panel to IS 456 in detail. Face-Off uses the same rules
                 in a quick pre-design form for six systems, then hands the winner to Slab Sketch.
               - Rate Rocket (rates), Shutter Plan (formwork), Crew Curve and Pour Planner (cycle time),
                 Tendon Tracer (PT), Kit Cutter (precast): the numbers it reads.
               - Quake Preview: shares the stiffness check, so a flat plate that fails drift is flagged.
             Trend: AEC trade news this year is about AI "options" in early design and about embodied
             carbon rules for tenders. "Show the client three options with cost and carbon on day one"
             is the common ask. I could not measure LinkedIn or Instagram counts; sources are web pages,
             so this is not a social-media claim.
             The wow: drop the DXF or IFC, and in a minute six floor plates appear side by side in 3D, each
             tinted by cost, with one bar chart that moves live as you drag "floor-to-floor 3.0 m -> 2.8 m":
             "Flat slab saves 200 mm per floor; over 14 floors that is 2.8 m less building, a floor you
             can add under the same height band, and 410 m2 of façade not built." Click Select, and the
             model is built with that system.
```

### Description

**The pain.**
- The floor system is chosen by habit, because comparing is slow.
- The choice moves many other numbers, but nobody sees them in one place: structural depth (headroom and
  floor-to-floor), steel and concrete, formwork area and reuse, cycle time, cost, carbon, drift and punching.
- The client asks "can we save height?" and "what if we use PT?" and the honest answer takes days.
- Free web tools exist, but they are generic, take typed bay sizes, and stop at a chart.

**The idea in one line.** Read the real column grid and spans, pre-design six floor systems on it with the
Indian rules, show them side by side with every number that matters, and build the winner in 3D.

### How it works (user view)

1. **Start from what you have.** An architect plan DXF with Column Compass columns, or an open IFC. The
   tool reads grid lines, column positions, slab outline, openings, lift and stair cores.
2. **Six systems, one grid.** Beam-slab (two-way), flat slab, flat slab with drop panels, PT flat slab,
   waffle, precast hollow-core on beams. Each gets a pre-design: thickness or depth, steel per m2 from IS 456
   span-to-depth and a simple strip method, punching check at columns, and a stiffness flag.
3. **The face-off board.** Six cards in a row, same scale. For each: concrete m3/m2, steel kg/m2, formwork
   m2/m2 and reuse count, structural depth, floor-to-floor, days per floor cycle, rupees per m2, carbon
   kgCO2e per m2. The cheapest, the lowest-carbon and the shortest are marked. A system that fails a check
   (span too long, punching, drift) is greyed with the reason, not hidden.
4. **Sliders, live.** Floor-to-floor, number of floors, concrete grade, steel rate, rate of PT, crane or
   pump limit, "no drop panels allowed" (the architect's rule). Every card and the chart update at once.
5. **Whole-building view.** For N floors: total height, façade area, lift travel, extra floors that fit the
   height band, and total cost and carbon. This is where a flat slab that costs more per m2 can still win.
6. **Select.** One click writes the chosen system as slabs, beams, drops or ribs into the model (IFC), with
   marks, and passes the pre-design to Slab Sketch for the full rebar and BBS. Nothing silently replaces
   work already done: it is a new model state that can be undone.
7. **One-page client note.** Six options, three numbers each, a recommended one and why, in plain words and
   in the client's language if set. Prints to PDF. Every number links back to its 3D proof.

### Why a BIM team would use it

- The modeller stops guessing. The first model is the right one, not the one that gets redone in week 3.
- The design lead gets six options in a minute instead of two in two days.
- The client sees height, cost and carbon trade-offs on day one, which often decides the project.
- A tender carbon limit becomes a number to hit, with the system that hits it.
- Easy to demo: the six plates and the moving bar chart get a reaction in every client meeting.

### Implementation

- **Input.** Reuse the CAD pipeline's grid and column reading and the IFC reader. Spans come from column
  centres; irregular grids are split into panels (rectangles first, other shapes flagged).
- **Rules as data.** A JSON rule pack per system: span-to-depth limits (IS 456 cl. 23.2 and the flat-slab
  clauses), minimum thickness, drop panel size rules, punching shear (IS 456 cl. 31.6), waffle rib spacing,
  PT span ranges, hollow-core spans from a vendor table. Rates and carbon factors are tables the firm edits
  (Rate Rocket and Carbon Lens tables when present). The tool never invents a rule; each card prints the
  clause or the table it used.
- **Quantity model.** A fast strip method gives steel per m2 from moment coefficients; it is a screen, not a
  design. Result bands are shown (for example +/-10 %) so nobody mistakes it for final quantities.
- **Calibration.** On each past project the user imports, the tool compares its pre-design with the real BBS
  and learns a correction per system. Reported openly: "on your last 6 projects the screen was 7 % low on
  steel for flat slab".
- **Cycle time and formwork.** Table of formwork area, reuse and days per floor per system, editable, fed by
  Pour Planner and Crew Curve when they exist.
- **3D and write-back.** Slabs, drops, ribs and beams are produced by the same builders as the DXF -> 3D
  pipeline, written to IFC with stable marks. A `FaceOff` property set records the system chosen, the
  assumptions and the date.
- **Where it runs.** In the browser, on the open file; nothing is uploaded. Packages: a new module in the
  engine for the rule packs and numbers, the app for the board and sliders. Not source-changing here; this
  file is only the idea.
- **Tests.** (1) Hand-check: five grids with known answers from a textbook or the firm's own past design,
  per system. (2) Back-test: 5 past projects, compare tool steel and concrete with the real BBS and
  quantities (kept out of git); report the error per system. (3) Any case the tool marks "passes" that the
  structural engineer says fails must be zero before release. (4) A timed trial before any time claim.

### Risks and limits (stated, not hidden)

- A pre-design is not a design. The tool says "screen" on every card, and Select hands off to the full
  design (Slab Sketch, ETABS) rather than replacing it.
- Rates and carbon factors differ by city, season and supplier. They are editable inputs; the tool ships
  no number as if it were a fact.
- Cycle time and reuse depend on the contractor; defaults are conservative and shown.
- Seismic, soil, fire rating and architectural needs (ceiling, services depth) can overrule the cheapest
  answer. Services depth is an input; the rest are checks the engineer must still do.
- Waffle, PT and hollow-core need local vendors; availability is a switch, not assumed.

### Future update

- **Phase 1 (about 2-3 weeks):** rectangular grids, beam-slab, flat slab and drop-panel, face-off board,
  sliders, PDF note.
- **Phase 2:** PT flat slab (with Tendon Tracer), waffle, hollow-core (with Kit Cutter); calibration from
  past projects.
- **Phase 3:** whole-building view (height band, façade, lift), tender carbon limit, "find the cheapest system
  under 2.8 m floor-to-floor" search.
- **Phase 4:** one-click build of the winner in IFC and hand-off to Slab Sketch and BBS.
- **Phase 5:** mixed systems per floor (flat slab on the typical floors, beam-slab at transfer and podium),
  steel-composite and timber options, and an MCP tool so an AI agent can ask "what is the cheapest floor
  system under 150 kg CO2e/m2 for this grid?"
- **Phase 6:** a learning mode that shows the junior why the winner wins, linked to the Dojo Drill exercises.

### Sources (market check)

- Concrete Centre, Cost and Carbon: Concept V4 — https://concretecentre.com/Resources/Design-tools-and-software/Cost-and-Carbon-Concept-V4.aspx
- CRSI, Reinforced Concrete Concept — https://www.crsi.org/online-design-tools/reinforced-concrete-concept/
- Nature Scientific Reports 2024, floor slab studies — https://www.nature.com/articles/s41598-024-58763-8.pdf
- Chalmers thesis on genetic algorithms in conceptual structural design — https://odr.chalmers.se/server/api/core/bitstreams/85c187f7-fbe4-4c52-98c6-976f0f25816e/content
- 2023 IJAME comparison of slab types — https://yadda.icm.edu.pl/baztech/element/bwmeta1.element.baztech-fea3e75a-d290-4e6d-b328-e86b9da32bcf/c/Ahmad_Comparison_IJAME_2_2023.pdf
