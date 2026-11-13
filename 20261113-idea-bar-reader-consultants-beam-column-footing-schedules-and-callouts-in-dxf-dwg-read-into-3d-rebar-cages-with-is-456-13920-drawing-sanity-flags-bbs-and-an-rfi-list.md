# 20261113 Idea: Bar Reader — the consultant's beam, column and footing reinforcement schedules and callouts (DXF/DWG) read into 3D rebar cages on the elements you already modelled, with drawing sanity flags (IS 456 / IS 13920), a BBS and an RFI list

## Entry 1

```
date-time:   2026-11-13 (scheduled run, second entry of the day)
idea name:   Bar Reader (reinforcement-schedule reader: the structural drawing's beam schedule, column
             schedule, footing schedule and bar callouts such as "3-T20 + 2-T16 ext", "T8 @ 100/150 c/c",
             "Ld = 47d" are read from DXF/DWG text and tables, matched to the 3D elements by mark, and
             built as rebar sets with hooks, laps, anchorage and confining zones; each mark gets a
             sanity check against the code, and every doubt becomes a numbered RFI)
potential:   Very high for the modeller who is asked for "rebar in the model" (LOD 400, BBS, clash-free
             cages). It is the longest single task after the concrete model itself, and today it is
             re-typing the consultant's drawing into Revit, mark by mark. Hours per tower: an estimate
             from how this work usually runs, NOT measured here; calibrate on one real job before quoting.
             Why it is a real gap (checked 2026-11-13, one-session search, not exhaustive):
               - Revit rebar add-ins (Naviate Rebar, GRAITEC, AGACAD, CADS RC3D) generate bars from
                 settings you type in Revit; none of the results read the consultant's 2D schedule table.
               - ConstructAI (vendor claim, unverified) reads PDF structural drawings and rebuilds rebar
                 as native Revit; it is a paid, cloud-style service.
               - In this repo: Detail Drop starts from ETABS results, BBS Weaver from an Excel BBS, Slab
                 Sketch designs slabs itself. **Nobody reads the drawing the consultant actually issued.**
                 Plan Tracer lists "detailing sheets to bar data" as a future line (this idea is that
                 line, made a product).
             Trend honesty: a 2026 newsletter says AI is entering BIM tools and robots are tying rebar on
             live jobs (Nomic, March 2026); "AI reads drawings" is a loud topic. I found no LinkedIn post
             about it directly, so check LinkedIn before posting a claim.
             The wow: open the consultant's DXF with the model. Click "Read bars". Beam B12's schedule row
             flies into a glowing cage inside the 3D beam; hoops pack at the supports, bars hook into the
             column core. A tally counts up: "412 marks read, 398 built, 14 questions". Click one of the 14:
             the schedule row lights on the drawing and the element lights in 3D, with a one-line reason.
```

### Description

**The pain.**
- The structural consultant's drawing set is 2D: a beam schedule (one row per mark: size, top and bottom
  bars, stirrups, extra bars at support), a column schedule or chart, footing schedule, slab and wall
  notes. The BIM modeller reads each row and builds the cage by hand. (From the team's description of
  the work; the hours are an estimate.)
- Rows are inconsistent on paper: a hoop spacing that is wider than the code allows in the plastic hinge
  zone, a top-bar count that does not fit the beam width at the cover, a footing mark missing from the
  schedule. These surface when the cage will not fit the model, or on site. By then the BBS is wrong.
- After a revision, the schedule changes in 30 rows and nobody knows which cages to redo.

**What exists today (checked 2026-11-13; not exhaustive).**
- Revit add-ins listed above generate cages from settings, one element type at a time. Their users still type
  the schedule in (my reading of the vendor pages and the search; not tested).
- MaxiBIM reads DWG lines to make families (geometry, not rebar).
- ConstructAI reads PDF drawings to rebar in Revit (vendor claim, paid).
- cad2bim already reads **tables and schedules from DXF/DWG** (CH v2 schedules, `*T` table blocks in
  `core/src/tables.rs`) and builds **rebar cages** on the Revit side and reads rebar from IFC. The two halves
  exist; the join is missing.
- Not claimed as unique worldwide. The claim that fits: a free, in-browser, no-upload route from the
  consultant's DXF/DWG schedule to checked cages and a BBS, tied to the model you already have.

**The idea in one line.** Read what the consultant wrote; build it; say plainly where the drawing does not
add up.

### How it works (user view)

1. **Open** the structural DXF/DWG next to the 3D model (the pipeline already holds the elements and marks).
2. **Pick the sheets**: the tool lists the tables and note blocks it found ("Beam schedule L2-L5", "Column
   chart", "Footing schedule", "General notes: cover, grades, lap rule"). Tick or untick each.
3. **Check the reading** in a split view: the table on one side, the parsed row on the other, shown as a plain
   sentence ("B12: 300 x 600, bottom 3 bars 20 mm, top 3 bars 16 mm, 2 extra 16 mm top at support,
   stirrups 8 mm at 100 mm in 2d zones, 150 mm elsewhere"). Fix any cell by typing; the fix is kept as a rule
   for that consultant.
4. **Read bars**: cages grow in the 3D elements. Each mark is built once as a type and placed on every
   element with that mark, with the grid and level it sits at.
5. **Review the questions**: a list with a reason and a pin on both the drawing and the 3D element. Answer one
   by choosing (assume the schedule, assume the code, mark as RFI); an RFI goes out as BCF or a table.
6. **Outputs**: rebar in the .c2b and IFC (IfcReinforcingBar, as Revit writes it), a BBS (the existing BBS
   module and Excel writer), a steel kg per m3 per mark, an RFI sheet, and the reading kept so a revision
   re-reads only the changed rows.

### What it reads (first release, narrow on purpose)

- **Beams:** mark, b x D, top/bottom bar counts and diameters in a table, extra bars at support and mid-span,
  stirrup diameter and the two spacings, side face bars, with the common Indian shorthand "3-T20", "2#16",
  "T8 @ 150 c/c", "2L-T8", "8 dia".
- **Columns:** mark, size, bar count and dia (or a chart's rows), tie dia and spacing, confining zone
  spacing, per level range.
- **Footings:** mark, size, depth, bottom mesh in each direction.
- **Notes:** clear cover by element, grades, lap length rules ("Ld = 40d for Fe500 in M25") if written;
  otherwise the code tables, marked "assumed" on every output.
- Not in the first release: slab bars from plans, walls, bent-up bars, PT, round columns with spirals.

### Rules-first, AI second

1. **Rules do the reading.** A small grammar for bar callouts, table-header synonyms ("Bottom", "BT", "Main
   steel"), and merged cells. Deterministic, testable on the team's own drawings, runs on the device.
2. **Cage building is geometry, not guessing.** Standard cover, hook (135 degrees for ties, IS 2502 style),
   laps staggered, anchorage into the column core, ties tight in the confining zone. Constants come from the
   notes or the user, never silently (the same rule as Tendon Tracer).
3. **Optional AI for the messy rows only** (scanned image text, free-form notes): with the user's own key or
   their own agent over MCP, as already decided for cad2bim. The AI proposes a parsed row; a person confirms.
   Nothing leaves the device unasked.

### Sanity checks on the drawing (the part that sells it)

Each row passes small, explainable tests. A failed test is a question, not a verdict (the designer signs):
- Bars fit the width: count x dia plus spacing and cover against b (a 230 mm beam with 5 bars of 20 mm in one
  layer does not).
- Minimum and maximum steel ratio (IS 456 cl. 26.5), side face bars over 750 mm depth.
- Stirrup spacing limits, and the closer spacing in the end zones for ductile detailing (IS 13920) where the
  drawing says ductile.
- Column bar count vs minimum steel (0.8 % of area) and tie spacing limits.
- Mark in the drawing but not in the schedule, mark in the schedule that no element carries, two rows for the
  same mark that disagree.
- Schedule size vs the modelled element's size (the model says 300 x 600, the table says 300 x 650).
Limits are quoted from the code and shown with the clause; the owner or the structural engineer confirms
every clause before the output says "checked". (I have not re-verified the clause numbers here; check
them against the standards before building.)

### Implementation

Where it goes: the CAD core reads text and tables (`packages/cad-engine`); a new "bar callout" grammar and a
mark-to-element matcher sit beside the CH v2 reader in `packages/pipeline/core/src/v2/` (the schedule reading
is already there for heights and grades); cages are built as in the engine's rebar and BBS modules
(`packages/engine/src/bbs`); the app adds a "Read bars" panel in the cad2bim tab. This note only proposes; no
source was changed.

**Phase 1 (about 2 weeks, estimate).** Beam schedule only. Grammar for callouts, one table layout, match by
mark, cages with fixed cover and hooks, a BBS. A list of unparsed cells. Test on two of the team's real
drawings (kept out of git).
**Phase 2 (about 2 weeks).** Columns and footings. Sanity checks with clause references. The split view with
cell fixes kept as rules. RFI list as BCF.
**Phase 3 (about 2 weeks).** Confining zones, laps and anchorage into joints; clash flags where cages meet at a
joint (the repo's Joint Congestion Doctor idea). Revision re-read of only the changed rows.
**Phase 4.** PDF schedules (through the Plan Tracer route), optional AI parse with the user's key, Revit
native rebar through the add-in.

**Risks.**
- Every consultant draws schedules differently. Start with a template of five layouts and the per-consultant
  rules; do not promise "any drawing".
- A wrong cage looks convincing in 3D. Every output says "from the drawing, not designed"; the sanity
  checks and the assumed values are listed on the BBS sheet. The structural engineer signs.
- DWG text can be exploded or stored as lines. Report what could not be read, with a count, rather than
  skipping it quietly (the Trace Lens idea does the same for the model).

### Future update

- Read the same schedule back from the model: **write the schedule** from edited cages, so the drawing and
  the model stay one (Drawing Autopilot).
- Feed Bar Optimizer (laps, cutting plan, carbon), Cage Manual (fixing steps) and Cover Cop (cover checks).
- Mark Merge: suggest consolidating similar marks after reading (fewer cages to build and check).
- Compare two consultant revisions row by row and show "30 rows changed, 41 cages to redo" (Change Radar).
- Learn per consultant: the next drawing from the same office needs fewer cell fixes.

### Sources

- [Naviate Rebar (Symetri), settings-driven rebar in Revit](https://store.symetri.com/naviate-for-revit/naviate-rebar/)
- [AGACAD column reinforcement tool](https://agacad.com/?p=2617)
- [CADS RC3D for Revit (PDF)](https://cads.co.uk/wp-content/uploads/2019/07/CADS-RC3D-for-Revit.pdf)
- [ConstructAI (vendor page; claims unverified)](https://www.aecplustech.com/companies/constructai)
- [Nomic, AI in the Built World, March 2026 (newsletter)](https://www.nomic.ai/newsletter/ai-in-the-built-world/march-2026)
- [Brightsurf summary: AI framework turns paper drawings into 3D digital twins (beams and columns, not rebar)](https://www.brightsurf.com/news/LDEM26G8/new-ai-framework-transforms-paper-drawings-into-3d-digital-twins-advancing-smart-city-construction.html)
- In this repo: `docs/design/ch-template-v2.md` (schedules), `docs/design/rebar.md` (rebar in IFC), `docs/design/bbs.md`
