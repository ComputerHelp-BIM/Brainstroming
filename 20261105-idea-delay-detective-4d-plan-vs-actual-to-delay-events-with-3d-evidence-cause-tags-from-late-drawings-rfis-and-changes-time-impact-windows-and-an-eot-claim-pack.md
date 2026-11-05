# 20261105 Idea: Delay Detective — compare the 4D plan with what really happened, find the delay events, tag each one with its cause (late drawing, open RFI, design change, rain, material) and 3D evidence, run a time-impact check, and write the extension-of-time (EOT) claim pack

## Entry 1

```
date-time:   2026-11-05 (scheduled run, second entry of the day, separate idea from Save-to-See)
idea name:   Delay Detective (Delay Analysis from the BIM Model: Baseline 4D vs Actual Progress, Delay Event
             Register, Cause Tags from Drawing Issue Dates / RFIs / Revision Diffs / Rain Days, Windows and
             Time-Impact Analysis, Who-Caused-What Split, EOT and Cost Claim Pack with 3D Stills and Clips)
potential:   High, and different from most ideas in this branch: it does not make a model faster, it makes
             the model the evidence in the argument that costs the most money on a project, which is
             "who delayed whom". Claims and delay disputes are a regular topic on construction feeds; read
             that as a trend I recall, not a measured statistic (not verified in this run). On most Indian
             and Gulf projects the EOT file is built by hand, months later, from emails, site diaries and a
             scheduling export; the BIM team is asked for "the drawing dates and what changed" and loses
             days digging. The model already holds the three things a claim needs and nobody joins: what
             was planned (4D), what was built when (progress), and what changed and when (revisions, RFIs,
             issue register). Estimated saving (an estimate, not measured): a claim file of 20 to 40 delay
             events takes a planning engineer and a BIM coordinator about 2 to 4 weeks by hand; with the
             events found and the evidence attached, perhaps 3 to 5 days of review. The larger value is
             being able to notify a delay in the week it happens (most contracts require notice within a
             set number of days), not a year later. Wow: pick a late slab pour, and the 3D shows the
             slab, the drawing revision that arrived 19 days after the task needed it, the RFI that sat
             open for 11 days and the rain days, with the days split between them.
```

### Description

**The pain (what teams do today).**
- The contractor's planning engineer keeps a schedule in Primavera P6 or MS Project. The site keeps a diary.
  The BIM team keeps the models and the drawing issue register. The architect and the consultant keep their own
  RFI logs. In a claim or a monthly review these four are joined **by hand in Excel**, using a date typed from
  memory.
- The common question is "which of these 40 late tasks are our fault, which are the client's, which are
  neutral (rain, strike)?" Without the join, everything becomes "late drawings" or "late contractor" by
  argument, not by record.
- Notices of delay are time-barred in many contracts. A delay noticed a week late can lose the claim. A tool
  that sees the delay while it grows is worth more than one that explains it afterwards.
- Design changes (a column grew, a wall moved) are buried in revisions of the model. Their effect on the
  task (more concrete, more steel, more days) is rarely measured on the day the change arrived.

**What exists today (one thin search plan; every "nothing like it" claim below must be re-checked before it
is published).**
- Delay and forensic scheduling tools exist (Oracle Primavera P6's own tools, Acumen / Fuse, Phoenix Project
  Manager, Powerproject, and consultants' in-house sheets). I did not verify features or prices here; they
  are commonly described as paid and expensive, and they work on **schedule files only**: they do not
  know which drawing revision a task was waiting for or what changed in the 3D model.
- 4D tools (Navisworks TimeLiner, Synchro, Fuzor and others) play a schedule on a model. They show *when*
  something was planned or built; I did not find one that **explains a delay** by joining the issue register,
  the RFIs and the revision diff, or that writes the claim pack.
- In this repo (read from the files in this folder and from cad2bim's `docs/design/next-milestones.md` and
  `docs/design/revision-timeline.md`):
  - **4D timeline** is on the roadmap (0.110.0+, "tasks and dates on elements, played back"), not built.
  - **Order Clock** (2026-10-30) and **Ready Board** (2026-10-31) look **forward** (will the material and
    the drawings be there). **Crew Curve** (2026-11-01) levels manpower. **Pour Planner**, **Bill Painter**,
    **Site QR Pass** and **Chat-to-Model** record progress and show planned-vs-actual colours.
    **Change Radar** (2026-10-14) and **Rev Merge** (2026-11-02) know what changed between revisions.
    **Issue Desk** (2026-10-31) keeps the drawing register with issue dates. **Meeting Mind** (2026-10-26)
    makes dated BCF issues. **Demo Map** (2026-10-31) mentions a "hindrance and delay log" for brownfield
    works only.
  - **Nothing here looks backward at the delay itself**: no finder for "the task finished late", no
    attribution, no time-impact method, no claim pack. Delay Detective is the *reader of the records those
    ideas create*. That is its strength and its limit: it needs them (or imports of them).

**The idea.** A **Delay** tab beside the 4D timeline in the cad2bim web app. It reads, on the device and with
no upload: the baseline schedule, the actual dates (from progress marks), the drawing issue register, the
RFI list, the revision history of the model, and a rain/holiday calendar. It then does five things.

1. **Find the delay events.** For each task compare baseline start/finish with actual start/finish. Mark
   late starts, long durations and late finishes; follow predecessors to find which late task actually moved
   the project end (the critical path of the *actual* programme). Show the result as a ranked list: "Slab L7
   pour: started 9 days late, finished 12 days late, moved the project end by 6 days."
2. **Tag every event with candidate causes, from records, not opinion.** For a late-starting task, ask
   "what was it waiting for?" and look it up:
   - **Drawing** — the last IFC / DXF revision the task's elements needed, and the date it was issued, from
     the issue register; if issued after the baseline start, the days between are a *drawing-late* candidate.
   - **RFI / query** — open RFIs pinned to the same elements (BCF issues) with the days they were open.
   - **Design change** — the revision diff (Change Radar): elements of this task whose size, count or position
     changed after the task was planned, with the extra quantity and the extra days at the planned output.
   - **Weather and holidays** — rain days above the contract's normal-rain allowance, public holidays,
     curfew or strike days from a calendar the team loads once.
   - **Contractor** — material or crew late, from Order Clock and Crew Curve records, if those exist.
   - **Unexplained** — the days left over. The tool never hides them.
3. **Split the days — carefully.** Concurrent causes are the hard part of delay analysis. The tool offers the
   standard methods by name and shows which one it used: **as-planned vs as-built**, **windows analysis** (the
   programme cut into slices, the critical path rechecked each slice) and **time impact** (insert each
   owner-side event into the baseline and see how far the end date moves). Concurrent delay is shown as
   *shared*, never silently given to one side. The method list follows the SCL Delay and Disruption Protocol
   (2nd edition, 2017) and AACE 29R-03 as published guidance; confirm the contract's own rules and the
   latest editions before anyone relies on the output.
4. **Show it in 3D.** Click an event: the elements light up, the drawing revision and RFI pins appear in the
   same view, and a small slider scrubs "baseline" against "actual" so the gap is visible. Export a still or a
   short clip for the claim.
5. **Write the claim pack.** One zip: a delay event register (Excel), a narrative per event (what was
   planned, what happened, the record that shows it, days claimed, clause field left blank for the
   claimant), the time-impact charts, the drawing and RFI evidence list, 3D stills, and an audit trail
   (which file, which date, which hash). A notice-of-delay **draft** is created the week an open event first
   crosses a threshold the team chooses (for example 3 working days on a critical task), so the notice is
   not forgotten.

**What makes it a "wow".**
- The 20-second demo: the 4D plays at plan speed; a red ring appears on the L7 slab; click it and four chips
  open: *Drawing R4 issued +19 days*, *RFI-112 open 11 days*, *Change: +8 m³*, *Rain 3 days*. A bar splits the
  12 days between them. One button writes the pack.
- It is a **change in what BIM is for** on a project: from drawings to dispute-proof records. That is a very
  different story on LinkedIn from another viewer or another export.
- It works without the claim: the same list is a weekly **delay watch** for the project manager.

**Honest limits (stated up front, because the subject is legal).**
- It **suggests**, it does not decide. Whether a day is compensable depends on the contract, the notices
  given and the facts on site, none of which the model knows. The pack is evidence and a draft for the
  claimant's planner and lawyer, never a finished claim. The tool must say so on every page it writes.
- Its answers are only as good as the dates: actual dates entered from memory weeks later are weak evidence.
  The tool therefore marks each date with its source and age (*logged on the day by phone*, *entered in
  bulk*, *imported*), and lists weak ones first.
- It does not replace a forensic scheduler on a large dispute. It does the 80 % that is data gathering.
- It does not give legal advice; clause names are left for the user to fill in (clause numbering differs by
  contract and edition; FIDIC and CPWD forms both have extension-of-time clauses, check the exact clause).

### Implementation

- **Where it fits.** `apps/web` (Delay tab and the claim-pack export); `packages/engine` (a new `delay/`
  module: pure functions over plain data, no UI); the 4D timeline and the progress marks (roadmap, to be
  built) as the source of baseline and actual dates; the revision diff (`docs/design/revision-timeline.md`)
  for the change evidence. No server, no upload. No new dependency for V1.
- **Data model (small and explicit).**
  - `Task { id, name, baselineStart, baselineFinish, actualStart?, actualFinish?, predecessors[], elementKeys[] }`
    (from the 4D timeline; for V1 also importable from a P6 `.xer` or MS Project XML / CSV export, whose
    formats are documented; write the reader from the spec, no GPL code).
  - `Evidence { kind: drawing|rfi|change|weather|contractor|other, date, source, sourceAge, elementKeys[], ref }`.
  - `Event { taskId, startSlip, finishSlip, endDateImpact, causes[{kind, days, evidenceIds[], shared}], unexplainedDays }`.
  - All dates are stored as calendar days **and** working days with a named calendar; the calendar is data,
    not code (Sunday and holiday rules differ by country and contract).
- **Engine steps** (`packages/engine/src/delay/`).
  1. `criticalPath(tasks, calendar)`: forward and backward pass on the baseline, then on the actual dates;
     the *driving* chain of the actual programme is the one that decides the end date.
  2. `findEvents(tasks)`: slips per task, ranked by effect on the end date, not by size of the slip.
  3. `attach(evidence, events)`: match evidence to tasks by shared `elementKeys` (GlobalId) and by date
     window (evidence dated between the task's baseline start and actual start).
  4. `windows(tasks, evidence, sliceDays)`: cut the programme into slices (say 2 weeks), recompute the critical
     path at each cut, and record which cause moved it.
  5. `impact(baseline, events)`: insert each owner-side event as a fragnet (an added or delayed activity) into
     the baseline and measure the end-date change; report sums with and without overlaps so concurrency is
     visible.
  6. `claimPack(...)`: Excel (existing `exceljs`-style path in `apps/web/src/lib/excel.ts`), narratives from
     templates with the numbers filled in, PDF through the existing PDF path, stills and clips from the
     viewer, and a manifest with SHA-256 of every source file (already in the toolchain for pinning).
- **Evidence sources, in order of how soon they exist.**
  - V1: a CSV the user fills: drawing register (sheet, revision, issue date, task or level it serves), RFI
    list (id, date raised, date answered, element or level), rain days. Plus the schedule file.
  - V2: Issue Desk's register, BCF issues and the Revision timeline fill these automatically.
  - V3: the site diary from Chat-to-Model and Site QR Pass statuses give logged-on-the-day actual dates.
- **Rules and wording.** The methods and the field names follow public guidance (SCL protocol, AACE 29R-03).
  Narrative templates are neutral in tone and are editable; the tool never writes the words "entitled to".
  Every page carries "Draft for review. Not legal advice." This text is a requirement, not a nicety.
- **Tests.** (1) A tiny synthetic programme of 6 tasks with known answers: critical path, slips and end-date
  effect equal hand-worked values. (2) A concurrency case: owner delay of 10 days and contractor delay of 10
  days over the same 10 days gives *shared 10*, not 20. (3) A float case: a delay on a task with 15 days of
  float does not move the end date, and the tool says so. (4) A drawing issued before the baseline start is
  never offered as a cause. (5) Evidence with no element link goes to "project-wide", not to a random task.
  (6) Working-day maths across a holiday and a Sunday. (7) The manifest hash changes if one source file is
  changed by a byte. (8) A P6 `.xer` and an MS Project XML of the same programme give the same tasks.
  Test with files in a scratch folder; real programmes are never committed.
- **Measure before claiming.** Run it on two real finished projects where the team already knows the answer
  (one with a settled claim, if the owner agrees and the data is anonymised): how many of the real delay
  events did it find, how many false ones did it add, and how many hours did building the first draft take
  against the file the team made by hand. Publish those numbers, not the estimates above.
- **Risks.** (1) The method names promise rigour the data cannot give: show the evidence grade (source and
  age) beside every number, and lead with weak-evidence warnings. (2) A contractor and an owner reading the
  same output differently: both sides get the same record, and shared concurrency is never assigned. (3)
  Confidential data: all on the device; the pack is a file the user chooses to send. (4) Scheduling
  accuracy: calendars, constraints and lags in P6 are many; V1 supports finish-to-start, start-to-start
  and lags and says plainly which P6 features it ignores. (5) Misuse in bad faith (back-filling dates):
  source and age tags and the hash manifest make back-filling visible; they cannot prevent it.
- **First 2 weeks.** A CSV-in, list-out version: tasks with baseline and actual dates, the critical path on
  both, ranked slip events, and evidence attached from a hand-filled drawing and RFI CSV. No windows
  method, no 3D scrub, no pack beyond one Excel sheet and one PDF page. Record on one real project: events
  found against events the team already knew, and the minutes from file load to first list.

### Future update

- **Live Delay Watch**: with progress marks logged on site, show the "days at risk" per task every morning
  and the first day a task crosses the notice threshold (a WhatsApp / email reminder draft for the planner,
  never sent by the tool).
- **Cost side (5D)**: price each event (prolongation: staff, plant, overheads per day from the contract
  rates; disruption: lost productivity from Crew Curve), with the rate source named, feeding the 5D cost curve.
- **Mitigation advice**: for a delay still in progress, offer re-sequencing options from Pour Planner and
  Crew Curve ("start the L8 east pour while L7 west waits for R4") with their end-date effect.
- **Counter view for the owner / consultant**: the same data shown as "contractor-side slips" (late
  materials, crews), so a PM can check a claim with the same records.
- **Lessons library**: after handover, store anonymised patterns ("drawing issue after the pour card is
  the top cause on podium slabs") to improve Ready Board warnings on the next project.
- **Claim-grade video**: a Model Reel clip with a date caption and a "baseline ghost" overlay, for hearings
  and board meetings.
- **Link to Scope Scout**: record real hours per project and feed back into the next quotation, so the
  consultancy sees where its own team was the late party.
