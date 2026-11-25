# 20261125 Idea: Schedule Spark — the model writes the whole-building baseline programme, with takt trains, a flowline chart, gang sliders and an instant 4D

## Entry 1

```
date-time:   2026-11-25 (scheduled run, fourth idea of the day)
idea name:   Schedule Spark — "Drop the model, get the programme."
             One click turns the 3D model's quantities into a complete baseline construction schedule: every trade
             from excavation to painting, with durations worked out as quantity / (productivity x gang), logic ties
             and curing lags, floors run as a takt "train", critical path, calendars and a flowline chart. Slide the
             number of gangs or the target finish date and the programme, the manpower and the 4D replay move live.
             Exports MS Project XML, Primavera P6 (XER / Excel) and Excel; every task already knows its elements.
potential:   High. A great many of the toolkit's ideas CONSUME a schedule (Task Tie, Ready Board, Crew Curve,
             Cash Curve, Order Clock, Sky Slot, Delay Detective, Model Reel, the 4D timeline on the roadmap), and
             Pour Planner makes one only for the RCC shell. The whole-building programme is still built by hand in
             MS Project or Primavera by a planner, or not built at all in the tender and concept stage. Schedule
             Spark is the missing source: it makes every one of those ideas work from a model alone.
             Time saved (my estimate, not measured): a first baseline for a G+12 residential tower takes a planner
             one to three days in P6/MSP; this gives a first draft in minutes and the planner spends the day
             improving it, not typing it. In tender and concept stage the saving is bigger: "can we finish in 20
             months?" is answered in a minute, not after a week.
             Trending: AI scheduling is a live topic. ALICE Technologies sells "generative scheduling" and added an
             AI assistant in 2025; Autodesk added an MCP-connected assistant to Revit 2027; "AI agents for the
             site" is a high-engagement theme on LinkedIn. The picture is made for a reel: a tower, a coloured
             staircase of trades climbing it in the flowline chart, and a slider that pulls the finish date in
             while the 4D replay speeds up.
             Market (from my own knowledge and one search today; prices not published or not re-checked):
             ALICE is paid, quote-based pricing, a separate tool you set up and feed; Synchro, Navisworks
             Timeliner and Vico-style takt tools animate or plan a schedule you must already have or build, and
             are paid; MS Project and P6 start from a blank page. None is a free, browser-based "model to
             programme" with Indian site defaults whose files stay on the PC.
```

### Description

**The pain (day to day of a BIM modeller, planner and site engineer).**
- The client asks "how long will it take, and what if we add a second shuttering set?" The modeller has the
  quantities (cad2bim's BOQ gives them) but not the time. A planner is needed, and a planner is days away.
- 4D is the most-posted BIM topic on LinkedIn and Instagram, yet nobody can show it without first typing a
  schedule. The animation is the easy part; the schedule is the work.
- Rough programmes are built from the planner's memory, so they are not repeatable: the same tower gets 16 months
  from one person and 22 from another, and nobody can say why.
- When the design changes (one more basement, two more floors) the programme is rebuilt by hand.

**The idea.** A **Schedule Spark** window in cad2bim (browser, files stay on the PC):

1. **Reads the model, not a schedule.** Quantities per element, level and zone come from the same place as the BOQ,
   so a number in the programme and a number in the bill can never disagree. Elements are tied to tasks by GlobalId,
   so no matching step (Task Tie) is needed.
2. **An activity library with plain, editable rules.** About 60 starter activities for Indian residential and
   commercial buildings: excavation, PCC, raft / footing, plinth beam, column, slab-and-beam cycle, shear wall,
   brickwork, door frames, conduit, plaster, waterproofing, screed, flooring, painting, lift install, façade,
   external works, snag and handover. Each has a unit, a default productivity per gang, a gang size, and
   predecessor rules with lags: "shuttering strikes at day 7 or by the cube result", "brickwork starts two floors
   below the slab being cast", "waterproofing before screed", "curing 7 days before loading". The defaults are
   starting points (estimates, labelled as such), by city and season, and every one is editable and shows its
   source.
3. **Duration = quantity / (productivity x gang), shown.** Click a task and see the sum: "Plaster, L7: 1,840 m2 /
   (28 m2 per mason-day x 12 masons) = 5.5 days". No black box; a planner can check and change the one number
   that is wrong.
4. **Takt trains.** Floors (or zones) are the "stations". Each trade is a gang that moves from floor to floor at one
   takt time (say 6 days). The tool balances the gang sizes so every trade fits the takt and no gang waits or
   blocks the next. The flowline (time-location) chart shows the trades as slanted bands climbing the tower: parallel
   bands are a healthy train, a bend is a clash or a bottleneck, a gap is idle labour.
5. **Critical path and calendars.** CPM runs in the browser; working days, holidays, the monsoon and Sundays are
   set once per project. If Sky Slot (weather) is present its lost days feed in; if not, a simple monsoon factor.
6. **The sliders.** Target finish date, takt time, number of shuttering sets, number of masonry gangs, one or two
   shifts. Move one and the finish date, the manpower histogram, the cost and the 4D replay update at once. A
   "make it N months" button solves for the cheapest set of changes and says what it costs: "To finish in 20
   months: second set of shuttering (+1.9 crore rental), plaster gangs 8 to 12, no night shift."
7. **Three options side by side.** A: economical, B: balanced, C: fast-track, compared on finish date, peak
   manpower, cost and cash need (Crew Curve and Cash Curve read the same schedule).
8. **Learns your way.** Import a finished project's actual schedule (MSP / P6 / Excel). The tool compares its own
   productivities with what really happened and offers to adopt yours, per trade and per site. Next time the first
   draft is closer; the calibration stays in the team's own library.
9. **Out.** MS Project XML, Primavera P6 (XER or the Excel import format), Excel Gantt, flowline chart PNG / SVG, and
   the 4D replay inside cad2bim (and as a Model Reel video). Re-export keeps task IDs stable so a planner's edits
   in P6 survive the next model revision.

**Why it is practical.** It answers the question asked in every meeting ("how long, and what if") in the
meeting. It removes the blank page. It makes 4D, cash flow, material orders and manpower possible for any model.

**Who gets what.**
- Modeller: a 4D that plays without a typing day; a number to quote when the client asks.
- Planner: a defensible first draft with the formulas visible, in the format they already use.
- Site engineer: a three-week look-ahead that starts from a real programme (Ready Board).
- Client / sales: a flowline picture and a replay that make the finish date believable.

### Implementation

1. **Scheduler core (TypeScript, a pure module, no UI).** Task graph, CPM forward and backward pass, calendars,
   lags, resource-constrained levelling (simple: serial gangs per trade). Unit-tested against hand-worked
   examples and against Pour Planner's shell durations (they must agree for the RCC floors).
2. **Activity library as one JSON file** (`.c2s`, versioned, diff-friendly), with the units, productivities, gang
   sizes, predecessor rules and their sources. Users add their own and share them in the team folder. No code
   change to add a trade.
3. **Quantity feed.** Read the BOQ's grouped quantities per level and zone from the live model (the BOQ window's own
   data, not a second calculation). Elements keep their GlobalId on each task for 4D, Task Tie-style look-ups and
   change detection.
4. **Takt balancer.** For a chosen takt, choose the smallest gang per trade that fits; report the trades that cannot
   (and why: quantity too large, or a hard lag). Flowline chart drawn in the design system's tokens.
5. **Sliders and the "make it N months" solver.** Start with a greedy search over a few levers (takt, sets of
   shuttering, gangs per trade, shifts); each step is shown so the user can accept or refuse it. No AI is needed;
   the MCP agent can drive the same tools ("finish in 20 months, no night work") when the person brings their own
   agent, as the project already decided.
6. **Exporters.** MSP XML (documented schema) first, then P6 Excel import format, then XER, then Excel. Each
   export is read back by the same code in a round-trip test; real MSP / P6 are used by hand for a final check.
7. **Calibration reader.** Parse an actual schedule, match task names to library activities by name and unit,
   and show a table: library productivity vs. achieved, with an "adopt" tick per row.
8. **Honesty built in.** Every number carries a tag: default (an estimate), team value or calibrated from project X.
   A one-page "assumptions" sheet is printed with every export. The tool says in plain words that it is a
   baseline for discussion and a planner must sign it.
9. **Rough order and size (my estimate, not measured).** Core and library: three weeks. Flowline chart, sliders and
   MSP export: two more. P6 and calibration: two more. First release can be the RCC tower with finishes; MEP,
   façade and external works come as library files, not code.
10. **Proof of done.** On a real G+12 model: first programme in under two minutes on a laptop; every task's quantity
    equals the BOQ's; shell durations equal Pour Planner's; MSP XML opens in MS Project with the same finish date;
    a planner changes one gang and the finish date moves as predicted.

### Future update

- **Resource and cost loading** from the rate profile, so the S-curve, cash flow and the bill-wise forecast come out
  of the same file.
- **Risk view:** run the programme many times with the weather and productivity ranges (the idea behind Sky Slot's
  P50 / P80) and show the finish as a range, not a date.
- **Live re-forecast:** feed in progress (Bill Painter, Chat-to-Model, Photo Placer) and re-plan the remaining work;
  Delay Detective explains the slip.
- **Takt for MEP and façade** as ready-made library packs (Live Wire, Cool Load, Skin Cutter, Fire Line), so the
  whole building is one train.
- **Multi-tower and podium** sites with shared cranes and shared gangs (Crane Canvas): the tool balances gangs
  between towers.
- **Team library hub:** a firm shares calibrated libraries by city and project type, with the numbers anonymised.
- **Hindi / Hinglish labels** for the site copy of the look-ahead (Bhasha Sheet).
- **Plan, do, check loop:** weekly Friday PPC (Ready Board) trains the library: the tasks that always slip get a
  slower default for that site.

### Notes and sources

- Existing ideas in this repo that Schedule Spark feeds or replaces the manual input of: Pour Planner (shell only),
  Task Tie, Ready Board, Crew Curve, Cash Curve, Order Clock, Sky Slot, Delay Detective, Model Reel.
- cad2bim's roadmap already lists the "4D timeline" (tasks and dates on elements, played back) and "5D cost"; this
  is the generator that gives that timeline something to play. It fits the owner's rule: one button, no extra
  screens, files stay on the device, the user's own AI key or agent over MCP.
- Market facts used today (vendor-reported, not independent): ALICE Technologies describes generative scheduling,
  Alice Plan / Optimize / Model products, an AI Insights Agent added in 2025, and quote-based pricing.
  - [ALICE review (pricing on request)](https://thedigitalprojectmanager.com/tools/alice-review/)
  - [ALICE launches Insights Agent](https://www.forconstructionpros.com/construction-technology/project-management/product/22951912/alice-technologies-alice-technologies-launches-aipowered-insights-agent-for-construction-scheduling)
  - [McKinsey and ALICE on generative scheduling](https://finance.yahoo.com/sectors/technology/articles/mckinsey-alice-technologies-partner-generative-130500698.html)
  - [BIM trends: AI in Revit 2027 via MCP and drawing-to-model work](https://www.bdcnetwork.com/sponsored/article/55343350/the-bim-trends-that-will-define-aec-project-delivery-in-2026)
- Not checked today: Synchro, Navisworks Timeliner and Vico prices and features (from my own knowledge); productivity
  defaults (to be sourced and reviewed by a planner before release).
