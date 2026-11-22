# 20261122 Idea: Plan Passer — The model (or the drawing) becomes a submission DXF that passes the municipal portal's geometry pre-check the first time: state layer packs, closed polylines, plain text, no blocks or xrefs, with every failure pinned to the line and fixed in one click. On the device, nothing uploaded.

## Entry 1

```
date-time:   2026-11-22 (scheduled run, UTC)
idea name:   Plan Passer (cad2bim model / DXF to authority-ready submission DXF: a layer pack per
             state portal, a geometry pre-check run on the device, one-click fixes, and a pass report)
potential:   High for Indian architecture and BIM teams, because every project needs a sanction
             drawing and a rejected upload costs a day. Medium-high "wow": a red list of 40 portal
             errors turns green in one press, before the file goes anywhere. Fits cad2bim's
             "nothing leaves the device" rule: the portal sees only the clean file.
             Time saved: NOT measured. The pain below comes from the portals' own manuals and from
             team experience, not from a timed study.
             First test: take 5 real sanction drawings that were rejected once (kept out of git).
             Count the rounds and hours each took to pass the portal. Run Plan Passer on the
             originals and count the errors it found, how many it fixed, and how many a portal
             still rejected. Do not quote hours until this is done.
             Market check (2026-11-22). Two standard web searches. Found: the portals' own
             manuals (Kerala PreDCR, West Bengal, Punjab Smart DCR, Assam CAD SOP, DIGIT-DCR)
             each list the same kind of geometry rules: closed LWPOLYLINE only, named layers
             (for example FLOOR01, FLOOR02 in West Bengal and Punjab, underscore layers in
             Kerala), colour codes, zero line thickness, plain Text not MText, no blocks, regions,
             wipe-outs or xrefs on floor layers, model space only, DXF 2004 and up to 30 MB in
             DIGIT-DCR and Assam. The West Bengal manual says its pre-check only checks geometry,
             not the rules. Paid products exist: SoftTech's AutoDCR and its RuleBuddy Plan
             Assist (described as a pre-check before submission); I did not verify price or how
             far it goes. A freelance listing for "smartdcr autocad drawing fix" suggests people
             pay for hand-fixing. What I did NOT find: a free tool that makes the file from a
             model and covers several states' layer packs. "I did not find one", not "none exists".
             Not verified: whether each portal takes DXF or only DWG (Punjab's manual says DWG);
             rules change by state and by year. Each pack must be checked against the current
             manual before it ships.
             Trend: "AI for building approvals" and "digital plan approval" posts are regular on
             LinkedIn in India. I could not measure counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Area Statement Pro: makes the area table and FSI sheet from the model. Plan Passer
                 makes the whole submission DXF pass geometry checks and puts that table on it.
               - Layer Whisperer: maps a messy incoming DXF to the CH template (the way IN).
                 Plan Passer maps the model to a portal's layers (the way OUT).
               - Trace Lens / DXF checks: find lines the pipeline skipped. Plan Passer finds lines
                 a portal would reject.
               - Sanction Gap / Plot Pilot / Exit Map / Access Pass: check or plan the rules.
                 Plan Passer only makes the file acceptable to the portal's geometry check.
             The wow: open a model or a drawing, press "Plan Passer", choose "Kerala PreDCR".
             A list shows 37 errors: 12 open polylines, 9 MText, 6 blocks on floor layers, 4 wrong
             layers, 3 non-zero thickness, 3 off-model-space. Press "Fix all": 34 turn green, 3 stay
             with a reason and a pin on the line. Press "Export": a DXF 2004, 4 MB, with a pass
             report you can attach to the file.
```

### Description

**The pain.**
- Each state portal wants its own layer names, colours and rules. Teams keep a different template
  per state, or fix every file by hand.
- The portal's pre-check says "Multiple Polyline Geometry Problem" and little else. Finding the line
  in a 30 MB drawing takes longer than drawing it.
- A model drawn for design is not a submission drawing: hatches, blocks, MText, xrefs and open
  outlines all fail.
- A rejected file means a new round, often days, and the same mistakes return on the next project.

**The idea.** Five moves.
1. **A pack per authority.** A small, editable, versioned file (JSON or YAML) lists layers, colours,
   mandatory items, text rules, file limits and version. Packs start with one or two states and grow
   by contribution. Each pack states the manual it was written from and its date.
2. **Build the file.** From the open model, cut the plan per storey and write the outlines (plot,
   proposed work, floor, staircase, lift, parking, setback line) as closed polylines on the pack's
   layers. From a drawing instead, map its layers to the pack (Layer Whisperer's mapping).
3. **Pre-check on the device.** Run the geometry rules a portal runs: closed, no duplicate vertices,
   matching vertices between neighbouring polygons, zero thickness, plain Text, no blocks or xrefs
   on floor layers, model space only, colour codes, file size and version.
4. **Fix and pin.** Each error shows on screen at its line. One-click fixes: close with a stated
   tolerance, explode blocks, convert MText to Text, change layer or colour, drop hatches and
   wipe-outs, purge. Every fix is listed and undoable; anything uncertain stays red with a reason.
5. **Pass report.** A one-page PDF: pack name and date, errors found, errors fixed, errors left.
   It does not claim the portal will pass. It says what was checked.

**What the person sees.**
- A *Plan Passer* tab: pick the authority, press Check.
- A split view: error list on the left, the drawing on the right with the line lit.
- Counts by kind, a "fix all safe ones" button, and one button per kind.
- Buttons: DXF, Pass report (PDF).

**What it is not.** It is not a bylaw scrutiny engine, and it does not approve a plan. It makes the
file acceptable to the geometry check. The rules (setbacks, FSI, fire) stay with Area Statement Pro,
Plot Pilot and Exit Map, and with the architect's judgement.

### Implementation

- **Where it lives.** A new module in the CAD engine (`@cad2bim/cad-engine`), using the reading it
  already does for DXF and DWG; a panel in the CAD workspace. Nothing leaves the device.
- **Rules as data.** The pack is a plain file, not code. The check engine reads it. This keeps a
  state change to a one-file edit and lets a firm keep its own packs.
- **Checks.** Closed-polyline test with a tolerance; vertex match between neighbours on a spatial
  index; entity-type scan for blocks, regions, wipe-outs, MText and xrefs; colour, layer, thickness
  and space checks; size and version check on write.
- **Fixes.** Each fix is a small function that returns what it changed, so the report and undo come
  from the same list. Closing a gap above the tolerance is never automatic.
- **Writing DXF 2004.** The engine already reads DXF; the writer is the new part. Check the licence
  of any writer library before adding it (MIT, Apache-2.0, BSD, ISC only; no GPL).
- **First version (smallest useful).** One pack, DXF in and DXF out, checks for open polylines,
  MText, blocks and layers, with the pass report. Add the model-to-plan step second, more packs after.
- **Test.** Rejected drawings kept out of git; a before and after error count; for every pack, a
  review by someone who has used that portal; a portal upload by the team, not by the tool.
- **Version.** New minor version of the CAD engine and the app; changelog, README and guide entry as
  CLAUDE.md asks. Not done here; this repo holds ideas only.

### Future update

- More packs: Maharashtra, Karnataka, Gujarat, Tamil Nadu, Delhi, Telangana, as people share them.
- DWG output through the DWG route the CAD engine already has, for portals that want DWG.
- Learn from a portal's real rejection messages: paste the message, get the matching fix.
- A "drawing diet" that cuts a 30 MB file under the limit without losing a required item.
- Re-check on every revision and show only what changed since the last pass.
- A shared pack library for a firm, with a version stamp on every pass report.
