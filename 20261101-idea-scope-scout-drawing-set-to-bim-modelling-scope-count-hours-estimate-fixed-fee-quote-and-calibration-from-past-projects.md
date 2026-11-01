# 20261101 Idea: Scope Scout — drop the client's drawing set and get a counted modelling scope, a man-hours estimate, a fixed-fee quote and a risk list in minutes, tuned by your own past projects

## Entry 1

```
date-time:   2026-11-01 (scheduled run, UTC)
idea name:   Scope Scout (Drawing Set to BIM Scope Count, Hours Estimate and Fixed-Fee Quote)
potential:   High. Every BIM job starts with "how many hours, and what do we charge?". Today a senior
             person opens the drawings, counts columns and beams by eye, guesses, and sends a number a day
             later. Under-quoting eats the margin; over-quoting loses the job. Scope Scout counts from the
             drawing itself (cad2bim already reads DXF/DWG and runs the pipeline), converts counts to hours
             with the team's own productivity, and prints a quote page. Estimated saving (an estimate, not
             measured): 2-6 hours per enquiry for the person who prices it, and a first reply the same day.
             It also gives the sales side a "wow" in the first call: drag the drawing in, see the count.
```

### Description

**The pain.**
- A BIM service team gets 5-20 enquiries a month. Each one arrives as a zip of DXF/DWG/PDF with a one-line
  mail: "need LOD 300 structural model, how much and by when?".
- Pricing is done by feel or by "last time it took X". Nobody knows the true size of the job until
  modelling is half done: 420 columns or 280? Is the basement in scope? Are the beam schedules on the
  drawings or only in a PDF note? Scope surprises are the commonest reason a fixed-fee job goes over.
- The numbers that decide the price (element counts, number of levels, typical-floor repeats, drawing
  quality) are exactly what a drawing reader can produce in seconds.
- Fast, accurate quoting and "AI takeoff" are recurring topics in AEC posts on LinkedIn; read this as a
  trend I have seen in discussions, not a measured statistic.

**What exists today (one thin search, to be re-checked before any "nothing like it" claim).**
- Quantity takeoff tools (PDF measuring, AI plan counting) exist, mostly paid and mostly aimed at
  contractors estimating *construction cost*. I am not aware of one that estimates the *modelling effort*
  of a BIM job from the drawings, with the firm's own productivity numbers, and keeps the files on the
  device. I did not confirm this or any prices.
- Earlier ideas in this repo come from the other end: Model Pulse measures actual hours after the work;
  RFI Forge audits drawings before modelling; Layer Whisperer cleans layers. None predicts the hours or
  writes the quote. Scope Scout is the front door of that chain, and Model Pulse is its feedback loop.

**The idea.** A **Scope** tab in cad2bim's CAD workspace. Drop one or many drawings (DXF, DWG, and later
PDF); get:

1. **A counted scope sheet.** Per discipline and per level: columns by mark, beams by mark, slabs, walls,
   footings, stairs, openings, doors/windows, grids, levels. Counts come from the same reading the
   DXF-to-3D pipeline already does (CH template layers, tables, notes such as "UNO"). Anything not
   countable is listed as "not found" instead of silently skipped.
2. **A complexity read.** Simple signals, each shown with its evidence on the drawing (click a flag, the
   viewer zooms to it):
   - Typical-floor repeats (floors that are the same, so the model is built once and copied).
   - Curved or skewed members, sloping slabs, level changes, transfer girders.
   - Messy layers, text-only dimensions, missing schedules, blocks exploded to lines.
   - Revisions on the sheet that disagree with each other.
3. **An hours estimate with a range.** `hours = sum(count x minutes-per-element x complexity factor)`
   plus fixed items (setup, families, QA, sheets, review). Shown as **low / likely / high**, never as one
   false-precise number. Starter minutes-per-element are *placeholders*, to be replaced by the team's own.
4. **A fixed-fee quote page.** Hours x blended rate x margin, by phase (set-up, modelling, QA, sheets),
   with a delivery date from the team's current load, the assumptions listed, the exclusions listed, and
   the open questions to send back to the client. One page, PDF, your logo.

**The wow 1: the client's own drawing becomes the quote.** In the first call, drop the drawing on the
screen: *"I count 312 columns, 640 beams, 9 levels, 4 of them typical. Likely 410 hours, range 340-520.
Three things in your drawing will change the price"*, each with a click-to-zoom pin. It builds trust
before any proposal is written.

**The wow 2: "what if" scope sliders.** Toggle *include basement*, *LOD 300 vs 350*, *with rebar*, *with
sheets*; the hours and fee change live. The client sees why rebar detailing costs what it costs, which
ends many negotiation rounds.

**The wow 3: it learns your team.** After each job, Model Pulse's real hours (or a simple "actual hours"
box) are compared with the estimate. Minutes-per-element are adjusted from that history, per modeller
level and per project type, and the quote page shows a plain "our estimates have been within 12% on the
last 8 jobs" line (only once enough history exists; not before). A wrong early estimate becomes the
cheapest training data the team has.

**The wow 4: a scope contract.** The counted scope (marks and counts) is saved with the quote. When the
client later says "we also wanted the podium", the tool shows the *scope change* against the saved count
and prices the delta, the same idea as Issue Desk's change catcher, but for money.

**Who uses it, when.**
- **BIM manager / proposal lead:** every enquiry; minutes instead of hours.
- **Sales / founder:** live in the first meeting.
- **Modeller:** receives the counted scope and the flagged risks as a start checklist.
- **Project lead:** compares planned and actual hours at the end of each job.
- **Client:** sees a quote with visible evidence rather than a lump sum.

### Implementation

**Stage 1: counts (3-4 weeks).**
- Reuse the CAD engine's reading and the pipeline's `analyze` result in `packages/cad-engine` and
  `packages/pipeline`: they already produce element counts for the CH template (the v1 counts equal
  template §12). Add a read-only `scope_summary()` that returns counts by kind, level and mark, plus
  "not found" and "uncertain" lists. No change to v1 `Entity` or `analyze`; v2 gets its own summary in
  `v2/`.
- UI: a table plus the existing viewer for click-to-zoom. Design-system tokens only.
- Test with a scratch folder of real drawings; none go in git.

**Stage 2: complexity flags and hours (3-4 weeks).**
- Flags are plain rules over the same data: repeated level fingerprints (hash of mark/position sets),
  non-orthogonal members, level differences, layer-name health, missing schedule tables, text-only
  dimensions. Each rule returns evidence (entity ids) so the pin has somewhere to point.
- Estimate model: a small editable table `kind -> minutes`, factors `LOD`, `with rebar`, `with sheets`,
  `drawing quality`. Range from the spread of past actuals; before history exists, a fixed +/-25% band,
  labelled as a guess.
- Store as a `scope.json` inside the project file; versioned with the project.

**Stage 3: quote page and learning (2-3 weeks).**
- Quote PDF with the same renderer used by the other report ideas; rate card, margin, tax and terms are
  settings the firm edits once.
- Calibration: read actual hours from Model Pulse or a manual entry; adjust `minutes` by a simple
  per-kind ratio with a floor and cap, and never change a number from fewer than 3 jobs.
- Scope-change view: diff two `scope.json` files and price the delta.

**Later.** PDF and scanned-drawing input via Plan Tracer; an IFC/Revit input for "we already have a
model, quote the changes"; a team capacity view so the delivery date comes from real availability; an
MCP tool so an AI agent can draft the covering email from the scope sheet.

**Risks and honesty notes.**
- A count is only as good as the drawing. The tool must show "not found" and "uncertain" loudly and never
  fill a gap with a guess.
- Minutes-per-element differ widely between teams; the starter values are placeholders. The estimate is a
  decision aid, not a promise.
- Drawings are confidential: counting runs in the browser, nothing is uploaded (cad2bim's rule: files stay
  on the device).

### Future update

- **Template library by building type** (residential tower, warehouse, hospital) with typical hours, so
  a rough quote works from a plot area and floor count before any drawing arrives.
- **Win/loss tracker:** quote vs. job won, so the margin and discount that actually win work become
  visible.
- **Scope-to-schedule:** the hours feed a delivery plan across the team (with Crew Curve's levelling idea
  applied to modellers).
- **Client portal link:** a read-only page of the counted scope with approve/comment, stored as a BCF-style
  record.
- **Cross-discipline scope:** architecture, MEP and rebar scopes counted together for one package price.
