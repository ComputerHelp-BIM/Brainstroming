# 20261014 Idea: Pour Planner — one-click 4D pour sequence, RMC order sheet and formwork/steel cycle

## Entry 1

```
date-time:   2026-10-14 (scheduled run)
idea name:   Pour Planner — auto 4D pour sequence from the 3D model, with RMC order, bar-cutting lists and formwork reuse cycle
potential:   High. Every site engineer asks the modeller the same questions each week: "how many pours on L5, how
             much concrete in each, when do I order steel for which pour, how many sets of shuttering do I need?"
             Answered today by hand in Excel + a Gantt in MS Project/Primavera. Pour Planner answers them from the
             model in under a minute, and plays the result as a 4D animation the client and contractor both
             understand. Paid 4D tools (Synchro, Navisworks Timeliner, Bentley SYNCHRO, Autodesk Construction Cloud
             schedule) are heavy, expensive and generic: they animate a schedule you must build yourself, and they
             know nothing about pours, formwork cycles or bar bending schedules.
```

### Description

**The pain.** 4D / 5D simulation is one of the most-posted BIM topics on LinkedIn and Instagram (animated
construction sequences, "digital twin of the site", lean construction, concrete cost tracking). But the day-to-day
reality for a structural modeller is smaller and more practical: take a structural model, decide the pour
breaks, give the site a pour-by-pour list (concrete m3, steel kg, shuttering m2) and a believable time plan.
Nobody has this one button.

**The idea.** A **Pour Planner** mode in cad2bim (works on IFC, DXF-to-3D output, and PlanWin AI Pro exports),
running fully in the browser with files staying on the PC:

1. **Auto pour zoning.** Per level, columns/walls first (vertical pour, lift height configurable), then
   beams + slab as one monolithic pour split by editable rules: max volume per pour (e.g. 60 m3 = one RMC pump
   day), construction-joint locations (at L/3 of beam span, at expansion joints, at gridlines), maximum
   area per pour. Each pour gets an id such as `L3-P2` and a colour; drag a joint line in the 3D view and the
   volumes recompute live.
2. **Per-pour numbers.** Concrete m3 by grade, steel kg (from the BBS data), shuttering m2, props/staging m2,
   and a cube-test count (IS 456 sampling rule: how many cubes per pour). One table, copy to Excel.
3. **Cycle engine.** Inputs: gang size, shuttering sets available (e.g. 1.5 sets), days to strip per element
   (slab soffit 14 d, beam soffit 21 d, sides 1 d; editable), steel-fixing productivity, crane/pump days.
   The engine builds a Finish-to-Start network with resource levelling and shows: floor cycle time (e.g. 7 days
   per floor), critical path, and what happens if you add one more shuttering set ("saves 21 days on a G+12").
4. **4D playback.** Three.js timeline: elements appear in pour order, active pour pulses, curing/stripped
   status shown by colour. Scrub bar, speed control, export a 20-second MP4/GIF/WebM for a client WhatsApp message.
5. **5D cash-flow.** S-curve of cumulative cost (concrete, steel, formwork hire, labour) from the same rate table
   as the BOQ; weekly cash requirement chart for the contractor and bill-of-the-month for the client.
6. **Site-ready outputs** (this is the daily-use part):
   - **RMC order sheet** per pour: grade, m3 (+ wastage %), slump, pump type, date/time, delivery address,
     truck count from the plant's truck size. Plain text and PDF for one-tap WhatsApp share.
   - **Bar-cutting + delivery list per pour** (group the BBS by pour, order of fixing), so the steel yard
     sends exactly what the next pour needs.
   - **Pour card** (one A4 per pour): mini plan with joints marked, sections, quantities, checklist
     (cover blocks, sleeves, MEP boxes, curing). Optional QR code back to the pour in the viewer.
   - **Schedule export** to Excel and MS Project XML/CSV (Primavera XER later) so planners keep their tools.
   - **ICS calendar** so pour dates drop into the site team's phone calendars.
7. **What-if panel.** Move a pour date, change shuttering sets, or change pump capacity and see delivery days,
   floor cycle and S-curve change at once. Save scenarios (A: 1 set, B: 1.5 sets, C: table-form system).

**Why it is a wow.** Load a model, press *Plan pours*, and a 3D building rises in pour order with a rupee S-curve
building beside it; one more click produces the RMC order message and the formwork-sets recommendation. It
turns the model into the contractor's weekly work plan, which is the moment BIM proves its value on site.

### Implementation

1. **Pour model** (`packages/engine/src/pour/`): pure functions over the existing element model. Types:
   `Pour {id, level, elementIds[], grade, volume, steel, formworkArea, joints[]}`. Zoning algorithm: group by
   level + class, sort by gridline sweep (x then y), greedy fill to max volume/area, cut at nearest allowed joint
   location (L/3 on beams, grid, expansion joints). Unit-tested with fixture towers from `tools/fixtures/`.
2. **Quantities**: reuse the existing quantity and BBS-to-Excel code per element; sum per pour (steel from BBS marks
   mapped to element ids). Wastage and rate tables reuse the BOQ rate sheet.
3. **Scheduler** (`packages/engine/src/schedule/`): small CPM + resource levelling (activities: formwork, steel,
   MEP boxes, pour, cure, strip), durations from productivity inputs; deterministic and unit-tested (no ML
   needed). Calendar with weekends/holidays (editable, default India holidays optional).
4. **4D viewer**: per-element visibility/colour by time via instanced colour attribute (no geometry rebuild, so
   50k elements stay smooth); timeline UI in `packages/ui` as a bottom-panel tab "Pours" next to QA and BOQ.
5. **Joint editing**: gizmo for a cutting plane per level; recompute on release; undo/redo through the existing
   command stack. Pour assignments stored in `.shkp` project file as a new optional section (spec in
   `docs/format/`), so it is open and version-controlled.
6. **Exports**: Excel via existing writer (sheets: Pours, Order sheet, Cycle, S-curve); PDF pour cards via the
   existing PDF path; MP4/WebM via `MediaRecorder` on the canvas (no server); ICS and MS Project CSV are
   text templates.
7. **PlanWin AI Pro hook**: CLI `PlanWinAIPro.exe cli pours model.pwai --max-volume 60 --out pours.json`, so the
   same zoning runs before the 3D model even exists; cad2bim imports the JSON.
8. **Perf target**: zoning + schedule <= 2 s for a G+20 tower, playback at 60 fps with 50k instanced elements.
9. **Docs**: `docs/design/pour-planner.md`, rule defaults in `docs/research/` with IS 456 clauses referenced
   (stripping times cl. 11.3, sampling cl. 15, construction joints cl. 13.4), roadmap update.

MVP (2–3 weeks): auto pour zoning by level + max volume, per-pour m3/steel/formwork table, Excel export, RMC
order text. Then cycle engine, 4D playback, S-curve, pour cards, MS Project export.

### Future update

- **Live progress (4D actual vs plan)**: site engineer ticks a pour as done on a phone (PWA); the model turns
  green and the delay shows against the S-curve; optional photo per pour attached to the element.
- **IDS Gatekeeper link**: pour cards include the model-readiness checks (missing rebar, sleeves, MEP
  openings) for that pour only: "Is L4-P2 ready to cast?" (see `20261014-idea-ids-qa-autofix-bcf.md`).
- **Change Radar link**: a revision that edits a pour already cast is flagged as rework with its cost
  (see `20261014-idea-change-radar-revision-diff-boq-delta.md`).
- **Formwork systems library**: conventional vs aluminium vs table-form with hire rates, reuse counts and break-even
  floor count, to answer "is aluminium shuttering worth it for this G+14?".
- **Weather / monsoon calendar**: avoid casting windows by city rainfall data; temperature-adjusted striking times.
- **Multi-trade 4D**: add masonry, plastering, MEP first-fix and finishes as additional layers with
  simple leads/lags, using the same timeline.
- **Revit Bridge**: write pour id and planned date into native Revit parameters for Revit schedules.
- **AI assistant**: "plan this G+12 with 45 m3 pours and 1 set of shuttering" via the existing action-based
  assistant pattern (Claude/Ollama/offline), numbers stay deterministic.

### Market check

- Synchro 4D, Navisworks Timeliner, ACC schedule/Touchplan, Bentley SYNCHRO: paid, desktop/enterprise,
  generic task-to-element linking; you still build pours, quantities and the schedule by hand.
- Free tools: IfcOpenShell scripts and simple Gantt-IFC viewers exist but animate a schedule you supply; none
  derive pours, RMC orders, bar lists, formwork cycles or S-curves from the structural model.
- Excel + MS Project is what sites really use; Pour Planner feeds those tools instead of replacing them.
- A free, in-browser, structural-aware **pour zoning + RMC order + BBS by pour + formwork cycle + 4D/5D**
  flow is not offered today. It extends the existing cad2bim quantities/BBS/BOQ and PlanWin AI Pro
  revisions and rates, so most data already exists in the toolkit.
