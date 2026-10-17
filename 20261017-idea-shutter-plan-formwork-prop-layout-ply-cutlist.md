# 20261017 Idea: Shutter Plan — auto formwork and propping layout from the 3D model: ply cut-list, joist/prop grid, cuplock count, stripping schedule

## Entry 1

```
date-time:   2026-10-17 (scheduled run, second idea of the day)
idea name:   Shutter Plan — pick a pour (slab + beams + columns) in the 3D model and get a ready-to-send formwork and
             propping layout: plywood sheet nesting with cut-list, secondary/primary joist and prop grid, cuplock /
             scaffold count, beam-bottom and column-box schedule, stripping dates, and a 3D "kit-of-parts" preview
potential:   High. Formwork is 20-25 % of RCC structure cost and the biggest site-labour item, yet the layout is
             still decided by the foreman by eye: "put props at 1 m, add some extras under the beams". The result is
             over-ordering (idle props and ply), under-propping (deflection, a safety risk), and no number the
             modeller can give the planner when asked "how many sets of ply for L5?". Today this needs Doka Tipos,
             PERI Planor / PERI CAD, MEVA, Tekla Structures formwork or Revit add-ins - all paid, vendor-locked
             (they plan only their own system) and aimed at big contractors. A free, in-browser, vendor-neutral
             layout for conventional ply + timber/steel joist + cuplock/prop shuttering - the system 90 % of
             small and mid Indian / Asian / African sites actually use - does not exist. It also extends two
             things already in the toolkit: Pour Planner (formwork reuse cycle) and the BOQ shuttering m2.
             Strong LinkedIn / Instagram fit ("from 3D slab to ply cut-list in 10 seconds", "how many props do I
             need?", "stop wasting plywood"): a 15-second reel of a slab turning into numbered ply sheets and a prop
             grid is easy to share and highly practical.
```

### Description

**The pain (daily life of a BIM modeller / planning engineer / site engineer).**
- The modeller issues drawings; the site asks "how much shuttering for the L4 slab, how many props, how many sheets of
  ply, when can I strip?" - answered with a rough m2 figure from the BOQ and a guess.
- Ply is cut on site with no plan: offcuts pile up, 12-18 % of every 8 x 4 ft sheet is wasted, and the same sheet size
  is cut differently by each carpenter.
- Prop and joist spacing is copied from the last job regardless of slab thickness, floor-to-floor height or beam depth.
  Thick transfer slabs get the same grid as a 125 mm residential slab: unsafe in one place, wasteful in another.
- Stripping time is "7 days, 14 days" folklore, not linked to the pour date, the element type, the span or the actual
  cube-test results.
- Formwork sets (how many floors of ply and props to buy) are decided without seeing the real floor plates.

**The idea.** A **Shutter Plan** mode in cad2bim (browser, files stay on the PC; Structura models can use the same
data) that turns a pour into a formwork kit:

1. **Pick the pour.** Select slab(s), beams and columns of one pour (or one level) in the 3D model - or reuse the
   Pour Planner pour boundaries. The tool reads each element's geometry: slab thickness and outline, beam width /
   depth / soffit level, column size, floor-to-floor height, openings and cantilevers.
2. **Choose the formwork system (editable presets).** Conventional: 12 / 18 mm ply 8 x 4 ft, 50 x 100 timber or
   steel-box secondary joists, primary ledgers, adjustable steel props or cuplock standards. Panel presets for
   steel shuttering sheets are possible later. All sizes, safe prop loads and spacing limits live in one JSON preset
   so the office can match its own kit.
3. **Auto-layout, rule based first, checked by a simple load check.**
   - Slab soffit: ply sheets nested over the slab outline (offcut-minimising nesting, grain direction rule, numbered
     sheet tags S-01...), secondary joists at ply-span spacing, primary ledgers across, props on a grid sized from the
     load (self-weight + construction live load, e.g. IS 14687 / BS 5975 / ACI 347 style defaults, editable).
   - Beam bottoms and sides: bottom + two cheek boards, support heads, props at spacing derived from beam depth and
     width; special notes where the beam is deeper than the slab thickness class.
   - Columns: box schedule by size, clamp or yoke spacing from the concrete pressure rule (height of pour, rate of
     rise), tie-rod count.
   - Camber / level notes for spans above a limit, props kept under beams and at slab openings, clear gaps at
     stair cores and lift shafts.
   - Every prop gets a utilisation % (load on prop / safe working load) - green, amber, red; red fails the plan and
     says why ("prop under transfer beam B12 overloaded: add 2 props or use 40 mm joist").
4. **Outputs for the site.** A **ply cut-list** (sheet number, piece sizes, offcut re-use flag) with a nested sheet
   drawing per pour; **material summary** (ply sheets, joist metres, ledger metres, props, cuplock standards /
   ledgers / jack bases, tie-rods, clamps, nails / tie wire) with total weight for crane / truck planning; the
   **layout plan as DXF + PDF** (prop grid, joist direction, sheet tags, level notes) and a **3D kit preview** that
   explodes into layers (props, ledgers, joists, ply) in the viewer.
5. **Reuse and stripping plan.** Using the Pour Planner cycle, the tool shows the number of formwork "sets" required
   by pour order and adds a **stripping schedule** per element: sides after N hours, soffit after N days by span /
   element type (IS 456 cl. 11.3 style table, editable), with early striking flagged and a **cube-strength override**
   for projects that strip on test results. Props that must stay as re-props are marked.
6. **Cost and BOQ link.** Shuttering m2 per element (already in BOQ), ply sheets, props and labour hours feed the
   costing sheet, so the quantity surveyor sees "BOQ 2,140 m2 vs. planned 1,980 m2 + offcut 11 %" before purchase.
7. **Safety stamp (optional).** A one-page "Falsework check" for the safety officer: loads, spacing, utilisation,
   assumptions and a signature box; the tool never claims it is a certified design, only an engineer's aid.

**Why it is a wow.** The first time a planning engineer clicks one slab and the model "unfolds" into numbered ply
sheets, a prop grid with green utilisation dots and a cuplock count they can email to the store, the 3D model
stops being a drawing tool and becomes the site's kit planner. For the BIM team it is a new paid service line
("formwork planning from the model") with a short learning curve - the carpenter understands a numbered ply sheet
far better than a section drawing.

### Implementation

1. **Geometry reader** (`formwork/geometry.js`, a new module beside the existing exporters): for each pour element
   return plan outlines (slab, beam, column), thickness / depth, soffit and top levels, openings. Reuses the same
   IFC / DXF-to-3D / Structura ETABS data the viewer already holds - no new importer.
2. **Preset schema** (`formwork/presets.json`): ply size and thickness, joist section and permissible span, prop SWL
   curve vs. length, cuplock member capacities, concrete density, construction live load, stripping table. Validated
   with a JSON schema; loaded from the project, overridable by the user.
3. **Slab layout engine** (pure functions, unit-tested): (a) pick joist direction along the shorter span of each
   bay; (b) place secondary joists at spacing = min(ply span limit, joist capacity); (c) place primary ledgers across
   joists; (d) place props from load per prop = tributary area x total load, shrinking the grid until utilisation is
   below the limit; (e) snap props under beams, edges and openings. The same rules go in a small test fixture of
   five typical slab / beam / column combinations with hand-calculated answers.
4. **Ply nesting**: start with guillotine bin-packing (first-fit decreasing) over rectangles produced from the slab
   outline strips, then add re-use of offcuts with a minimum size filter; later an optional better nesting using the
   same cutting-stock solver as Bar Optimizer (shared module = one maintenance path).
5. **Column and beam kit generators**: closed-form rules (pressure p = rho g h or a rate-of-rise formula; clamp
   spacing from p and board span) producing schedule rows rather than geometry - fast and testable.
6. **3D preview and DXF output**: add the kit as extra instanced meshes in the viewer on a toggled layer, with an
   explode slider; DXF writer reuses the layer / text style conventions used by Drawing Autopilot so the formwork
   plan drops into the same sheet set.
7. **Stripping + reuse engine**: take the Pour Planner pour order and dates, compute the number of sets and the
   strike date per element from the editable table; export a date column into the schedule and Excel.
8. **UI**: "Shutter Plan" side panel with the pour picker, preset dropdown, an "Assumptions" accordion, result cards
   (ply sheets, props, weight, planned vs. BOQ m2) and download buttons (Excel, DXF, PDF, ply cut-list CSV).
9. **Validation**: compare against two real, already-built floors from a Computer Help project (actual ply and prop
   counts vs. the tool) and publish the difference; add a "calibrate to my site" factor for waste and damage.

### Future update

- **Panel and system presets** for aluminium formwork and table forms (flying forms), including a "what if I used
  aluminium?" cost comparison against ply.
- **Crane and truck load plan**: group the formwork kit into lift packages with weights and sequence so the
  planning engineer can see crane hook time and truck trips per floor.
- **Live prop tag** with Site QR Pass: scan the prop-count tag to confirm layout photos, stripping date and who
  inspected it.
- **Bill Painter link**: formwork stage % billed automatically when "shuttering" is painted on the pour.
- **Early-age strength stripping**: connect maturity-sensor or cube-test CSVs and let the tool propose the safe strike
  date (the "smart formwork" direction predicted for the industry).
- **Photo check**: upload a phone photo of the actual prop grid; the tool compares spacing against the plan and
  flags missing props.
- **Revision guard** with Change Radar: when the slab outline or beam depth changes, the layout, ply list and
  quantities update with a "what changed" note.
