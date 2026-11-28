# 20261128 Idea: Green Load — the landscape drawing (trees, planters, lawns, pools, play areas, fire-tender lane) becomes a podium and terrace load map, drop-slab recesses, soil and drainage layers, 3D trees and a consultant-ready package, before the podium slab is poured to the wrong depth

## Entry 1

```
date-time:   2026-11-28 (scheduled run)
idea name:   Green Load — "The landscape architect's drawing arrives after the podium is designed. Find what it
             does to the slab in the first hour, not at the pour."
             Drop the landscape DWG/DXF (or PDF) for a podium, terrace or roof. Green Load reads the planting and
             hardscape zones (lawn, shrub bed, planter box, tree pit, paver, water feature, swimming pool, kids'
             pool, play area, fire-tender lane, DG/AHU pads) and builds, per zone, the full build-up as 3D layers:
             structure, waterproofing, protection screed, drainage cell, filter fabric, soil or growing medium,
             finish. It then adds up the dead and live load per square metre for each zone and paints a LOAD MAP
             over the podium slab (kN/m2, green / amber / red against the load the slab was designed for).
             Trees are the wow: each tree is placed in 3D at its mature size, with its root ball, its pit depth
             and its weight (wet soil included), and the slab beneath it is checked as a point load. Where the soil
             is deeper than the slab's drop allows, it proposes a drop-slab recess, a raised planter, a mound
             over a beam or a lighter growing medium, with the load and cost change of each option. The output is
             a clash-free "landscape loads" package for the structural consultant (zone table, point loads,
             recess outlines as DXF, a 3D model), the waterproofing/drainage BOQ for the contractor and a
             client picture of the garden.
potential:   High in India's apartment townships and commercial podiums, and in any city where roof gardens are
             sold as a feature; the same problem repeats on every such project.
             The daily pain: the structure is designed with a podium live load and a nominal soil allowance. The
             landscape consultant is appointed later and draws 600 to 1200 mm of soil, mature trees, a pool and
             a fire-tender lane over it. Every one of those either overloads the slab, needs a recess that nobody
             allowed for in the slab levels, or fouls a drain, a lift pit or a beam below. The BIM modeller is the
             person asked to "fit the landscape in the model", and does it by eye: a few extruded boxes, no loads,
             no layers, no check. The cost of getting it wrong shows late: a podium slab cast with no recess
             where the tree should go, then chipped out, or planters that leak into the parking below.
             My estimate (not measured; to be tested on three past podiums): the first pass, from the landscape
             drawing to a load map and a consultant package, takes under an hour instead of two to three days of
             hand counting and coordination. It also removes the "which drawing is current" argument, because
             every zone points back to the drawing it was read from.
             Market: to my knowledge (not verified for this run) the landscape tools (paid CAD and BIM landscape
             add-ons, plant libraries and rendering tools) draw gardens but do not turn them into structural load
             maps, and the structural analysis tools do not read landscape drawings. Revit has planting and
             topography tools, and neither gives a load take-down per zone. A free, in-browser, on-device tool
             for this link between landscape and structure is not one I know of.
             Trending: "this tree weighs 8 tonnes on your slab" and "your rooftop garden, with its weight,
             in 3D" are visual and surprising, and sustainability and green-roof posts do well with architects and
             developers on LinkedIn and Instagram. Treat that as a signal of interest, not a measured trend.
```

### Description

**The pain (a BIM modeller's day).**
- The podium slab is 200 to 250 mm thick, with a drop of 600 mm kept for the garden over most of its area. Then the
  landscape drawing arrives: a lawn here, a 1.2 m planter there, a mature rain tree in the middle, a 1.2 m deep
  kids' pool on the third tower's podium, and a 6 m fire-tender lane across the lot.
- The modeller is asked three questions that nobody can answer quickly. How heavy is it? Where does the slab need a
  recess? Does the water end up where it should?
- The weight is not only the soil. It is the wet soil, the drainage layer, the paving and the planters, plus a tree
  that will grow for 20 years. The live load on a garden zone is not the same as on a pool deck, a play area or a
  fire lane. Each of those has its own load, and the consultant needs them as a map, not as a note.
- A recess, a raised planter or a mound is the usual answer for a deep tree, but each changes the slab, the beams
  below, the waterproofing and the finish levels. The earlier the modeller shows it, the cheaper it is.

**What Green Load does.**

1. **Read the landscape drawing.** The CAD engine already reads DXF and DWG, and the PDF route (a PDF with a stated
   or calibrated scale) already exists. Green Load takes closed outlines on landscape layers (and their hatches,
   blocks and tags) as zones. Trees come in as blocks or circles with a species tag and a girth or canopy size.
   Where layers are messy, the layer-mapping step (*Layer Whisperer*, proposed) is reused.
2. **Name each zone and its build-up.** Every zone gets a *build-up recipe* from a small table the team owns:
   - *Lawn:* 300 mm growing medium, filter fabric, 40 mm drainage cell, protection screed, waterproofing.
   - *Shrub bed:* 450 mm medium, same layers beneath.
   - *Tree pit:* depth from the tree size (a root-ball and soil volume rule from the species table), with a drop or
     a mound.
   - *Paver:* paver, bedding, protection, waterproofing.
   - *Pool:* water depth plus the pool shell, from the tank rules in *Tank Smith* (proposed).
   - *Fire lane:* paving build-up plus the fire-tender wheel loads as the authority or consultant gives them.
   Each layer is a real 3D slab with its own material, thickness and unit weight, so the model and the load agree.
3. **Add up the load, per square metre and per point.** Dead load from the layers (soil at its saturated unit
   weight), live load from the zone's use, and point loads from trees (root ball, soil and water), planters
   and water features. Numbers come from a rules table the team edits; **defaults are placeholders, and the
   structural consultant confirms every one**. Each number cites its source, so no number is "the tool's".
4. **Compare with what the slab was designed for.** The structural model or the consultant's design note gives the
   allowed load per bay. Green Load paints the podium: green within, amber near, red over. Clicking a red bay
   shows what stands on it, and what the load is made of.
5. **Propose the smallest fix.** For a red zone: move the tree to a column or beam line, raise the planter on a
   mound over a beam, swap to a lighter growing medium, shallow the soil and choose a smaller species, or cut a
   drop-slab recess. Each option shows the load change, the steel and concrete change, the waterproofing area and a
   rough cost, and nothing is changed without a click.
6. **Check the water.** Build-up slopes and outlets are checked against the drain points under the podium, with
   the same rules as *Rain Run* (proposed) for falls and ponding. A planter with no outlet is flagged.
7. **Hand off.** A one-page *landscape load table* (zone, area, build-up, kN/m2), a *point load list* (trees,
   water features, pool), the recess outlines as a DXF in the CH template, a 3D IFC model (trees as simple
   proxies with a species property, layers as slabs), the waterproofing and drainage BOQ, and a client picture.

**What makes it a daily tool, not a demo.**
- It starts from the drawing the landscape consultant actually sends, not from a clean model.
- It gives the BIM modeller a map, a table and a fix in one place, so the conversation with the consultant takes
  minutes rather than days of emails.
- A revision of the landscape drawing re-runs the whole pass and shows what changed: "tree T14 moved 2 m, bay B7
  went from amber to red".

### Implementation

**Reuse what exists.** (Docs and folder layout in this repository, the cad2bim docs and the existing idea files
were read for this idea; the code was not built or run.)
- `packages/cad-engine` reads DXF/DWG and draws layers, blocks and hatches. The zone outlines, tree blocks and tags
  come from the same reading the pipeline uses. The PDF-with-calibration route is already described in the
  decisions for the AI chat.
- `packages/pipeline` (Rust) already finds closed outlines, assigns them to layers and handles an overlap of the
  same shape drawn twice; a new, small landscape reader sits beside the CH reader and never changes the CH v1 or
  v2 types (the repository rule).
- `packages/engine` writes IFC and already has the BOQ and the viewer, so layers become `IfcSlab`/`IfcCovering`
  elements with materials and unit weights, trees become `IfcGeographicElement` (or a proxy with a species
  property), and the BOQ gets the soil, drainage and waterproofing lines.
- The viewer's QA colouring already paints elements by a rule; the load map is one more rule, on the podium slab's
  bays instead of elements.
- The phasing and the 4D Timeline already exist: landscape layers become a phase, so the planting date can follow
  the structure and the garden finishes last.
- The agent helper (MCP) can expose `landscape_load(drawing)` that returns the zone table and the red bays, and an
  agent can draft the consultant email.

**New pieces.**
1. **Zone reader:** layer names and hatches to zone kinds, with a confirm panel for anything unsure ("this hatch
   is lawn or shrub bed?"). Kept per project and per consultant, like layer maps.
2. **Rules as data:** a JSON table of build-up recipes, unit weights, species sizes and live loads, per project and
   per code. Each value carries a source note and a "to confirm with the consultant" flag.
3. **Load take-down:** a small, pure, tested module (no I/O). It takes the zones and the slab bays and returns the
   load per bay and the point loads, with a range, never one false-precise number.
4. **Bay loader:** bay load limits read from the design note (typed in, or from a table the structural model
   gives), then the comparison and the colours.
5. **Fix proposer:** a short, ordered list of changes. It re-runs the take-down after each one and returns the
   cheapest that passes. It never edits the model without a click.
6. **Tree placer:** places each tree in 3D from its species size, with mature and planting-day sizes so the
   picture can be shown at planting and at year ten.
7. **Package writer:** the load table (PDF and Excel), the recess DXF, the IFC, the BOQ lines and a client page.

**Privacy.** Everything runs in the browser on the device. Nothing is uploaded to read or analyse a drawing.

**Test plan (what must be true before it is trusted).**
- Take three past podiums where the final landscape differed from the structural allowance. Run Green Load on the
  landscape drawing and ask the project's structural engineer which red bays were real problems on site. Record the
  catches and the false alarms.
- Hand-check 30 bays and 10 trees for load against the consultant's own numbers. The tool must match on at least
  27 of the 30 bays, and every miss must read as "cannot tell", not as a wrong "pass". Measured, not estimated;
  record the result in the repository.
- A podium with 400 zones and 120 trees must give its first answer in under 10 seconds on a normal laptop; measure
  and record it.
- Show the output to two structural consultants and ask only: "would you act on this?". Keep the answers, the
  negative ones too.

### Future update

- **Roof and terrace loads together** with *Sun Roof* (rooftop solar) and *Rain Run* (terrace slopes), so one
  roof gets one load map with garden, panels, tanks and plant.
- **Live link to the structural model**: read the design loads from the ETABS/SAFE model (*Frame Ferry*,
  proposed), so the limits come from the analysis rather than a typed note.
- **Plant growth over time:** the 3D tree grows from planting to year ten, with its root ball and weight, so
  the load map shows the building ten years on, not only at handover.
- **Irrigation and electrical:** the sprinkler layout, the water tank sizing and the garden lighting points, from
  the same zones.
- **Fire-tender route check:** the swept path and the turning radius on the podium lane, with the wheel loads from
  the authority, as a second screen in the same panel.
- **Site check:** the planter and tree pit levels checked after the pour from the surveyor's readings
  (*Plumb Line*, proposed), so a missed recess is found before the soil goes in.
- **Client card:** a one-page "your garden, and what it weighs" for the developer, with the picture, the
  load and the cost of each change, so value engineering does not quietly cut the recesses later.

*References read for this run: the repository's `CLAUDE.md`, `docs/decisions.md` and `docs/design/` folder
listing (cad2bim), and the existing idea files in this folder, to avoid repeating them (Yard Smith covers the ground
outside; this idea covers the structure's own garden decks). Nothing in the source repository was changed.*
