# 20261122 Idea: Maquette Maker — The open 3D model becomes a physical scale model: pick a scale and a style, get a watertight STL for a 3D printer, or numbered laser-cut DXF sheets (floor plates, walls, cores) with slots and an assembly guide. In the browser, nothing uploaded.

## Entry 1

```
date-time:   2026-11-22 (scheduled run, UTC)
idea name:   Maquette Maker (IFC / .c2b / drawing-built model to scale-model STL and laser-cut
             DXF, with a "peel the floors" cutaway and an assembly guide; on the device)
potential:   Medium-high for BIM teams that pitch, win and present (client meetings, society
             and authority presentations, sales offices, competitions). High "wow" per minute
             of work: a model on the table beats a screen. Medium for daily modelling speed;
             this is a presentation-time saver, not a modelling accelerator. Say so plainly.
             Time saved: NOT measured. Today the job is a half to one day of manual clean-up
             (see "The pain"). Claim hours only after the first test below.
             First test: take 3 finished models (one tower, one low-rise, one with a podium),
             kept out of git. For each, time a person doing it today (Revit view to DWG, clean
             up, scale, lay out sheets, number parts) and time Maquette Maker. Then print or cut
             one and count parts that did not fit or hold. Do not quote hours until this is done.
             Market check (2026-11-22). One standard web search only. Found: Autodesk has a free
             STL exporter add-in for Revit (not for Revit LT); paid STL exporters exist
             (Cadsum, GeoExporter); batch exporters list STL and DXF as formats; forum threads
             say STL carries no scale or units (set in the slicer) and that the laser-cut DXF
             step is "pretty manual"; one Dynamo script for it was called outdated; some firms
             sell the conversion as a service. These are forum posts, marketplace listings and
             vendor pages; I did not verify prices or how far any tool goes. What I did NOT
             find: one free tool that takes the model to a scaled, printable-or-cuttable,
             numbered kit with an assembly guide. "I did not find one", not "none exists".
             Trend: 3D-printed buildings and architectural models are a regular LinkedIn and
             Instagram topic. I could not measure counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Model Capsule / Room Link: a client sees the model on a screen. This puts it
                 on a table, for the meetings where there is no screen or a client who wants
                 to hold it.
               - Drawing Autopilot (3D to 2D DXF set): the same idea of cutting the model into
                 2D, but for site drawings. Here the 2D is parts to cut, with slots.
               - Kit Cutter (precast panels): panels for building; here parts for a model.
               - Roadmap "More file formats" lists STL out as an idea; this is the first
                 real use of it.
             The wow: open a tower model, press "Maquette", choose 1:200. A cutaway tower
             appears in 3D with the floors pulled apart and numbered. Press "Cut sheets": a
             DXF with 12 floor plates and the core nested on a 600 x 300 mm sheet, each part
             numbered, with slots where the core passes through. A second button gives a
             watertight STL, split to fit a 220 mm printer bed.
```

### Description

**The pain.**
- A Revit or IFC model is not a printable solid: walls, slabs and columns overlap, doors and windows
  leave holes, and tiny parts (rails, tiles) make the mesh heavy.
- STL has no units, so the scale is guessed in the slicer.
- For laser cutting, a modeller exports a view and cleans lines by hand: closing outlines, removing
  hatches, making parts fit a sheet, adding numbers.
- Nobody writes the assembly order, so the model maker guesses which floor goes where.

**The idea.** Five moves.
1. **Pick scale and style.** 1:100 to 1:500; style is *massing* (outer shell and floors), *shell and
   structure* (columns and cores visible) or *cutaway* (floors lifted). Parts under a size at that
   scale (railings, bars, small families) are dropped by a stated rule, listed in a "left out" panel.
2. **Make it one solid.** Union overlapping members, close door and window openings to a simple
   recess or fill, remove hidden inner faces and report any open edges. Output is a watertight mesh.
3. **STL out.** Written at real millimetres at the chosen scale (so the slicer needs no guess), split
   at a chosen bed size with pegs so the halves line up, base plate with a name and the north arrow.
4. **Laser-cut out.** Floor plates, core and shear walls sliced into flat parts with slots and tabs
   at the intersections; parts nested on a sheet of a chosen size and thickness (foam board, MDF,
   acrylic), kerf offset set by the person, cut lines and engrave lines on separate layers, each part
   numbered. DXF only, so any cutter software reads it.
5. **Assembly guide.** A one-page PDF with an exploded 3D view, part numbers and the order of stacking.

**What the person sees.**
- *Maquette* tab with the scale, style, sheet size and thickness.
- A 3D view of the cutaway with part numbers; click a part to see which cut sheet it is on.
- A "left out" list and a fit report ("2 parts thinner than the sheet; widen to 1.5 mm?").
- Buttons: STL, Cut sheets (DXF), Assembly guide (PDF).

### Implementation

- **Where it lives.** A new module in `@cad2bim/engine` (geometry in, files out), using the geometry
  the viewer already holds. A panel in the app. No server; nothing leaves the device.
- **Mesh work.** Boolean union is the hard part. Start with the cheap route: per-storey slabs and walls
  extruded from their footprints (the model already knows these), union as 2D polygons per storey,
  then extrude. This avoids heavy 3D booleans and stays fast on big models. Check the licence of any
  geometry library before adding it (MIT, Apache-2.0, BSD, ISC only).
- **Slicing for the cutter.** Cut each storey at its plan height to get the 2D outline; add slots
  where the core crosses a plate. Nest with a simple shelf-packing first; better nesting later.
- **Scale and units.** STL in millimetres at the chosen scale; the DXF header carries the units.
- **First version (smallest useful).** Massing style only: floor plates + outer wall shell, DXF
  sheets and STL. Ship that, then add cutaway and shell-and-structure.
- **Test.** Part count and total area of parts compared with the model's storey areas; every output
  mesh checked for closed edges; one real cut and one real print on a model kept out of git.
- **Version.** New minor version of the engine and the app; changelog, README and guide entry as
  CLAUDE.md asks. Not done here; this repo holds ideas only.

### Future update

- Terrain base with contours cut as stacked layers.
- Neighbouring buildings as plain blocks from the site context idea in the roadmap.
- Colour-coded parts (print in multi-material, or paint guide per part).
- Lit models: LED slot and a wire channel for presentation tables.
- A "client version" with the floors labelled by use (from Plot Pilot / Area Statement output).
- Send the cut file to a nearby fabricator as a quote request, only when the person presses send.
