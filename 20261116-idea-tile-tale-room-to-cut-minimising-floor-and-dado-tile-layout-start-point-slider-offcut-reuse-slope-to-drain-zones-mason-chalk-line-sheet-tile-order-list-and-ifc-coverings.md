# 20261116 Idea: Tile Tale — pick a room, get the tile layout with the fewest cuts (floor and bathroom walls), slide the start point and watch the cut count fall, reuse offcuts, respect the slope to the drain, then print the mason's chalk-line sheet and the exact tile order

## Entry 1

```
date-time:   2026-11-16 (scheduled run, UTC)
idea name:   Tile Tale (room-by-room tile layout planner, in the browser, nothing uploaded)
potential:   High, daily, and every flat has it. Tiles are one of the largest finish items by cost and
             by site argument: how many boxes, where to start, what small slivers will show at the door,
             who pays for the breakage. In Revit a floor "pattern" is only a picture; it does not know
             the cuts. Today the estimate is "area + 10 or 15 percent" and the mason decides the start
             point on the day. The result: slivers at the door, ordering too many or too few boxes, and a
             second lot of tiles with a different shade.
             Time saved: NOT measured here. First test: take 10 real flats (bedroom, living, toilet,
             kitchen), count the boxes the site actually used, and compare with Tile Tale's order list
             and with the "area + 10 percent" rule. Also ask the mason whether the chalk-line sheet was
             clear. Do not quote percent saved until that is done.
             Market check (2026-11-16, one search; not exhaustive, so "I did not find one", not "none"):
               - Autodesk App Store "Floor Finish & Cutting Optimization" (NEXORA), listed at about
                 $10 a month: tile layout, cut optimisation, offcut reuse and take-off inside Revit.
                 I did not verify its 2026 status or reviews. It is a floor-only Revit add-in.
               - TilePlan and web tile calculators give a cut list for one rectangle. Not tied to a model.
               - Academic work (ASCE paper; a 2025 Elsevier paper, closed access) shows the algorithm
                 works in BIM. Research, not a daily tool for Indian flats.
               - One site claims "up to 85 percent less waste". That is marketing, and I do not use it.
                 The 15 to 25 percent waste figure in planning guides has no study cited either.
               - I did not find a tool that does floor AND bathroom wall (dado) tiles, the slope to the
                 floor drain, and a mason's sheet, from a model that is not Revit.
             Extends tools already in this repo (does not repeat them):
               - Wall Wizard: counts tile area and a rough cut waste per room. Tile Tale draws the real
                 layout, so the waste is counted from actual cuts, and Wall Wizard can read it.
               - Drain Line: puts the floor trap and the sunken slab. Tile Tale reads the drain and lays
                 the slope zones around it (tiles cut on the diagonal or small tiles near the trap).
               - Level Lens: finish levels and build-ups. Tile Tale uses the finish thickness and the
                 door threshold height from it.
               - Window Wright and Wall Wizard: the door, window and niche openings the dado must go round.
               - Order Clock: it can take the box count and the lead time of the chosen tile.
               - Model Capsule / Site QR Pass: the mason's sheet can open on a phone by QR.
             Trend: AI takeoff is the loudest AEC topic this year (Trimble reports 60 percent saving in MEP
             takeoff; Autodesk says Forma Takeoff halves quantity takeoff; both vendor figures, not
             independent). Takeoff counts areas. Tile layout is the next step: what is actually cut.
             I could not measure LinkedIn or Instagram counts from here, so this is not a social-media claim.
             The wow: click a bathroom in the 3D model. The floor shows its tiles. Drag the start point:
             the number "cuts: 31, slivers under 50 mm: 4" changes live, the slivers glow red, then at the
             best point it reads "cuts: 19, slivers: 0". One more click, and the walls unfold into four
             elevations with the tiles, the niche and the shower mixer already round-cut.
```

### Description

**The pain.**
- The tile count is area plus a guess. Two flats with the same area need different boxes because of the shape.
- The start point is decided by the mason, with no plan. The usual result is a thin sliver at the door,
  the most visible place in the flat.
- Bathroom walls are worse: tile height must stop at a lintel or a beam soffit, a niche, a window, the
  mixer, the geyser point. Each cut is on the wall, in front of the owner.
- Floor slope toward the trap needs smaller tiles or diagonal cuts. Nobody draws this; the mason improvises.
- Modellers lose time drawing a tile pattern in Revit that still gives no count.

**The idea in one line.** Treat tile laying as a small puzzle the model can solve: the room outline, the doors,
the drain, the tile size and the joint are the inputs; the layout with the fewest cuts and no slivers is the
output, with a sheet the mason can follow.

### How it works (user view)

1. **Pick rooms** (one, a type, or a whole floor) and a tile from a small table: size (600 x 600, 800 x 800,
   300 x 600, 300 x 300 and so on), joint width, thickness, pieces per box, price. The table is the user's
   own; there is no built-in price list.
2. **Auto layout.** For each room the tool tries start points (door centre, room centre, a wall, the longest
   clear wall, a corner) and two or three rotations (straight, brick offset, diagonal). It scores each by
   cut count, sliver count (a cut piece narrower than a chosen limit, default 50 mm), waste, and whether the
   door threshold gets a full tile. It shows the best three; the user picks.
3. **Slide the start point.** Drag a handle on the plan; cut count, slivers and waste update live. The user can
   lock a rule, for example "full tile at the door line" or "centre a tile on the room's axis".
4. **Offcut reuse.** A cut piece that is left over goes into a small pool; the next cut that fits takes it
   (the same piece size and the same shade lot). The order list counts only the boxes actually needed.
5. **Bathroom walls.** The walls unfold as elevations. Rows start at the floor or from the top (the user
   chooses which edge must be a full row). The tile height stops at the lintel or beam soffit, goes round
   windows, niches, the shower mixer and the geyser point, which the tool reads from the model's openings
   and MEP points. A listello or border row can be set.
6. **Slope to drain.** In a wet area the floor zone around the trap is marked as a slope zone; the user picks
   "small tiles", "diagonal cut" or "screed only". The cut list for the zone is separate.
7. **Outputs.**
   - **Mason's sheet:** one page per room at 1:20 (floor) or 1:10 (wall), start point, chalk lines marked A, B,
     C, cut sizes written on the tiles, and a note per room ("start from door, full tile at the threshold").
   - **Tile order list:** boxes per tile type, per floor and total, with the extra the user chooses (default
     5 percent, a setting, not a promise), the cut count and the offcut reuse.
   - **Excel and the model:** quantity per room goes into the finish schedule, and the layout can be written
     as IFC coverings with the tile pattern so the 3D view shows the real tiles.
   - **Phone link:** a QR on the mason's sheet opens the room layout on the phone (the Site QR Pass idea).

### Why a BIM team would use it

- The finish quantity becomes a real count, not area times a factor. The BOQ and the purchase list agree.
- The modeller no longer draws pattern lines by hand; one room takes seconds.
- The site gets one clear sheet. Fewer arguments about the start point and the sliver at the door.
- Client demos: bathrooms in 3D with real tile joints look much better than a flat colour.
- The data feeds Wall Wizard, Order Clock and the BOQ tools already in the toolkit.

### Implementation

- **Input.** Room polygons come from the model (floor boundaries, walls, doors, openings) or from the
  drawing pipeline's rooms. Doors and windows are read with their hosted position. Drain and mixer points are
  read from the model if present, or placed by the user with a click.
- **Layout core.** A tile grid is placed on the room polygon in the chosen rotation and offset. Each cell is
  clipped to the polygon (a standard polygon clipping routine; a MIT or BSD licensed library is fine, GPL is
  not). A clipped cell smaller than the full tile is a cut; the cut area, its two side lengths and the
  sliver flag are stored. For the offset search, the start point moves in steps of 10 mm over one tile pitch in
  each direction, which is a small search (a 600 mm tile gives 60 x 60 trials), all run in the browser in
  a worker. The score is a weighted sum the user can change.
- **Offcut pool.** A greedy match: sort cut pieces by size, assign each a leftover piece that is larger in both
  sides; this is a simple heuristic, not an optimal nesting. It is reported as "estimated reuse", and the
  order list shows the boxes with and without reuse, so the site can choose to trust it or not.
- **Walls.** Each wall face is a rectangle with holes (openings, niches, fixtures). The same clip routine is used;
  the row direction and the full-row edge are inputs.
- **Slope zone.** A small polygon around the drain, offset by a distance the user sets; the layout there uses
  the chosen method. The fall check itself is Drain Line's job; Tile Tale only reads the result.
- **Output.** The mason's sheet uses the same PDF path as the drawing outputs; the layout goes into the IFC as
  coverings with a tile type and quantity set (area, count, cuts). Tile type data is kept in the project, so
  it goes to the BOQ with the other items.
- **Privacy.** All of it runs in the browser on the model that is open. Nothing is uploaded.
- **Tests.** (1) A rectangle of known size and tile: cut count and sliver count by hand. (2) A room with a
  door recess and a column: check that cells are clipped correctly. (3) The offset search finds a layout at
  least as good as the centre start, on 20 test rooms. (4) The order list matches a hand count on 5 real
  flats (the real-flat check needs the site's data; I have not done it).

### Rollout

1. **Phase 1 (about 2 weeks).** Rectangular and L rooms, floor only, one tile, start point search, cut and
   sliver counts, order list, Excel.
2. **Phase 2.** Mason's sheet PDF, door and column cut-outs, the offcut pool, brick offset and diagonal.
3. **Phase 3.** Bathroom wall elevations with openings, niches and fixtures; the slope zone; IFC coverings.
4. **Phase 4.** Phone link by QR; a whole-floor run; read the finish build-up from Level Lens.

### Risks and honest limits

- Tiles are not all the same: a shade lot, a rectified edge or a patterned tile (directional) changes the
  rules. Phase 1 ignores directional patterns and says so on the sheet.
- Real rooms are not perfect: walls are out of square by 5 to 15 mm. The sheet says "check the room
  squareness before marking", and the tool lets the user add a measured error per wall.
- The offcut reuse number is an estimate. Do not use it to cut the order below what the site accepts.
- The tool does not decide the tile's suitability (slip rating, wet area use). That is the architect's spec.
- Prices and waste percents are the user's inputs; the tool ships with none.

### Future update

- Read the owner's tile choice from Mood Swap, so the picked finish becomes the layout in one step.
- Skirting, nosing, threshold strip and edge trim counted per metre from the same layout.
- Stairs: tread and riser tiles, using Stair Smith's geometry.
- Terrace and external tiles with the falls from Rain Run, and paver layouts.
- A "same shade lot" tracker per flat, from the box labels read by phone photo.
- Compare the order list with the supplier's quote (Bid Mirror's idea, for finishes).
- Sheet of "tile heights for the plumber" so the mixer and the shower point land on a joint, not on a tile.
