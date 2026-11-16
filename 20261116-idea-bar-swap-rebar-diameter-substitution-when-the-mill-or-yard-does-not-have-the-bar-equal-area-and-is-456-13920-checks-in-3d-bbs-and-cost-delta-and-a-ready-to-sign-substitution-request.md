# 20261116 Idea: Bar Swap — "the yard has no 25 mm": pick the bars, say which diameters you can get, and get a code-checked substitution in 3D with the new bar marks, BBS, weight and cost change, and a substitution request ready for the structural consultant to sign

## Entry 1

```
date-time:   2026-11-16 (scheduled run, UTC)
idea name:   Bar Swap (rebar diameter substitution, checked in the 3D model, in the browser, nothing uploaded)
potential:   High, and it happens every week on every RC site. The site calls: "25 mm is not available for two
             weeks, can we use 20 + 16?" or "the mill rate for 32 mm jumped, make the job 28 and 20." Today a
             modeller or engineer does equal-area on a calculator, then checks spacing, cover, lap and anchorage
             by hand, edits bar marks in the model, re-runs the BBS and writes a letter. It is slow, it is done
             in a hurry, and the checks that people skip are exactly the ones that matter (fits in the beam
             width? lap length now longer? column bars still symmetric? bar count still ok for IS 13920?).
             Time saved: from how this work usually runs, NOT measured here. A first test: take one real
             substitution request from a past project, time a person doing it by hand and compare with the tool
             on the same members. Do not quote hours until a team has timed it.
             Market check (2026-11-16, one search on substitution tools, one on steel prices; not exhaustive).
             It looks open, so I say "I did not find one", not "none exists":
               - Equal-area is easy to find: web calculators (civilconcept, idecad AS Calculator, various
                 rebar area/weight pages). They give number-of-bars for an area. They do not know your
                 member, your cover, your neighbouring bars, your laps, or your BBS.
               - Graitec PowerPack for Revit (paid) detailing add-in lists average rebar diameter in 2025.1;
                 I found no description of a "swap diameters under code checks and write the substitution
                 request" feature. Not verified in the product itself.
               - An engineering forum thread notes that equal area does not mean equal performance
                 (bundles, development length, column layouts need care). That is the gap this idea fills.
               - Revit forum: users report the built-in bar-type list may lack a size they want.
             What I did not find: a free browser tool that works on the 3D rebar model, changes the diameters
             of chosen bars, re-checks the code rules on the real geometry, and outputs the BBS delta and the
             request letter. Only a claim from one search; verify before marketing.
             Extends tools already in this repo (it does not repeat them):
               - Bar Optimizer: cuts and laps the SAME bars better (cutting stock, lap placement, carbon).
                 Bar Swap changes WHICH bars. They chain: swap first, then optimise.
               - Mark Merge / Type Diet: fewer marks for the same design, with safe upsizing. Bar Swap is
                 driven by availability and price, per diameter, not by mark count.
               - Cover Cop, Joint Congestion Doctor, Core Cage, Slab Sketch: they make or check cages. Bar Swap
                 re-runs their checks on the swapped cage.
               - Steel Ledger: tells you steel was issued that the BBS did not explain. Bar Swap is the
                 BBS change that has to exist BEFORE a different diameter arrives on site.
               - Rate Rocket / Change Radar: price and revision deltas; Bar Swap feeds them a clean delta.
             Trend: press this year shows rebar prices moving fast in both directions (BigMint: up about
             INR 5,000/t in Feb 2026, then four-month lows by June, with buyers cautious). Price swings and
             "what can we buy this week" are what makes substitution a daily topic, and value engineering is a
             steady topic on LinkedIn. I could not measure LinkedIn or Instagram counts directly; the sources
             are press pages.
             The wow: select all 25 mm bars in columns C1–C14 in the 3D view. Type what the yard has:
             "16, 20, 28". The model re-bars in front of you: ghost bars (old) and solid bars (new), the
             cage turns green where all checks pass and amber where a rule bites ("beam B7: 5 x 28 does not
             fit in 230 wide, try 4 x 28 + 2 x 16"). A bar at the top reads: "Steel +1.8% (area never below
             required), weight +212 kg, cost +Rs 14,600, 3 members need your eye." One click makes the
             substitution request PDF with the numbers. Nothing changes in the real model until you accept.
```

### Description

**The pain.**
- Steel arrives in the diameters the mill and the yard have, not always the ones in the drawing. Prices change
  by diameter. Someone has to answer "can we swap?" the same day.
- The answer is not just area. A bigger bar needs more room (spacing, cover, bundling), a longer lap and a longer
  anchorage; fewer, bigger bars can break the symmetry a column needs or the bar count IS 13920 wants at a joint;
  smaller bars can mean more bars than the width allows.
- The model, the BBS, the bar marks and the cutting list all have to follow. If the model is not changed, the yard
  and the BBS disagree (Steel Ledger will later find that mismatch, but it is better never to create it).
- The structural consultant must approve, and wants numbers on one page, not a phone call.

**The idea in one line.** Say which bars and which diameters you can get; the tool proposes the fewest, safest
swaps on the real 3D cage, checks every rule it can, and gives you the model change, the BBS change and the
approval note.

### How it works (user view)

1. **Pick the bars.** In 3D: select members, a level, a bar mark, a diameter ("all 32 mm"), or paste a list of
   marks. The tool lists what it found (for example 412 bars, 14 columns, 6 beams) and which grade and ductility
   (Fe500 / Fe500D / Fe550D) each belongs to.
2. **Say what you can get.** Either type the diameters available (16, 20, 28), or drop a stock/price sheet (Excel
   or CSV: diameter, grade, available tonnes, rate). Optional: "avoid more than N sizes per member", "never swap
   in columns without a flag", "mill rate for 25 is high, make it expensive".
3. **See the options.** For each member the tool shows up to three layouts, for example "same area, 2 sizes",
   "fewest bars", "lowest cost", each as ghost-vs-new bars in 3D, with one line of reasons. The safe default is
   "area provided is equal or more than the original provided, and never below the required area where the
   design area is known".
4. **Rules that run on the real geometry.** Each is a pass / warn / fail with the clause named:
   - Area: provided at least original; for beams and slabs also the required area when the design is attached.
   - Count and layers: bars fit in the width with cover, clear spacing (bar diameter, aggregate size + 5 mm,
     25 mm style rules as the company chooses), number of layers allowed.
   - Columns: minimum bars (4 in rectangular, 6 in circular), bar spacing on a face not above the code limit,
     symmetry kept, percentage steel between the minimum and the maximum, link diameter and spacing still ok
     for the new longitudinal diameter (link not less than a quarter of the largest bar and 6 mm).
   - IS 13920 items: minimum two bars top and bottom through a beam, tension steel ratio limits, beam-column
     joint congestion (hands to Joint Congestion Doctor), lap not in the plastic hinge zone (reuses the lap rules).
   - Development length and lap length recalculated for the new diameter and grade; the drawn lap and anchorage
     are lengthened, and the tool says if the new length no longer fits in the column depth or the hook.
   - Bar bending: minimum bend radius for the new diameter; a stirrup hook or bent bar that no longer fits is flagged.
   - Cover and fire: clear cover to the new outer bar unchanged or larger (links count).
   - Crack width / service checks in water-retaining members (links to Tank Smith rules) are flagged "check the
     design", not decided.
   The tool never claims the design is approved. It says "the rules above passed; the engineer of record signs".
5. **Accept per member or all.** Accepting writes a new bar set into the model as a revision with the old one
   kept (so it can be undone and shown in the revision timeline). Bar marks, BBS and cutting list update.
6. **The numbers.** Before / after table by diameter: total length, number of bars, weight, rate, cost. Wastage
   effect when the new lengths are fed into the cutting-stock optimiser. A "Rs per tonne" line is shown only when
   rates were given; otherwise weight only.
7. **The substitution request.** One A4 PDF or an Excel: member list with marks, the original and the proposed
   bars with a small section sketch each, the rules checked and the result, the weight and cost change, the
   reason ("diameter not available until <date>" typed by the user), and signature boxes. It carries the model
   revision. It can also be written as BCF issues (one per member) for the consultant to answer in the same tool.

### Why a BIM team would use it

- The answer to the site call comes in minutes with the proof, not at the end of the day.
- The model, BBS, cutting list and request stay in step, so the "model says 25, yard has 20" mismatch never starts.
- It teaches juniors why equal area is not the whole check: each warning names the rule and shows it on the bars.
- Procurement can ask "what if we drop to three diameters?" and see weight, cost and the members that complain.
- A record of every substitution with who, when and why is kept, which helps at handover and in any dispute.

### Implementation

- **Reads the 3D cage already in cad2bim.** Rebar in the IFC (IfcReinforcingBar with diameter, steel grade and
  the bar mark property), and the rebar built by the pipeline. No new reader is needed. A model with bars as
  plain curves and no bar data is told: "cannot swap, no bar data".
- **A small deterministic rule engine.** Pure functions: `provided_area`, `fit_in_width`, `min_spacing`,
  `ld_and_lap(dia, grade, concrete grade, bar type, zone)`, `column_rules`, `bend_radius`. The same tables the
  rest of the app uses for laps (Bar Optimizer) and cover (Cover Cop), in one shared module, so they cannot
  disagree. Rule set is data (IS 456 / IS 13920 first; a company or country pack later).
- **Layout search.** Per member, a small search over combinations of the allowed diameters (at most two sizes in
  a layer, so it stays buildable) that meet the area and the fit rules, ranked by: bars changed, extra weight,
  cost if rates exist. It is brute force over a small set; it finishes in milliseconds per member in a web worker.
- **Write-back as a revision.** New bars are created as a new bar set in the project (.c2b) with the old set
  linked, using the revision timeline that the app already has. Export to IFC keeps the GUIDs of the bars it did not
  touch. The BBS tool reads the new set.
- **Stock sheet import.** Excel/CSV read on the device. The same importer as the other sheet tools.
- **Request PDF.** Built in the browser from the same tables (no server). Section sketches drawn from the cage
  geometry.
- **Revit side (optional).** With the Revit bridge, a preview-then-confirm "apply to Revit rebar" action, never
  silent. The first version is IFC and cad2bim only.
- **Where it sits.** New tool in the quantities and rebar group of the ribbon, with a "Swap" button on the
  context menu of a selected bar or member. Cannot be done without bar data, so it comes after the rebar model
  work already planned; the rule module can be built and tested first on its own with unit tests (known cases
  from IS 456 worked examples, written down by an engineer, not invented by the tool).
- **Tests.** (1) Rule unit tests with hand-checked cases. (2) Property test: for any swap the tool accepts, the
  provided area is not below the original. (3) Round trip: swap, accept, undo, the model is byte-equal to the start.
  (4) A real past substitution request redone by the tool and compared by an engineer. Report the false-alarm
  and miss rate; do not publish a time saving until a team has timed it.

### Future update

- **Whole-project diameter diet.** "Use only 3 diameters" or "make 32 mm go away" over the entire job, with the
  steel penalty slider (the same idea as Mark Merge) and a one-page cost comparison.
- **Live price feed.** Optional rates from a user-given sheet or a public index, so the tool can say "28 mm is
  Rs X per tonne cheaper than 25 mm today"; always shown with the date, never silent.
- **Bundles and couplers.** Bundled bars (where the code allows), mechanical couplers instead of long laps.
- **Slabs and walls by spacing.** "12 mm at 150 becomes 10 mm at 100" with the same rules for spacing and crack control.
- **Purchase hand-off.** The swap list goes to Order Clock as the new material order, and to Steel Ledger as the
  new theoretical quantity for the reconciliation.
- **Consultant mode.** The consultant opens the request link in the browser (no upload, device to device like Room
  Link), sees the ghost-vs-new bars, and replies approve / change / reject, which lands as BCF.
- **Learn from your approvals.** Layouts the consultant keeps approving rank higher the next time.
- **Other materials.** Structural steel section swaps (ISMB 300 to 350) with the same pattern, later.

### Reel idea for LinkedIn / Instagram

12 seconds: a column cage in 3D, bars blue. Text: "Site calls: no 25 mm for 2 weeks." Select the bars, type
"16, 20, 28". Ghost bars slide out, new bars slide in, the cage turns green, one member flashes amber with a note
"does not fit in 230". The top bar counts: "+1.8% steel, +Rs 14,600". Tap "Request" and a one-page PDF drops
in. End card: "Equal area is not the answer. The cage is." Caption: "The 'can we swap the bar?' call, answered
with proof." (Use a real or clearly marked example and say which; do not post invented numbers.)

### Honest limits

- It is a checking and drafting aid. It does not replace the structural engineer's judgement or signature. Anything
  outside its rules (dynamic loads, fatigue, water-retaining crack control, special members) is flagged for the
  engineer, not decided.
- Code details (clause values, ductility, bar grade, bundling) must be set by an engineer of the company; the
  first pack is for IS 456 / IS 13920 and must be reviewed against the current code editions before use.
- Rules need bar data in the model. Bars drawn as plain lines cannot be swapped.
- Design area is only known when the design (for example from ETABS or the schedule) is attached; without it the
  tool can only keep "area provided is at least the original", which is a weaker promise, and says so.
- Cost lines depend on the rates the user gives. Without them only weight is shown.
- The hours-saved claim is an estimate to test. The market check is two searches; paid tools I did not try may
  do parts of this.

### Sources (checked 2026-11-16)

- Equal-area bar calculators (area only, no member checks): [civilconcept: cross-sectional area to number of bars](https://www.civilconcept.com/cross-sectional-area-to-the-number-of-bar-converter-detailing-of-rebars/),
  [ideCAD AS Calculator](https://help.idecad.com/ideCAD/as-calculator).
- Paid Revit rebar detailing add-in, nearest I found: [Graitec PowerPack Rebar Detailing 2025.1](https://graitec.com/uk/blog/whats-new-in-powerpack-rebar-detailing-for-revit-2025-1/).
- Equal area is not equal performance (forum, a practitioner's view): [Eng-Tips thread on larger bars and bundles](https://eng-tips.com/threads/im-a-dummy-can-some-let-me-know-if-i-can-change-larger-diameter-rebar-with-2-piece-bundles.454050).
- Revit bar-type list limits: [Autodesk forum](https://forums.autodesk.com/t5/revit-structure-forum/revit-2023-problems-with-structural-area-reinforcement-bar-type/m-p/12319270).
- Indian rebar price swings in 2026: [BigMint, 3 Mar 2026](https://www.bigmint.co/insights/detail/india-primary-rebar-prices-rise-faster-than-hrc-higher-by-nearly-inr-5000-t-in-feb26-727859),
  [BigMint, 6 Jun 2026](https://www.bigmint.co/insights/detail/india-tier-1-steel-mills-reduce-rebar-prices-for-jun26-amid-rising-inventories-weak-enquiries-757688),
  [BigMint, 4 Jul 2026](https://www.bigmint.co/insights/detail/india-if-route-rebar-prices-decline-on-slow-demand-softer-raw-materials-766440).
- These are web and press pages. I could not measure LinkedIn or Instagram trends directly, and I did not search for
  bar-size shortages or value-engineering practice specifically.
