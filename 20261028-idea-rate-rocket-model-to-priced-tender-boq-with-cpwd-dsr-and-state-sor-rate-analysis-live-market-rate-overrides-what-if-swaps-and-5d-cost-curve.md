# 20261028 Idea: Rate Rocket: open the model, pick the rate book (CPWD DSR or your state's SOR), and cad2bim prices every element: it maps each BOQ quantity to the right schedule item, writes the full rate analysis (material, labour, lead, cess, overheads), outputs a tender-ready item-rate BOQ in the department's format, and lets the team swap a grade, a block or a thickness and watch the cost, the cost per sq ft and the 5D cash-flow curve move

## Entry 1

```
date-time:   2026-10-28 (scheduled run, UTC)
idea name:   Rate Rocket. "The client wants a budget by Friday and a tender BOQ by next week.
             Today the BIM modeller exports quantities, the estimator re-types them into
             Excel, looks up each item in the DSR book (a PDF of 1,000+ pages), adjusts for
             the cost index, adds lead and lift for the site, adds cess, GST and contractor's
             profit, and does it all again when the structure changes. Here: the modeller
             opens the model in cad2bim and presses Rate Rocket. A panel asks three things:
             the rate book (CPWD DSR 2023 + cost index for Pune, or Maharashtra PWD SOR, or
             the office's own rate sheet), the work type (government tender, private
             contract, budget estimate), and the site (city, lead distances for sand, stone
             and steel). In seconds every element has a rate: 'M30 RCC column, 450 x 900,
             4th floor: 3.24 m3 at 9,150 /m3 (item 5.1.3 + 5.9.2 + 5.22.1), steel 412 kg at
             84 /kg laid, centering 8.1 m2 at 710 /m2'. The total appears with a cost per
             sq ft of built-up area and a pie by trade. The estimator opens any rate and
             sees the analysis: cement 7.2 bags at ..., sand ..., labour ..., 15 % contractor
             profit and overheads, with the page and item number of the schedule. A column of
             'unmatched' items shows what the book does not have, with the three nearest
             items; one click picks one, or opens a non-schedule item with its own analysis
             and a note. Then the wow: 'what if' sliders. 'M30 to M25 on floors 5 to 14',
             'AAC block instead of brick', 'slab 150 to 130', and the cost, the cost per
             sq ft and the steel tonnage move live, with the delta against the base shown in
             rupees and in per cent. Press Tender Pack and out comes the BOQ in the
             department's own layout (item no., description in the book's words, unit,
             quantity, rate, amount, with the abstract of cost), a rate-analysis annexure,
             a measurement sheet with the model's quantity derivations (the 'MB' lines, with
             the element marks), and a price-bid Excel that the tender portal accepts.
             Press 5D and the cost lays over the 4D schedule: a monthly cash-flow curve and
             the bill amount due each month."
             One engine, five front ends:
               (1) cad2bim web app: a "Rates" dock panel (rate book, match review, what-if,
                   tender pack), using the BOQ the app already produces (packages/engine
                   BOQ) and the 4D/5D plan in docs/design/next-milestones.md,
               (2) cad2bim CH template v2: the same rates when the model comes from a drawing,
                   so a plan can be priced the day it is drawn, before any Revit work,
               (3) Revit (bridge add-in + pyRevit AnonGee button): writes the rate, the item
                   number and the amount into parameters on each element (or on a Revit
                   schedule), so the office's Revit schedule and the tender match,
               (4) agent helper (MCP): 'price this building with the 2023 DSR and tell me what
                   changes if we use M25 above the 10th floor',
               (5) Structura / etabs-mcp: when ETABS changes sizes, the cost change is shown
                   next to the new reinforcement, so design choices carry a price.
             Outputs: priced BOQ (Excel + PDF), tender-format BOQ, rate-analysis annexure,
             measurement sheets, abstract of cost, cost-per-sq-ft and trade pie, option
             comparison sheet, monthly cash-flow (5D) curve, and IfcCostItem /
             IfcCostSchedule written back into the IFC.
potential:   Very high, and used on every job from the first sketch to the last bill.
             Every project has a budget conversation, and every government and many private
             tenders need a priced BOQ.
               - Quantity take-off is the part BIM already helps with. The slow part is
                 pricing: the match of each item to the schedule and the rate analysis.
                 It is hours to days per estimate and it is repeated every time the design
                 changes.
               - The BIM modeller is often asked 'what will this cost' with no estimator
                 beside them. Today they have no answer; with Rate Rocket they can give
                 an indicative figure the same day, marked as indicative.
               - Value engineering is where the money is saved, and it needs fast what-ifs.
                 In Excel, a change from M30 to M25 across ten floors is a half-day of
                 edits. Here it is a slider.
               - It closes the loop with Bill Painter (progress to RA bill) and Carbon
                 Lens (carbon vs cost): they all need a rate on the element, and this
                 gives them one place for it.
             Market: from a quick web search (not a full survey; the sources are vendor pages
             and I did not verify prices, so check before relying on any of this):
               - Estimator India's 2026 guides describe the CPWD BOQ structure and the DSR
                 rate analysis as a manual Excel job
                 (constructionestimatorindia.com, 'CPWD BOQ Format & DSR Rate Analysis for
                 Civil Work (2026 Guide)').
               - Revit-based 5D services exist from outsourcing firms (for example united-bim
                 and bluentcad pages), which sell the work as a service, not a tool.
               - Paid estimating suites exist (for example CostX, Candy, Bluebeam-with-
                 takeoff, Autodesk Takeoff in ACC); none is listed in what I found as
                 shipping the CPWD DSR or the Indian state SORs, with the item matching,
                 the analysis and the tender format, in a browser on the user's device.
               - Revit itself has Material Takeoff and cost fields but no rate books.
             So I could not find a tool that does model to DSR-priced tender BOQ, which is
             the gap. If one exists, it is likely paid or a plug-in for one modeller.
             Why it makes people say wow: the budget number appears in the time it takes to
             say it; the slider turns the building cheaper or dearer live while the client
             watches; and 'a tender BOQ in the department's format, from the model, in two
             minutes' is the sentence estimators have not heard. A good reel for LinkedIn or
             Instagram: 'G+14 tower. Budget in 90 seconds. Swap M30 to M25 above floor 10:
             saves 41 lakh. Tender BOQ in the CPWD format. No Excel.'
description: Rate Rocket adds a pricing layer on top of the BOQ cad2bim already makes. The
             rate book is data, not code, so any state SOR or the office's own rates can be
             loaded.
             1. Rate books. A rate book is a JSON/CSV file: item code, description, unit,
                rate, optional analysis lines (material, labour, machinery with their
                coefficients), the source and the year. The app ships none that it has not
                the right to ship: the DSR and state SORs are government publications whose
                reuse terms must be checked. The first release should ship the loader and a
                sample book, and let the office load its licensed or public book on its
                device. Nothing leaves the device (the cad2bim rule).
             2. Match. Each BOQ line (element type, material, size, work: concrete, steel,
                shuttering, masonry, plaster, finishes) maps to a schedule item by a rule
                table ('RCC column, M30, above 3.5 m height' -> item X + steel item Y +
                centering item Z). The rules are editable and saved per office. Matching
                uses the existing property sets; ambiguous lines go in a review list with
                ranked suggestions (text similarity + unit check).
             3. Adjust. Cost index, lead and lift, floor-height extra, cess, GST,
                contractor's profit and overheads, as a short list of percentage and
                per-unit rules in order, shown on every analysis line.
             4. Price. rate x quantity per element, summed up by floor, trade and item.
                Each amount keeps the element ids behind it so a click on a number selects
                its elements in 3D (the same trick as the BOQ window).
             5. What-if. A scenario is a set of overrides over the base (grade, thickness,
                block type, rate change). Scenarios are kept in the .c2b, shown side by side
                with the delta; the model itself is only changed if the user accepts.
             6. Tender pack. Templates for the BOQ layout (department name, item numbering,
                abstract of cost, rate-analysis annex, measurement sheet) written to Excel
                and PDF by the existing writers.
             7. 5D. The 4D schedule's tasks give each element a month; the cost per month
                and the cumulative curve come from the same totals. Written to IFC as
                IfcCostItem with IfcRelAssignsToControl, which the roadmap already plans.
             Honest limits: a match is a suggestion; the estimator signs the estimate. The
             tool says 'indicative' on any line matched by similarity and not by a rule,
             and lists them. Rates in the schedule are not market prices; the live market
             override is the user's own entry (or a CSV), not scraped data.
implementation:
             Phase 0 (about 1 week): the rate book format and loader; a sample book with
             invented items clearly marked 'sample'. Price the existing BOQ lines with a
             one-to-one rule table; show amount per line and total.
             Phase 1 (about 2 to 3 weeks): the rules engine (conditions on type, material,
             size, height), several items per element (concrete + steel + centering),
             adjustments, the review list for unmatched lines, 'select the elements behind
             this amount'.
             Phase 2 (about 2 weeks): rate analysis view and the tender pack (Excel and PDF
             templates), the measurement sheet from the model's geometry.
             Phase 3 (about 2 weeks): scenarios and sliders; option comparison sheet; the
             cost per sq ft and trade pie; save scenarios in the .c2b.
             Phase 4 (after the 4D timeline ships): cash-flow curve and IfcCostItem write-
             back; link to Bill Painter and Carbon Lens.
             Phase 5: Revit parameters (via the bridge) and the agent tools.
             Where it goes in the code: a new `packages/engine/src/cost/` beside `boq/` and
             `bbs/`; the rate book and rules as plain data files; a "Rates" dock panel in
             `apps/web`; a design note `docs/design/rate-rocket.md` first. No new dependency
             is needed; spreadsheets use the writers already in the app. Licence check: any
             rate book shipped must have terms that allow it; otherwise user-loaded only.
             Proof: price three finished projects whose estimator-made estimates exist and
             report the difference per trade, measured, not claimed; the difference is the
             honest accuracy figure to publish.
future update:
- **Live market rates:** a CSV or API feed the office controls (cement, steel, RMC by
  plant), with the date shown on every rate, so the budget uses today's steel price.
- **Contractor mode:** subcontractor quotes against each item, a comparison matrix, and the
  lowest-bid pick per trade; the rate becomes the negotiated one.
- **Tender checking:** load a contractor's priced BOQ (Excel) and compare it with the
  model: missing items, quantity differences, rates far from the book, front-loaded items.
- **Variation orders:** change in the model (Change Radar) becomes a priced variation with
  the old and new quantity and the rate that applies.
- **Cost per sq ft benchmarks:** the office's past projects as a benchmark band
  ('this tower is 8 % above your last three').
- **Life-cycle and carbon:** add O&M cost and embodied carbon to the same what-if
  ('cheaper, but 12 % more carbon').
- **MEP and finishes:** extend the rule table to MEP quantities and the Wall Wizard / Ceiling
  Composer outputs, so the estimate covers more than the structure.
- **Other countries:** the rate book is data, so a UK NRM or a Gulf BOQ is a template.
- **Agent:** 'what is the cheapest way to get this under 30 crore' runs scenarios and
  returns the ranked options with the changes.

### Improvement suggestions for existing tools in the repos

- **cad2bim (BOQ):** every BOQ line should carry the list of element ids and a stable
  line key, so a rate can attach to it and survive a re-run; add a `trade` and a
  `work item` field (concrete, steel, shuttering, masonry, finish).
- **cad2bim (BOQ):** add shuttering (centering) area per element as a standard line; it is
  often missing and always priced. The Shutter Plan idea can feed it.
- **cad2bim (roadmap):** the 5D cost item in `docs/design/next-milestones.md` should say
  where the rate comes from (a rate book), so the 4D and 5D releases share this design.
- **cad2bim (export):** an Excel writer option for a merged-cell, department-style header
  and abstract sheet; today's writers are plain tables.
- **cad2bim (.c2b):** a place for scenarios and rate-book references in the project file
  (`docs/design/project-file.md`).
- **Bill Painter / Carbon Lens / Wall Wizard:** read the same rule table and rate book so
  one rate is entered once.
- **pyRevit-addons (AnonGee):** a button that writes rate, item number and amount to element
  parameters from the exported priced BOQ.
```
