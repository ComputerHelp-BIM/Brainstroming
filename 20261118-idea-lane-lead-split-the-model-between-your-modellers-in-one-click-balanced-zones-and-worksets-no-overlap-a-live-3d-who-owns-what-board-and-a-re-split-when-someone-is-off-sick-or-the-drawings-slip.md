# 20261118 Idea: Lane Lead — split the model between your modellers in one click: balanced zones with no overlap, worksets made for you, a live 3D "who owns what" board that fills as the work gets done, and a one-click re-split when someone is off or the drawings slip

## Entry 1

```
date-time:   2026-11-18 (scheduled run, UTC; third idea of the day)
idea name:   Lane Lead (plan the modelling work of a team, hand out the lanes, watch them fill, re-balance on the day)
potential:   High for any office with 3 or more modellers on one building, and it is a daily problem.
             Monday 9:30: the lead has a G+14 tower, 5 modellers and one drawing set. The split is made in
             the lead's head and on a WhatsApp group: "Ravi take L1-5, Neha take L6-10, Amit slab and stairs..."
             By Wednesday two people have modelled the same lift core, one is idle because his floors were
             not issued yet, one is buried under all the rebar, and nobody knows what is truly done.
             Every other tool in this repo helps AFTER the model exists or helps ONE person. This one is
             about the team's first hour and the team's daily hour.
             Time saved: NOT measured. First test (cheap): take 3 finished past projects. Ask the lead to
             write down, from the drawings only, who would have done what. Compare it with what happened:
             hours per person, overlap hours (same element built twice or fixed twice), idle hours waiting
             for drawings. Then run Lane Lead on the same drawings and compare the plan. Do not quote a
             percentage until that is done.
             Market check: I did NOT search the web in this run, so this is "I did not check", not "none".
             What I do know: Revit worksharing gives worksets and "borrowing" but does not plan or balance
             the work; ACC/BIM 360 and BIMcollab hand out issues, not model zones; Monday, Trello and Jira
             are generic and know nothing about the model. To do before building: search the Autodesk App
             Store and "BIM project management" tools for "work breakdown from model", "workload balancing".
             Extends tools already in this repo (does not repeat them):
               - Scope Scout: counts the scope and gives hours and a fee BEFORE the project. Lane Lead takes
                 the same counts AFTER the job is won and cuts them into per-person lanes.
               - Model Pulse: records what people did, after the fact, and draws a team board and timesheet.
                 Lane Lead makes the plan Model Pulse would measure against, and the lane colours it paints.
                 The two chain: Lane Lead plan (what should be done) vs Model Pulse actual (what was done).
               - Drop Zone: reads the drawing folder, a readiness board per level. Lane Lead uses that board
                 to decide who can start now and who must wait or switch to another lane.
               - Trace Lens / Mark Keeper: coverage and marks. Lane Lead uses mark and element counts as the
                 size of each lane, and asks Mark Keeper's rules before two people share one mark range.
               - BEP Bolt: sets the standards and worksets list. Lane Lead creates the live workset per lane.
               - Co-Model: several people editing live in the browser. Lane Lead works with it or without it
                 (Revit worksharing is the daily case in most offices).
               - Dojo Drill: skill map for juniors. Lane Lead can read it so a junior is not given the
                 transfer slab (a suggestion the lead can change, never a rule).
             Trend: "AI agents running the project office" and "BIM project managers using AI" are standing
             LinkedIn topics, and "who is blocking whom" boards are common on Instagram reels about site work.
             I could not measure LinkedIn or Instagram counts, so this is not a social-media claim.
             The wow: the lead drops in the drawing folder and types the team ("Ravi, Neha, Amit, Priya,
             Karan; Priya off Thursday"). In 20 seconds the tower in 3D is painted in five colours, one per
             person. Each colour is a clean block of floors and a trade (not 5 random patches): "Ravi: L1-4
             columns and beams, 410 elements, 11.5 h". Under it a fairness bar: all five between 10.8 and
             11.9 h. A red line: "Neha: L6-8 waits for sheet S-207 (not issued): moved to Amit's lane for
             now, 2.5 h". Press "Hand out". Each person gets a link: their lane, their sheets, their
             standards and a ten-line checklist. On Wednesday Karan reports sick; press one button: his
             lane is cut into five pieces and the whole board re-balances; moved elements keep their colour
             change and a note, nothing is lost.
```

### Description

**The pain.**
- The split of work is done on a call and kept in memory. It is not tied to the model or the drawings.
- Equal floors are not equal work: a transfer floor with 300 kg/m3 of steel is not a typical floor.
- Dependencies are invisible: the lift core is needed by every floor; sheets arrive late; one person's
  grid or levels must exist before the others start.
- Two people building the same thing, or two fixing the same clash, is the most common hidden waste.
- When someone is off or a revision lands, the lead redoes the split by hand again.

**What it does.**
1. **Count the work.** Reads the drawing set and, if present, the half-built model. For each level and
   trade: elements to model (columns, beams, slabs, walls, stairs, footings, rebar sets), by the same
   counting Scope Scout uses, with a difficulty factor per type that the office sets once (typical floor
   vs transfer vs podium vs lift core vs rebar).
2. **Pick the lanes.** The lead types or pastes the team, days, and optional limits ("Priya not before
   Thursday", "Amit does rebar only", "Karan max 6 h today").
3. **Cut cleanly.** A planner cuts the model into lanes that are:
   - *Contiguous*: whole floors, or a bay range, or a trade on a floor band, never a scattering.
   - *Non-overlapping*: each element belongs to one lane. A seam between two lanes is named ("grid 4-5").
   - *Balanced*: hours within a few percent of each other, using the difficulty factors.
   - *Ready*: a lane starts only where its sheets and levels exist (from Drop Zone). If not, it takes
     something else first, and the wait is shown with the sheet that is missing.
   - *Ordered*: shared things (grids, levels, core, typical column family) come first and go to ONE
     person, with the others shown as "blocked until 11:00" or given independent work meanwhile.
4. **Show it in 3D.** Each lane a colour, a fairness bar, a Gantt of hours per person, and a "critical
   path" line ("the core blocks four lanes until Ravi finishes it").
5. **Hand out.** One link per modeller: lane, sheets, naming and workset, checklist. A ready-made workset
   per lane is made on the Revit side (names from the BEP), with an owner set.
6. **Fill as they work.** Lane colours go from outline to solid as elements appear in the model (read
   from the model file or Revit bridge, never from screen watching). Done is counted by elements against
   the plan, not by self-report.
7. **Re-split in one click.** Someone off, a new revision, a slipped sheet, a rush ("do L9-10 first"):
   re-run on the unfinished part only. Finished work stays with the person who did it.
8. **End of day.** A two-line note per person ("done 62%, 1.5 h behind, waiting on S-207") for the lead,
   and a plan-vs-actual line that feeds Model Pulse and the next Scope Scout calibration.

**What it is not.** It does not watch people, rank them, or write a performance report. Hours are the plan,
not a score. The lead can drag any block of elements to another person; the tool then recounts. A person's
name appears only where the lead typed it, and the data stays in the project file on the device.

### Implementation

- **Where it lives.** A new "Plan" board in the web app, using the model already open, plus a small Revit
  bridge call to create worksets and read element counts by workset. No upload; no accounts.
- **Counting.** Reuse the counts already built (the DXF → 3D pipeline's per-level element lists, and Scope
  Scout's counting when it exists). Where there is only a drawing, count from the pipeline's dry run; where
  there is a model, count the elements and the unfinished parts from the IFC.
- **Difficulty factors.** A plain table (element type × complexity → minutes) that starts from published
  values the office can overwrite and is then calibrated from its own past projects (Model Pulse records).
  Until there is history, the first plan is labelled "estimate, not measured".
- **The cut.** A simple, explainable planner, not a black box:
  1. Build units (level × trade × bay range) with hours and prerequisites (a graph).
  2. Sort by prerequisites; assign by "longest job first to the least-loaded available person", respecting
     limits and days off; keep units of one person adjacent in space where the sizes allow.
  3. Refine with a few swaps that reduce the largest gap in hours. Everything is deterministic: the same
     input always gives the same plan, and the page can print its own reasoning ("Neha took L6-8
     because she had the most free hours after Priya's Thursday start").
- **Ownership.** A lane is a saved selection (level range, trade, grid zone) plus a person and a workset
  name; the 3D colours are one colour override per lane in the existing view settings.
- **Progress.** Poll the model or the bridge on Save/sync; count elements by lane that exist and pass a
  quick check (has mark, has type, on the right level). "Exists" is not "right", so done is labelled
  "modelled", and the check list in the handout still asks for a human review.
- **Revit side.** Create worksets, set the owner name as text, and put each lane's elements in its
  workset on request. If worksharing is off, the tool only prints the plan and the lane selections; it
  never changes a model without the lead pressing a clear button.
- **Hand-out links.** A static page built on the device (a "lane card" file or a link inside the office
  network share) holding only the lane's names, sheet list and checklist. No model data leaves the device.
- **Test first.**
  1. Three finished projects of different type (a typical tower, a podium with transfer, a small villa).
  2. Compare Lane Lead's plan with what the team actually did: spread of hours, number of "two people on
     one thing" cases, hours people waited for a drawing.
  3. Run five "something went wrong" scenarios (sick day, a late sheet, a big revision, a new joiner, a rush
     floor) and measure minutes to a new accepted plan by hand and with the tool.
  4. Report the plan's miss: planned vs real hours per lane. If the planner is not better than the lead's own
     split on at least two of the three, say so and fix the factors before building the board.
- **Risks.** People feeling watched (mitigate: no scores, names only as typed, plan-not-person view, easy
  to turn off); wrong difficulty factors giving a bad split (mitigate: show the factor, let the lead edit
  it, calibrate); a split that ignores a hidden dependency such as one tricky family (mitigate: the lead can
  pin an item to one person with a click); too many reassignments (mitigate: show the cost of a move as
  hours of hand-over).

### Future update

- **Calibrates itself.** Each finished project updates the difficulty factors, so the next quote (Scope
  Scout) and the next plan (Lane Lead) get truer. Show the miss, not only the plan.
- **Pair mode.** A senior and a junior share one hard lane, with a review point built into the plan.
- **Review lanes.** After a lane is "modelled", a second person is assigned a quick check of it; Warning
  Doctor and Sheet Sentry results go to the lane owner.
- **Cross-office.** Two offices (or a freelancer) share one plan: lanes are exchanged as a small file; the
  model still moves by the office's own channel.
- **Site mode.** The same lanes for the site: a pour or a bar-fixing crew per zone, with the 4D plan (Ready
  Board, Crew Curve) as the source.
- **Standup card.** A 30-second morning card per person (what is mine today, what is blocked, what changed
  overnight) with a voice summary for the lead.
- **Client-facing.** A "progress by floor" picture for the client report, from the same lane fill, with no
  names on it.
