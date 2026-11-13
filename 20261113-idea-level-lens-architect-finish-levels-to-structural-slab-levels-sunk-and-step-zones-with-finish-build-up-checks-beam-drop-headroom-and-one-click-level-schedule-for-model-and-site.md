# 20261113 Idea: Level Lens — architect finish levels (FFL) turned into structural slab levels (SSL) with sunk and step zones, finish build-up checks, a level map for the site, and one-click write-back to the model

## Entry 1

```
date-time:   2026-11-13 (scheduled run)
idea name:   Level Lens (read the architect's finish floor levels per room from the plan DXF or a
             level table, subtract each room's real finish build-up to get the structural slab top
             level, find every sunk, step, ramp and threshold zone, check the build-up fits, build the
             slab-level zones in 3D, and print a level map and schedule that the site can pour to)
potential:   High, daily use. Every floor of every job has toilets sunk 50 to 150 mm, balconies and
             terraces lower than the room, a lobby with a different finish, a ramp, a pooja or kitchen
             step. Structural modellers get these as spot notes ("FFL -75", "sunk 150") scattered on the
             architect plan. A missed or wrong level is a chipped slab or a re-pour. Checked the 103
             ideas in this repo (searched "FFL", "SSL", "finish level"): none turns finish levels into
             structural levels. Drain Line, Rain Run and Basement Box each need sunk or sloped
             levels, but each invents its own; none reads the architect's.
             Honest on "news": a quick search (2026-11-13) found no LinkedIn or Instagram trend for
             this topic, and I cannot read LinkedIn from here. The case is the daily pain, not virality.
             The wow: drop the architect's floor plan. The slab in 3D turns into coloured level zones
             (main floor 0, toilet -75, balcony -50, terrace -100 with slope). Click "Check": red on
             a sunk zone where the beam drop leaves no room for the sunk, on a door where the
             threshold does not close. Then one click: a level map DXF and a table for the site.
```

### Description

**The pain.**
- The architect's plan carries FFL (finish floor level) as small notes and hatches; the structural
  drawing needs SSL (structural slab level) = FFL minus tile, screed, bedding and waterproofing. The
  modeller subtracts by hand, room by room, from a list of rules found in a spec or a chat.
  (From the team's experience; not measured here.)
- Build-up differs by room: 20 mm tile on 30 mm screed in a living room, 12 mm tile on 60 mm
  waterproofed screed in a toilet, 25 mm stone on a lobby. One wrong default shifts the whole zone.
- The slab then gets modelled flat and the sunk is "noted". On site the shuttering carpenter and
  the plumber find the real level from a sketch.
- Mismatch shows up at the end: sunk meets a beam drop, a door threshold is higher than the room
  beyond it, the balcony slab is not lower than the room so water comes in (a classic handover snag).

**What exists today (checked 2026-11-13; not exhaustive).**
- Revit has levels, slab "Shape Editing" and "Offset from level". It does not read an architect
  drawing's level notes, and does not calculate SSL from finish layers. Paid plug-ins help with
  floor finish schedules, not structural levels.
- General BIM coordination tools compare geometry; none turns a 2D note like "FFL -75" into the
  structural level.
- In this repo: Wall Wizard and Ceiling Composer do finishes and ceilings. Drain Line,
  Rain Run, Basement Box and Slab Sketch assume a level. Flush Fit checks architecture against
  structure geometry, not levels. Plan Tracer / Layer Whisperer read plans. **Nothing reads level
  notes or computes SSL.** Level Lens is the shared level source those tools should use.
- Do not claim the market has nothing: Dynamo scripts exist for single jobs. What is not obvious is
  a free browser tool that goes from drawing notes to a checked level map without upload.

**The idea in one line.** Read the architect's finish levels, subtract the real build-up, give every
room a structural level, and prove it fits.

### How it works (user view)

1. **Drop** the architect floor plan DXF (and optionally a finishes schedule from Excel).
2. **Read.** The tool finds level notes (text such as `FFL -75`, `+0.000`, `SUNK 150`, `RL 100.450`),
   spot-level blocks, and room names or hatches, and joins each note to its room by position. Notes it
   cannot place go in a list for one-click assigning, not guessed.
3. **Build-up table.** A small table per room type (Living, Toilet, Kitchen, Balcony, Terrace, Lobby,
   Stair landing, Parking). Starts with common defaults shown clearly as *defaults to edit*; the
   project's own spec is the authority. Saved per project and reused on the next floor.
4. **Compute.** SSL = FFL - (finish + bedding + screed + waterproofing). The structure level is
   the floor datum plus that. Zones with the same SSL merge into slab level zones; the edge between
   two zones becomes a step, with its height.
5. **3D.** Slab zones show as coloured depth steps. Optionally writes IFC slab geometry with the
   steps (kerbs/upstands) so Revit opens it.
6. **Check** (every item pinned, goes to BCF):
   - sunk deeper than the slab thickness less cover and bar clearance (slab cannot take the sunk);
   - sunk meets a beam or its drop: remaining depth or width too small (opposite case: beam in a
     toilet that needs a bigger sunk);
   - door threshold step bigger than the user's limit, or no step where the water must stop;
   - balcony/terrace not lower than the room, or no slope where one is expected;
   - step height out of line with its neighbour (looks like a typing error);
   - the same room with two different levels on two sheets.
7. **Output.** A level map DXF (zones, step lines, SSL and FFL labels), a room-wise level table
   (Excel), and a one-line list "pour to" for the pour card. Optional: a sheet for the carpenter.

### Why it fits cad2bim

- Plan reading and layer mapping already exist (CAD engine, Layer Whisperer); text and hatches
  are plain entities. Room joins reuse the pipeline's closed-loop room finding.
- Slab geometry and IFC writing exist; zoned levels need a step on the slab outline, not a new
  object class.
- Becomes a shared source: Drain Line, Rain Run, Basement Box, Wall Wizard and Slab Sketch can
  read the level zones instead of each guessing.
- Nothing leaves the device; architect drawings are commercial files.

### Implementation

**Phase 1 (about 1.5 weeks, estimate).** Read FFL/SUNK text from a DXF, join to room loops, build-up
table with editable defaults, SSL per room, coloured zones in 3D, level table to Excel.
**Phase 2 (about 2 weeks).** Step edges and kerbs in IFC. Checks: sunk vs slab thickness, sunk vs
beam, threshold step, balcony fall. BCF export.
**Phase 3 (about 1.5 weeks).** Level map DXF and pour card line. Hand-off to Drain Line, Rain Run
and Slab Sketch. Revision compare through Change Radar (a level changed, which rooms, what
quantity).
**Phase 4.** Read levels from a PDF plan (through Plan Tracer) and from a finishes schedule.

Where it goes: the CAD core reads text and hatches (`packages/cad-engine`); level logic sits in the
pipeline beside room finding; the app adds a "Levels" panel in the Structure ribbon. This note only
proposes; nothing in the source was changed.

**Risks.**
- Notes are written in many ways ("-75", "(-75)", "75 BELOW FFL", metric or RL). Start with the team's
  last three jobs, kept out of git, and show unread notes, never silently skip.
- Build-up defaults can mislead. Show them as defaults on every output and require one confirm per
  project; the architect's spec decides.
- Waterproofing and slope screed thickness change along a fall (Rain Run); treat Phase 1 as flat
  zones and say so.

### Future update

- Sloped zones from Rain Run (terrace falls) as true plane zones with a pour-thickness map.
- Pour check on site: a phone laser/level reading per zone compared to the SSL (Cover Cop style
  checklist).
- Quantities: extra concrete or screed volume per zone into the BOQ (Rate Rocket, Measure Right).
- Level compare against a live Revit model through the bridge: show rooms where the modelled slab
  top differs from the computed SSL.
- Bylaw/handover link: step-free route levels feed Access Pass.

### Sources

- Quick web search (2026-11-13) for BIM level-coordination trends: no direct LinkedIn or Instagram
  trend found; it can only be checked on LinkedIn itself. General context only:
  [Constructible BIM, NBM&CW](https://www.nbmcw.com/product-technology/technologies-digitilization-software/constructible-bim-empowering-civil-engineers-to-build-with-confidence.html),
  [Structural BIM process in practice, NBM&CW](https://www.nbmcw.com/product-technology/technologies-digitilization-software/structural-bim-process-in-practice.html)
