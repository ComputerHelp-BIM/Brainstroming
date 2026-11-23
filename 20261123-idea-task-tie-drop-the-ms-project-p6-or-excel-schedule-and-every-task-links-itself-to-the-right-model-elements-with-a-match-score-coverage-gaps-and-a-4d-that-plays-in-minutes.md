# 20261123 Idea: Task Tie — Drop the contractor's schedule (MS Project, Primavera P6 or Excel) on the model. Every task links itself to the right elements (level, zone, element type, pour). A match score, a "tasks with no elements / elements with no task" gap list, and a 4D that plays in minutes, not days. On the device, nothing uploaded.

## Entry 1

```
date-time:   2026-11-23 (scheduled run, UTC)
idea name:   Task Tie (schedule file to model auto-link: task text + WBS + dates matched to
             element sets by level, zone, type and pour; match score; coverage gaps; one-click 4D)
potential:   High. Every BIM team asked for a "4D video" eventually (client, lender, PMC,
             tender). The slow part is not the video, it is linking each of 200 to 2,000 tasks
             to elements by hand. This removes that part. Medium-high "wow": drop a schedule file
             on an open model, watch the link board fill (green / amber / red), press Play, and
             the building rises in the order of the plan, with a "Coverage 94%" badge.
             Time saved: NOT measured. The pain below is from public sources and team
             experience, not from a timed study.
             First test: take 1 past project with its schedule (kept out of git). Time a modeller
             linking 50 tasks by hand in the team's usual tool. Then run Task Tie on the same 50
             and compare: minutes, links that match the hand links, links that differ, and
             who was right in each difference. Do not quote savings until this is done.
             Market check (2026-11-23). Two standard web searches. Found: Navisworks TimeLiner
             (Autodesk page lists Simulate at about USD 1,110 a year per user) connects tasks to
             model objects; Synchro (price not found); Fuzor 2024, whose makers say an AI
             workflow creates 4D sequences with one click; ALICE Technologies (schedule
             optimisation from P6); Innovaya (Revit to MS Project / Primavera). A Procore guide
             and an MDPI paper say the linking is still mostly manual or rule-based. All are paid
             or sales-led; the claims are vendor claims and not verified here.
             What I did NOT find: a free, in-browser tool that reads the schedule on the
             device, shows WHY each task matched (and a score), lists the gaps both ways, and
             learns from the user's corrections. "I did not find one", not "none exists".
             Trend: "4D BIM", "AI in construction scheduling" and "schedule vs model" posts are
             regular on LinkedIn. I could not measure counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - 4D timeline (roadmap, Next): puts tasks and dates ON elements and plays them.
                 Task Tie is the step before it: it FINDS which elements each task means.
               - Tag Tamer: tags elements with level, zone, pour, WBS by rules. Task Tie reads
                 those tags when they exist and works without them when they do not.
               - Pour Planner, Order Clock, Ready Board, Crew Curve, Cash Curve, Delay
                 Detective, Sky Slot, Crane Canvas, Model Reel: all start from "a 4D exists".
                 Task Tie is how it comes to exist, so they all get their input faster.
               - Sheet Sync / Drop Zone: Excel in, bulk edit; Drop Zone reads a drawing folder.
                 Task Tie reads the schedule file, a different input.
             The wow: open a G+12 model. Drop "Tower_Programme_R3.xml" (MS Project). In a few
             seconds: "412 tasks read, 371 linked, 26 need you, 15 are milestones." Click the
             task "L7 Slab + Beams Casting": the L7 slab, beams and the lift core wall lights up,
             "match 92: level 7, type slab+beam, 'casting' = pour". Press Play: the structure
             rises in plan order. A red bar says "Tasks with no elements: 'Fire tank waterproofing'
             (the model has no tank). Elements with no task: 38 parapet walls, roof".
```

### Description

**The pain.**
- The planner writes the schedule in MS Project or Primavera P6, in the planner's words:
  "Podium 2 - Col & Wall - Zone B", "Footing F3-F9 PCC". The model uses Revit's words: level
  names, type names, marks. The two never agree.
- Linking is a person selecting elements, one task at a time, in a 4D tool. For hundreds of
  tasks it takes days, so it is done once, late, and goes stale at the next revision.
- Nobody checks the links. A task can point at the wrong floor, or half of the roof can have no
  task, and the video still plays. The client sees a building that finishes with missing parts.
- When the planner sends revision 4 of the schedule, the links are redone or lost.

**What Task Tie does.**
1. **Read the schedule on the device.** MS Project XML, Primavera XER (the P6 export), Excel/CSV
   with a one-time column map (name, WBS, start, finish, predecessors). Nothing is uploaded.
2. **Understand each task name.** Split it into parts the model can answer: level or storey
   ("L7", "Podium 2", "Terrace"), zone or grid ("Zone B", "A-D / 1-5"), element type
   ("column", "beam", "slab", "footing", "wall", "stair"), activity ("rebar", "formwork",
   "casting", "curing", "strip") and WBS path. A small word list in plain files (English and the
   usual Indian site words), editable by the firm.
3. **Find the elements.** For each task, select the elements that match level + zone + type from
   the model's own data (storey, grid position, IFC class, type name, marks, tags from Tag
   Tamer). Several tasks may share elements (rebar, formwork and casting of the same slab): each
   shows its own activity stage, so 4D can colour them "formwork / rebar / cast".
4. **Show why, with a score.** Each link has a match score and the reason in one sentence. Green
   (clear), amber (a guess: ask), red (no match). Amber and red go to a review board, sorted by
   how many elements they affect.
5. **Gaps, both ways.**
   - Tasks with no elements (a task the model cannot show, or a model that lacks something).
   - Elements with no task (parapets, roof, stairs left out of the programme).
   - Elements in two tasks of the same activity (double counted).
   - Sequence sanity: a slab planned before the columns under it; a floor cast before the floor
     below; a task finishing after its successor starts. Shown as questions, not verdicts.
6. **Learn from corrections.** Fix a link once ("'Core' means the lift and stair walls") and the
   rule is saved for this project and, if the person agrees, for the firm. The next revision of
   the schedule keeps every confirmed link and only asks about the new or changed tasks.
7. **Revision-proof.** Drop schedule R4: tasks are matched to R3 by their ID or WBS, the changes
   are listed ("14 tasks moved, 3 new, 2 deleted"), links follow, dates update, the 4D replays.
8. **4D in one click.** Hands the linked tasks to the app's 4D timeline: colour by status, play
   by day or week, and the existing hand-offs (Model Reel video, Ready Board, Crew Curve, Cash
   Curve, Delay Detective) work on it.

**What the person sees.**
- A *Task Tie* tab: a file drop area, then a board with three columns of numbers (linked, to
  review, no match) and the tasks listed under them.
- Click a task: its elements light up in 3D, with the reason line. Click an element: the tasks
  that cover it.
- A coverage bar: share of elements (by count and by concrete volume) that have a task, so a
  forgotten roof shows as a gap of real size, not a count.
- A Play button, a "Questions for the planner" list (Excel) and a "Questions for the modeller"
  list, each with the task or element named.

**What it is not.** It does not make or optimise the schedule, does not check that durations are
realistic, and does not decide the build sequence. It only ties the planner's file to the model,
shows where they disagree, and keeps the ties alive when either one changes.

### Implementation

- **Where it lives.** A new module in the IFC engine (`@cad2bim/engine`) for reading, matching and
  gaps (pure functions, easy to test); a panel in the web app beside the planned 4D timeline.
  It uses the 4D data model the timeline will define (task id, start, finish, element GUIDs,
  stage); if that is not built yet, Task Tie ships its own small version and the timeline adopts it.
- **Reading files.** MS Project XML and Excel/CSV are plain. Primavera XER is a tab-separated
  text format: parse the TASK, TASKPRED and PROJWBS tables. Check any library's licence before use
  (MIT, Apache-2.0, BSD, ISC; no GPL); a short hand-written reader is likely enough. Binary
  `.mpp` and `.pmxml` are not first-version: the person exports XML.
- **Matching.** Rules first, no AI needed to start:
  (1) normalise text (case, "L7" / "Level 7" / "7th floor" / "07"); (2) read tokens into the five
  parts above; (3) query the model: storey name or elevation, IFC class, type name, grid cell
  from the element's position, Tag Tamer tags; (4) score = share of parts that agree, with
  weights the firm can change; (5) keep the best set and the runner-up for the review board.
  Zone maps (grid ranges to "Zone B") are a small table the person fills once.
- **Optional AI help, only when asked.** For red tasks, an "Ask" button can use the person's own
  API key (as the app's AI chat already does) to suggest which elements a task text means. The
  task text goes out only for that one task, after a notice; the app holds no key of its own; it is
  off by default. The result is shown as an amber suggestion, never applied silently.
- **Learning.** Corrections are stored as small rules (text pattern to element query) in the
  project file; no model training, no upload. Firm-wide rules are a file the team can share.
- **Revisions.** Match tasks by schedule ID, then WBS code, then name similarity; list moved,
  new and deleted; never drop a confirmed link without saying so.
- **Gaps.** Coverage by count and by concrete volume reuses the BOQ quantities. Sequence checks
  are a few explicit rules (support below, floor order, predecessor dates) each with a name and a
  plain sentence, kept in a rules file so a firm can edit them.
- **Agent helper.** Expose `task_tie_status()` and `link_task(task, query)` through the MCP helper
  so an agent can answer "which tasks have no elements?" and fix a link. Follows
  `docs/agent/protocol.md`.
- **First version (smallest useful).** Excel/CSV and MS Project XML in; level + element type
  matching; the link board with scores; the two gap lists; hand-off to the 4D timeline. Primavera
  XER, zone maps, learning, revision matching and the optional AI help come second.
- **Test.** 1 real project with a real schedule, kept out of git (people's drawings and schedules
  never go in git). Compare against the modeller's hand links: agreement rate, time, and who was
  right in each difference. Add a small synthetic schedule and model to the repo's tests.
- **Rules to respect.** Nothing leaves the device unasked. No GPL code. Plain, short wording in
  the app; no helper text or taglines (the design rule).

### Future update

- **Resource and cost links.** Tie BOQ rates to the same tasks so 5D cost over time (Cash Curve)
  appears without another link step.
- **Two-way.** Send the changed dates back as an updated MS Project XML / XER so the planner keeps
  one schedule, not two.
- **Progress back in.** Read the planner's percent complete, or site photos (Photo Placer), and
  colour the model by actual vs planned (Delay Detective).
- **Lookahead sheets.** From the linked tasks, print the 3-week lookahead per trade with the
  elements shown, one page per gang (Ready Board).
- **Template library.** A firm's own word list and zone maps per project type (tower, villa,
  podium, industrial), so a new project starts at 90% matched.
- **Beyond structure.** The same linking for MEP and finishes when "every discipline in one
  project" ships.
- **Honest scoreboard.** After 5 projects, show the team's measured link accuracy and time, and
  publish only measured numbers.
