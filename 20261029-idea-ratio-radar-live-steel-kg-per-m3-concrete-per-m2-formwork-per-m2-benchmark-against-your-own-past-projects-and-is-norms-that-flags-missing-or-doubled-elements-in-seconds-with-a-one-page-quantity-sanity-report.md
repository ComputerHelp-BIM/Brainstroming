# 20261029 Idea: Ratio Radar — the "does this model smell right?" meter: steel kg/m³, concrete m³ per m² of floor, formwork per m², all compared with your own past projects, live as you model

## Entry 1

```
date-time:   2026-10-29 (scheduled run, UTC)
idea name:   Ratio Radar (Quantity Sanity Meter). "Every model has a steel-per-cubic-metre. Know yours before the QS does."
potential:   High, and cheap to build. The question every structural lead, QS and client asks in the first
             five minutes of any review is "what is the steel in kg per m³ of concrete, and the concrete in
             m³ per m² of built-up area?" Today nobody can answer it from a model without exporting a BOQ,
             opening Excel and dividing by hand. And the same two numbers are the fastest way to catch the
             worst modelling mistakes: a missing floor of columns, a slab modelled twice, a wall that forgot
             its rebar, a typical floor copied with the wrong beam sizes. A model that is "75 kg/m³ when your
             last six G+9 towers were 105-125" is wrong or special, and either way someone must look at it
             today, not at the bill stage. Ratio Radar turns that habit into a live gauge. Competing tools:
             cost-estimating suites (paid, per seat, mostly cloud) show cost per area; none that I know of
             benchmarks quantities from an IFC against the office's OWN past projects, on the device, and
             points to the floor, the member type and the element that makes the ratio odd.
```

### Description

**The pain (daily life of a BIM modeller, structural lead, QS).**
1. *Silent model errors.* A tower is modelled over three weeks by three people. Nobody notices that level 7's
   shear-wall boundary elements have no rebar, or that a slab was pasted over another one (double volume), until
   the BOQ is compared with a rough estimate, often at tender or at the first bill. Rework found then is 10x the
   cost of rework found when it is modelled.
2. *No quick sense of scale.* A client or partner asks "is this design heavy?" The honest answer is a number the
   office has in its head from old projects, not in any tool.
3. *Design drift.* Between design stages a few "small" changes push steel up 8 percent; no one sees it until the
   tender. Change Radar (this repo) shows what changed; Ratio Radar shows whether the result is still in the
   range the office considers sane.
4. *Cost talk without data.* "Why is your steel higher than theirs?" needs per-floor, per-member-type ratios
   side by side, not a single total.

**Why now (trending).** LinkedIn and Instagram AEC feeds in 2025-26: (1) *embodied carbon and "design for
less material"*, where kg/m² of steel and m³/m² of concrete are THE numbers people post; (2) *AI model checking
and "BIM QA that actually finds something"*; (3) *data-driven early-stage design*, benchmarks instead of
opinions. Most posts are screenshots from Power BI or Excel dashboards that a BIM manager built by hand for one
project. Ratio Radar is that dashboard, automatic, inside the tool, on every model.

**The idea.** A small **Ratio Radar** panel (docked, like the existing QA and BOQ windows). It reads the live
model and shows a gauge row:

| Ratio | Example reading | Band from your past projects |
|---|---|---|
| Concrete per built-up area | 0.34 m³/m² | 0.30 - 0.38 (green) |
| Steel per concrete | 118 kg/m³ | 95 - 125 (green) |
| Steel per built-up area | 40 kg/m² | 33 - 46 (green) |
| Formwork per concrete | 8.2 m²/m³ | 6.5 - 9.0 (green) |
| Columns concrete share | 14 % | 10 - 17 (green) |
| Beams / slab / walls share | ... | ... |

Each gauge is a needle on a band: green inside the range of the office's past projects, amber at the edge, red
outside. Click any gauge and it **breaks down**: by floor (a bar chart where level 7 sticks out), by member type,
by grade and by diameter. Click a bar and the viewer isolates those elements.

Three things make it more than a dashboard:

1. **The benchmark is the office's own.** A folder of finished projects (`.c2b` files, or just their saved ratio
   cards, a few kilobytes each) builds the bands automatically: "G+9 residential, 2 basements, Mumbai: 6
   projects". The user can also pick the IS-practice rule-of-thumb bands that ship with the tool as a starting
   point (clearly labelled "rule of thumb, not a design check"). New office, no history: the shipped bands are
   used, and every finished project adds to the office bands with one click.
2. **It explains the outlier.** Not just "steel is low". It runs a short list of "smell" checks and says what is
   probably behind the number:
   - **Floors that break the pattern:** "Level 7 concrete is 41 % of level 6's; same plan area. Probably
     missing: 14 columns (compare marks) or slab S7 not modelled."
   - **Doubled elements:** two slabs or beams with the same bounds on one level (volume counted twice).
   - **Rebar gap:** concrete elements with no reinforcement attached, grouped by type ("38 walls, all on levels 5-9").
   - **Odd density:** a type whose kg/m³ is far from its own peers (beam B14 at 40 kg/m³ while the other
     400x700 beams are 150).
   - **Typical floor drift:** floors marked "typical" whose quantities differ by more than a tolerance.
3. **It watches over time.** A small timeline records the ratios at every Save (or every version, when
   Versions and history ships). "Steel/m³ went 112 to 129 after Tuesday's change" with the change named. The
   gauge becomes a design-drift alarm for a team lead, and the timeline is the evidence in a "why did the bill go up"
   talk.

**Outputs.**
- **One-page Quantity Sanity Report (PDF):** gauges, per-floor chart, the list of suspected issues with the
  element ids, and a signature line ("checked by"). Takes the place of the ad-hoc "quick check" that goes in the
  design review folder today.
- **Excel ratio card:** one row per project for the office's own benchmark sheet (ready to paste into the
  tender-cost database).
- **Share link/QR-free summary text** for the WhatsApp group: "G+9 Tower A: concrete 0.36 m³/m², steel 121 kg/m³.
  Within band. 2 notes."
- **BCF/RFI-style findings:** every suspected issue as a finding in the existing actionable-QA shape, so it
  appears in the same QA list and in BCF export.

**The wow moment (a 30-second reel).** Open a 24-floor tower model. Eight gauges glow on screen, seven green,
one pulsing red: "Steel / concrete: 71 kg/m³ (your band 98-124)". Tap the gauge: a floor-by-floor bar chart
drops, one bar (level 7) is half height. Tap the bar: the viewer isolates level 7 and 14 columns blink amber:
"no longitudinal bars". One click: **Select in Revit** or **Fix: add rebar from BBS**. The needle swings back to
green. A QS who watches this thinks of every tender he was handed a model that was "70 percent done"
and nobody knew. Total time: under a minute.

**Why this is unique / extends what is already here.**
- cad2bim already has the BOQ (with rates), BBS, QA findings, mark/grade rules, colour-by-parameter and heat
  maps by calculated value (quick-wins B8 in the repo's own design notes: "heat map by a calculated value (concrete
  per bay, steel ratio)"). Ratio Radar is the *benchmarked, explained, time-tracked* layer over all of those:
  none of them says what a number should be.
- Nearby ideas in Brainstroming each use one ratio for one job: Carbon Lens (kg CO2), Mark Merge (steel penalty),
  Rate Rocket (cost bands), Load Path X-Ray (axial ratio), Model Pulse (timesheet benchmark). None is a general
  quantity-sanity meter whose bands come from the office's own finished work. Ratio Radar can feed them: the same
  ratio card becomes the "before" for Carbon Lens and Rate Rocket.
- Nothing leaves the device: the benchmark is a small file the user keeps. Paid cost-modelling products compare
  against their own cloud data; this one is yours.

**Day-to-day use cases.**
- Every Friday: the BIM lead opens the model, glances at the gauges. Green means go home; red means the Monday
  to-do is written for them.
- Before sending a model to the QS or the client: print the one-page report; it answers the first five
  questions before they are asked.
- Design stage changes: re-run after each revision; the timeline shows the quantity effect of the change in
  steel kg and rupees (with Rate Rocket rates, if present).
- Tender comparison: load a competitor's IFC (or the architect's concept model with structure) and see whether
  its ratios are plausible.
- Onboarding: a junior modeller sees right away that "missing rebar" shows up as a red gauge, a quiet teacher.
- Freelancers: add "ratio-checked" to the deliverable as a mark of quality.

### Implementation

Fits the architecture: the browser, a worker, no server, files stay on the device. The cad2bim repo was read for
reference only; nothing was changed there.

1. **Inputs (all exist):** element volumes, areas and categories from the engine's IFC reading
   (`packages/engine`, web-ifc); rebar mass from the BBS engine (`packages/engine/src/bbs/`); formwork area from
   the BOQ's face areas; built-up area from slab/floor area per level (or a user-entered area when the model
   has no slabs yet; the panel says which it used). Everything is read from the live model ("one model", 0.80.0),
   so edits move the needles.
2. **Ratio engine (new, TypeScript in `packages/engine/src/ratios/`):** a pure function
   `ratios(model) -> { totals, byLevel, byType, byGrade }` with units noted and a clear "not measured" for any
   input that is missing (no rebar in the file = "steel unknown", not zero, so an IFC without rebar does not show
   a false red).
3. **Benchmark store:** a JSON "ratio card" (project type, storeys, basements, region, area, the ratios, date,
   an optional note), saved from any open project by a button, and read from a folder or from the cloud
   projects of a team (the accounts system already exists). The band is the 10th-90th percentile of the
   matching cards (type + storey range), with fewer than 3 cards falling back to the shipped rule-of-thumb band.
   The report prints how many cards back each band, so nobody mistakes a guess for data. Measured and
   estimated are kept apart, as the documentation rule requires.
4. **Smell checks (new, small functions):** per-level pattern comparison using mark and bounds; duplicate-bounds
   finder (a hash of rounded bounding boxes per level); concrete-without-rebar by host grouping from
   geometry proximity, because a Revit IFC carries no relation between a bar and its host (the BBS design note's
   measured finding); peer-density outliers by median absolute deviation. Each returns the actionable-QA finding
   shape so the existing list, select and zoom, and BCF code are reused.
5. **Timeline:** append a row to the project's local history at each Save (ratios only, a few hundred bytes),
   with the version message when one exists. No images are stored.
6. **UI (React, the design system):** a dock panel with a gauge row (SVG, tokens for colour: a green/amber/red
   from the status tokens, not raw values), a breakdown chart (reuse the BOQ chart component), a findings list
   and a "Benchmark" tab to manage cards. Report as PDF through the existing PDF writer. An MCP/Python tool
   `ratios()` so an AI agent or macro can ask "is this model in band?" and file the answer.
7. **Tests:** generated models with known ratios (Vitest), planted errors (a deleted column row, a doubled
   slab, bars removed from a wall type) with expected findings, and a no-rebar IFC that must show "unknown",
   not red. Real client models never go in git.

**Phases.**
- *Phase 1 (about 1 week):* totals and per-level ratios, shipped rule-of-thumb bands, gauges and breakdown, PDF
  one-pager. This alone is the wow demo and is useful the day it ships.
- *Phase 2 (about 1-2 weeks):* ratio cards, office bands, smell checks, findings in the QA list.
- *Phase 3:* timeline across Saves/versions, Excel benchmark sheet, MCP tool, team-shared cards.

**Honest risks.** Ratios depend on how the model is built (are walls in? is the basement counted?), so the panel
shows exactly what went into each number and lets the user switch inclusions. Rule-of-thumb bands must be
written as context, never as a design pass or fail; the report carries the sentence "not a structural design
check". Few past projects mean wide bands, so the card count is always shown. A false red costs trust, so
every finding has a "dismiss with reason" that the project remembers.

### Future update

- **Live design target:** the user sets a target (for example "steel ≤ 105 kg/m³ for the tender") and the
  gauge shows the margin; Mark Merge and Bar Optimizer can then work towards it.
- **Region and cost layer:** with Rate Rocket rates, each gauge also shows rupees per m², and the report gives a
  "range of the bill" before the BOQ is final.
- **Carbon bridge:** the same card feeds Carbon Lens, so embodied carbon per m² appears next to steel per m³.
- **Anonymous benchmark exchange (opt-in):** teams may share ratio cards (no geometry, no names) to a public
  pool, so a small office gets bands from a hundred projects. Nothing leaves the device unless the user presses
  the button, as cad2bim already requires.
- **ETABS comparison:** design-model quantities against the BIM's, using the ETABS import, as a second opinion on
  "is the model complete?" (pairs with Twin Check).
- **Floor-by-floor 4D:** the ratio per pour (Pour Planner) so the site sees the heaviest pours first.
- **"Explain to the client" card:** a plain-language image ("this tower uses 12 % less steel than your last
  one") for LinkedIn and client decks, made from the same data.
