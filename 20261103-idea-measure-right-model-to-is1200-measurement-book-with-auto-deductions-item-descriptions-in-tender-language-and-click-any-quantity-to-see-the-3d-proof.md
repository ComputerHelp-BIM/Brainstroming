# 20261103 Idea: Measure Right — turn the model into an IS 1200 / CPWD Measurement Book with the deductions done by rule, tender-language item descriptions, and a click on any quantity that shows the elements, the formula and the rule behind it

## Entry 1

```
date-time:   2026-11-03 12:00 UTC (scheduled run)
idea name:   Measure Right (Method-of-Measurement Engine: Model to IS 1200 / CPWD Measurement Book, Auto
             Deductions, Tender-Language Item Descriptions, "Prove This Number" Click-Through to 3D)
potential:   Very high. A BOQ from a model is only as good as its rules. A QS or site engineer does not
             accept "the software says 412.6 m3"; they ask "did you deduct the openings, did you measure
             to the face or the centre, what is in this number?". Today that argument is settled by
             hand-measuring in Excel (the Measurement Book, "MB" sheet: Nos x L x B x H = Qty). Measure
             Right makes the model answer that argument itself. Estimated saving (an estimate, not
             measured): 1 to 3 days per bill for a QS or BIM modeller on a mid-size building, and far
             fewer "your BOQ doesn't match mine" meetings. It also makes BIM quantities acceptable in
             tenders and running-account bills, which is the real reason many teams stop trusting them.
```

### Description

**The pain.**
- Every country measures differently. In India the method is IS 1200 (parts for earthwork, concrete,
  brickwork, plaster, paint and so on) plus CPWD and state SOR conventions: openings under a stated area
  are not deducted, deduction rules differ for brickwork and plaster, concrete is measured net of steel
  volume only when stated, and each item has a fixed unit (cum, sqm, rmt, nos, kg). Elsewhere it is
  NRM2, SMM7, CESMM or POMI. (Read these as the rules as I know them; the exact clauses must be checked
  against the current published editions before anything is built or sold.)
- A model's raw quantity is geometric. A tender BOQ is a rules-based quantity. The gap between them (door
  and window deductions, column-beam-slab junction overlaps, skirting along wall length minus doors,
  plaster on both faces, dado and wall-tile heights) is where hours go, in Excel, after every revision.
- "Where did this number come from?" is the most common question a QS asks about a BIM BOQ. Nobody can
  click a number and see.
- "BIM quantities are not accurate" is a common comment in AEC posts and groups (a trend I read in
  feeds, not a measured statistic). In most cases the model is fine and the measurement rules were not
  applied, or were applied silently.

**What exists today (one thin check, to be re-verified before any "nothing like it" claim).**
- Revit schedules and Dynamo/Excel give geometric quantities. They do not apply a method of measurement.
- Paid tools: CostX, Bluebeam Revu, Candy, iTWO, Navisworks Quantification, Costlab and local
  Indian estimating packages measure from drawings or models, usually on subscription or licence, and I
  did not verify here which of them apply IS 1200 or output an MB sheet. Treat their feature lists as
  unchecked.
- Free route: export to Excel, then a QS re-measures by hand in an MB sheet.
- In this repo: the BOQ in `@cad2bim/engine` (`packages/engine/src/model/boq.ts`) gives quantities by
  group; Rate Rocket prices a BOQ; Wall Wizard does blockwork, plaster and paint deductions for walls
  only; Ratio Radar sanity-checks totals; Change Radar shows BOQ deltas between revisions. None applies a
  named method of measurement across all trades, writes the MB sheet, or lets a person click a number
  and see its proof. Measure Right is the rule layer under all of them.

**The idea.** A **Measure Right** panel in the cad2bim web viewer. The BOQ gets a "Measured by" switch:
*Raw geometry* (today) or a rule set: *IS 1200*, *CPWD*, *Client's own*, with *NRM2* and others later.

1. **Rule sets as data, not code.** A rule set is a plain, versioned file: item code, description template,
   unit, what elements feed it, how to measure (net volume, face area, centre-line length, count), and
   the deduction rules (for example: "openings up to X m2 not deducted; beyond that deduct the whole
   opening", "wall measured to face of column", "slab measured net of beam rib below"). The exact numbers
   come from the published standard or the client's tender; the file records the clause it came from.
   Teams can fork a rule set for their client.
2. **Measure.** The engine walks the model and applies the rules: junction handling (column / beam / slab
   / wall overlaps counted once, in the order the rule says), opening deductions with their reveal
   returns where the rule asks, both-face plaster, finish areas by room, skirting and dado by perimeter
   less doors, bar weight from the BBS where steel is measured in kg.
3. **Write the item description in tender language.** From the element and type properties: "Providing
   and laying in position cement concrete of grade M30 for columns above plinth, including formwork,
   curing and finishing, as per drawings and specifications." Templates are editable so the team keeps
   its house wording and its client's schedule-of-quantities numbering.
4. **The Measurement Book.** One click writes the MB sheet as Excel: item, then one line per measured
   element group with its mark, level and the Nos x L x B x H columns, deduction lines in negative, and
   the item total. The QS can recompute any line by hand and get the same answer. A summary tab gives the
   abstract (item, unit, quantity), ready for pricing or Rate Rocket.
5. **Prove this number.** Click any quantity (in the panel, the MB or the abstract) and the 3D view
   isolates exactly the elements that feed it, colour-coded: counted, deducted, excluded by rule. A small
   card shows the formula for that line and the clause of the rule used. This is the part a client's QS
   will remember.
6. **Rule diff.** Switch from IS 1200 to the client's rule set and see which items move and by how much:
   "your concrete is unchanged, plaster is down 4.1% because their rule deducts openings above 0.5 m2".
   This ends the argument about whose rules are right with a number.
7. **Gaps, not guesses.** Items the model cannot measure (anything not modelled: waterproofing
   thickness, painting coats, loose furniture) appear in a "not measured from the model" list, with a
   manual-entry row, so the BOQ is complete and nothing is silently missing or invented.

**The wow 1: click a number, see the proof.** In a client meeting, someone challenges the plaster
quantity; you click it and the walls light up with the openings that were deducted shown in red. The
debate ends in about ten seconds.

**The wow 2: the 20-minute bill.** A running-account or tender bill that used to take a QS days of
re-measuring comes out as an MB sheet in minutes, and when the architect revises the plan the MB is
regenerated and Change Radar shows exactly which lines moved.

**The wow 3: "switch the rules" slider.** One toggle re-prices the building under IS 1200, CPWD, or the
client's own rules, with the difference listed item by item. A bid manager sees the commercial effect of
a measurement clause, which no raw BIM quantity can show.

**Why this fits LinkedIn and Instagram right now (read as a trend from feeds, not a measured one).** QS
and 5D/"BIM to cost" posts do well, and the usual comments are "great, but is it measured to the
standard?". A 30-second reel of "click a quantity, see why" and "switch the rules, watch plaster change"
answers that exactly and works well as a Model Reel clip.

### Implementation

Everything below is a plan; nothing was built or measured.

- **Place in the toolkit.** The rule engine is a pure function in `@cad2bim/engine` (`model/` beside
  `boq.ts`): elements + rule set in, measured items with lines and provenance out. It is testable without
  a browser and can run in a worker. The panel is in `apps/web`, next to the BOQ. Add-in and agent helper
  (MCP) can call the same function, so an AI agent can answer "what is the M30 column concrete under IS 1200
  and why".
- **Provenance is the core data structure.** Every measured line stores {itemCode, elementGUIDs,
  formula inputs, rule id, clause, sign (+/-)}. "Prove this number", the MB sheet and the rule diff are all
  views of it. Design this first; the rest is presentation.
- **Geometry needs.** Net volumes and areas from the loaded IFC geometry; junction overlaps need boolean or
  voxel/sampled overlap of columns, beams, slabs and walls. Start with the cases the standard names
  (opening deductions, column-beam-slab overlaps, wall faces) and the model quantities IFC already holds
  (`IfcElementQuantity` / `Qto_*`), then fall back to geometry. State the tolerance against hand
  measurement honestly after the first test.
- **Rule files.** JSON or YAML, schema-checked, each rule with `source` (standard, edition, clause) and a
  `verified_by` field the user fills in. Do not ship a rule file as "IS 1200" unless its author has read the
  current edition; mark unverified rules as drafts in the UI. The standards themselves are copyrighted:
  store the rule logic and the clause number, not the standard's text.
- **Descriptions.** Template strings with fields from properties (grade, size, level, finish). Optional AI
  rewrite with the person's own key (roadmap: "AI chat with your own key"), clearly marked, never required.
- **MB writer.** Reuse the BOQ Excel writer; lay out Nos / L / B / H / Qty columns with deduction lines in
  negative and formulas left live in the cells so the QS can audit them in Excel.
- **Where Wall Wizard and Rate Rocket plug in.** Wall Wizard's finish quantities become rule-set items
  (not a second engine); Rate Rocket prices the abstract by item code; Change Radar diffs MB line by line.
- **Privacy.** Local only. Files stay on the device, as in the repo rules; sharing an MB is an explicit
  export.
- **Licences.** Rule files are our own data. Any geometry or spreadsheet library must be MIT, Apache-2.0,
  BSD or ISC; no GPL (repo rule).
- **Risks.** (1) A wrong rule silently misprices a tender: show the rule and clause on every line, ship
  rule sets as drafts until a QS signs them off, and keep a "difference from raw geometry" figure visible.
  (2) Standards differ by state, client and edition: rule sets are forks, never one global truth.
  (3) Models with sloppy modelling (overlapping duplicates, walls through columns) give wrong nets: run the
  existing duplicate/overlap QA first and show its warnings on the panel.
  (4) Trust: never present the output as a certified measurement; the QS stays the signing person.
- **First 2 weeks.** Provenance data structure, one rule set with five items (RCC columns, beams, slabs;
  brickwork with opening deductions; internal plaster), the MB Excel writer, and "prove this number" on those
  five items. Test on one real project model kept outside git against that project's hand-made MB, and
  record the difference per item, in the next entry. Ask one practising QS to review the rules before any
  wider use.

### Future update

- Add more rule sets: NRM2, CESMM, and state SORs (Maharashtra, Karnataka, Tamil Nadu and so on), each as a
  community-reviewed file the team can fork.
- Import the tender's own BOQ (Excel) as the item list, and map model groups to its numbering automatically,
  so the measured quantities drop straight into the client's format.
- Running-account bills: connect to Bill Painter so each month's progress is measured by the same rules and
  the RA bill carries the same item codes and proof links.
- Variation orders: Change Radar + Measure Right produce a "VO quantity statement" with before/after MB
  lines for each revised item.
- Shared review link: a read-only page where the client's QS clicks quantities and sees the proof without
  installing anything (explicit share, never automatic).
