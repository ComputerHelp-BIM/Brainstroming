# 20261113 Idea: Load Loader — masonry wall, parapet and finish loads counted from the 3D model and put on the exact beam or slab as ETABS-ready line and area loads, with opening deduction, a "wall with no beam under it" finder, and a model-vs-ETABS load audit

## Entry 2

```
date-time:   2026-11-13 (scheduled run, second entry of the day)
idea name:   Load Loader (read every non-structural wall, parapet, cladding, finish and fixed
             equipment in the architect-aligned 3D model, work out its weight with the openings taken
             out, drop it on the beam or slab that really carries it, and give ETABS a load table
             it can take in, plus a list of walls that stand on a slab with no beam under them)
potential:   High, daily use, and a quiet source of structural error. Every ETABS / SAFE model needs
             "wall load on beam" (a brick wall's kN/m) and "floor finish and partition load" (kN/m2).
             Today the engineer or modeller measures them by hand from the architect plan:
             wall length x height x thickness x density, less doors and windows, then types the
             number onto each beam by label. It is slow, and it goes stale the moment the architect
             moves a partition. Checked the 104 ideas in this repo (searched "wall load", "line load",
             "superimposed", "masonry load"): Load Path X-Ray takes column loads down to footings and
             Mark Merge mentions loads in passing; none works out wall and finish loads from the
             model or writes them for ETABS. Detail Drop, Quake Preview and Pad Packer all need
             these loads but assume someone typed them.
             Honest on "news": a quick search could not confirm a LinkedIn or Instagram trend for
             this. The case is the daily time saved and the real risk of a missed wall load.
             The wow: select the model, press "Load it". Every beam lights up with its wall load in
             kN/m (heavy walls red, light green). A parapet turns orange, a wall standing on a
             slab with no beam shows a flashing marker with "230 mm wall, 3.1 m long, 18 kN, no beam".
             Press "Send": the loads land in the ETABS file, with a table showing the working.
```

### Description

**The pain.**
- ETABS gives dead load of beams, columns and slabs by itself. It does not know about brick or block
  walls, parapets, cladding, water proofing, tiles, screed or false ceilings. The engineer adds
  them by hand. (From team experience; not measured here.)
- The usual hand method: "230 mm wall x 3 m high = 13 kN/m, less 20% for openings". A flat 20% is a
  guess. A wall with a full-height window has 60% out; a blank wall has 0%.
- Walls get moved in every architect revision. The load table in ETABS is not updated, or is
  updated for some floors only.
- Walls built on a slab with no beam under them (a toilet partition, a kitchen platform wall) need
  extra steel or a hidden beam. They are found on site or not at all. This is a well-known cause
  of cracks in slabs and one of the first things a proof checker looks at.
- Heavier-than-assumed walls (fly ash, AAC, solid block, stone cladding, parapet with a water tank
  behind) change beam and column sizes only after the analysis is already "done".

**What exists today (checked 2026-11-13; not exhaustive).**
- Revit analytical model and some paid add-ins can carry model loads to analysis software, but the
  usual result is area loads or per-element loads set by hand; they do not look at which beam
  carries which wall, nor take out the openings of the actual doors and windows.
- ETABS has a manual "Frame line load" and area "Wall load" assigned by hand. Nothing in ETABS reads
  the architect's drawing to find them.
- Paid structural-BIM suites do have wall-load transfer, but they sit in a full licence and a
  Revit-to-analysis workflow. I have not found a free, browser-based, no-upload tool that does it
  from an IFC and writes an ETABS-ready table. If you know one, this idea should be re-checked.
- In this repo: Load Path X-Ray (column takedown), Detail Drop (rebar from ETABS forces) and
  Twin Check (design vs BIM) work after the loads exist. **Nothing creates the loads.** Load Loader
  is the missing first step of that chain.

**The idea in one line.** The model already knows every wall; let it tell ETABS how heavy each one is
and which beam carries it.

### How it works (user view)

1. **Open** the project (IFC or .c2b) with structure and the architect's walls (from Drawings → 3D,
   Every discipline in one project, or an IFC import). Walls that are not structural become
   "loads", not members.
2. **Unit weights table.** A small table of editable defaults: brick 230/115 mm, AAC 200/150/100,
   solid block, stone/ACP cladding, plaster each side, parapet, tile + screed per room type,
   false ceiling, waterproofing and soil on terrace, water tank on slab. Shown as *defaults to edit*;
   the project's spec and IS 875 Part 1 values are the authority. Saved per project.
3. **Read walls.** For each wall: length, thickness, height (storey height minus beam depth under
   it, or to the slab above when there is none), material, plaster, and its hosted doors, windows and
   openings (cad2bim already hosts them as Revit does), so each opening is taken out exactly.
4. **Find the carrier.** The wall footprint is intersected with beams and slabs below:
   - on a beam: its whole length or the part overlapping it becomes a **line load** (kN/m) on that
     beam, split when two beams share the line;
   - on a slab with no beam: marked **wall on slab**. The tool shows the load as a line over the slab and
     proposes: a hidden beam, extra bottom bars below it (indicative), or a lighter wall;
   - at a wall end hanging past the beam (a cantilever stub) the overhang is flagged, not hidden.
5. **Area loads.** Floor finish, ceiling and terrace build-up by room, from the same table, give
   kN/m2 per slab panel. Live loads stay as the engineer's choice (IS 875 Part 2 type per room is a
   dropdown), not guessed.
6. **See it.** Beams coloured by kN/m. Click a beam: the walls that load it, each with length,
   height, openings out, and weight, as a short sum on screen. Click "Why?" on any number and the
   3D proof highlights the wall and its openings.
7. **Send.** Writes the load table:
   - **ETABS**: an .e2k text block of frame line loads and area loads by label, or a CSV import for the
     bulk-load table (version-checked against the user's ETABS; the format is read from the user's own
     file, not assumed).
   - **Excel**: the same table with the working per row, ready for the calc folder.
   - Back into the IFC as a load pset on the wall and on the carrier beam, so Revit and others see it.
8. **Audit mode.** Drop the ETABS model the team already analysed and the tool compares the loads typed
   in it to the loads it computes from the model. It shows a list: "Beam B214 floor 6: ETABS 9 kN/m,
   model 13.4 kN/m (+49%)", with a status of "missing" (no wall load at all), "stale" (the wall moved),
   or "ok within 10% (editable)". This is the part most teams do not have.

### Checks it can make (each is pinned and goes to BCF)

- wall standing on a slab, no beam, longer than a set length;
- wall load over a beam larger than the beam was sized for (a simple capacity screen, not a design);
- parapet or heavy cladding at a slab edge with no edge beam or upstand;
- terrace with a build-up heavier than the one in the analysis model;
- a wall in the architect's model that has no carrier on any floor below (floating wall at a transfer);
- a wall assumed in ETABS that no longer exists in the model (load left behind after a deletion).

### Why it fits cad2bim

- Walls, doors, windows and their hosting already exist in the 3D model (0.101); no new geometry
  is needed, just reading it.
- Beam and slab geometry plus overlap tests are the same ones Flush Fit, Joint Congestion Doctor and
  Load Path X-Ray need; one shared "who carries what" function serves them all.
- ETABS text files are read by the planned ETABS import; writing loads is a small second step.
- Nothing leaves the device: architect models and analysis files stay in the browser.
- It is the first link of a chain already proposed here: **Load Loader → Quake Preview → ETABS
  → Detail Drop → BBS**. A modeller can say "from drawings to bar schedule" and mean it.

### Implementation

**Phase 1 (about 1.5 weeks, estimate).** Unit-weights table, wall volumes with openings out, carrier
search against beams, per-beam kN/m, colouring in 3D, Excel table with the working.
**Phase 2 (about 2 weeks).** Wall-on-slab finder with fix suggestions, parapet and cladding rules,
area loads from finishes, BCF export, load pset written to IFC.
**Phase 3 (about 2 weeks).** ETABS write (.e2k block and CSV) tested on 3 of the team's finished
projects; audit mode reading an ETABS model and listing differences.
**Phase 4.** Live link: change a wall and see the beam load change; Change Radar adds "load changed
on 14 beams" to each revision note.

Where it goes: reading and overlap logic in the engine beside the QA and BOQ code; the unit table and
the audit panel in the app under the Structure ribbon; the ETABS writer next to the planned
ETABS import. This note only proposes; nothing in the source was changed.

**Risks.**
- The load number depends on the unit weights. The tool shows defaults loudly, and a person confirms
  them once per project. It is a loading calculator, not a design stamp.
- A wall on a deep beam may load only part of it by arching; Phase 1 uses the plain full line load
  and says so (safe side).
- ETABS versions differ in text format. Start with the version the team uses, read the user's own file.
- Walls modelled loosely in the architect's IFC (wrong height, no openings) give wrong loads; the
  tool lists walls missing height or material, not guesses.

### Future update

- Load takedown on a transfer girder with several walls and point loads from columns above.
- Wall material swap what-if: "AAC instead of brick" shows the change in kN/m, column axial load,
  steel kg and carbon (links Carbon Lens, Mark Merge, Rate Rocket).
- Hoarding, façade and glazing wind-and-weight hand-off to the façade vendor (Skin Cutter).
- Phone check on site: photo of a built wall thickness compared to the loaded one.
- SAFE and STAAD writers, after ETABS.

### Sources

- Quick web search (2026-11-13) could not confirm a LinkedIn or Instagram trend for this topic;
  that can only be checked on those apps. General practice (IS 875 Part 1 unit weights, hand
  calculation of wall loads) is the author's knowledge of Indian structural design practice, not a
  measured number from the team.
