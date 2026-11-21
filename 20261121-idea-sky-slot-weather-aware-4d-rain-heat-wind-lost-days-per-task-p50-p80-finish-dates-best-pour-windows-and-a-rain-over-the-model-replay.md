# 20261121 Idea: Sky Slot — The 4D schedule meets the weather: for each task, the days rain, heat or wind will cost at this site, a P50 / P80 finish date instead of one hopeful date, the best pour windows, and a 3D replay where the rain falls on the model day by day.

## Entry 1

```
date-time:   2026-11-21 (scheduled run, UTC)
idea name:   Sky Slot (weather-aware 4D: lost days per task, P50/P80 finish, pour windows,
             rain-over-the-model replay; in the browser, only the site's location is sent)
potential:   High for planners, contractors' BIM teams and PMC in India (monsoon, summer heat);
             medium-high for the toolkit, because it makes the planned 4D timeline (roadmap
             0.119.0+) more believable with one extra button. Every site team already asks
             "will the rains hit our slab cycle?" and answers it from habit.
             The pain: a 4D schedule is drawn as if every day is a working day. The
             monsoon, summer concreting limits and high wind (crane stop) are added later as one
             flat "15 % buffer" or not at all. The client sees one date; the site misses it.
             Time saved: NOT measured.
             First test: take 2 finished projects (kept out of git). Run Sky Slot with the
             weather that actually happened. Compare its lost days per activity with the site
             diary, and its P50/P80 against the real finish. Do not quote accuracy before this.
             Market check (2026-11-21), two standard web searches:
               - Found: research on weather-aware 4D/5D BIM with AI, mostly frameworks and
                 papers, e.g. a 2026 review-based framework not yet tested in practice; a 2018
                 study that turns historical weather into delay coefficients in spreadsheets;
                 Allplan's blog says weather forecasting in 4D "could" show likely patterns;
                 a vendor article on re-sequencing with 4D BIM before a hurricane (US).
               - Found: 2026 news that the delayed, uneven monsoon hit cement demand and some
                 city water supply to construction sites; a cement-maker blog advises a flat 15 %
                 extra buffer for weather (vendor advice, not a standard).
               - NOT found: a ready tool that gives per-task weather loss, P50/P80 finish and
                 pour windows from a 4D model, for Indian monsoon, in the browser. "I did not
                 find one", not "none exists". Oracle's delay prediction (Primavera side) is
                 based on project data; I did not see weather in it. Paid suites I did not
                 check feature by feature.
             Trend: monsoon and heat delays are live news in Indian construction (above). I could
             not measure LinkedIn or Instagram counts, so no trend there is claimed.
             Extends tools already in this repo (does not repeat them):
               - Pour Planner, Raft Run, Cube Clock: they plan a pour once the day is chosen.
                 Sky Slot chooses which days are good, and tells the whole schedule.
               - Order Clock, Ready Board, Crew Curve, Cash Curve: they read the 4D dates.
                 Sky Slot moves those dates by weather, and they update.
               - Delay Detective: explains delay AFTER it happened. Sky Slot forecasts it BEFORE,
                 and its lost days become evidence later if a claim is made.
               - Rain Run: rain on the terrace of a finished building. Sky Slot is rain on the
                 construction programme.
               - Crane Canvas: crane geometry. Sky Slot adds the wind-stop days it needs.
             The wow: press "Play the year". The 3D model builds day by day as in 4D, but clouds
             and rain move over the site; the slab being cast on a rainy day is outlined red and
             slides to the next dry slot; the finish date drifts from "30 Jun" to a band "P50 12 Jul,
             P80 28 Jul". Drag one pour earlier and the band moves live.
```

### Description

**The pain.**
- A 4D timeline is a straight line. Rain, heat and wind are applied as one flat buffer, if at all.
- Each task reacts differently: excavation and plastering stop in heavy rain; a slab pour needs a
  dry window and a cover plan; steel fixing goes on in light rain; crane lifts stop in high wind;
  summer concreting needs a temperature plan. One buffer is wrong for all of them.
- Client dates are promised as one date. Later, the delay is argued on memory and the site diary.

**The idea.** Four moves.
1. **Pick the site.** The user gives a city or pin (only the location is sent; the model never is).
   The tool pulls daily history for it from a free public weather source and keeps it on the device.
   Final choice of source is made after reading its licence (a commercial product must not
   break a non-commercial-only rule).
2. **Weather rules per task kind (editable).** A small table: "slab pour: stop above X mm rain on the day,
   hold if forecast rain in the next hours; summer: warn above Y °C; crane: stop above Z km/h".
   Starter values are copied from public codes and guides and marked "check with the site's
   engineer"; every output prints the rule set and its version.
3. **Run the schedule through many weather years.** For each historical year, replay the site's
   weather over the 4D tasks, count lost days per task, push the successors along the logic, and
   take the P50 and P80 finish from all the runs. Result: lost days per task, the critical tasks that
   weather hurts most, and a finish band instead of a point.
4. **Suggest, never force.** "Move level-7 slab pour 3 days earlier: 18 % less chance of a wet pour";
   "start waterproofing before the first week of June"; "order the pour cover sheets for these 5
   pours". Each suggestion shows its change on the P80 date. The planner accepts or ignores.

**What a user sees.**
- A "Sky Slot" panel: site pin, rule table, a calendar strip coloured by risk, the finish band.
- 3D replay with weather over the model and risky pours outlined; click a task for its lost-day
  count and the years that caused it.
- Output: a one-page "schedule risk from weather" sheet (PDF) for the client, and an Excel of lost
  days per task. A 14-day look-ahead mode uses the live forecast for the next pours.

### Implementation

Inside the cad2bim app; no model or schedule leaves the device. Only latitude and longitude go to
the weather source, on a button the user presses, in line with "Site context" (roadmap).
- **Base:** the 4D tasks and their links to elements (packages/engine, roadmap 0.119.0+). Until
  that ships, a CSV or MS Project XML schedule read by the same reader that 4D will use.
- **Weather data:** daily rain, max/min temperature, wind for 10-30 past years, cached in the
  browser (IndexedDB) per site. No outside CDN for code. Source licence read before choice.
- **Engine:** a pure function: (tasks, logic, weather year, rules) -> task dates and lost days.
  Run for each year in a worker; P50/P80 from the sorted finish dates. Vitest on small hand-computed
  schedules, including a task that straddles the dry/wet boundary.
- **Rules:** JSON pack per task kind, with a source line and date; editable in the panel.
- **3D:** existing viewer colour overrides for risky pours; rain and cloud as a light overlay,
  switched off for low-end devices. The replay uses the planned camera-moves player.
- **Writes back:** an optional shifted schedule as a new 4D version, never over the original;
  lost days saved as task notes (an `IfcTask` property set) for later claims.
- **Agent access:** MCP tools in packages/mcp: "which tasks lose the most days to rain", "best
  pour window for the level-7 slab in June", "P80 finish if the foundation starts 10 days late".
- **Build order:** (1) weather cache and rule pack; (2) engine on a CSV schedule with a table
  output; (3) P50/P80 band and calendar strip; (4) link to 4D elements and 3D colours; (5) replay
  with weather; (6) suggestions; (7) MCP.
- **Risks:** historical weather does not predict a given year (the tool says "range", not "will
  happen"); El Niño-type years differ from the past, so the user can bias the years used; rules
  are rules of thumb and the site engineer decides; the free weather source's limits and licence;
  small sites have no nearby station, so the grid data is modelled, not measured; says so.
- **Check before release:** the 2 finished projects above, and a site engineer's review of every
  default rule.

### Future update

- **Live site diary:** the actual rain days from the site diary replace history, and the finish
  band narrows as the project goes on (Bayesian-style update, kept simple).
- **Delay Detective link:** when a weather delay happens, attach the day's data and the rule as
  evidence in the time-impact pack.
- **Cost of weather:** with 5D, price the lost days (idle gang, hire of crane and props) and the
  cover and dewatering items, shown in the Cash Curve.
- **Heat and labour:** a heat-stress rule for site gangs, with a shift-timing suggestion.
- **Template for tenders:** a "weather days allowed" table per month, filled from data, to put
  in the tender and contract instead of a flat percentage.
- **Other countries:** rule packs for frost, snow and typhoon seasons.

Sources used for the market check (not verified beyond these pages):
[Allplan: AI-based weather systems in construction planning](https://allplan.com/blog/can-ai-based-weather-systems-improve-construction-planning),
[Adaptive construction scheduling with AI in 4D/5D BIM (Portsmouth research portal)](https://researchportal.port.ac.uk/en/publications/adaptive-construction-scheduling-with-ai-in-4d5d-bim-integrating-/),
[Oracle: predicting schedule delays with AI](https://oracle.com/construction-engineering/construction-intelligence-cloud/using-ai-and-machine-learning-to-predict-construction-schedule-delays),
[United Rentals: BIM and severe weather](https://es.unitedrentals.com/project-uptime/data/4-ways-bim-helps-contractors-react-fast-severe-weather),
[AP: delayed rains in India, June 2026](https://wtop.com/world/2026/06/delayed-rains-bring-water-shortages-to-cities-and-uncertainty-to-farms-across-india/),
[JK Cement: best season for construction in India](https://www.jkcement.com/blog/construction-planning/best-season-for-home-construction-india/).
