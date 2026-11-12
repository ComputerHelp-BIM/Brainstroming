# 20261112 Idea: Raft Run — the big raft or transfer-slab pour planned from the 3D model: truck and pump roster, a cold-joint clock, a heat-of-hydration check (peak and gradient), thermocouple and cooling plan, and one live board for pour night

## Entry 1

```
date-time:   2026-11-12 (scheduled run)
idea name:   Raft Run (mass-concrete pour planner for rafts, pile caps and transfer slabs: pour front and
             layer plan from the 3D element, plant/truck/pump roster against the placing rate, cold-joint
             time limit per layer, heat-of-hydration check of peak temperature and core-to-surface
             difference, mix and ice/chilled-water options, thermocouple points, curing plan, and a
             pour-night board that re-plans when a truck is late)
potential:   High, and it is in the news now. Record continuous raft pours keep going round LinkedIn and the
             trade press: about 24,100 m3 in 54 hours at the Umiya Dham temple raft near Ahmedabad (2025,
             reported with 26 RMC plants, 285+ transit mixers and 600+ workers, "without a single cold
             joint"), and 6,000 m3 in 50 hours for a Hyderabad tower raft (Sumadhura, January 2025).
             Every one of these is planned by a few people with Excel, a phone and experience. The 3D
             model already holds the raft's exact volume, shape, rebar and levels. Nothing here turns that
             into the plan.
             The wow: select the raft. A "pour front" sweeps across it in layers while trucks stream in on a
             timeline, and a temperature heatmap of the core rises and falls on the section. Drag the
             start time to night, add ice, or drop a pump, and the red "too hot" and "cold joint" zones
             move live. A 100-second reel of it is the post a contractor wants to show the client.
```

### Description

**The pain.**
- A big raft is one pour, planned once, and a mistake cannot be repaired. The checks that matter are
  trucks per hour, pump reach and standby, how long the previous layer can wait (initial set), and how hot
  the middle gets. They are done by hand, from different sheets.
- Volume is quoted from a quantity sheet that may be a revision old. Raft level changes, a lift pit or a
  thickened strip, and the order of 6,000 m3 is wrong by a few hundred (the size of the error is an
  estimate, not measured here).
- Heat is the quiet risk. A 2 to 3 m thick raft in an Indian summer can pass the common limits (often
  quoted as a peak near 70 °C and a core-to-surface difference near 20 °C; the project specification
  decides, and the tool takes its own numbers). The usual answer is "ice in the mix" and some
  thermocouples, chosen by feel.
- The site team works the pour night on WhatsApp. The plan on paper does not know a truck broke down.

**What exists today (checked 2026-11-12 from memory and a quick search; not exhaustive).**
- Finite-element packages (midas Civil, ANSYS, Abaqus) can model hydration heat. They are paid, slow to set
  up, and used by specialists on the biggest jobs. ConcreteWorks (US research tool) is free but desktop,
  US mixes and codes, no 3D model, no truck plan.
- Pour planning in 4D tools (Navisworks, Synchro, Bentley) animates a pour as a date. None of them
  checks truck arrivals against placing rate, or a cold-joint time.
- RMC suppliers give a delivery plan. They do not see the raft's shape or the pump positions.
- In this repo: Pour Planner (4D RMC formwork cycle) splits slabs by volume and writes an RMC order. Joint
  Judge places construction joints. Cube Clock reads cube results and site temperature. Crane Canvas does
  the lifting side. Basement Box makes the raft and its rebar. **No idea here plans a single big pour as a
  continuous operation, or checks its heat.** Raft Run sits between those: it takes the raft Basement Box or
  Pile Pilot made, and gives Pour Planner a tested plan.

**The idea in one line.** Pick a raft or transfer slab; get a layered pour plan, a truck and pump roster,
a cold-joint clock, a heat check, and a board to run it on the night.

### How it works (user view)

1. **Pick the element(s)** in the cad2bim viewer (the raft, or a group of pile caps or a transfer slab).
   Volume, area, thickness by zone, rebar mass and levels are read from the IFC. The rebar mass matters: it
   takes the place of concrete and is shown in the order, so the order is net, not gross.
2. **Set the mix and the rules** (all editable, all shown):
   - grade, cement type and content, fly ash / GGBS replacement, w/c, initial and final set time at the
     expected temperature, slump retention;
   - placing temperature limit, peak and gradient limits from the spec (default: the common figures above,
     marked "check your spec").
3. **Plan the pour front.** The tool cuts the raft into layers (default 300 to 500 mm) and a sequence
   (steps, or one slope, from one end). Rule: each fresh layer must cover the previous one before it
   sets. It computes the minimum placing rate per hour for that, from layer area and set time.
4. **Roster.** Needed trucks per hour from the placing rate and the round trip (plant to site, from a
   distance and a traffic factor you enter), plants and their capacity, and pumps: boom pumps and line pumps
   with reach and output, positions on the site plan, cleaning and changeover, and **standby**. A pump
   output below the needed rate is a red bar, with the fix (add a pump, or slow the front and use a retarder).
5. **Cold-joint clock.** A moving timer per layer: "layer 4, last concrete at 02:10, initial set 04:30 at 31 °C,
   next layer due before 04:00". If the roster cannot make it, the tool says where the first likely cold
   joint falls on the 3D model, and proposes a planned stop-end (hands over to Joint Judge).
6. **Heat check.** A fast 1D / 2D estimate of the temperature through the thickness (heat from cement
   content, placing temperature, ambient, form and cover insulation, curing). It shows a core curve, a
   surface curve, the difference, and the day the difference drops below the limit. Levers: ice or chilled
   water in the mix (temperature drop per % ice), night pour, blanket or water-pond curing, lower cement
   with fly ash or GGBS, and splitting the raft in two lifts. Each lever shows its effect and its cost
   from the BOQ rates.
7. **Thermocouple and curing plan.** Suggested sensor points on a plan and section (core, 50 mm from the
   top, 50 mm from the side, one at the bottom), numbered, with a log sheet to fill (or a CSV from the
   data-logger). Curing and blanket quantities go to the BOQ.
8. **Pour-night board.** One web page, phone-friendly, no install: truck arrivals (type the ticket number
   or scan the batch ticket QR), m3 poured against plan, layers done, pump status, temperatures from the
   logger CSV. A late truck re-plans the next few hours and warns "layer 5 will cold-joint at 04:20".
9. **Reel and report.** The pour animation (front, trucks, heat) as an MP4 for the client or the group, and
   a signed-off PDF: plan, roster, assumptions, limits, the sensor map. After the pour the log is attached
   and the report is the record for the file and the engineer's acceptance.

### Why it is practical for a BIM modeller / team

- The BIM team already owns the raft model. This turns that model into the thing the site team asks for
  in the week before the pour, so BIM is seen as the pour partner and not as the drawing office.
- It re-runs in seconds when the raft changes. The roster and the heat check follow the model.
- It cuts the week of Excel and phone calls the planner spends. (A guess: about 2 to 3 days saved for a big
  raft; not measured. A first project would show the true number.)

### Build notes (inside cad2bim)

- **Reads:** IFC solids and rebar mass (existing viewer and QA), BOQ rates (existing), levels, the 4D
  schedule when it exists.
- **Runs on the device,** like the rest of cad2bim: the plan, the heat model and the board data stay in the
  browser and in the project file. The pour-night board has no server; a second phone joins from a shared
  link in the same way Room Link does (peer to peer), or it exports a CSV.
- **Heat model:** an adiabatic temperature rise curve from cement content and type (a published form, with the
  constants named in the panel), explicit finite differences through the thickness, daily ambient curve.
  Written in Rust or TypeScript, no GPL code. It is an **estimate**, not a design check; the report says so
  and asks for sensors. It is calibrated on the thermocouple log of the first real pour (below).
- **Truck model:** a small discrete-event simulation (arrivals, travel time spread, unloading, pump
  hold-up). Fifty lines of logic, thousands of runs in a second, gives a chance of a cold joint, not only
  a single answer. That "chance" number is the useful new thing.
- **Licences:** only MIT, Apache or BSD maths and charting; no copied code from the research tools named above.

### Future updates

- **Calibrate from real logs.** Each pour's thermocouple CSV improves the cement-rise constants for that
  supplier's mix; after a few pours the estimate is the team's own.
- **Weather feed.** Pull the forecast for the pour date (an opt-in call, the only thing that goes out;
  nothing from the model is sent). Suggest the night with the smallest temperature rise.
- **Plant and traffic.** Plug in plant locations; a map gives the round trip; flag a plant that is more than
  an hour away.
- **Other mass pours:** dams' spillway blocks are not our market, but pile caps, lift pits, transfer girders
  and thick core walls use the same engine.
- **Link with other ideas.** Joint Judge for planned stops; Cube Clock for strike and cube results;
  Order Clock for ice, cement and admixture orders; Delay Detective for a record of the night; Model Reel
  for the post.

### Risks and honest limits

- A fast 1D/2D model is not a full thermal-stress analysis. Large or irregular rafts, or ones with a
  tight spec, still need a specialist. Raft Run says so when the thickness, size or limits go past its
  range, and exports its inputs for one.
- Mix constants vary by plant. Defaults are marked as defaults.
- The claims above about records and competing tools come from a quick search and memory on 2026-11-12;
  they should be re-checked before they go on a public page.

### Sources (trend)

- Umiya Dham raft, Ahmedabad: <https://www.constructionworld.in/policy-updates-and-economic-news/psp-projects-sets-world-record-with-umiya-dham-foundation/79013>
- Sumadhura continuous pour, Hyderabad: <https://www.nbmcw.com/news/sumadhura-group-sets-record-for-fastest-concrete-pour-in-hyderabads-real-estate.html>
