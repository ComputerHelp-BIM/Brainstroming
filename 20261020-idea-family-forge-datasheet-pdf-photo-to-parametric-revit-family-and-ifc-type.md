# 20261020 Idea: Family Forge — manufacturer datasheet / PDF / photo to a parametric Revit family, type catalog and IFC type in minutes

## Entry 1

```
date-time:   2026-10-20 (scheduled run, first entry of the day)
idea name:   Family Forge — "Drop the supplier's PDF. Get a flexed, parametric .rfa with all 14 sizes."
             A new AnonGee pyRevit tool (Essential panel) plus a cad2bim web page that turns the things a BIM
             team actually receives from site and suppliers (a manufacturer datasheet PDF with a dimension table,
             a DWG block, a phone photo of a fixture with two tape measurements, an Excel size list) into a
             proper parametric Revit family: reference planes, labelled dimensions, type parameters, a type
             catalog (.txt) for every size row, LOD 100/200/300 geometry switches, correct category, IFC class
             and shared parameters, and a 3D web preview that the BIM lead approves before it lands in the library.
potential:   Very high, used every week by every modeller. Family creation is the single most common "stop and
             wait" in a Revit project: a structural team needs a precast element, an anchor-bolt group, a
             cast-in channel, a base plate, a bearing pad, a lift-hook insert, a rebar coupler, a manhole cover,
             a pump, a lift car, a DG set, a water tank; the supplier sends a PDF, not an RFA. A competent
             modeller needs 1-4 hours per family (reference planes, constraints, flex test, type catalog,
             parameters, IFC mapping) and juniors often model "dumb" in-place geometry that breaks schedules and
             quantities.
             Time saved: a 14-size datasheet table becomes 1 family + 14 types in ~5 minutes of review instead
             of ~3 hours; the team library grows from what it receives, not from what someone found time to build.
             Trending angle: "AI made my Revit family from a PDF / photo" before-and-after reels (flat datasheet
             on the left, flexing 3D family spinning on the right, type count ticking up) are exactly the
             high-engagement LinkedIn / Instagram format of 2025-26 AEC-AI posts. The built-in "flex reel"
             (GIF of the family stepping through every type) is made to be posted.
             Market: free options are generic content portals (BIMobject, BIMsmith, manufacturer sites) that only
             help if the exact product is already modelled, and rarely for Indian suppliers. Building families
             from a datasheet is manual; the AI "text/image to family" tools that exist are paid, cloud-only,
             early-access, or produce non-parametric meshes (no reference planes, no type catalog, no flex). To
             my knowledge nothing free produces a constrained, flex-tested RFA + type catalog + IFC mapping from a
             supplier's dimension table, and nothing ties it to a structural toolkit's own generators.
```

### Description

**The pain (daily life of a BIM modeller).**
- The model needs an object, the library does not have it, the supplier sends a 6-page PDF. Work stops.
- A family is not just geometry: reference planes, aligned/locked faces, labelled dimensions, formulas
  (e.g. `Bolt_Spacing = (Plate_Width - 2 * Edge_Distance)`), visibility per detail level, materials, a category
  that schedules correctly, shared parameters for the BOQ, IFC export class. Juniors skip half of it.
- Size tables: datasheets list 10-30 sizes. Typing them as types or a type catalog by hand is slow and error prone.
- Families built by different people use different parameter names, so `Export Schedule`, `Parameter Combine` and
  the BOQ tools in this repo see "Width", "W", "width_mm" and "B" for the same thing.
- Libraries rot: no preview, no metadata, nobody knows which version is approved.

**The idea.** One guided flow, three input routes, one output standard.

1. **Ingest.**
   - *PDF datasheet*: extract the dimension table (text layer first, OCR fallback via the same pipeline idea as
     Plan Tracer) and the labelled dimension sketch (A, B, H, D, ...). The user sees the table in a grid and fixes
     any cell before anything is built.
   - *DWG block / DXF*: reuse cad2bim's DXF parsing to read the plan + elevation views of the block; matching
     edges in two views give a 2.5D extrusion/sweep proposal.
   - *Photo + two measurements*: phone photo of the object with a reference length; an LLM/vision step proposes a
     "primitive recipe" (box + cylinder + fillet + array) with dimensions as parameters. This route is clearly
     marked LOD 200 "placeholder", never shop-drawing geometry.
   - *Excel size list*: straight to the type catalog.
2. **Recipe, not mesh.** The core is a small JSON "family recipe" (category, template, reference planes,
   parameters with units and formulas, solids built from those planes, detail-level visibility, materials, IFC
   class, type rows). AI may propose a recipe; a deterministic builder executes it. That is what keeps the
   output parametric and repeatable, and lets the BIM lead edit a recipe instead of a family.
3. **Build.** pyRevit opens the right family template (`Application.NewFamilyDocument`), creates reference planes,
   extrusions/sweeps/revolves locked to them, `FamilyManager.AddParameter` for each parameter (shared parameters
   from the office file where a name matches), formulas, materials, `Visible` per detail level, and writes the
   type catalog `.txt` next to the `.rfa`.
4. **Flex test.** Before saving, the tool steps through every type, regenerates, and checks: no failure, volume
   changes the way the formula says, bounding box matches the datasheet's overall size within tolerance. A failed
   row is shown red with the reason. The flex run is captured as the "flex reel" GIF.
5. **Approve.** A cad2bim web page shows the family in 3D (glTF export of each type), the datasheet side by side,
   and an "Approve to library" button; approved families get a version, a thumbnail, a source-PDF link and a
   QR-style id, and go into the office library folder.
6. **Use.** Load into the open project; or, from the Excel bridge, `families.place` drops instances at grid
   intersections / coordinates from a sheet (e.g. 48 anchor-bolt groups under 48 columns).

### Implementation

- **Recipe schema** (`family-recipe.v1.json`): versioned, documented, with examples for 5 starter archetypes that
  cover most structural/MEP requests: plate + bolt array (base plate, anchor group, cast-in plate), prism/void
  (precast block, pit, tank, manhole), profile sweep (channel, bearing, kerb), revolve (coupler, pipe fitting,
  bollard), box-with-face-connectors (equipment, pump, DG set).
- **pyRevit tool `Family Forge`** (IronPython/CPython engine as already used in AnonGee): `recipe -> .rfa` builder,
  flex tester, type-catalog writer, shared-parameter mapper (one office mapping file so every family uses the same
  names that `Export Schedule` and BBS/BOQ tools expect). One transaction per stage, everything previewed.
- **Bridge commands**: `families.build` (recipe in, rfa path out) and `families.place` (rows in, instances out),
  following the declared/built pattern of the existing Excel bridge.
- **cad2bim web**: datasheet upload, table extraction grid, recipe editor, Three.js preview of glTF per type,
  library browser with approval status. The web side never needs Revit; the build step does.
- **AI layer (optional, swappable)**: table extraction and photo-to-recipe proposals via an LLM/vision model; output
  is always a recipe that passes JSON-schema validation and is shown to the user. No AI step writes a family directly.
- **Library health scan** (bonus, small): read every `.rfa` in the office library, report missing type catalogs,
  non-shared parameters, wrong categories, files with no preview; feeds the same Health Card idea as Warning Doctor.
- **MVP (2-3 weeks)**: Excel size list + archetype "plate + bolt array" and "prism/void" -> rfa + type catalog +
  flex test + thumbnail. That alone covers base plates, anchor groups, embed plates, pits and precast blocks.
- **Tests**: recipe round trip (recipe -> rfa -> read back parameters == recipe), flex test on all starter
  archetypes, golden type catalogs.

### Future update

- DWG-block route for arbitrary 2-view profiles; IFC `IfcTypeObject` export straight from the recipe for non-Revit
  users (Bonsai / web viewer), so the same recipe serves Revit and openBIM.
- Nested families and connectors (MEP connectors on pumps/equipment; rebar host faces on precast).
- Supplier packs: publish recipes for common Indian suppliers' catalogues (anchor fasteners, couplers, bearings,
  precast) as a free shared library; a community recipe gallery grows the wow effect on LinkedIn.
- "Family from model": select an in-place or generic model element and convert it to a proper parametric family
  (a big clean-up win in received models).
- Carbon and cost fields per type (EPD / rate), so placed families flow into Bill Painter and BOQ deltas.
- Version diff between two recipes (what changed in the supplier's new catalogue), reusing Change Radar's diff UI.
