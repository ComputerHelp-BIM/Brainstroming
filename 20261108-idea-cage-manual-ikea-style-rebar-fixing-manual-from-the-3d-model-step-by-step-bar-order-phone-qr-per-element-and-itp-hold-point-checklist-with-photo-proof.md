# 20261108 Idea: Cage Manual — an IKEA-style, step-by-step rebar fixing manual for every beam, column, slab and footing, made from the 3D model, opened by QR on the bar-bender's phone, with the pre-pour hold-point checklist and photo proof

## Entry 1

```
date-time:   2026-11-08 (scheduled run, first entry of the day)
idea name:   Cage Manual (auto "fixing order" manual for each rebar cage: which bar goes in first, spacer and
             chair points, stirrup start/end, lap stagger, cover blocks; step-by-step 3D that the site gang
             can follow on a phone; ITP / pre-pour hold-point checklist with photos; all from the model and
             BBS cad2bim already holds)
potential:   High, daily on every RCC site and every modelling team that hands over rebar. The 3D rebar model
             and BBS are finished, yet the person tying the cage still works from a 2D section and memory.
             Wrong bar order (top bars tied before the stirrups are threaded, a lap in the wrong zone, a
             missed spacer) is found at inspection, when it costs rework, or after the pour, when it costs
             far more (the size of this loss is an estimate from how sites usually run, not measured here).
             The wow: scan a QR stuck on the formwork, and the beam's cage builds itself on the phone, bar by
             bar, like a furniture manual, in Hindi, Marathi or English, with a "Done" tick and a photo
             after each stage.
```

### Description

**The pain.**
- Modellers spend days getting 3D rebar and BBS right (this repo has many ideas for that: BBS Weaver, Detail
  Drop, Slab Sketch, Core Cage). The model then stops at the office door.
- On site, the bar-bender and the foreman read 2D sections. Congested joints (beam-column, beam-beam, core
  walls) are hard to picture, and the fixing order is not on any drawing. It lives in the head of the best
  foreman.
- Quality checks are a paper checklist, often filled after the fact. Photos sit in WhatsApp, not tied to an
  element or a pour.
- When the consultant revises a bar, nobody tells the gang. The printed section is out of date.

**What exists today (checked 2026-11-08, not exhaustive).**
- In this repo's `idea` branch (99 files read by file name and keyword search): Joint Congestion Doctor finds
  rebar clashes; Joint Judge plans construction joints; Pour Planner and Ready Board plan pours and
  readiness; Site QR Pass shows a pour card; Steel Ledger reconciles steel issued; Bar Optimizer cuts bars.
  Keyword searches for "fixing order", "hold point" and "ITP" found no entry that turns the rebar model into
  a **build sequence** or a **hold-point checklist**. Cage Manual fills that gap.
- In cad2bim (`docs/design/`): rebar (`rebar.md`), BBS (`bbs.md`), 4D (`next-milestones.md`), the
  Python console and the agent API. Rebar bars with positions, marks, diameters, hooks and laps are in the
  model. The viewer already isolates by element, section boxes and exploded views (0.10).
- In the market (web search 2026-11-08; marketing claims unverified): rebar tools for Revit such as Graitec
  PowerPack generate 3D cages and detailing; an Autodesk University class lists "3D cage generation" and
  cage templates. I found **no** result showing animated bar-by-bar fixing sequences or an automatic
  fixing-order manual. A 2024 peer-reviewed study reports a BIM-based bar bending schedule cut preparation
  manpower by about a third ([Buildings, 2024](https://doaj.org/article/6ef0862c559a4440aa0e79a507c6d127)),
  which supports value in pushing rebar data to the person doing the work. QR-to-model-view tools exist
  (for example [Buildagil](https://knowledge.buildagil.com/en/article/1qk2l6/); Doxel uses QR plus computer
  vision for progress tracking), but they open a view or log progress; they do not explain **how to build the
  cage**. Honest limit: this search was shallow, and Tekla and Allplan were not checked. Verify before any
  claim of "nobody has this".
- Trend note: I could **not** measure LinkedIn or Instagram trends; the search found no 2026 trend data. What
  is safe to say: field BIM on tablets and phones, QR links to models, and "AI in construction" are widely
  posted topics (informal sources). A short phone screen recording of a cage building itself is exactly the
  kind of clip that suits those feeds.

**The idea.** A "Cage Manual" button on any rebar host (beam, column, slab, footing, wall). It produces a
numbered sequence and a phone page. Everything is computed on the device from data already in the model.

1. **Sequence rules (the engine).** A small, editable rule set per element type gives the usual order, for
   example for a beam: place corner/hanger bars on the stirrups' set-out; thread stirrups at the start/end
   zone spacing (closer in the confinement zone from IS 13920); tie bottom main bars; add extra top bars at
   supports; add side-face bars; fit cover blocks; then end hooks into the column. The rules are a starting
   point written down openly, then the office's senior foreman edits them. The tool does **not** claim to
   replace site engineering judgement.
2. **Step player.** Each step shows the bars added in that step (others ghosted), the bar mark, diameter,
   length, count and cut-length from the BBS. The camera moves to the right side. A "next / back" and an
   exploded view for congested joints.
3. **Clash-aware order.** Where bars cross in a joint, the order is chosen so each bar can physically enter
   (use the geometry already used by congestion checks). When no order works, the step says "needs a design
   fix" and creates an issue.
4. **Phone page by QR.** One QR per element on the formwork. The page works offline once loaded, has big
   buttons, bar marks in large type, and a language switch. No account is needed for the gang; a signed link
   limits it to that pour.
5. **Hold points.** After defined steps the page asks the checks an inspector would make (cover, spacing,
   lap length and stagger, bar count, chairs, clean formwork, embedded items present). Each check takes a
   tick and a photo, stamped with time, element id and, if the phone allows it, position.
6. **Pre-pour clearance.** When all hold points are ticked, the pour card shows "clear to pour" and exports a
   one-page PDF with photos for the client's engineer. Missing items block it in red.
7. **Revision safety.** If a bar changes in a new revision, the manual for that element is marked "changed",
   the QR page shows a warning banner, and the change is listed for the foreman.

### Implementation

- **Data.** Reuse rebar, host, mark and BBS data from the model (GlobalId stays the key). Add three small
  stores: sequence rules (JSON), step results (tick, photo reference, time), and QR link tokens.
- **Sequence engine.** A pure function: rebar set + host geometry + rule set -> ordered steps. Start with
  beams and columns (the most common), then footings, slabs, walls. Write it in TypeScript in the engine
  package with unit tests using sample beams; rules edited in a plain table in the app.
- **Order check.** Ray or sweep tests to see if a bar can enter without crossing an already placed bar;
  fall back to "needs design fix". Reuse the clash code planned for Joint Congestion Doctor.
- **Viewer.** Step player uses existing isolate / ghost / camera-fit calls; no new rendering engine.
- **Phone page.** Static, small page generated per pour, with the steps as glTF or simple instanced
  geometry. Photos stored on the phone and shared as the person chooses; nothing is uploaded unasked (fits
  the repo's "nothing leaves the device unasked" rule). Sharing a pour package is an explicit export (a file
  or link the user creates).
- **Export.** PDF manual per element (printed A4 "IKEA" sheets), and a pour clearance PDF.
- **Checks before building.** Test on three real beams and one column with the senior foreman and measure:
  minutes to read the manual, wrong-order errors found at inspection before and after. Report what was
  measured and what was estimated.
- **Rough order of work:** beam rules and step player (about 2 weeks), phone page and hold points (about
  2 weeks), PDF and revision banner (about 1 week). These are estimates, not measurements.

### Future update

- Voice steps in Hindi and local languages ("ab stirrup daalo") for gangs who do not read English.
- Link to 4D and Order Clock: a step becomes a task with man-hours, and hold points feed the Ready Board
  and Pour Planner.
- Learn from sites: record how long each step really took (from tick times) and tune labour output.
- Tablet AR overlay on the real formwork to check the cage against the model (needs testing; not promised).
- Precast and prefabricated cage yards: the same sequence becomes the jig assembly order for cage tables.
- Steel Ledger link: bars fixed vs bars issued per pour, so shortages show before the pour.

### Sources

- [Autodesk University: Best Rebar Detailing Tools Revit 2018](https://www.autodesk.com/autodesk-university/class/Best-Rebar-Detailing-Tools-Revit-2018)
- [Buildings 2024, BIM-based bar bending schedule study](https://doaj.org/article/6ef0862c559a4440aa0e79a507c6d127)
- [Buildagil: QR codes to model views](https://knowledge.buildagil.com/en/article/1qk2l6/)
- [Doxel: QR codes and computer vision for progress tracking](https://doxel.ai/blog/qr-codes-computer-vision-can-streamline-construction-progress-tracking)
