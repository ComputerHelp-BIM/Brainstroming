# 20261019 Idea: Wall Wizard — infill blockwork course layout, lintels, openings, plaster / putty / paint / tile and the room finish schedule, straight from the structural model

## Entry 1

```
date-time:   2026-10-19 (scheduled run, first entry of the day)
idea name:   Wall Wizard — "The frame is modelled. Now the walls fill themselves, brick by brick, with a cut-list."
             Open the cad2bim / Structura / IFC frame (columns, beams, slabs) plus the architectural plan DXF (or a
             Plan Tracer output). The tool finds every panel the frame leaves open and fills it with infill masonry:
             course-by-course block layout (AAC / fly-ash / red brick / concrete block, any size and mortar joint),
             openings from door/window tags, lintel and sill-band placement, cut-block and half-block count,
             sunk-slab/toilet kerbs, plaster (internal/external, thickness, mesh at RCC-masonry junctions), putty,
             primer, paint, tile and skirting areas per room, and a Room Finish Schedule. Output: IFC walls with a
             real layer/course structure, a 3D "exploded wall" preview, an Excel BOQ, a block cut-list per wall,
             and a DXF elevation sheet for the mason's gang.
potential:   High. After concrete and steel, masonry and finishes are the next-largest quantity block on every
             building job and the one that is almost always measured by hand: "wall length x height minus openings",
             copied from a PDF into Excel, redone at every revision. Typical cost is 1-2 days per floor-set for a
             modeller or quantity surveyor, and the usual error is 5-10 % in blocks, 8-15 % in plaster and paint
             (openings, reveals, junction mesh and deductions forgotten). Over-ordering blocks and mortar, or running
             short on site, both cost real money and days. Wall Wizard turns "re-measure after every revision" into a
             one-click re-run that also shows what changed.
             Trending angle: "AAC block layout in 10 seconds", "no more brick count in Excel" and "see the wall build
             course by course" are natural short-video hooks (exploded wall assembling itself, then the cut-list
             printing). Fits the daily-productivity posts that BIM and site-engineering communities engage with.
             Market: to my knowledge, Revit and ArchiCAD model walls as solid layers and do not place blocks;
             block-layout add-ons exist as paid, per-seat plug-ins or inside paid take-off platforms (PlanSwift,
             Kreo, Cost-X, CostX, Togal-type subscriptions) that measure 2D drawings. No free, open, IFC-native
             tool that derives infill from the structural frame, lays out courses and outputs a mason-ready cut-list
             plus finish BOQ and room schedule in one flow.
```

### Description

**The pain (daily life of a BIM modeller, QS and site engineer).**
- The structural model has no walls. The architect's DXF has wall lines but no relation to the beam soffits,
  column faces, slab edges or the kerb at wet areas, so true wall height and length are guesses.
- Wall quantities are measured twice: once for blocks (volume less openings), once for plaster/paint (area, both
  faces, plus reveals). Every plan change repeats it. Different people deduct openings differently.
- Lintels, sill bands, door-frame fixing and RCC-masonry junction mesh are scattered across notes and get missed in
  the BOQ; site then re-orders emergency material at higher rates.
- Block wastage is argued instead of calculated: the number of half/cut blocks depends on opening positions and
  panel lengths, so "5 % wastage" is wrong in both directions.
- Finishes are asked by room ("how much vitrified tile, skirting and paint for flat 904?"). That answer is another
  half-day of manual summing.
- Tender comparisons and contractor bills need a measured basis; the model should be that basis.

**The idea.** A **Wall Wizard** panel with five steps; each step is reviewable and re-runnable.

1. **Find the panels.** For each storey the engine builds the 2D "open frame" polygons from columns, beams and slab
   edges (with beam depth known, clear wall height = slab-to-beam-soffit or slab-to-slab less slab/beam). The
   architectural DXF (wall layers, door/window blocks and tags) is registered on the grid and each wall line is
   assigned to a panel. Panels with no wall line (open, glazed, parapet-only) are skipped; shafts and toilets are
   flagged for kerb rules.
2. **Rule pack (YAML/JSON, editable, versioned).** Block family and size (e.g. 600x200x150 AAC, 190x190x390
   concrete, 230x110x75 brick), mortar joint, thin-bed or conventional, wall type by thickness and location
   (external 200, internal 100/150, toilet partition), course bond (stretcher, half-lap), lintel rule (bearing,
   depth by span, precast vs cast-in-situ), sill band, door-frame detail, kerb height in wet areas, plaster
   thickness and mesh strips at dissimilar-material junctions, putty/primer/paint coats, waterproofing at toilet
   walls. Shipped packs: AAC 200/100, red brick 230/115, concrete block 190/100 — all editable. Every output prints
   the pack name and version.
3. **Lay the courses.** Within each panel the layout algorithm lays courses from the lowest course up: starts from
   the column face, keeps the bond, respects openings with lintel and sill, and cuts the last block to fit. It
   returns, per wall: number of full, half and cut blocks, the length of each cut (so the mason sees "cut 3 blocks
   to 310 mm"), mortar volume or bags, lintel list, and a 2D elevation with the course lines. Wastage is *computed*
   from the cut-list; a small allowance for breakage is added separately and shown.
4. **Finishes by room.** IfcSpace or room polygons (or auto-generated from wall faces) carry a finish code from
   a mapping table (e.g. "Bedroom: putty + 2 coat emulsion, vitrified 600x600, skirting 100 mm"). The engine
   computes wall area per room face (less openings, plus reveals), ceiling, floor, skirting length, dado height and
   tile count (with tile cut waste from room dimensions and layout direction), and writes the **Room Finish
   Schedule** with a row per room and a total per flat/floor/block.
5. **Outputs.**
   - IFC: `IfcWall` for each panel with `IfcMaterialLayerSet`; optional `IfcBuildingElementPart` for each course
     (or only for the elevation view, to keep the model light), properties for block type, plaster, paint code.
   - Excel BOQ: block counts and cut-list, mortar, lintels, plaster, putty, paint, tile, skirting, waterproofing,
     by floor/flat/room/wall type, with rates column for 5D hand-off (reuses the Sheet Sync column format).
   - DXF/PDF: mason's elevation sheet per wall (course lines, cut sizes, lintel schedule) in the Drawing Autopilot
     sheet standard; room finish plan with hatches.
   - Viewer: "Build wall" animation (courses stack in order), and a heatmap of wastage per wall (green = clean fit,
     red = many cuts) so the architect can nudge an opening by 50 mm and save blocks.

**Why it is unique / extends the repo.** The frame is already modelled by cad2bim/Structura and the architectural
plan arrives as DXF (Plan Tracer, DXF → 3D). The missing piece is everything that sits *between* the frame — walls
and finishes. It reuses: IFC read/write (cad2bim), DXF read (cad2bim 2D → 3D), room polygons and area logic (Area
Statement Pro), Excel round-trip (Sheet Sync), drawing sheets (Drawing Autopilot), delta reports (Change Radar),
progress painting (Bill Painter, which can now bill "blockwork L4 flat 3 = 60 %").

### Implementation

1. **Phase 1 (MVP, 1-2 weeks).** Python + `ifcopenshell` + `shapely` + `ezdxf`. Read columns/beams/slabs per
   storey; compute open panels as polygon difference; take the architectural DXF wall lines on named layers, match
   to panels by overlap; compute wall length x clear height less openings (door/window blocks with width/height
   attributes); output a simple Excel BOQ (blocks by volume, plaster area, paint area). One rule pack (AAC 200).
2. **Phase 2.** Course layout engine: 1-D bin layout per course with opening cut-outs; cut-list; lintel/sill band
   placement; mortar bags; DXF elevation per wall. Unit tests with 5 hand-laid reference walls (count must match a
   mason's tally within one block).
3. **Phase 3.** Room finish engine and Room Finish Schedule; tile layout waste by direction; skirting; waterproofing;
   IFC `IfcMaterialLayerSet` output and optional per-course parts; viewer "build wall" animation with wastage
   heatmap (three.js / web-ifc, instanced meshes for light rendering).
4. **Phase 4.** Interactive tweak: drag a door in the viewer, see block count and wastage update live;
   rule-pack editor with schema validation and a "compare two packs" cost view (AAC vs red brick).
5. **Validation.** Golden-file tests on three real floors (hand BOQ vs tool within 2 %); warnings for panels where
   frame and architectural DXF disagree by more than a tolerance (wall off-grid, wall over a beam-less slab edge);
   all outputs stamped "Computed from model <hash>, rule pack <name/version> - verify before ordering".
6. **Safeguard.** Masonry here is *non-structural infill*. The tool does not design walls for lateral load or
   check frame-infill interaction; it only measures and lays out. Structural infill effects stay in Twin Check /
   the design software.

### Future update

- Change Radar link: "Rev C moved the door in W-12: 14 blocks, 2.4 m2 plaster, 1 lintel changed; BOQ delta Rs. ...".
- Procurement pack: order list by delivery date, using the Pour Planner sequence (blocks needed on L6 week 3), and
  a block-stock planner for site stacking and floor loading.
- Services chase and sleeve awareness (Sleeve Punch openings become cut-outs in the course layout).
- MEP-aware finishes: recessed boxes, niches and splash-back tile areas.
- Facade module: external plaster, texture, scaffolding area, and cladding panel layout for the same engine.
- Contractor rate library and 5D roll-up; measured basis for RA bills (Bill Painter) and variation notes.
- Image-to-finish: photo of a finished room estimates % completion of plaster/putty/paint for the progress video
  (Model Reel) and DPR.
- Community rule-pack library (AAC makers, thin-bed systems, local brick sizes) with human review of each pack.
