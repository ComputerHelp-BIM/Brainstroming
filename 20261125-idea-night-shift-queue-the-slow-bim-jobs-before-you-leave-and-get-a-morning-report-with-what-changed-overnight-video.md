# 20261125 Idea: Night Shift — queue the slow BIM jobs before you leave; next morning the QA, BOQ, BBS, clash and "what changed" video are waiting

## Entry 1

```
date-time:   2026-11-25 (scheduled run)
idea name:   Night Shift — "Drop the folder, go home. The model checks itself overnight."
             A small local runner (`cad2bim-run`) that opens the team's project folder with no one at the screen
             and does the slow, repeated jobs from a one-file recipe: convert every new or changed DXF/DWG to 3D,
             read every IFC, run QA / IDS checks, rebuild BOQ and BBS, diff against the last run, render a short
             "what changed" video. At 8 a.m. there is one page per project: green / amber / red, the numbers that
             moved, and a list of who must look at what.
potential:   High. Today these jobs are done by hand, one after another, with a modeller watching a progress bar.
             My estimate (not measured): 45-90 minutes a day per modeller goes to waiting on exports, re-running the
             same checks after each consultant drop, and rebuilding the same BOQ/BBS Excel. A BIM team of eight
             loses about one person-day a day to this. The bigger win is not hours: checks happen EVERY night, not
             "when someone remembers", so a wrong column size on Tuesday is caught Wednesday morning, not at the
             Friday coordination meeting.
             Trending angle: "AI agents that work while you sleep" and "morning brief" posts are among the most
             engaged on LinkedIn right now; the before/after is simple to show: a screenshot of the 8 a.m. report
             and a 20-second video of last night's changes. It also fits the Instagram "day in the life of a BIM
             modeller" reel format: 6 p.m. press Start, 8 a.m. coffee and a green board.
             Market (from my own knowledge; I did not re-check prices today): Autodesk Model Checker, Solibri and
             Navisworks batch utility can run scheduled checks, but each is paid and checks one thing (rules or
             clashes), and none rebuilds BOQ + BBS + a change video in one pass. Revit has no unattended mode
             without Dynamo/Revit Server scripts or Design Automation (a paid cloud service that uploads the model).
             cad2bim runs on the device, so nothing is uploaded.
```

### Description

**The pain (daily life of a BIM modeller / BIM coordinator).**
- A consultant drops new DXFs and an IFC at 5 p.m. The modeller waits through the conversion, the QA, the BOQ,
  and often does it again because a setting was wrong.
- The same checks are run by hand, in the same order, on every revision. When people are busy they skip them.
- Nobody knows what changed between yesterday's model and today's until the meeting. Reports are rebuilt
  by hand the evening before.

**The idea.** One recipe file in the project folder says what to do. The runner does it, unattended, on the
team's own PC or an office machine, on a schedule or when the folder changes.

```
project/
  night-shift.yml      # the recipe (about 15 lines)
  incoming/            # consultants' DXF, DWG, IFC, XLSX land here
  reports/2026-11-26/  # written by the runner: report.html, report.xlsx, changes.mp4, bcf/, ifc/
```

Example recipe:

```yaml
watch: incoming/
jobs:
  - dxf-to-3d: { template: ch-v2, out: ifc/ }       # new or changed drawings only
  - qa:        { ids: rules/office.ids }            # clash, missing data, naming
  - boq:       { grade-by: level }
  - bbs:       { format: xlsx }
  - diff:      { against: last-green }              # element-level change list
  - video:     { what: changes, seconds: 20 }       # new red, removed grey, moved amber
notify: report.html  # opens the morning page; optional email/WhatsApp text of the summary
```

**What the person sees at 8 a.m.** One page, top to bottom:
1. **Traffic light per model** (green: nothing new to fix; amber: review; red: do not issue).
2. **Numbers that moved:** concrete m³, steel kg, BOQ cost, element count, each with last night's value and the change.
3. **Fix list:** QA findings grouped by root cause and assignee (BCF files, one per person).
4. **The 20-second video:** the model with changes coloured; click an element to see the before/after values.
5. **Failures, honestly:** a drawing that could not be converted is listed with the reason, never hidden.

**Why it makes a "wow".** It turns a pile of separate tools into one habit: the model is checked every night
without anyone asking. The morning page is something a manager can read in a minute and a client can be sent.

### Implementation

How it can be built from what the repo already has (all to be verified against the code before starting; I
read `docs/agent/protocol.md` and the package list, not the whole engine).

- **Where the pieces already live.**
  - DXF/DWG → 3D: the Rust pipeline (`packages/pipeline`) and the CAD core (`cad-extract` CLI in
    `packages/cad-engine`) are already native or WebAssembly code with no UI. This is the first step of the runner.
  - IFC reading, QA, BOQ, BBS: `packages/engine` (web-ifc, which also runs in Node). Needs a check that these
    modules do not import browser-only code; if some do, the smallest step is to run them in headless Chromium.
  - Agent tools: `packages/mcp` already has a catalogue of 41 operations (`docs/api.md`) answered by an open
    tab. Night Shift reuses the same operation names, so a recipe step and an agent tool call are the same thing
    (one vocabulary: good for agent experience).
- **Runner (`cad2bim-run`).** One Node program, shipped like `cad2bim-mcp.mjs` (a single file served by the app,
  "Manage → Night Shift", with a copy-paste scheduled-task line for Windows Task Scheduler / cron). It starts a
  headless Chromium with the app (Playwright is already used for smoke tests) bound to `127.0.0.1`, pairs with the
  existing helper protocol, and calls the recipe's operations in order. No new network surface; the same origin
  and pairing rules as the AI agent helper (`docs/security.md`).
- **Incremental by hash.** Each input file is hashed (SHA-256). Only new or changed files are converted; results
  are cached in `.night-shift/` so a typical night with two changed drawings takes minutes, not hours.
- **Diff.** Use the model's stable ids (IFC GlobalId; for DXF-built elements the pipeline's own stable key) to
  match last night's elements to tonight's. Same as Change Radar's matching (see
  `20261014-idea-change-radar-revision-diff-boq-delta.md`): reuse it, do not build a second one.
- **Video.** The three.js viewer renders frames to a canvas in headless Chromium and encodes with
  WebCodecs; the "Camera moves and video" roadmap item is the same engine. The first version can ship without
  video and write a still image strip instead.
- **Report.** One self-contained HTML (no outside CDN, per the licence and privacy rules) plus XLSX. Uses design
  tokens and the existing report styles; no helper text on the page (the design rule).
- **Rules to respect.** Nothing leaves the device unasked: the runner never uploads and has no cloud mode.
  Licences: only MIT / Apache-2.0 / BSD / ISC / MPL-2.0 dependencies; an MP4 encoder must be checked (use
  WebCodecs from the browser, not a GPL ffmpeg). Windows: no two files differing only by case in `reports/`.
- **First version (smallest useful).** Folder watcher + `dxf-to-3d` + `qa` + `boq`, one HTML page with traffic
  light and the numbers that moved. No video, no email, no BBS. Measure how many minutes a night of real work
  takes against the same work done by hand.
- **Test.** Use a scratch folder with three real consultant drops kept out of git ("people's drawings never
  go in git") and a small synthetic set in the repo's tests: one drawing changed, one unchanged, one broken.
  Report only measured values: seconds per step, cache hit rate, the changes the diff found against the
  changes planted, and the minutes a modeller needs for the same morning check by hand. State clearly what is
  estimated.

### Future update

- **Multi-project league table.** Run the same recipe on all projects in a server folder and rank them by risk
  (links to Warning Doctor's multi-model idea and Ratio Radar).
- **Rules from your own history.** Feed closed BCF/RFI history back so the nightly QA learns the office's usual
  mistakes (links to Repeat Offender and Deja Model).
- **Hand-off steps.** After a green night, auto-build the issue set and transmittal (Issue Desk) and the model
  capsule for the client (Model Capsule) and leave them as drafts for a person to approve. Never send on its own.
- **Pull in site data.** Add a morning step that reads the day's pour, cube and delivery records (Pour Proof,
  Cube Clock) and shows plan vs actual beside the model changes.
- **Phone push.** A one-line summary to the team's phone: "Tower B: 2 red, 5 amber, 3.2 m³ added".
- **Honest scoreboard.** After a month, publish measured hours saved and checks caught per project, not estimates.
