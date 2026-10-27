# 20261027 Idea: Model Pulse: Revit records what the team actually did (elements added and changed, views, commands, syncs, active time), and cad2bim turns it into a draft timesheet per project and level, a team board with progress against the LOD milestone, a 3D model coloured by "who built what, and when", a replay of the building being modelled, a finder for the repetitive work that an AnonGee button could do, and an ROI meter that shows how many hours the toolkit saved this month. Everything stays on the office's own disk

## Entry 1

```
date-time:   2026-10-27 (scheduled run, UTC)
idea name:   Model Pulse. "Friday, 6 pm. The BIM lead opens cad2bim and drops in the office's
             'pulse' folder (on the office NAS, never uploaded). The board shows the week:
             Project Sunrise G+12, 142 active modelling hours across 5 people; Level 5-8
             structure 100% (columns, beams, slabs vs the CH template count), Level 9-12 60%,
             rebar 25%: 'at this rate the LOD 350 milestone on 14 Nov slips by 3 days'. The
             draft timesheet is already filled: Ravi 38.5 h Sunrise / 2 h Office, Neha
             31 h Sunrise / 7 h Greenfield, split by level and by category (walls, rebar,
             sheets). One click on '3D' and the Sunrise model opens coloured by who built each
             element; a slider replays the week and the tower grows floor by floor in 40
             seconds. The 'Repeated work' card: 'Neha placed 486 beam stirrup sets one beam at
             a time: about 6.5 h. The AnonGee Detail Drop / BBS Generator flow does this from
             the ETABS design in minutes. Show me how.' The 'Toolkit ROI' card: 'AnonGee
             buttons ran 1,312 times this month and touched 48,900 elements; estimated 211 h
             saved (baseline per button below, measured in the office's own timing test).'
             The lead exports the timesheet to Excel for billing, posts the 40-second build
             replay to LinkedIn, and closes the laptop."
             One record, four readers:
               (1) pyRevit-addons (AnonGee): a "Pulse" hook set, always on once the office
                   switches it on: element added / changed / deleted (by category, level,
                   workset), view and sheet activity, command use (ribbon and AnonGee
                   buttons, with run time and elements touched), sync with central, and
                   active time (Revit in front and the person giving input). Written as
                   one small JSON-lines file per person per day to a folder the office
                   chooses,
               (2) Revit bridge add-in (C#, .NET 8): the same events for offices that
                   run the bridge, with the element's IFC GUID so the 3D view can match
                   it,
               (3) cad2bim web app: a "Pulse" workspace: the board, the timesheet, the
                   3D colour-by-author / colour-by-date view, the replay, and the cards.
                   Opens the folder with the browser's folder picker, reads it locally,
                   nothing leaves the device,
               (4) cad2bim agent helper (MCP): "how many hours did rebar on Sunrise take
                   per tonne?", "who is free next week?", "estimate a G+10 like Sunrise".
             Outputs: draft timesheet (Excel / CSV, per person, project, level, category),
             weekly PDF report for the client or the director, progress vs milestone chart,
             3D heatmap (IFC opened in cad2bim with colours), a 20-60 s build replay video
             (MP4 / WebM), the "repeated work" list, the toolkit ROI sheet, and an
             estimating table (hours per m2, per tonne of rebar, per sheet) from the
             office's own history.
potential:   High, and it is the one idea that helps every other tool, because it measures
             them. Every BIM office has the same three questions every week: where did the
             hours go, are we on time for the milestone, and what should we automate next.
               - Today: timesheets are filled from memory on Friday (or on the 30th), by
                 project only, so nobody knows that rebar took 3x the walls. Progress is
                 "about 70%" in a WhatsApp message. Estimates for the next job are a guess
                 plus a margin. The value of an in-house toolkit like AnonGee is "it feels
                 faster", not a number (from how offices work, not measured).
               - Time on timesheets: 15-30 min per person per week, plus 2-4 h for the lead
                 to chase, check and bill (estimate). For a 10-person team: about 6-9
                 person-hours a week, 300-450 a year, before counting better estimates and
                 caught slips.
               - Repeated manual work: in most models a few tasks (rebar, tags, dimensions,
                 sheets, renaming, view setup) take a large share of the hours. The
                 finder shows exactly which ones, in this office, with the hours, so the
                 next AnonGee button is built for the biggest real cost, not a guess.
             With Model Pulse (target, to be measured in the pilot): timesheets drafted
             automatically (review and fix in 2-3 min), progress per level and category
             every day, an estimating table from real history after 3-5 jobs, and a
             monthly "toolkit saved X hours" number that is defensible because its baseline
             is measured.
             Trend: "BIM ROI", "data-driven BIM management", "AI productivity", "what gets
             measured gets automated" and model-building time-lapses are steady LinkedIn
             topics for BIM managers and AEC firms; time-lapse reels of a building growing
             floor by floor do very well on Instagram. A firm posting "our in-house tools
             saved 211 hours this month, here is the build replay" is exactly the kind of
             post that gets shared.
             Wow effect: you open a folder and see the whole week: the tower grows on screen
             in 40 seconds, each element coloured by who built it, and a card tells you the
             six hours someone spent clicking that a button could have done.
             Market (to our knowledge, not a full survey):
               - BIMBeats (Revit / AutoCAD usage telemetry and dashboards): paid, per user,
                 data in its cloud.
               - Unifi Project Analytics (formerly Kinship): paid, model and family
                 analytics, cloud.
               - Autodesk Construction Cloud / Insight dashboards: paid, cloud, not activity
                 per person per element, no timesheet.
               - Revit journal file readers (free scripts): show raw commands, no project,
                 level, timesheet or 3D.
               - Timesheet apps (Harvest, Clockify, Zoho): paid or freemium, typed by hand,
                 no link to the model.
             We know of no tool, free or paid, that goes from Revit activity to a draft
             timesheet, a 3D who-and-when heatmap, a build replay, a repeated-work finder
             with suggested automation and a measured ROI of the office's own toolkit, with
             all the data kept on the office's own disk.
```

### Description

**The pain (a normal Friday in a BIM office).**
- Timesheets: everyone fills them from memory. "Sunrise 40 h". Nobody knows how the 40 h split between
  structure, rebar, sheets and rework after the architect's revision. The client billing for the revision is
  argued, not shown.
- Progress: the lead asks in the group "how far is Level 9?". The answer is a feeling. The milestone slips
  quietly; the client hears about it last.
- Estimating: the next G+10 is quoted at "about the same as Sunrise", but nobody knows what Sunrise really took.
- Automation: the office builds tools (AnonGee, cad2bim, Structura), but cannot say which manual task costs the
  most hours, or how many hours each tool saved. The director cannot see the return on the tool time.
- Hand-over between people: when someone is on leave, nobody knows exactly which levels and views they were
  in the middle of.

**What Model Pulse does.**
1. **Record (in Revit, quietly).** A pyRevit hook set (or the bridge add-in) records events, not keystrokes:
   - elements added, changed and deleted, by category, family type, level, workset, phase, with the IFC GUID
     (from Revit's `IfcGUID` parameter or the element's UniqueId converted the same way the IFC exporter does),
   - views and sheets opened, and how long each was in front,
   - commands run: ribbon command name, AnonGee button name, run time, elements touched,
   - sync with central (time and duration), opening and closing the model,
   - active time: Revit in front and the person giving input in the last N minutes (N set by the office,
     default 5); no screenshots, no keystroke text, no window titles of other programs.
   One JSON-lines file per person per day, in a folder the office chooses (local or the office NAS).
2. **Tag the time.** Each active minute goes to the project (from the model's Project Information or a mapping
   file), the level and category most worked on in that minute, and a task: "modelling", "rebar", "sheets and
   annotation", "coordination / review", "fixing warnings", "rework after revision" (changes to elements that
   already existed before the last issue date).
3. **Board.** In cad2bim's Pulse workspace: hours per project, person, level, category and task this week;
   progress per level against a target count (from the CH template or the previous issue: "Level 9: 48 of 52
   columns, 81 of 96 beams, slab 0 of 1"), and a burn-down against the milestone date with the forecast.
4. **Timesheet draft.** Per person per day per project, rounded the way the office wants (15 or 30 min). The
   person checks and fixes it; the edit is kept so the draft learns (e.g. "meetings on Tuesday 11-12 are
   Greenfield"). Export to Excel / CSV for payroll or billing.
5. **3D views.** The project's IFC opens in cad2bim with colours: by who created it, who last changed it, by
   date (old blue to new orange), by "changed after the last issue" (the revision rework, priced for the client
   with the hours). Click an element: created by, date, changed 4 times, last by, 12 min of work around it.
6. **Build replay.** A timeline slider plays the model being built in creation order; record it to video (the
   Model Reel idea's recorder) for the director, the client, or LinkedIn.
7. **Repeated work finder.** Looks for long runs of the same manual pattern: the same command on the same
   category many times (placing rebar one host at a time, tagging one by one, copying views, renaming sheets),
   and gives the hours and a suggestion: an existing AnonGee button that does it, a cad2bim feature, or "make
   this a button" (the Toolsmith idea).
8. **Toolkit ROI meter.** Each AnonGee button logs its runs and the elements it touched. The office times the
   manual way once per task (a short timing test: e.g. "dimension one grid by hand: 4 min"), and Pulse shows
   the monthly hours saved, per tool, with the baseline visible.
9. **Estimating table.** After a few jobs: hours per m2 of floor, per tonne of modelled rebar, per sheet, per
   level, by building type and LOD. New job: "G+10, 9,800 m2, LOD 350 with rebar: 640-760 h, based on 4 past
   jobs".

**Trust and privacy (designed in, not added later).**
- The office decides, writes it in the staff policy, and switches it on per machine; each person sees exactly
  what is recorded about them (their own day view first) and can mark a period as private (break, personal).
- No keystrokes, no screenshots, no other programs, no file contents leave Revit; only event counts and IDs.
- Data stays in the office's folder; cad2bim reads it in the browser and uploads nothing, the same rule the
  app already keeps for drawings.
- The board shows teams and tasks first; per-person views are for the person and the lead, set by a role.
- The aim is "find the work to automate and bill fairly", not "rank people". The cards never say "slow", they
  say "this task took 6.5 h; here is a faster way".

**Daily use cases.**
1. *Friday timesheet*: two minutes to check the draft instead of 20 to remember the week.
2. *Client revision claim*: "the architect's revision R3 cost 46 h of rework; here are the 312 elements in
   orange and the hours". A fair, shown claim.
3. *Monday stand-up*: progress per level and the forecast for the milestone on one screen.
4. *Quoting a new job*: hours from the office's own history, not a guess.
5. *Choosing the next tool to build*: the top 5 repeated tasks with their hours.
6. *Proving the toolkit*: the monthly ROI sheet for the director and a LinkedIn post with the replay.
7. *Hand-over*: "Ravi was last in Level 7 rebar, views 'L7 Beam Rebar' and sheet S-207"; the colleague
   picks up in minutes.
8. *Training*: a new modeller's time per task drops week by week; the lead sees which tasks need a
   short training session.

**Not in the first version.** AutoCAD and ETABS activity (later, see Future update), payroll integration,
automatic billing to clients, and any per-person ranking.

### Implementation

**1. Recorder (pyRevit, IronPython or CPython, inside AnonGee).**
- pyRevit extension hooks in `AnonGee.extension/hooks/`: `doc-changed` (Revit's `DocumentChanged` event:
  `GetAddedElementIds`, `GetModifiedElementIds`, `GetDeletedElementIds`, `GetTransactionNames`),
  `doc-synced`, `doc-opened`, `doc-closed`, `view-activated`, and `command-before-exec` /
  `command-exec` for commands. Transaction names already say a lot ("Place Structural Rebar", "Tag All").
- Active time: a light timer (Revit's `Idling` event, throttled to once every 30-60 s) checks that Revit is
  the foreground window and the last input time (Windows `GetLastInputInfo`), and writes one "active" tick.
- A small `pulse` module in `AnonGee.extension/lib/`: `pulse.event(kind, **fields)` buffers events and
  writes them to `pulse/<user>/<yyyy-mm-dd>.jsonl` every minute and on close; a failure to write never
  blocks Revit (log and drop). A schema version on every line.
- AnonGee buttons call `pulse.tool_run(button, elements, seconds)` at the end of a run (one line added to
  each script, or a decorator), which feeds the ROI meter.
- Element facts read once per change: category, level (`LevelId` or the host's level), workset, phase,
  family/type name, and the IFC GUID. Creator and last changed by come from
  `WorksharingUtils.GetWorksharingTooltipInfo` for workshared models.
- Cost: events are small (a few hundred bytes); a busy day is a few MB per person at most (estimate, to be
  measured). The handler must stay under a few ms per change: no geometry reads in the event.

**2. Bridge add-in (C#, .NET 8), optional.**
- The same events and file format from `bridge/revit`, for offices that run the bridge instead of pyRevit.
  The format is one shared JSON schema in the cad2bim repo, so both writers stay the same.

**3. Reader and model (cad2bim web app).**
- A "Pulse" workspace in `apps/web` (its own ribbon and dock layout, like the CAD workspace). The folder is
  opened with the File System Access API (or a zip of the folder dropped in); parsing in a web worker.
- Sessions: active ticks grouped into minutes, each minute tagged with project, level, category and task by
  simple rules first (most-touched category in the minute, transaction names, "rework" when the element
  existed before the last issue date). No AI needed for the MVP; rules are visible and editable.
- Board and charts from the design system (`packages/ui`), colours from tokens.
- 3D: the project IFC opens in the existing viewer; colour by a property map (GUID -> author / date /
  rework) and a timeline slider for the replay (show elements with created <= t). Recording reuses the
  canvas recorder from the Model Reel idea.
- Exports: timesheet and ROI sheet to Excel (the app's existing BOQ / BBS Excel writer), weekly PDF.

**4. Repeated work finder.**
- Runs of the same (command or transaction name, category) with short gaps, counted and timed; a table maps
  patterns to known tools ("Place Structural Rebar" on beams x many -> BBS Generator / Detail Drop flow;
  "Dimension" x many on grids -> AutoDIM; renames x many -> Bulk Rename). Unknown patterns are listed with
  their hours as candidates for a new button.

**5. Agent helper (MCP).**
- Read-only tools over the parsed data: `pulse.hours(project, by)`, `pulse.progress(project, level)`,
  `pulse.estimate(building)`. The data is read from the same local folder.

**MVP (3-4 weeks).**
1. pyRevit recorder: element changes, transactions, active time, AnonGee button runs, to JSON lines.
2. cad2bim Pulse workspace: the board (hours by project, person, level, category), the draft timesheet and
   the Excel export.
3. 3D colour-by-author and colour-by-date with the replay slider, on the project IFC.
4. ROI meter for 5 AnonGee buttons with measured manual baselines.
5. Pilot: 2-3 people for 2 weeks on one live project; compare the draft timesheets with what they would
   have filled; measure recorder overhead in Revit (time per change event, file size per day).

### Future update

- **AutoCAD and cad2bim CAD viewer activity** (drafting hours on the 2D side), and C2B conversion runs, in the
  same file format.
- **ETABS / etabs-mcp**: time in analysis and design iterations per project; the agent's tool calls logged
  with their duration, so "the agent saved X hours" is measured too.
- **Learning tags**: a small local classifier trained on the office's own corrected timesheets for better
  task tagging (still on the office's machines).
- **Milestone planner**: from the estimating table and who is free, propose the next month's staffing per
  project, and warn early when two milestones clash.
- **Quality link**: join with the Warning Doctor and IDS QA ideas: hours spent fixing warnings and QA items
  per project; which rules cause the most rework.
- **Client portal view**: a read-only progress page (levels done, replay video) the office can choose to share.
- **Office benchmark**: compare this year with last year (hours per m2, per tonne) to show the effect of the
  toolkit over time.

### Improvement suggestions for existing tools in the repos

- **pyRevit-addons (AnonGee):** add the `pulse` module to `lib/` and one `tool_run` call to every button
  (or a decorator in the shared script template that `Create Button` uses), so every new button is measured
  from day one. A small "Pulse" toggle and "My day" view in the Dev panel.
- **pyRevit-addons (AnonGee):** `Bridge Check` could also report whether the recorder is on and writing,
  so the office sees at a glance that the data is complete.
- **cad2bim (viewer):** a general "colour by property" and "timeline by property" feature in the IFC viewer
  (any date or text property, not just Pulse data): useful for 4D, for revision dates and for QA status.
- **cad2bim (QA / Change Radar idea):** mark elements changed after the last issue in the same orange the
  Pulse rework view uses, so the revision diff and its hours are one story.
- **cad2bim (agent helper, MCP):** log each tool call's duration and result size locally; the same ROI meter
  can then show the agent's savings.
- **etabs-mcp:** the same optional local log of tool calls and run times.
- **Structura, C2B, planwin-ai-pro:** write the shared Pulse event format on each run (input file name only,
  duration, elements produced), so one board covers the whole toolkit.
