# 20261114 Idea: Gang Pay — the model and the BBS become labour-contractor work orders and weekly pay sheets (bar bending per kg, shuttering per m², concrete per m³, blockwork per m²), gang by gang, every line clickable to its 3D proof, with a "claimed vs done" check before anyone is paid

## Entry 1

```
date-time:   2026-11-14 (scheduled run, UTC)
idea name:   Gang Pay (labour-contractor work order + weekly measured pay sheet, from the 3D model, the BBS and
             the pour/progress status; the "other bill" that sits under the client RA bill)
potential:   High for the site-side BIM team; medium-high for the modeller who is asked "give me quantities for the
             labour contractor". Every site pays its gangs (bar benders, carpenters, masons, concrete gang) by
             measured quantity: per kg of steel, per m² of shuttering, per m³ of concrete, per m² of block work.
             Today the site engineer builds this by hand in Excel from drawings, BBS and a notebook, every week,
             and every week there is an argument about the number. The model already holds every quantity. Nobody
             should re-measure it.
             Time saved: from how this work usually runs, NOT measured here; calibrate on one real site before
             quoting. A half to one day a week per site engineer is a fair guess to test, not a claim.
             Market check (2026-11-14, one session, not exhaustive): payroll tools with piece-rate wages exist,
             but are aimed at wages, not contractor billing; accounting/ERP tools (Tally, Procore, QuickBooks) were
             not described as producing per-kg / per-m² labour bills from a model; the search found Excel
             templates and rate lists only. I found no tool that does this from a 3D model. Not claimed as unique
             worldwide; the claim that fits is: free, in the browser, no upload, tied to the model you already have.
             In this repo it is NOT the same as:
               - Bill Painter (client RA bill: what the client pays the main contractor),
               - Measure Right (IS 1200 measurement book for tender/billing),
               - Crew Curve (how many workers are needed, planning),
               - Steel Ledger (steel issued vs used, a material check).
             Gang Pay is the money going out to the gangs, measured against what the model says was built.
             Trend: LinkedIn / news this week is about AI agents inside Revit, MCP connectors, and above all
             governance: previews, confirmations and read-only modes before an agent changes anything (see the
             sources at the end). Gang Pay fits that mood: it only reads, shows a preview, and a person signs.
             The wow: the site engineer on Friday opens the model, picks the week, and the elements built that
             week glow. A pay sheet writes itself, gang by gang: "Gang A (bar bending): 6,820 kg x rate = amount".
             Click 6,820 kg and the 3D shows the exact bars. The contractor claims 7,900 kg; a red line says
             "1,080 kg claimed on elements not yet marked done" and lights them up.
```

### Description

**The pain.**
- Gangs are paid by measured quantity. Bar bending is typically per kg, shuttering per m², concrete per m³ or
  per element, block work per m² (from common site practice and the rate sources below; units differ by
  contract, so the tool must follow the work order, not assume).
- The site engineer measures from drawings and the BBS, in Excel, by hand. The sheet is redone every week and after
  every revision. The contractor measures too, and the two numbers rarely match.
- The common leaks (stated as typical site problems, not measured): paying for elements not yet finished, paying
  twice for the same element in two weeks, paying for a revised bar length at the old quantity, paying for steel
  that the model says is not in that element.
- The BIM team is asked to "send quantities", then the same quantities are asked for again next week.

**What exists (checked 2026-11-14; one search, not exhaustive).**
- Rate lists and Excel templates for labour quotations ([Construction Estimator India](https://constructionestimatorindia.com/?p=15171),
  [HouseYog RCC labour rates](https://www.houseyog.com/blog/rcc-work-labour-cost-india/), [InfraLens rate analysis](https://infralens.in/rate-analysis/rcc-reinforcement-steel)).
- Payroll tools with piece-rate wages, aimed at wages ([Petpooja guide](https://blog.petpooja.com/industry-business-guides/best-payroll-software-construction-firms-india/)).
- Nothing found that reads a 3D model and a BBS and makes the gang bill with proof.
- cad2bim already has the pieces: IFC quantities, the BBS, element ids, 4D status and the viewer's highlight.

**The idea in one line.** A work order says "per kg, per m², per m³, at these rates". The model says what was built.
Gang Pay multiplies the two, shows the proof, and stops the bill that does not match.

### How it works (user view)

1. **Make a work order (once per gang).** Pick the gang or contractor, the scope ("steel fixing, columns and beams,
   Tower B"), and a rate card: unit, rate, and what is included ("shuttering included in RCC labour", "binding wire
   by contractor"). Start from a small template; the rates are typed by the user. The tool does not ship market
   rates and does not claim any are right.
2. **Mark progress.** Elements become "done" from the 4D status, the pour card, a phone check (Site QR Pass,
   Cage Manual) or a simple multi-select in 3D. Each "done" has a date and who marked it.
3. **Pick the week.** The elements marked done in that week light up. The pay sheet builds:
   - per gang, per item: quantity (kg, m², m³), rate, amount;
   - per element, so any number can be checked: click "6,820 kg" and the exact bars glow, with the BBS marks.
4. **Claimed vs done.** The contractor's own sheet (Excel) is dropped in. The tool matches rows by element mark
   and shows the gap: more claimed than done, done but not claimed, the same element billed twice, quantity
   changed by a revision since last bill. Each gap is a line with a reason and the elements in 3D.
5. **Hold, adjust, approve.** The engineer can hold an item (with a reason), add a manual line (extra work,
   deduction, advance, retention, recovery from the Steel Ledger), and approve. The tool prints the sheet (PDF
   and Excel) with a number and a date. Nothing is paid by the tool; a person signs.
6. **After a revision.** A bar length changes, a column moves: the tool shows the effect on past bills ("Gang A
   was paid 1,240 kg for B12; the model now says 1,310 kg; +70 kg next bill") and never rewrites a signed sheet.

### Why a BIM team would use it

- The modeller stops answering "send me quantities for the labour" every Friday: it is a button the site team presses.
- Quantities have one source, the model, so the BOQ, the BBS, the client bill and the gang bill agree.
- Fewer arguments: every number has a 3D proof and a date.
- It makes the model *used on site every week*, which is what keeps the model alive and the BIM budget safe.

### Implementation

Runs in the browser on the open IFC and BBS; nothing is uploaded (the repo's rule: files stay on the device).

1. **Quantities.** Read per element from IFC base quantities where they exist, and from the cad2bim BBS for steel
   (kg per mark, per element). Formwork area comes from the member faces minus joints (reuse the Shutter Plan
   logic if it exists by then); block work from the Wall Wizard idea or IFC wall quantities. If a quantity is not
   in the model, the line says "not in model" and is left for a manual entry. No silent zero.
2. **Rate card and work order.** A small JSON file: gang, scope selector (IFC class, type, level, mark pattern),
   unit, rate, inclusions. Saved on the device and exportable, so a company keeps its own templates.
3. **Progress.** One table: element GUID, status, date, who. Sources: the 4D task link, the Pour Planner card,
   phone check-ins, manual select. Same table feeds Bill Painter and Ready Board, so progress is marked once.
4. **Pay sheet engine.** Group elements by work order and week, sum quantity x rate, keep a per-element ledger.
   Billed quantity per element is stored, so "billed twice" and "revision delta" are plain set maths.
5. **Claimed vs done.** Read the contractor's Excel with the existing document viewer's reader; match on element
   mark with a fuzzy-match step and a person confirming the unclear ones. Output: a gap list with a reason code.
6. **Output.** PDF (print-ready sheet) and Excel with formulas intact. A QR on the sheet opens the elements in 3D
   in Model Capsule or the app, offline.
7. **Safety.** Read-only on the model. The ledger is a separate file. Approval is a person, with a name and a date;
   signed sheets are locked. Matches the "preview, confirm, audit trail" mood in the trend note.
8. **Test plan.** Take one finished floor from a real project; compute the pay sheet by hand and with the tool; the
   total must agree to the kg and the m². Report the difference. Do not publish a time-saving figure until a
   site engineer has timed one week both ways.

### Future update

- **Rate analysis link.** Pull rate cards from Rate Rocket so labour rates and tender rates are the same table.
- **Retention, advances and taxes** (GST / TDS in India): fields only at first; rules added with an accountant's
  review, never guessed.
- **Productivity learning.** Compare billed quantity per day with the Crew Curve norm; show which gang is
  slower or faster than its own average (private to the company).
- **Phone-first mode.** The foreman marks "done" on a phone with a photo; the engineer signs on a laptop.
- **Multi-project view.** One page for a contractor's gangs across sites: paid, held, claimed.
- **Subcontractor portal-lite.** A read-only Model Capsule for the gang's contractor showing only their elements
  and measured quantities (client-safe profile), so the argument starts from the same picture.

### Reel idea for LinkedIn / Instagram

15 seconds: split screen. Left, a site engineer with a notebook and a calculator on Friday night. Right, the 3D
model: the week's built elements glow, a pay sheet writes itself, the contractor's claim loads, a red line
lights 1,080 kg of bars that are not built yet. Caption: "The labour bill with a 3D receipt." (Check the claim
against one real project before posting; do not post made-up numbers.)

### Honest limits

- Units and inclusions differ by contract; the work order is user-defined, and the tool only follows it.
- Formwork and block work quantities are only as good as the model; where the model has none, the line is manual.
- Rates are never supplied by the tool. Rate sources above are for reference only.
- The hours-saved figure is an estimate to test, not a measurement. The market check was one search.

### Sources (checked 2026-11-14)

- AI agents, MCP and governance trend in Revit: [Autodesk Revit API DevCon 2026](https://forums.autodesk.com/t5/revit-api-forum/revit-api-agentic-ai-sessions-amp-resources-from-devcon-2026/td-p/14127696),
  [AEC Magazine: Symetri BIM Summit 2026](https://aecmag.com/bim/symetri-bim-summit-2026),
  [Ideate: AI assistants and deterministic tools](https://ideatesoftware.com/ai-in-revit-what-autodesk-assistant-means-for-revit-workflows).
  These are vendor and press pages, not LinkedIn counts; I could not measure LinkedIn or Instagram directly.
- Labour rate and unit practice: the three rate pages listed under "What exists".
