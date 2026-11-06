# 20261106 Idea: Bid Mirror — drop the contractors' priced BOQ Excel files next to the 3D model; every bid line is matched to model elements, quantity gaps and front-loaded rates light up in 3D, and the bidders' quantities double as a free audit of your own model

## Entry 1

```
date-time:   2026-11-06 (scheduled run, first entry of the day)
idea name:   Bid Mirror (Tender Bid Excel vs Model Quantities: On-Device Line-to-Element Matching,
             Quantity Gap Heatmap, Unbalanced / Front-Loaded Rate Detector, "Model Probably Missing Something"
             Cross-Check, Side-by-Side Bidder Leveller, One-Page Award Note)
potential:   High for any team that issues a tender off its own model. The model is built for coordination, but the
             day it pays for itself is the day the bids come back: 3 to 6 Excel files, each laid out differently,
             each with its own line wording, units and totals. Today someone spends 2 to 5 days (an estimate from
             how this usually runs, not measured here) pasting them into one sheet, matching lines by eye, and
             asking "why is bidder B 18 % lower?" The answer is usually a quantity the bidder left out or priced
             low, a line they front-loaded, or an item the model never had. Bid Mirror turns that into minutes and
             shows each finding on the 3D element, so the BIM team becomes the owner's quantity referee and not
             only the drawing producer. It is also a two-way check: when all three bidders say 410 m3 of concrete
             and the model says 300, the model is the odd one out.
```

### Description

**The pain.**
- Bidders return the owner's BOQ template, but they edit it: lines added, merged, split, re-worded; units changed
  (cum / m3, RMT / m, MT / kg); "provisional" or "lump sum" slipped in; totals typed over formulas.
- Leveling is Excel work: copy, paste, VLOOKUP, hope the line numbers survived. Two bidders' "RCC M25 columns" may
  not be the same scope (one includes shuttering, one does not).
- The expensive surprises are not the headline totals. They are:
  1. **Quantity gaps**: a bidder's quantity is far below the model's (they will claim "extra" later).
  2. **Unbalanced rates**: high rates on items done first (excavation, PCC, mobilisation), low rates on items done
     late or likely to shrink; the bid looks fine in total and costs more when the job changes.
  3. **Missing lines**: waterproofing, anti-termite, curing, chairs and spacers, wastage, nothing priced.
  4. **Model errors the bids expose**: all bidders agree with each other and not with the model.
- The BIM modeller has the quantities and the element graph, but no one asks them to join the leveling meeting
  because the tool for it is Excel.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Rate Rocket (2026-10-28) builds the **owner's** priced tender BOQ from the model with
  rate analysis. Measure Right (2026-11-03) makes the IS 1200 measurement book with a 3D proof per quantity. Ratio
  Radar (2026-10-29) sanity-checks quantity ratios against past projects and norms. Bill Painter (2026-10-17) turns
  **site progress** into the running-account bill. Steel Ledger (2026-10-29) reconciles **steel received** with the
  BBS. Order Clock (2026-10-30) plans orders. **No entry takes the bidders' returned Excel files in** (searches for
  "bidder" and "submittal" in this folder return nothing).
- In cad2bim: the BOQ has element-wise quantities, rates and overrides and exports to Excel with live formulas
  (`docs/roadmap.md`, BOQ with rates, 0.6). "More to import" (roadmap, Next) plans reading property values from
  Excel, and `docs/design/documents.md` already reads Excel on the device with nothing in the file run. Clicking a
  BOQ group already selects its elements, which is the 3D link this idea needs. **Bid Mirror is the other end of
  the BOQ: from "what we ask" to "what they answered".**
- In the market (not exhaustively checked): bid-leveling and bid-comparison features exist in paid estimating and
  project-management suites and in cost-management add-ons, and QS firms do it in templated Excel. Those I know of
  compare **line to line between bidders**, not **line to the 3D model**, and they are paid, per-seat or
  per-project. A free, in-browser tool that matches bids to model elements and shows the gap on the 3D view is not
  something I found; treat that as unverified until a proper search is done.

**The idea.** A **Bids** panel beside the BOQ. Everything runs in the browser; no bid file is uploaded. Bids are
commercially sensitive, so "nothing leaves the device" is the selling line (and the repo's own rule).

1. **Drop in the Excel files.** One per bidder. The reader finds the header row, the description, unit, quantity,
   rate and amount columns by itself (and lets you correct it with two clicks), and ignores totals, notes and
   blank sections. Each file becomes a bidder column set. File names stay on the device.
2. **Match every line to the model.** For each bid line, find the BOQ group (and so the elements) it prices:
   - **Exact first:** the owner's template item codes, when the bidder kept them (most do).
   - **Then rules:** unit + words ("RCC M25 column" -> concrete, grade M25, category column), using the same
     grade / category vocabulary the model already has.
   - **Then fuzzy and, if the person allows, an on-device or own-key AI pass** for the rest, always shown with a
     confidence and a one-click "this is wrong, match it to ..." that is remembered for the next bidder.
   - Lines that match nothing stay visible as **"bidder's own line"** (e.g. a price for temporary power) and are
     listed, not dropped.
3. **Normalise before comparing.** Units converted (cum -> m3, RMT -> m, MT -> kg), lump sums flagged, "rate only"
   and "provisional" flagged, formulas re-checked (does quantity x rate equal the amount? does the column add up?).
   Arithmetic errors are a finding of their own.
4. **Show four things, not forty columns.**
   - **Quantity gap heatmap.** In 3D, elements are coloured by how far the bidders' quantities sit from the model:
     green within 3 %, amber 3 to 10 %, red beyond, grey no bid line matched. Click one: "Footings: model
     84.2 m3; A 79.0, B 61.5, C 84.0. B is 27 % short."
   - **Unbalanced rate detector.** For each bidder, rate vs the median rate of all bidders, and vs the owner's
     estimate (from Rate Rocket or the BOQ rates), weighted by the **model quantity**. It then runs a
     what-if: "if the quantity of earthwork falls 20 % and the quantity of finishes rises 20 %, bidder A's total
     moves +4.1 %, bidder C's moves -0.6 %." Front-loading shows as a simple chart of cumulative value against
     the 4D order of work, once the 4D timeline exists.
   - **Missing-scope list.** Model items nobody priced, items only some priced, and items priced that the model
     does not contain, each with a "who might have included it in another line" hint.
   - **Leveled comparison.** One sheet, bidders side by side, **at the model's quantity**, so the totals are
     comparable on scope: "adjusted total" next to "as quoted", with the adjustments listed line by line and
     open to the person to switch off.
5. **Use the bids to audit the model.** When two or more bidders agree with each other and differ from the model by
   more than the tolerance, raise a **model check card** ("Concrete in footings: model 300 m3; the three bidders
   say 405 to 412. Possible causes: footings not modelled at a grade change, a missing pile cap, a unit slip in
   the model's type"). It points at the elements and offers a BCF issue to the modeller. This is the "free audit"
   half and the part that earns the BIM team a seat in the leveling meeting.
6. **Write the award note.** One page, plain words, with the numbers and where each came from: the recommended
   bidder, the three biggest gaps to clarify, the three rates to negotiate, the items to confirm in writing, and a
   **clarification letter draft per bidder** ("Please confirm the quantity of line 3.2 ...") ready to paste into
   email. Every claim links to the 3D elements or the Excel cell.

**What the person sees.** A bar of bidder chips (A, B, C) with the as-quoted and adjusted totals; a 3D view coloured
by gap; a list of findings ranked by rupees at stake; and a split view where clicking a bid line selects its
elements and clicking an element shows every bidder's line for it. A **Replay** button sweeps the gap colours from
the lowest to the highest bidder, which makes a good three-second clip for the owner's meeting.

**Why it can wow people (and is honest about it).**
- LinkedIn-friendly demo: drag in three bidders' Excel files and a model; in about ten seconds the view lights up
  red on the footings and a card says the lowest bid is short by a hundred cubic metres. That is a real
  rupee-and-programme conversation, not a screenshot of a pretty model.
- It uses the thing teams already own (the model) for the thing they care most about (the contract price).
- It is on trend with the "5D BIM", "AI in tendering", "cost certainty" and "BIM that pays for itself" posts
  that appear in AEC feeds; I cannot browse LinkedIn or Instagram from here, so this is a judgement, not a count.

**Limits and honesty (written on the cards).**
- A bid line can legitimately differ from the model (a bidder's method, a different wastage, an alternative
  design). Findings say "differs", not "wrong". Nothing here is a ruling on a bid; the award stays with people.
- Matching is the hard part and will be imperfect. Every match shows its confidence and its reason, and the
  unmatched share is shown as a number on the panel ("142 of 160 lines matched, 18 need you").
- It compares quantities and rates, not methods, programme, safety, references or finance.
- Only as good as the model's quantities. The model audit card is a hint to look, not proof the model is wrong.
- Commercial data: kept in the session or in the project file only if the person saves it; never sent anywhere.

**Success measures.** Hours from the bids landing to a leveled sheet (target: under 30 minutes for 3 bidders and
200 lines, against the 2 to 5 days estimated above); share of lines matched without a human click; gaps found
that a manual pass missed (checked on 3 old tenders where the outcome is known); model errors found by the
cross-check and fixed before award.

### Implementation

Build on what cad2bim already has; only the reader, the matcher and the findings are new.

1. **Excel reader (new, small).** Use the Excel reading already planned in `docs/design/documents.md` and "More to
   import". Output a plain table per sheet, then a `BidLine` (`bidder`, `code`, `description`, `unit`, `qty`,
   `rate`, `amount`, `source cell`, `flags`). Header detection by keyword scoring across the first 30 rows
   (description / item, unit / uom, qty / quantity, rate, amount / total), with a manual override and a saved
   per-owner template so the second tender is one click.
2. **Unit and arithmetic normaliser (new, small).** One table of units and factors (cum, m3, cu.m; sqm, m2; rmt, m;
   mt, tonne, kg; nos, no., each) shared with the BOQ; flags for lump sum, provisional, rate-only; recompute
   `qty x rate` and column sums and raise an `ArithmeticFinding`.
3. **Matcher (new, the core).** Input: bid lines and the live BOQ groups (category, type, grade, level, unit).
   - Stage 1: exact `code` match against the owner's template (the BOQ export writes the code into a hidden column
     so a bidder's returned file carries it back).
   - Stage 2: rule match with a small vocabulary built from the model itself (grades M20 / M25 / Fe500, categories,
     storey names) and unit compatibility (a concrete line must be volume).
   - Stage 3: fuzzy text (token set ratio) with a threshold; optional AI pass through the existing
     "AI chat with your own key" path, sending **only the unmatched line texts and the BOQ group names**, never
     quantities or rates, and only after the person agrees.
   - Output a `Match` (`bidLine -> boqGroup`, `confidence`, `reason`) and a learned-corrections store keyed by
     normalised description, kept in the project file.
4. **Findings (new, pure functions, testable).**
   - `quantityGap(group, bidderQty[])` -> per-bidder `%` and a status against a tolerance the person sets.
   - `unbalance(bidder, rates, modelQty, medianRates, ownerRates)` -> unbalance score and the sensitivity table.
   - `missingScope(modelGroups, matches)`.
   - `consensusCheck(modelQty, bidderQty[])` -> model-audit card when at least two bidders agree within 5 % and
     differ from the model by more than the tolerance.
   Each returns the same card shape the QA panel already uses (`measured`, `limits`, `elementIds`), so cards, the
   "select elements" action and BCF export come for free.
5. **3D colouring (reuse).** The viewport's per-element `overrides` (colour, transparency) already colours by
   status in Visibility / Graphics; add a "Bid gap" colour-by source.
6. **Leveled sheet and award note (new, small).** Export to Excel with live formulas (the BOQ exporter's pattern):
   as-quoted, adjusted-at-model-quantity, and a Notes sheet listing every adjustment. The award note is a template
   filled from the findings; the clarification letters are one template per finding type.
7. **Where it lives.** A tab beside BOQ in the web app; the engine side is pure TypeScript in `packages/engine`
   (reader output -> matcher -> findings) so the Python console and the agent's tools can call it too
   (`bids.load`, `bids.match`, `bids.findings`), in line with "one model, every view".
8. **Tests.** A synthetic tender (model of 40 elements; three generated bidder files with known planted faults: one
   short footing, one front-loaded earthwork, one missing waterproofing, one unit slip, one arithmetic error) and a
   unit test that each fault is found and nothing else is. Then 3 real old tenders (kept outside git, as for
   drawings) to measure the match rate and the false-alarm rate.
9. **Phasing.**
   - Phase 1: read one bid, match to the BOQ, quantity gap heatmap, arithmetic checks. (1 to 2 weeks, estimate.)
   - Phase 2: several bidders, leveled sheet, unbalanced-rate detector, missing-scope list.
   - Phase 3: model-audit cross-check, award note and clarification letters, learned corrections.
   - Phase 4: optional AI matching pass; 4D front-loading chart once the timeline ships.

**Risks.** Messy Excel (merged cells, images of tables, PDFs instead of Excel) -> start with .xlsx / .csv only and say
so; a wrong match makes a wrong finding -> confidence and the unmatched count always visible; commercial
sensitivity -> device-only, no telemetry on contents; legal weight of an "unbalanced" label -> wording is
"rate pattern differs from the median", not an accusation.

### Future update

- **4D link:** with the timeline, front-loading is shown as cumulative bid value against the work order, and a
  cash-flow curve per bidder (with Crew Curve and Order Clock for the contractor-side view).
- **Variation tracker after award:** the winning bid's rates become the contract rates; Change Radar, Rev Merge and
  Issue Desk changes are priced at those rates, so every revision shows "this change costs X at contract rates".
- **Bill Painter and Measure Right:** the contract BOQ is the starting point for the running-account bill and for
  the measurement book, with the same 3D proof.
- **PDF bids and scanned BOQs:** read tables from PDF with the documents viewer; scanned files through Plan
  Tracer's tracing stack once it exists.
- **Subcontractor packages:** split the BOQ by trade package, send each package as its own template (Excel with
  item codes), and level each trade separately.
- **Learned norms:** across a firm's past tenders, keep per-item rate bands on the device and say "this footing
  rate is at the 85th percentile of your last 14 tenders".
- **Tender Q&A log:** every clarification letter and reply stored against the 3D element, so the contract file
  has a 3D index.
