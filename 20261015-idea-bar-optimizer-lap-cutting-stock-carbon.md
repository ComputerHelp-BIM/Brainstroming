# 20261015 Idea: Bar Optimizer — laps, cutting-stock plan, offcut reuse and steel-waste / carbon meter

## Entry 1

```
date-time:   2026-10-15 (scheduled run)
idea name:   Bar Optimizer — from "BBS says 100 t" to "buy 96.8 t, cut it this way, here are the tags"
             (lap splice placement + 1-D cutting-stock optimiser + offcut reuse + waste and CO2e meter)
potential:   High. Rebar is 25-35 % of the structure cost and 8-12 % of the embodied carbon of an RCC building,
             and every site loses 3-8 % of it as cut-off waste because the BBS lists "cut lengths" but nobody
             plans how 12 m stock bars are cut. Cutting 3 % of waste on a 100 t job is 3 t saved (about
             INR 1.9 lakh and 7+ tCO2e) for ~10 seconds of work. Fills a gap the toolkit states itself:
             cad2bim's BBS design doc and Excel notes say "laps not modelled, wastage and rolling margin are not
             included". Paid alternatives exist only inside Tekla / Allplan / dedicated yard software.
```

### Description

**The pain (daily life of a BIM modeller / detailer / site engineer).**
- The BBS (from cad2bim, Structura or the pyRevit BBS Generator) gives *net* cut lengths per bar mark. It does not
  know that stock comes in 12 m (sometimes 9 m or 11.7 m), that a 14 m column bar needs a lap, or that a 3.1 m
  piece cut from a 12 m bar leaves a 2.9 m offcut that a different mark could use.
- Result: the store buys by weight with a guessed "5 % wastage", the yard cuts bars in the order the BBS lists
  them, and offcuts pile up. Reconciliation at the end shows 4-7 % loss and a dispute with the contractor.
- Laps are the other hole: lap length depends on bar diameter, grade, zone (tension / compression) and member
  (IS 456 cl. 26.2.5, ACI 318 development / splice, BS 8110/EC2), but the model draws bars as continuous or at the
  length Revit happens to give. The drawing, BBS and site disagree on where the joints are.
- "Steel waste" and "embodied carbon" are trending LinkedIn / Instagram topics (net-zero construction, EPD,
  CO2e per m2 posts, "material efficiency", lean construction). Almost nobody gives a modeller a number they can
  post that comes from their own model.

**The idea.** A **Bar Optimizer** mode that takes the BBS the toolkit already produces and turns it into a
buyable, cuttable, taggable plan in three steps:

1. **Lap splice engine (rules, not drawing).** For every long bar (columns, wall verticals, beam top/bottom, slab
   bars) split at the stock length minus allowance so laps fall at code-preferred places: columns just above slab
   level and staggered (max 50 % of bars at one section), beam bottom bars near supports, top bars at mid-span,
   never in the plastic hinge zone (seismic rule pack, IS 13920 / ACI 318 ch. 18). Lap length from a rule table
   (e.g. Ld x 1.0 / 1.3 / 1.5 by tension, compression, 50 % lapped), with mechanical couplers as an option for
   >= 25 mm. Output: new bar pieces and a lap schedule; each lap shown in 3D as a small marker.
2. **Cutting-stock optimiser.** Pool all pieces by diameter and grade across the whole job (or per level / per
   pour from the Pour Planner idea) and solve the 1-D cutting-stock problem for the chosen stock lengths:
   first-fit-decreasing as the instant baseline, then a column-generation / local-search improvement running in
   a worker for up to 5-10 s. Constraints: min usable offcut (say 300 mm) counts as scrap, max pieces per bar
   (yard cutter limit), group by pour/level so bars are not stored for months, optional "mixed stock lengths".
   Output: **cutting plan** ("Pattern 14: 12 m bar -> 3.45 + 3.45 + 3.45 + 1.20 offcut, x 86 bars").
3. **Waste and carbon meter (the wow screen).** One dashboard:
   - *Naive plan* (BBS listed order, 12 m bars) vs *optimised plan*: bars to buy, tonnes, waste %, offcut by
     diameter, rupees (using the BOQ rate the app already stores).
   - **Big number**: "Optimised plan saves 3.1 t of steel = INR 1.9 lakh = 7.4 tCO2e" (factor from an editable
     table; default per tonne of rebar, user can switch to their EPD / project factor).
   - **Offcut ledger**: usable offcuts are kept in an inventory and *reused* by later pours (the solver sees
     them as extra stock), so the number goes down the longer the project runs.
   - **3D**: click a cutting pattern and every piece it feeds lights up in the model; click a bar mark and see
     which stock bar it will be cut from.

**Site hand-off (practical, not just a report).**
- **Cutting & bending list for the yard**: one sheet per cutter machine shift, ordered by pattern, with bar
  mark, count, length, shape code and a **QR tag per bundle**. Scan on site -> opens a read-only page showing the
  bundle's pieces highlighted in the 3D model ("these go to Column C14, level 6"). No sign-up, files stay local
  or in a shared link the user chooses.
- **Machine export**: CSV/XML for common cut-and-bend machines (bar-bending schedule interchange such as
  BVBS/`.abs`-style files where the format is public), so the yard does not retype.
- **Store requisition**: "buy 540 x 12 m T16, 130 x 12 m T20" ready to send to the supplier, plus a wastage
  reconciliation sheet at the end ("planned 2.9 %, actual (entered) 3.4 %").

**Why it is a wow.** Drop the BBS, press **Optimise**, watch the waste bar fall from 6.8 % to 2.1 % in a few
seconds, and read "saves 3.1 t / INR 1.9 lakh / 7.4 tCO2e". Then scan a QR on a bundle and see where it goes in
the 3D model. It is the first time the modeller delivers something the *yard* and the *purchase* team use
directly, and it produces a sustainability number a BIM team can post.

### Implementation

1. **Reuse, don't rebuild.** Bar list comes from the existing BBS builders:
   cad2bim `apps/web/src/lib/` (`bbs` module + `bbsExcel.ts`, shape codes 00/11/21/23/46/51/51A/99), Structura
   `rebar.js` / `boq.js`, and the pyRevit BBS Generator (BS 8666 / IS 2502 / ACI 318 modules). Define one neutral
   input type `BarPiece {mark, dia, grade, shape, netLength, count, member, level, pour?}` shared by all three.
2. **Pure-function engine** `packages/engine/src/bars/` (TypeScript, no DOM, easy to unit test):
   - `lap.ts` — rule tables (IS 456 / IS 13920, ACI 318, BS 8110 / EC2) as data in `docs/research/`, function
     `splitAtLaps(bar, stockLen, rules) -> BarPiece[]` with stagger logic; golden tests against hand-worked
     examples from the code books.
   - `cutting.ts` — `packCuttingStock(pieces, stocks, opts)`: FFD baseline, then improvement (pattern merge,
     swap, or a small LP-guided column generation); deterministic with a seed so results are reproducible.
     Target: 20k pieces in < 3 s baseline, < 10 s improved, in a worker.
   - `waste.ts` — waste %, tonnes, cost and CO2e from the plan; editable factor table; offcut inventory type.
3. **UI**: new "Bars" tab in the bottom panel next to QA / BOQ (or a Model -> Bar Optimizer window like the BBS
   window): input summary, stock-length chips (6 / 9 / 11.7 / 12 m), minimum offcut, grouping (job / level /
   pour), Optimise button, result table, dashboard, 3D highlight via the existing selection/instanced colour
   approach. Component work in `packages/ui`.
4. **Outputs**: Excel (existing writer, new sheets: Lap schedule, Cutting plan, Store requisition, Waste
   summary), PDF cutting sheets with QR (existing PDF path + a small QR lib), CSV for bar-bending machines.
5. **QR deep link**: a hash-only URL (e.g. `cad2bim.in/#/bundle/<id>`) where the bundle data lives in the
   `.shkp` project file or an exported JSON; opening it with the project loaded highlights the pieces. No server
   needed for MVP; optional shared link later.
6. **Revit**: via the existing Revit Bridge write the lap positions back as shared parameters on rebar
   (`Lap_Count`, `Stock_Bar_ID`) and optionally split `DB.Rebar` sets; the pyRevit BBS Generator gets a
   "Use optimised plan" switch.
7. **Privacy and offline**: all computation in the browser; same promise as Structura / cad2bim.
8. **Docs**: `docs/design/bar-optimizer.md`, rule tables and carbon factors in `docs/research/`, roadmap and
   homepage update; add the missing "laps, wastage, rolling margin" lines in the BBS notes to point here.

MVP (1-2 weeks): stock-length input, FFD cutting-stock on the current BBS, naive-vs-optimised waste dashboard,
Excel cutting plan and requisition. Then lap rules (IS 456 first), offcut ledger, 3D highlight, QR tags, machine
export, carbon factors.

### Future update

- **Pour Planner link** (see `20261014-idea-pour-planner-4d-rmc-formwork-cycle.md`): optimise per pour so only
  the bars needed this week are cut, and release the cutting list together with the RMC order.
- **Twin Check link** (see `20261015-idea-twin-check-design-vs-bim-ast-heatmap.md`): do not cut for members with
  open design mismatches; feed the "steel above design" number into the optimiser as a saving target.
- **Change Radar link** (see `20261014-idea-change-radar-revision-diff-boq-delta.md`): when a revision changes
  bars, show "already cut: 3.2 t affected, usable as offcut stock: 1.1 t" so the cost of a late change is visible.
- **IDS Gatekeeper link** (see `20261014-idea-ids-qa-autofix-bcf.md`): rules such as "every bar over 12 m has a
  lap or coupler" and "stock length parameter present".
- **Whole-building carbon**: extend from rebar to concrete (grade-based factors), formwork and the full BOQ for
  an A1-A3 embodied-carbon report per level and a "design alternatives" compare (M30 vs M35, Fe500 vs Fe500D).
- **Schedule-aware ordering**: purchase and delivery windows from the 4D schedule; lock stock lengths to what the
  nearest supplier stocks.
- **2-D extension**: nesting for plates/shuttering sheets (plywood cutting plan) with the same engine.
- **Yard feedback**: scan-to-confirm a bundle as cut and tied, giving real wastage vs planned and a live steel
  progress view.

### Market check

- Cutting-stock optimisation, lap placement and bar tags exist inside paid products: Tekla Structures rebar and
  its "Tekla Structures for fabricators" tooling, Allplan Reinforcement / Precast, Revit extensions from Autodesk
  and third parties, and dedicated yard / cut-and-bend software from machine vendors. They are licence-heavy,
  mostly desktop or vendor-tied, and rarely go from a free open IFC to a QR tag.
- Free options are generic 1-D cutting-stock calculators (not connected to a BBS, shape codes, laps or the model)
  and per-project Excel macros.
- A free, in-browser, privacy-first **BBS -> laps -> cutting plan -> carbon number -> QR tags** loop connected
  to a 3D model is not a standard tool today. The inputs already exist in this toolkit (cad2bim BBS with BS 8666 /
  IS 2502, Structura rebar and BOQ, pyRevit BBS Generator, BOQ rates, Pour Planner idea). (Competitor claims here
  are from general market knowledge; verify before external marketing. Carbon factors and waste percentages above
  are indicative planning values; replace with project / EPD data.)
