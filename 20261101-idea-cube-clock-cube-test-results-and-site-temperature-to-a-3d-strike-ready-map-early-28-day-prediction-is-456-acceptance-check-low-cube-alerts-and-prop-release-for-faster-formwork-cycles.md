# 20261101 Idea: Cube Clock — type the cube results once and the 3D model shows which slabs and beams can be struck today, predicts the 28-day strength from the 3-day and 7-day cubes, checks IS 456 acceptance, and pins low-cube alerts to the pour

## Entry 2

```
date-time:   2026-11-01 (scheduled run, UTC)
idea name:   Cube Clock (Cube Results + Site Temperature -> Strike-Ready Map, Early Strength Prediction, Acceptance Check)
potential:   High. Every cast-in-situ site in India cubes every pour, and the results live in a lab register,
             a PDF from the testing lab, a WhatsApp photo and the QA engineer's Excel. The model never sees
             them. The two daily questions, "can we strike this slab?" and "did this pour pass?", are answered
             from memory and a fixed day-count table. Striking late costs a floor cycle (props and shuttering
             tied up, the whole 4D slips). Striking early is a safety risk. A tool that ties the cube and
             temperature data to the element on screen helps both. Estimated saving (an estimate, not measured):
             0.5 to 1 day on a typical floor cycle where props are held "to be safe", and the ten minutes a day
             a QA engineer spends hunting for "which pour was that cube from".
```

### Description

**The pain.**
- Formwork is struck on a fixed number of days (soffit 7 or 14 days, props 14 or 21, from the usual code table),
  whatever the weather. In a hot week the concrete is ready sooner. In a cold or monsoon week it may not be.
  Either the site waits for no reason, or it strikes on the calendar and hopes.
- Cube results arrive as separate sheets: 3 cubes at 7 days, 3 at 28 days, per pour. Matching them to
  "L5 slab, grid A-D / 1-4" depends on a hand-written cube ID. When a 7-day result comes back low, the question
  "which elements does this affect?" costs half an hour of looking through records.
- A failed 28-day cube is found 28 days after the pour, when the floors above may already be cast. The
  early warning that a 7-day cube gives is read by eye and rarely written down.
- Concrete strength and quality control are steady discussion topics on LinkedIn and Instagram site-reel
  pages (cube failures, early striking collapses, "smart concrete sensors"). Read this as an impression of
  trends, not a measured statistic.

**What exists today (a thin check, to be re-verified before any "nothing like it" claim).**
- Wireless maturity sensors with apps exist (for example Giatec SmartRock, Converge and Maturix; I did not
  check their current prices). They need hardware per pour and a subscription, and the data sits in their
  cloud. Desktop QA registers and Excel templates track cube results, but none that I know of links them to an
  IFC element in a 3D view, uses the free weather history, and runs in the browser with the file kept on the
  device.
- Earlier ideas in this repo touch the topic from the side: Pour Planner counts the cubes per pour (IS 456
  sampling rule), Shutter Plan sets a strike day, Ready Board checks readiness, Audit Eye plans NDT. None of
  them takes the results back in.

**The idea.** A **Cube** panel in cad2bim, in the 4D and QA workspaces. One pour = a set of elements (the
existing selection, or the Pour Planner's pour). For that pour the person types or pastes:
1. **Cube results** (Excel paste or CSV from the testing lab): age (3, 7, 28 days) and N/mm2 for each cube.
2. **Temperature**: pick the site on a small map or type the town. The tool reads the hourly air temperature
   since the pour. Only the location is sent, never the model (the same rule as Site Context). Without a
   connection, type the day's min and max, or load the CSV from a cheap logger left on site.
3. The pour's **mix grade** (M25, M30, ...) and cement type, if the model does not already know them.

**What the person sees.**

- **Wow 1: the strike-ready map.** The 3D model is coloured by state: *wait* (red), *close, check tomorrow*
  (amber), *ready by strength* (green), *struck* (grey). Click a slab: *"L5 slab, cast Tue 06:00. Maturity says
  about 68 percent of the 28-day strength now. The code day-count would let you strike on Tue; the data says
  Sat."* Click a green slab: *"props can be released to the next level's re-propping plan."* The decision stays
  with the engineer: the tool shows its working and never says "safe", only "meets the strength you set".
- **Wow 2: early 28-day prediction.** A 3-day or 7-day result is projected to 28 days from the mix's own
  strength-gain curve, which the tool keeps learning from the team's past pours. A 7-day cube far below
  expectation raises an alert on day 7, not day 28: *"L7 columns, grid C-F: 7-day mean 14.1 N/mm2. Expected
  about 20 for M30. Request a rebound test or an extra cube now?"* The alert is pinned to the elements, with
  the cube IDs and photos.
- **Wow 3: pass or fail on one page.** The tool applies the acceptance rule of IS 456 (the group-of-four mean
  and the single-cube minimum, with the project's established standard deviation), shows the running
  statistics per grade and per month, and marks each pour *pass*, *fail*, or *waiting*. For a fail it lists the
  elements, the floors cast above them since, and the next steps the code gives (core test, load test), as a
  checklist for the engineer.
- **Wow 4: a card for WhatsApp.** "Today's strike list": one line per element, with the data that backs it.
  The site engineer taps *Struck* and the 4D, Ready Board and Shutter Plan update.

**Maturity, in plain words.** Concrete gains strength with time and warmth. The usual way to account for both
is the Nurse-Saul maturity: add up (temperature minus a datum temperature) times hours. A mix has a curve that
maps maturity to strength. The team calibrates it once from their own cubes (the 3, 7, 28 day results at the
logged temperatures), so no lab trial is needed to start. Defaults are *placeholders to be verified*:
the datum temperature (ASTM C1074 gives a default I recall as -10 degrees C, to be checked) and the starting
curve. Air temperature is not concrete temperature, especially in thick pours (heat of hydration) and in
shade. The first version says so on the screen and lets the person add a correction. A data logger buried in
the pour is the better source, and the tool accepts its CSV.

**Who uses it, when.**
- **QA/QC engineer:** every pour, every day. Replaces the cube register and the "which pour?" search.
- **Site engineer / planner:** the strike list each morning, props release and the 3-week lookahead.
- **BIM modeller:** sets up the pours once (the Pour Planner already does), and the rest follows.
- **Structural consultant / PMC:** a ready evidence pack for a doubtful pour, with the exact elements named.
- **Contractor director:** fewer idle props, and a cleaner defence if a cube result is later disputed.

### Implementation

**Stage 1: cube register tied to the model (3 weeks).**
- Data model in the project file: `Pour { id, elementIds, castAt, grade, cementType }`, `Cube { pourId, ageDays,
  strengthMPa, labId, photo? }`. Elements come from the existing selection and the IFC GlobalIds, so the link
  survives revisions (the same id rule as in `docs/design/project-versioning.md`).
- Excel and CSV paste in the browser (file stays on the device). A column mapper remembers each testing lab's
  layout, in the same spirit as the Layer Whisperer's profile idea.
- Colour-by and a property panel on the existing engine's "colour by" mechanism, so no new viewer code for the
  first release.
- IS 456 acceptance check as a small pure function with unit tests: input a list of results and the grade,
  output pass or fail and the margin. The clause numbers and limits are written into the tool as an editable
  table, with a visible note "verify against the current code and the project specification".

**Stage 2: temperature and maturity (3 weeks).**
- Temperature sources, in order: a logger CSV (timestamp, degrees C), the typed daily min and max (a sine
  curve between them), and a hourly weather history for the site's lat/lon.
- Weather source: the cad2bim rule is that nothing leaves the device unasked, so this is a button that sends
  only the coordinates and shows what it will send first. Check the licence of the weather API before
  shipping (some free APIs allow only non-commercial use, and cad2bim is commercial; there is a public
  Indian source, the IMD, and paid options). Make the source swappable.
- Nurse-Saul maturity per hour per pour, in a small worker so the UI does not stall. Calibration: least-squares
  fit of strength against log maturity from the team's own cubes, with a clear "not enough data" state until
  there are 3 or more pours of the same grade.

**Stage 3: strike-ready map and alerts (2-3 weeks).**
- State machine per element: *wait, close, ready, struck*. A person-set rule decides *ready*: required
  strength as a percentage of the grade, or "twice the stress it will carry" from the structural model's
  design stress, if the project has one (the code allows both; the person picks).
- Alerts on the QA panel and in the 4D/Ready Board (a task cannot be marked *ready* while its precondition
  pour is red).
- Strike card for WhatsApp: a rendered image and a text, shared by the Web Share API. Nothing is uploaded.

**Stage 4: team learning (2 weeks).**
- Keep per-site, per-mix curves and show the *prediction error* of past pours, so trust is earned in public. A
  curve that mispredicted by more than the set tolerance is not used for the strike map until re-calibrated.

**Risk and honesty.**
- A wrong "ready" is a safety issue. Mitigations: the tool only recommends, a registered engineer signs the
  strike decision (a field for name and time is stored with it), a conservative default (use the lower of the
  code day-count and the data), and the screen states what is estimated and what was measured.
- Maturity from air temperature is an approximation. Never present it as a lab result.
- The code table values and the acceptance limits are typed from memory of IS 456:2000; they must be checked
  against the current standard and the project specification before any release.

**Fit with the repo.**
- `packages/engine`: the QA and "colour by" parts, the BBS/BOQ takeoff for the pour's concrete volume (the
  cube count rule uses it).
- `apps/web`: a new panel on the design system's tokens and components; the 4D timeline (roadmap, "Building
  now / Next") for the strike dates.
- `packages/mcp`: add `get_pour_status` and `add_cube_result` so an AI agent (or a WhatsApp bot in the Chat to
  Model idea) can fill the register from a photo of the lab report.

### Future update

- **Photo of the lab report to cube rows.** OCR on the testing lab's PDF or photo, matched to the pour by the
  cube ID or the date. Verified by a person on screen before saving.
- **Sensor-ready.** Import from open low-cost loggers (a DS18B20 probe on a small board costs a few hundred
  rupees; I did not check exact prices) and a one-page guide to build one. Later: direct Bluetooth readout from
  the browser.
- **Rebound hammer and UPV tests.** Add NDT readings to the pour, with the correlation to cube strength
  learned from the team's data. Joins Audit Eye for doubtful pours.
- **Curing log.** Water-curing days, curing compound, and a daily photo check. A pour with missed curing gets a
  lower predicted strength and a visible flag.
- **Props re-use plan.** Combine with Shutter Plan: as green zones appear, produce the move-up list (which
  props go to which floor tomorrow) and the idle-props count.
- **Cold and hot weather alerts.** A forecast-based warning before a pour: *"39 degrees C and wind on Saturday,
  plan early morning pour, add curing crew"*. Ties to Pour Planner and Order Clock.
- **Supplier scorecard.** RMC plant by plant: average cube margin, failed-cube rate, slump compliance, delivery
  delay. Useful in the next tender.
- **Regional codes.** ACI 318 and ACI 301 striking and acceptance rules, Eurocode 2 / EN 206 conformity
  criteria, as selectable rule packs, so non-Indian teams can use it.
- **Live cross-check with design.** If an ETABS model is linked, compare the required striking strength with
  the actual construction-stage load from the slab above (the shoring/reshoring load) instead of a flat
  percentage.
