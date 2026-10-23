# 20261023 Idea: Tower Stacker — model one typical floor, fill in a floor matrix in Excel, and get the whole tower in Revit with column zoning from ETABS, refuge / service floors, slab and wall changes, and live sync when the typical floor changes

## Entry 1

```
date-time:   2026-10-23 (scheduled run, second idea of the day)
idea name:   Tower Stacker — "Model the typical floor once. Fill in one Excel sheet: which floors are
             podium, typical, refuge, service, terrace; which column sizes apply on which floors (straight
             from ETABS); where shear walls get thinner; where slab thickness or floor height changes; which
             flats are mirrored or merged. Press Stack. Revit builds every level and every floor of the
             tower with the right sizes, and when the architect moves a wall on the typical floor, Stack
             again pushes only that change to all 34 floors, without breaking the floors that are
             different."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, Core panel, next to AutoLevel and BIM Generation): pick the
                   source typical floor, edit the floor matrix in a WPF grid or Excel, preview, stack,
                   re-sync,
               (2) etabs-mcp bridge: reads storeys, column / wall / beam section assignments per storey
                   and design results, and fills in the zoning columns of the floor matrix,
               (3) cad2bim web viewer: an "exploded tower" 3D view of the stack, coloured by floor type
                   and by zoning, with a floor-by-floor diff after every re-sync,
               (4) structura / BBS / Pour Planner: floor-wise quantities, because every floor now carries
                   its true sizes instead of the typical floor's.
             Outputs: levels with names and elevations, all floors modelled (walls, floors, columns, beams,
             shear walls, openings, rooms, room tags, flat numbers), per-floor overrides applied, a stack
             report (what was created, overridden, skipped), a re-sync diff, and a floor matrix sheet that
             the whole team edits instead of 30 floor plans.
potential:   Very high for any office doing residential or commercial towers (G+12 to G+60), which in
             India is most BIM work today. Every tower has a typical floor and every tower has exceptions.
             Time today (one G+30 residential tower, 4-6 flats per floor):
               - levels and floor-to-floor heights by hand or AutoLevel: 0.5 day,
               - copy the typical floor up with Copy / Paste Aligned to Selected Levels or Model Groups:
                 1 day, then fight groups ("group has been changed outside group edit mode", excluded
                 members, hosted elements lost),
               - column size changes every 5-8 floors from ETABS (C1 900x900 to 750x750 to 600x600), shear
                 walls thinning, beam depths changing at transfer / refuge floors: 2-4 days of floor-by-
                 floor type swaps, and they are often missed,
               - refuge floors, service floor, podium, terrace, mumty, OHT: 1-2 days of manual edits,
               - every architect change on the typical floor (a toilet moved, a balcony grows 300 mm):
                 0.5-2 days to repeat on all floors, 10-20 times in a project,
               - total: 2-4 weeks of modeller time per tower, plus quantity errors because "typical"
                 sizes were left on floors that are not typical.
             With Tower Stacker: first stack in under an hour after the typical floor is ready; each
             architect change re-synced in 5-15 minutes; column zoning pulled from ETABS in one click.
             On a 3-tower project that is 6-10 weeks of team time saved, and the quantities of concrete
             and steel per floor become right (column zoning alone changes concrete by 8-15 % against a
             model that kept the typical size).
             Quality saved: wrong column sizes on upper floors (the most common BIM-vs-structure mismatch
             in Indian towers), refuge floors modelled as typical (no NBC refuge area, wrong slab), flat
             numbers and room tags not updated per floor, groups that silently corrupt, and drawings of
             floor 22 that still show the floor 5 column.
             Trending angle: "generative design", "automate the boring part of BIM" and Revit-group
             horror posts are constant on LinkedIn; time-lapse reels of a tower rising floor by floor are
             some of the most shared BIM clips on Instagram. The reel here: one typical floor on screen ->
             an Excel row changes "C1 700x700 from L18" -> press Stack -> the tower grows floor by floor
             with columns visibly slimming at L8, L18, L26 -> the architect drags a balcony on the typical
             floor -> 34 floors flash and update -> counter "34 floors, 18,420 elements, 6 min".
             Market: Revit Model Groups (free, fragile, no zoning, no exceptions per floor); Dynamo /
             Grasshopper scripts (custom, one-off, each office rewrites them); Revit "Paste Aligned"
             (copy only, no sync); TestFit, Finch3D, Hypar (paid, early-stage massing and layouts, not
             detailed Revit floors with structure from ETABS); CSiXRevit (free, but imports the ETABS
             model as its own analytical-driven geometry, not as overrides of the architect's typical
             floor, and has no floor-type matrix or re-sync of architecture). To our knowledge nothing
             free, or even paid, gives a floor matrix + typical-floor master + ETABS zoning + safe
             re-sync in one tool.
```

### Description

**The pain (daily life of a BIM modeller on a tower project).**
- The typical floor is 80 % of the tower, and the remaining 20 % is where all the time goes: podium,
  stilt, refuge floors every 7th floor (NBC), service floor, terrace, mumty, OHT, and the top floors where
  the client merges two flats into a penthouse.
- Revit groups look like the answer and become the problem: hosted doors drop, excluded members are
  forgotten, a single "ungroup to fix" on floor 14 breaks the sync forever, and the file gets slow.
- The structural engineer sends a column schedule from ETABS: "C1 to C24, size changes at L8, L18, L26".
  The modeller swaps types floor by floor; one or two floors are always missed, and nobody sees it until
  the BBS or the concrete quantity is wrong.
- The architect changes the typical floor every week. Each change means doing it again on 30 floors, or
  deleting and re-copying them, which loses the floor-specific edits.

**The idea, in seven parts.**
1. **Typical floor as master, tagged once.** The modeller picks one level as the master (or several:
   "Typ-A for L2-L15, Typ-B for L16-L30" when the plate changes). Tower Stacker stamps every element on it
   with a stable `TS_Key` (master element id + role) in an extensible-storage field, so copies on other
   floors can always be traced back to their master element.
2. **The floor matrix (the heart).** One Excel / WPF grid, one row per floor:
   `Level | FFL | Floor-to-floor | Floor type (Podium / Stilt / Typical-A / Refuge / Service / Terrace)
   | Master | Column zone | Wall zone | Beam zone | Slab | Flat numbering | Skip / Mirror / Merge rules`.
   Zones are small tables: `Zone Z2: C1=750x750, C2=600x900, SW1=250 thk` from L8 to L17. Floor-type
   templates say what changes: a Refuge floor drops the flat walls, keeps the core and columns, adds
   refuge area boundary, parapet and 1.2 m railings; a Service floor uses a different height and slab.
3. **ETABS zoning in one click.** Through etabs-mcp, read storeys, frame and wall section assignments per
   storey and the design sections; group consecutive storeys with the same sections into zones; match
   ETABS labels (C1, SW3) to Revit columns / walls on the master by label, grid intersection or nearest
   location; fill in the zone columns of the matrix. Mismatches are listed ("C17 in ETABS has no column
   at grid D/7 in Revit").
4. **Stack.** For every floor: create or update the level, copy the master elements with the level offset
   (`ElementTransformUtils.CopyElements` with a transform, no groups), then apply overrides in order:
   floor type template -> zone sizes (swap types, creating `RCC_Column_750x750` if missing) -> slab
   thickness -> flat numbering (`Flat No` = floor x 100 + unit, room names and tags updated) -> local
   exceptions. Hosted elements (doors, windows, openings, sleeves from Sleeve Punch) are copied with their
   hosts so nothing drops.
5. **Exceptions that survive.** Any element on floor N that the modeller edits by hand is marked
   `TS_Local` (detected on re-sync by comparing with what the stack would have produced). Local edits are
   never overwritten; the re-sync report shows them in a separate list: "floor 12: kitchen wall moved by
   hand, not synced, keep / reset?".
6. **Re-sync (live sync).** When the master changes, Re-sync computes a diff of the master since the last
   stack (added, deleted, moved, type changed, parameter changed) and applies only that diff to every
   floor that uses the master and does not have a local override on the same element. 30 floors update in
   minutes, without deleting anything else, in one `TransactionGroup` with a single undo.
7. **Tower view and report.** The cad2bim viewer shows an exploded tower coloured by floor type and by zone
   (columns visibly slimming up the tower), a click on any floor shows its matrix row, and the re-sync diff
   is highlighted ("balcony slab +300 mm on L2-L30, 29 floors updated, L31 skipped: penthouse local").
   The stack report goes to Excel / PDF with floor-wise counts and quantities (concrete per floor by
   zone).

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)** already has **AutoLevel** (levels from data), **BIM Generation** and
  **FramewinToBIM** (creating structure in bulk), **Rotate Column**, **Join Priority** and **Unjoin by
  Category** (joins after copying are the classic mess), **Export Schedule**, the WPF UI templates and the
  `bridge/` to Python 3. Tower Stacker turns AutoLevel into "AutoTower".
- **etabs-mcp (ETABS Bridge)** already exposes storeys, sections, design results and the full database
  tables through its REST API and MCP server; zoning is a read of what is already reachable.
- **cad2bim** web viewer already renders IFC / glTF in three.js; the exploded tower and diff colouring are
  a viewer plugin. **C2B** can supply the typical floor from a client DXF.
- **structura**, **BBS Weaver**, **Bar Optimizer**, **Pour Planner** and **Bill Painter** all get correct
  floor-wise sizes and quantities; **Twin Check** (design vs. BIM) becomes "green" by construction;
  **Drawing Autopilot** can produce floor plans per floor type instead of per floor.

**Wow moments.**
- One typical floor and one Excel sheet become a 34-floor tower in a few minutes, rising floor by floor.
- Change "C1 750x750 from L8" to "from L10" in the sheet: two floors change size, nothing else is touched.
- The architect moves a toilet wall on the typical floor; Re-sync updates 29 floors and politely skips
  the penthouse that was edited by hand.
- The exploded tower view coloured by column zone, shown next to the ETABS model with matching colours,
  in the client / structural coordination meeting.

### Implementation

**Phase 1 — Matrix and first stack (3 weeks).**
- Floor matrix schema (`tower_matrix.xlsx` + JSON mirror): floors, floor types, zones, numbering rules;
  WPF grid with validation (heights add up, every floor has a type, zones do not overlap).
- Level creation and update reusing AutoLevel logic (name pattern, elevation, plan views from a view
  template).
- Master tagging with extensible storage (`TS_Key`, `TS_Master`, `TS_Floor`, `TS_Local` flag, a hash of
  the generated state per element).
- Copy engine without groups: `ElementTransformUtils.CopyElements(doc, ids, translation)` per floor in
  batches, rebinding level parameters (base / top constraints, offsets) to the new level; walls and
  columns attached to the next level by rule.
- Floor type templates in YAML (`floor_types/refuge.yaml`: keep categories, drop rooms by name, add
  railings, slab type, height).
- Tests on a sample G+12 model: element counts per floor, every copy maps to a master key, levels and
  constraints correct, hosted doors present on every floor.

**Phase 2 — Zoning from ETABS (2 weeks).**
- etabs-mcp client in the bridge: storeys, frame section per storey, area (wall) section per storey,
  labels and coordinates.
- Zone builder: group consecutive storeys with identical section sets; storey name to Revit level by
  elevation with a tolerance.
- Matching ETABS frames to Revit columns by label, then by grid intersection, then nearest point
  (configurable tolerance); report unmatched in both directions.
- Type manager: find or create column / wall / beam types by size with the office naming convention.

**Phase 3 — Re-sync, local edits, viewer (3 weeks).**
- Master snapshot after every stack (element key -> geometry fingerprint + type + key parameters); diff
  on re-sync gives added / removed / modified elements.
- Local-edit detection: compare each copy's current fingerprint with the stored "generated" fingerprint;
  differences mark `TS_Local`.
- Apply diff per floor in a `TransactionGroup`, with progress, cancel, and a report; warnings from Warning
  Doctor run after the sync.
- Export the stacked tower to IFC / glTF for the cad2bim exploded view; floor-type and zone colouring from
  the matrix; diff highlight from the last re-sync report.

**Tech notes / risks.**
- Performance: copying 15,000-25,000 elements; batch per floor, suspend regeneration where possible, and
  keep the copy in batches of floors with progress so Revit stays responsive. Worksharing: one floor
  range per user, borrow elements via worksets per floor band.
- Joins and attachments are the usual breakage after copying; run Join Priority rules and
  attach-top/base by level after each floor.
- Hosted and face-based families (doors, sleeves, fixtures) must be copied with the host in the same call;
  test with every family type used in the master.
- Mirror / merge rules (penthouse, mirror flats in a wing) start as "local templates" defined once and
  applied per row; full generative rules come later.
- Never deletes anything it did not create; elements without a `TS_Key` are invisible to the tool.

### Future update
- Typical-floor variants: A/B plates for setbacks, and wing-based stacking for L / U shaped towers.
- MEP stacking: Drain Line, Live Wire and Sleeve Punch layouts follow the same matrix (stacks, risers and
  shafts continuous across floors, refuge floor offsets checked).
- Façade stacking: chajjas and fins sized by Sun Compass per floor and orientation.
- 4D: the floor matrix becomes a floor cycle (7-10 days per floor) for Pour Planner and Model Reel, so the
  tower-rising reel is also the construction programme.
- Two-way ETABS sync: when the stack changes a floor height or adds a refuge floor, push storey changes
  back to ETABS through etabs-mcp for re-analysis.
- AI mode: "Make floors 28-30 two penthouses by merging flats 1-2 and 3-4": the tool writes the local
  template and previews it before stacking.
