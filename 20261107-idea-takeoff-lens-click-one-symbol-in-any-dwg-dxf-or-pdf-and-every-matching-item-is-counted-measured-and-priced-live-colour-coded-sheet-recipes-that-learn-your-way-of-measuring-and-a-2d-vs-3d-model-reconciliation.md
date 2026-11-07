# 20261107 Idea: Takeoff Lens — click one symbol in any DWG, DXF or PDF and every matching item is counted, measured and priced live, with a colour-coded sheet, recipes that learn your way of measuring, and a 2D-vs-3D check

## Entry 1

```
date-time:   2026-11-07 (scheduled run, second entry of the day)
idea name:   Takeoff Lens (One-Click 2D Quantity Takeoff for CAD and PDF drawings, Live Running Total,
             Colour-Coded Takeoff Sheet, Learned Recipes, Takeoff-vs-Model Reconciliation)
potential:   High, and it is used every week by every modeller, not once per project. "How many doors on
             this floor?", "How much shuttering is that?", "Give me the wall length by thickness for the
             tender, we have no model yet." Today the answer is a hand count with a highlighter, the COUNT
             command in AutoCAD (if you have it open), or a paid takeoff app. A modeller loses 1 to 4 hours
             per drawing set on these side questions (an estimate from how this usually runs, not measured
             here), and the numbers are rarely written down in a form anyone can check later. Takeoff Lens
             turns the drawing already open in cad2bim into a quantity sheet in minutes. You click ONE
             example (a door symbol, a wall hatch, a beam line); the drawing lights up every match in one
             colour, the total ticks up, and it goes to the BOQ with a rate. The wow is the drawing painting
             itself in colours as the numbers appear, and a red "the model says 212 m3, your drawing says
             226 m3" flag when 2D and 3D disagree.
```

### Description

**The pain.**
- The quick quantity question arrives before there is a model: tender stage, early budget, a client call, a
  site query. The modeller is the person who "can read the drawing", so they count.
- Hand counts are not repeatable. Two people get two numbers; nobody can show what was counted.
- Exact CAD data is thrown away. A DWG already knows every block, polyline length and hatch area exactly, yet
  the common workflow is to count by eye or trace over it again in another tool.
- Paid takeoff apps need the drawing uploaded to their cloud and a seat per estimator, and the AI ones guess
  on dense, heavily annotated drawings (source below). Many Indian consultants send DWG and the free tiers
  often limit DWG.
- When the 3D model exists later, nobody compares it with the early takeoff, so a wrong early number is never
  found.

**What exists today (checked 2026-11-07, not exhaustive).**
- In this repo's `idea` branch: Measure Right and Rate Rocket go **model to quantities and rates**. Scope
  Scout counts a drawing set into **modelling hours**, not construction quantities. Plan Tracer turns a scanned
  plan into DXF. Wall Wizard does blockwork from the model. A search for "polyline length" returns nothing and
  no entry takes quantities **from the 2D drawing itself** with no model. Takeoff Lens fills that gap.
- In cad2bim (`docs/design/cad-viewer.md`): the CAD viewer already has Count, Select Similar, Quick Select,
  Measure (Distance, Area, List), layers and the reading of DXF and DWG on the device; BOQ with rates and the
  Excel writer exist for the 3D side; the documents core reads PDF on the device. The pieces are there; the
  **takeoff workflow on top** is the new part.
- In the market: Kreo, Togal.AI, PlanSwift, STACK and Bluebeam do 2D takeoff. Reported prices (vendor and
  comparison pages, which conflict, so verify before quoting): Kreo about $35 to $175 per user per month,
  PlanSwift about $59 to $249 per month, Togal.AI about $299 per user per month or quote-only. STACK's free
  tier is manual with no AI; Bluebeam's takeoff is mostly manual measurement. Sources: [takeoff software
  comparison](https://hackceleration.com/labs/best/takeoff-estimating-software), [Kreo and Togal
  alternatives](https://oneestimate.ai/en/blog/kreo-construction-takeoff-alternatives-2026), [free takeoff
  software, Capterra](https://www.capterra.com/takeoff-software/s/free/), [AI takeoff
  guide](https://www.technbrains.com/blog/ai-construction-estimating/). So it exists, but paid, seat-based and
  cloud. A free tool that does it on the modeller's own laptop with nothing uploaded, from the **exact CAD
  data** and feeding a model-aware BOQ, I did not find. Treat "nothing free like it" as unverified until a
  proper search.
- Trend: AI takeoff and "estimating in minutes" are heavily marketed in 2026 (the comparison pages above are
  mostly vendor SEO). One guide reports only about 23% of firms use AI for estimating, so it is early.
  I could not verify what is trending on LinkedIn or Instagram specifically; the before/after clip below is the
  post, not a measured trend.

**The idea.** A **Takeoff** tab in the CAD viewer (and in the PDF viewer for vector PDFs). Everything runs in
the browser; the drawing never leaves the device.

1. **Click one example.** Click a door block: Takeoff Lens finds every instance of that block (also nested and
   in other layouts, with the same name or the same geometry fingerprint) and paints them. Click a wall hatch:
   it sums the area of every hatch with the same pattern and layer. Click a beam line: it sums the length of
   polylines on that layer, split by linetype or colour if the drawing does. The match rule is shown in plain
   words ("block D1, any layout, any scale") and can be changed in one click.
2. **Live total and colour.** Each takeoff item gets a colour and a row: name, unit (nos, m, m2), quantity,
   rate, amount. The total updates as you add items. Items can be tweaked: height or thickness to turn a length
   into an area or a volume ("wall length x 3.0 m x 0.23 m"), a waste %, a deduction for openings found by
   the door and window blocks.
3. **Exact, not guessed.** Because the data is CAD, the numbers are exact for what is drawn. What is **not**
   exact is said out loud: an unclosed polyline "area" is flagged, a block exploded into loose lines is shown
   as "looks like a door but is not a block, 14 found by geometry", text-only items are listed for you to
   decide. Nothing is silently dropped; an Audit-style list shows what was skipped and why.
4. **PDF with a scale.** For a vector PDF the same works on its vector paths after you pick two points of a
   known length ("this grid span is 4,500") to set the scale; a scanned PDF is offered to Plan Tracer first.
5. **The takeoff sheet.** One click gives (a) the drawing re-printed to PDF with every item in its colour and a
   legend with totals, ready to send to a client or contractor, and (b) an Excel with live formulas: item,
   rule, quantity, rate, amount, and the sheet and layer it came from, so anyone can re-check it.
6. **Recipes that learn.** The way you measured ("doors = block starting D, all layouts; brickwork = layer
   A-WALL length x height from the level schedule x thickness 0.23; RCC beams = layer S-BEAM") is saved as a
   **recipe**. On the next revision or the next project from the same consultant you press "Apply recipe": the
   whole sheet is rebuilt in seconds and **changes against the last run** are shown (see Change Radar).
7. **Takeoff vs model.** When a 3D model of the same job exists (built by the DXF to 3D pipeline, or an IFC),
   Takeoff Lens lines up the 2D takeoff with the model's BOQ per item and shows the difference: "RCC beams: 2D
   226 m3, model 212 m3, difference 6.6%, 14 beams not found in the model, click to see which". It catches both
   a model that missed something and a drawing that was not what it seemed. It uses the same rates as BOQ.
8. **Nothing uploaded, nothing changed.** The drawing is read only. No AI is needed for the core (it is rules
   and geometry); an optional AI helper can name items and suggest a recipe from the layer names, with your own
   key and the consent used elsewhere in cad2bim.

**Time saved (estimate, to be measured).** First takeoff of a 10-item sheet on a typical floor plan: from 1 to
3 hours by hand or in a trial app to about 10 minutes. A revision re-run with a recipe: from the same hour
again to about a minute. The saving that is not an estimate: the numbers can be checked, because each one is a
list of objects you can click.

**Why it is a "wow".**
- The 20-second clip: a messy architect's floor plan, one click on a door, all 38 doors flash orange and a
  counter runs to 38; one click on a hatch, the walls turn blue and "184.6 m2" appears; the sheet prints to a
  colourful PDF with a legend. Then "Apply last project's recipe" and the whole sheet fills in.
- The red 2D-vs-3D flag is a result paid takeoff apps cannot show, because they have no model.

### Implementation

- **Where it lives.** `packages/cad-engine` (the Rust core, compiled to WebAssembly) gets a `takeoff` module
  that works on the document the core already has: blocks with their handles, polylines, hatches, layers,
  layouts. The match rules are pure functions over that data, so they are testable without a screen. The React
  Takeoff tab and its panel live in `apps/web` and use the design system tokens; no new package.
- **Matching.** (a) Block match by name and by a geometry fingerprint (so renamed or exploded copies are
  caught), (b) polyline length by layer / linetype / colour, (c) hatch and closed-polyline area with islands
  subtracted, (d) text-pattern items (for example "C1" labels counted as columns) using the same text list the
  viewer already builds. Results are lists of handles, so the existing selection, highlight and List tools work
  on them unchanged.
- **Units and scale.** Take drawing units from the DXF header (already read), allow a per-drawing override, and
  for PDF a two-point calibration stored with the project.
- **Rates and Excel.** Reuse the BOQ rates table and the Excel writer used for the 3D BOQ (live formulas), so a
  takeoff and a model BOQ look the same and can be summed or compared.
- **Takeoff vs model.** Map a takeoff item to a BOQ line by IFC class and type / mark; unmapped items are shown
  and can be mapped by hand once; the mapping is stored in the recipe.
- **Recipes** are small JSON files saved in the `.c2b` project and exportable; they contain rules, never the
  drawing. Applying one to a revision uses Change Radar's matching of handles when the drawing is a revision of
  the same file.
- **Steps (one developer).** (1) block count + length-by-layer + area with a panel and Excel (about 1 week),
  (2) live colour and the printed PDF legend (about 1 week), (3) recipes and revision re-run (about 1 week),
  (4) PDF calibration (about 1 week), (5) takeoff-vs-model reconciliation (about 1 to 2 weeks).
- **Testing.** Small hand-counted DXF fixtures (a room with 4 doors and 2 windows, a hatch with a hole, an
  open polyline, an exploded block), each with the exact expected number; a real consultant drawing counted by
  hand once and compared; no drawing files go in git (`*.dxf` is ignored), only synthetic fixtures.
- **Rules to respect in cad2bim** (`CLAUDE.md`): nothing leaves the device, tokens only for colours, a
  `@cad2bim/cad-engine` version bump with a changelog entry, and the decision recorded in `docs/decisions.md`.

### Future update

- **Click-to-trace** for drawings with no clean data: trace a wall or room by clicking, snapped to the lines.
- **Assemblies.** "1 m2 of 230 mm brickwork" expands to bricks, mortar, plaster and labour by rate-analysis
  tables (CPWD DSR style) so the sheet reaches a priced BOQ in one step.
- **Takeoff for rebar from a detail sheet:** read bar callouts ("8-16 dia + 2-12 dia") and hand them to BBS.
- **Multi-sheet roll-up.** One sheet across all levels of a Drop Zone folder, with typical floors multiplied by
  the floor matrix (see Tower Stacker).
- **Revision delta:** "this revision adds 6 doors and removes 41 m of wall: +Rs 3.2 lakh", straight from two
  runs of the same recipe (see Change Radar and the roadmap's "What changed, in quantities").
- **Share a read-only takeoff** as a Model Capsule so a contractor can click each item and see what was counted.
- **Recipe library** shared inside a team, so the whole office measures the same way; Model Pulse can show how
  often each recipe is used.
- **Connect to existing ideas:** Drop Zone feeds it the right drawing; RFI Forge can flag items that conflict
  between plan and schedule; Rate Rocket prices it; Measure Right is the 3D side of the same comparison.
