# 20261121 Idea: Neighbour Shield — Before the dig, piling or demolition starts, the buildings next door are recorded in 3D (photos and cracks pinned to their walls); during the work, crack-gauge and vibration readings are typed or pasted in and light up amber/red against limits; afterwards, a "who moved first" evidence pack answers the neighbour's complaint.

## Entry 1

```
date-time:   2026-11-21 (scheduled run, UTC)
idea name:   Neighbour Shield (pre-condition record + crack/vibration watch + dispute evidence pack
             for adjacent buildings; runs in the browser, nothing uploaded)
potential:   High for contractors, PMC and BIM teams on urban basement, piling and demolition jobs
             in India; medium-high for the toolkit, because it reuses the site-context, dig and
             demolition work already planned and adds a part nobody models: the neighbour.
             The pain: when excavation, piling or demolition starts next to an old building, the
             first crack the neighbour finds becomes "your fault". The record that protects the
             site team (a dated condition survey) is usually a PDF of photos on a shared drive,
             not tied to a wall, a date or a reading. Work stops for days while both sides argue.
             Time saved: NOT measured.
             First test: take 1 real urban basement or piling job (kept out of git). Time how long
             it takes to (a) assemble the pre-dig photo record by hand and (b) answer one mock
             complaint "this crack is new" from files. Repeat with Neighbour Shield. Compare minutes
             and whether the answer is backed by a dated photo on the same wall. Do not quote hours
             until this is done.
             Market check (2026-11-21), two standard web searches:
               - Found: dilapidation / pre-construction condition surveys are standard practice;
                 the idea of a before-and-after comparison is documented (UK construction-damage
                 wiki, Intertek, Applus+ pages). News and court records in India show the dispute
                 itself is real: cracks in adjoining buildings during basement digging (Tribune,
                 Amritsar), deep excavation putting houses at risk (Tribune, Mandi), and court cases
                 asking whether damage came from the neighbouring construction (Delhi High Court
                 records; Bombay High Court on access for a pre-construction survey and monitoring
                 instruments). I read search snippets only, not the full judgments.
               - Found: vibration and crack monitoring is sold as sensors plus alert software
                 (Specto, Svantek, SoftDB, Eyrus, Intertek, Applus+). These are paid services or
                 hardware dashboards.
               - NOT found: a tool that puts the condition photos and the readings on a 3D model of
                 the neighbour's building and checks them against your own dig/piling schedule.
                 "I did not find one", not "none exists".
             Trend: I could not measure LinkedIn or Instagram counts, so no trend is claimed beyond
             the news pieces above. Basement digs and neighbour disputes are recurring local news.
             Extends tools already in this repo (does not repeat them):
               - Dig Deep (excavation, shoring, truck trips): it plans the hole. Neighbour Shield
                 watches what the hole does to the building beside it.
               - Demo Map (brownfield demolition planner): same idea for demolition next door.
               - Pile Pilot: the piling that causes the vibration; its schedule is the clock here.
               - Photo Placer and Snag Sweep: photos placed in 3D and pinned to elements. Reused for
                 the condition record, but dated, locked and tied to a limit.
               - Audit Eye (crack finder): finds cracks in your own structure. This is about
                 someone else's, with a before-baseline.
               - Roadmap "Site context" (terrain and nearby buildings): gives the neighbour boxes
                 this idea fills with evidence.
               - Model Capsule / Delay Detective: the evidence pack and the delay claim pattern.
             The wow: draw (or pull from site context) the neighbour's building as a box. Walk the
             boundary with a phone; each photo lands on the right wall by tap and is locked with a
             date. Then drag the dig/piling date slider: neighbours go green, amber, red by distance
             and activity (a pile rig at 6 m turns the old house amber a week early). Paste a
             crack-gauge reading and the pin on that wall changes colour. A complaint arrives: click
             the wall, and the page shows the dated baseline photo, every reading since, and what
             work happened that day. One click makes the evidence pack.
```

### Description

**The pain.**
- The condition survey is a folder of photos. Which wall, which crack, which date, taken by whom is
  in file names at best.
- Readings (crack tell-tales, vibration, settlement points) live in a separate Excel or a sensor
  vendor's portal, away from the schedule of what the rig was doing that day.
- Nobody tells the site team in advance which neighbour is most at risk from the next activity.
- When a complaint arrives, the answer takes days of digging through files, while the work waits.

**The idea.** Four moves.
1. **Model the neighbour.** A simple box with walls, from site context, a quick sketch, or the
   plot sheet. Not a full model: just walls, floors and the distance to the work.
2. **Record the condition.** Photos placed on walls (Photo Placer), a crack marked as a line with
   width typed in, the observer and the date. Records are locked after saving: edits become new
   dated entries, never overwrites.
3. **Watch during the work.** Paste or import readings (crack-gauge width, vibration peak velocity,
   settlement). Each point carries a limit set by the user from a standard they choose (for
   example a national or foreign vibration code); the tool shows the limit's name and does not pick
   one. Colours: below warning, above warning, above alarm. A "what-if" slider uses the 4D dates
   to show which neighbours the next activities approach.
4. **Answer the complaint.** Click a wall: baseline photos, later readings, the work done within a
   few days (from the 4D tasks), and the distance from the rig or the dig edge. Output: a dated
   evidence pack (PDF plus Excel).

**What a user sees.**
- A "Neighbour Shield" panel: neighbour list with a status colour, a date slider, a reading
  table, and a "log a complaint" button.
- 3D: the neighbour boxes on the plot edge, walls coloured by status, pins for photos and readings.
- The pack: baseline photos per wall, a reading chart per point against its limits, a timeline of
  site activity, and the plain line "record, not a structural opinion".

### Implementation

Inside the cad2bim app; nothing leaves the device. The tool never decides who is at fault.
- **Neighbour geometry:** simple IFC walls and slabs, written as an extra context discipline in
  the .c2b (the multi-discipline work on the roadmap), or a lightweight overlay if that is not
  ready. Distances to the dig/pile footprint are plain geometry.
- **Evidence records:** append-only tables in the project file: photo (hash, date, wall id,
  observer), crack (line, width, date), reading (point, value, unit, date). The hash and date make
  later edits visible. This is a record, not a legal proof; the pack says so.
- **Limits:** a user-edited table (name, unit, warning, alarm, source note). No built-in claim that a
  limit suits a given building; starter rows are copied from public codes and marked "check".
- **Schedule link:** the 4D tasks (roadmap) give "what happened that day"; until then, an Excel
  daily log read by the Sheet Sync reader.
- **Reading import:** CSV paste or file with a point id, date and value; nothing connects to a
  sensor network in version 1.
- **Output:** PDF and Excel through existing export code; 3D colours through viewer overrides.
- **Agent access:** MCP tools in packages/mcp: "which neighbours are above warning", "what happened
  near the north wall on 12 March", "make the pack for building B".
- **Build order:** (1) neighbour boxes, locked photo and crack records; (2) limits and CSV
  readings with colours; (3) distance and schedule view; (4) complaint view and pack; (5) MCP.
- **Risks:** a structural engineer must set limits and interpret readings; the tool must not
  suggest it replaces an independent surveyor; photo dates from phones can be wrong, so the
  observer's own date and the file date are both shown; legal weight differs by court. Check with a
  geotechnical consultant and the project's lawyer before release.
- **Check before release:** 1 real job, the stopwatch test above, and a mock complaint reviewed by
  someone who has handled a real one.

### Future update

- **Sensor feeds:** read common vibration or tilt sensors' exports on a schedule (user-owned
  data only), same colours.
- **Phone-guided survey:** an on-phone walk-round that asks for each wall in order and flags
  missing sides before the first rig mobilises.
- **Auto crack widening alert:** a gentle trend line per crack and a warning before the alarm.
- **Insurance pack:** the same record laid out for a contractor's all-risk insurer.
- **Neighbour-friendly view:** a read-only Model Capsule for the neighbour showing the plan and
  the monitoring schedule, if the developer chooses to share it.
- **Reuse:** a saved neighbour-risk template per city or soil type from the team's own jobs.

Sources used for the market check (search snippets only, not verified beyond these pages):
[Tribune, cracks in adjoining buildings while digging hotel basement](https://www.tribuneindia.com/news/amritsar/cracks-appear-in-adjoining-buildings-while-digging-hotel-basement-323260),
[Tribune, deep excavation puts houses in Mandi at risk](https://www.tribuneindia.com/news/himachal/deep-excavation-by-builder-puts-houses-in-mandi-at-risk-of-collapsing/amp),
[Designing Buildings, damage caused by construction works](https://www.designingbuildings.co.uk/wiki/Damage_caused_by_construction_works),
[Delhi High Court judgment (RFA 528/2022)](https://delhihighcourt.nic.in/app/showFileJudgment/NBK10042026RFA5282022_173308.pdf),
[Intertek, construction vibration monitoring](https://www.intertek.com/blog/2026/09-25-Construction-Vibration-Monitoring/),
[Applus+, vibration monitoring services](https://www.applus.com/ae/en/what-we-do/service-sheet/vibration-monitoring-services),
[Eyrus, vibration monitoring equipment guide](https://eyrus.com/guide-to-construction-vibration-monitoring-equipment),
[Specto, construction vibration monitoring](https://www.spectotechnology.com/specto-academy/vibration-academy/construction-vibration-monitoring/).
