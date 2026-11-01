# 20261101 Idea: Crew Curve — the 4D schedule plus the model's quantities become a day-by-day manpower histogram (bar benders, carpenters, masons, helpers), gang sizes, a labour-levelling "flatten the peak" slider, and a what-if crash tool, with no manual resource loading

## Entry 1

```
date-time:   2026-11-01 (scheduled run, UTC)
idea name:   Crew Curve (Auto Manpower Histogram and Labour Levelling from 4D + Model Quantities)
potential:   High. Every tender and every monthly review asks "how many workers do you need, and when?".
             Today a planner types resource numbers into a schedule by hand, or a site engineer guesses.
             The model already knows the quantities (m3 concrete, kg steel, m2 shuttering, m2 block work)
             and 4D already knows the dates. Quantity x productivity norm / duration = crew per day. Nobody
             should be typing that. Estimated saving (an estimate, not measured): half a day to two days
             per tender or per re-baseline, and fewer "labour peak" surprises on site. It also turns 4D
             from a nice video into a decision tool, which is what a client pays for.
```

### Description

**The pain.**
- The 4D shows *when*. The first question a contractor's director asks is *how many people*. A labour peak of
  120 in week 14 and 35 in week 15 means idle camps, overtime and missed pours. Nobody sees it until it
  happens, because the histogram is built by hand from a bar chart.
- Resource-loading a schedule in desktop planning tools is manual: pick a task, type a gang, type
  output-per-day. For 400 tasks that is days of work, and it is wrong the moment the pour sequence moves.
- Labour shortage and productivity are steady LinkedIn topics in Indian construction (sites short of
  skilled bar benders and shuttering carpenters, festival-season attrition). Read this as a trend I saw in
  discussions, not a measured statistic.

**What exists today (one thin search, to be re-checked before any "nothing like it" claim).**
- Resource loading and histograms exist in paid planning suites and in some 4D/5D tools. I could not
  confirm what they auto-derive from the model, or their prices. None that I know of is free, runs in the
  browser, keeps the file on the device, and uses Indian trades and norms out of the box.
- Earlier ideas in this repo produce the inputs (Pour Planner dates, Bar Optimizer and BBS Weaver steel,
  Shutter Plan area, Wall Wizard block work, Rate Rocket rates) but none turns them into people.

**The idea.** A **Crew** panel in cad2bim's 4D workspace (planned on the roadmap: "4D timeline", "5D cost").
For each 4D task:
1. **Quantity** comes from the linked elements (concrete m3, steel kg from BBS, formwork m2, block m2,
   plaster m2). No typing.
2. **Productivity norm** per trade = output per person-day (for example kg of rebar fixed per bar-bender per
   day). Starter values come from a small editable table; the team's own numbers overwrite them. Defaults are
   *placeholders to be verified* against the company's own records or published labour constants (CPWD
   analysis of rates is a public starting point; I did not check exact figures here).
3. **Crew per day** = quantity / (norm x task duration). Sum by trade, by day, over the whole programme.
4. Output: a **stacked histogram** (one colour per trade), the **peak**, the **average**, a "labour-days
   total", and a labour cost line that feeds 5D (rate from Rate Rocket).

**The wow 1: drag the peak flat.** A *Level labour* slider. The tool shifts tasks inside their float (and
splits repeatable zone tasks) to lower the peak, then shows before/after histograms and the finish-date
impact: *"Peak bar benders 64 to 41 by moving L9-L11 steel fixing by 1-3 days. Finish date unchanged."*
The planner accepts or rejects; nothing moves without a click. Simple greedy levelling is enough for v1.

**The wow 2: "I have 30 carpenters" what-if.** The person enters the real crew they have. The tool turns
every task's duration into what that crew can do, replays the 4D, and shows the new finish date and the
weeks that fall behind. The reverse also works: *crash to finish by 15 Jan* and read off the extra crew
needed per trade per week. Best case for a tender discussion: change one number live in the meeting.

**The wow 3: the site board.** A one-page "this week's crew" card per gang for WhatsApp (*"Week 14: bar
benders 38, carpenters 52, masons 20, helpers 70"*), and an "ask vs. actual" bar after the foreman taps the
headcount once a day (the same phone check-in as Ready Board). After a month the tool shows the real
output per person-day, so the norms improve from the team's own data.

**Who uses it, when.**
- **BIM / 4D modeller:** tender stage, after the 4D is linked; ten minutes to a labour plan.
- **Planner:** weekly re-baseline, levelling instead of rebuilding resource sheets.
- **Contractor director / PM:** one number that matters in a meeting (the peak).
- **Client / PMC:** evidence that a stated finish date has the people behind it.

### Implementation

**Stage 1: quantities and histogram (3-4 weeks).**
- Reuse the existing BOQ/BBS quantity takeoff in `packages/engine`; add `Task.quantities` per trade by
  summing linked elements.
- Data model: `Trade { id, name, unit, outputPerDay }`, `TaskResource { task, trade, qty, durationDays }`,
  stored in the project file. Spread crews evenly over the task by default; allow front-loaded and
  back-loaded profiles.
- Histogram: an SVG or canvas stacked bar chart, built with the design tokens. No new heavy dependency.

**Stage 2: levelling and what-if (3-4 weeks).**
- Greedy levelling: process tasks by float, shift each into the lowest-load window inside its float, never
  break a predecessor link. Keep a list of the moves so the result can be undone and explained.
- What-if crew cap: recompute durations as `qty / (norm x crew)` and run the existing forward pass of the
  4D logic.
- Crash: invert the formula to give the extra crew needed for a target date.

**Stage 3: cost and site loop (2-3 weeks).**
- Labour cost = labour-days x wage rate (editable per trade), shown as a line under the histogram and as a
  5D curve. Print PDF and PNG with the renderer the other ideas use.
- Static offline check-in page for daily headcount; data stays on the device until the person exports.

**Licences and rules.** Pure arithmetic and a small greedy loop; charts in canvas/SVG; no GPL code; nothing
leaves the device. Cad2bim's rule "say what was measured and what was estimated" applies: every default norm
is labelled *starter value* until the team confirms it.

**Risks, said plainly.** (1) Productivity norms vary a lot by site, floor height, season and gang skill; a
wrong norm gives a confident wrong histogram. So norms are visible, editable, and carry a "from your
records / starter" tag. (2) Even spreading is a simplification (pours are lumpy); Stage 1 offers profiles
and the tool says "indicative". (3) Levelling can hide a real constraint (crane, access), so it should read
Ready Board and Crane Canvas constraints when those exist.

### Future update

- **Learn from actuals:** after daily headcounts arrive, suggest norms from this team's own history per
  trade and per floor type.
- **Camp and transport planner:** peak headcount to labour camp beds, canteen size, shift buses and PPE
  stock.
- **Skill mix:** split each trade into skilled, semi-skilled and helper ratios, and flag a shortage week
  ahead so recruiting can start early.
- **Festival and weather calendar:** thin the crew in festival weeks and monsoon spells automatically.
- **Subcontractor view:** share only a trade's own curve with that subcontractor and track their ask vs.
  actual for the monthly review.
- **IFC export:** write crew-days per element into a property set so any viewer can colour by labour
  intensity ("where is the labour hungry part of this building?").
