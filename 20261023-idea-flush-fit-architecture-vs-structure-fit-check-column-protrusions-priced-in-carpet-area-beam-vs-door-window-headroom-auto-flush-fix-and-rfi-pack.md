# 20261023 Idea: Flush Fit — architecture vs. structure fit check: column and beam protrusions priced in lost carpet area, beam vs. door / window head clashes, sunken slab and shaft mismatches, one-click flush fix and an RFI pack for the structural consultant

## Entry 1

```
date-time:   2026-10-23 (scheduled run, third idea of the day)
idea name:   Flush Fit — "Load the architect's plan and the structural model. Press Fit. Every column
             that pokes into a bedroom, every beam that cuts a window head, every drop that hangs over a
             door, every toilet sunk on the wrong slab panel and every shaft with no slab opening lights
             up in 3D, with its cost in saleable carpet area. Accept the fixes you want and Flush Fit
             moves the columns flush with the wall face (within the limits the engineer set), lists the
             rest as RFIs, and sends both teams one sheet."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, StrucBIM stack, next to Rotate Column and Join Priority):
                   arch model or linked arch model + structural model, Fit, review list, apply fixes,
               (2) cad2bim web viewer: "ghost architecture" view (walls translucent, protrusions solid red,
                   headroom clashes amber) that the architect and the client open in a browser,
               (3) C2B / Plan Tracer: when the architect only sends a DXF, the wall, door and window lines
                   come from the DXF instead of a Revit model,
               (4) Area Statement Pro: carpet area lost to each protrusion, per flat and in rupees.
             Outputs: issue list (BCF 2.1/3.0 + Excel), colour-coded 3D view, flush-fix preview and apply,
             "carpet loss" sheet per flat and per floor, RFI pack (PDF with plan snips and proposed
             column offset / size, one page per issue type), and a fit score per floor that is tracked
             across revisions.
potential:   Very high, every day, in every office that models RCC buildings from separate architectural
             and structural inputs (almost all Indian residential and commercial work).
             Time today (one G+12 residential building, two wings, 8 flats per floor):
               - overlaying arch and structure plans floor by floor and marking where columns stick out of
                 walls: 0.5-1 day per typical floor set, by eye, in AutoCAD or Revit,
               - checking beam soffit against door heads (2100 + frame) and window heads / lintel bands:
                 1-2 days, usually done only for "important" rooms, often skipped,
               - sunken slabs (toilets, kitchens, balconies, terraces) vs. structural slab panels and
                 beam positions: 0.5 day, missed often,
               - shafts, ducts, lift pits, staircase openings vs. slab openings: 0.5 day,
               - writing RFIs with screenshots and chasing replies: 1-2 days per round,
               - total: 4-6 days per building per revision, 3-6 revisions per project.
             With Flush Fit: the full check in 2-5 minutes per building, the RFI pack generated in
             one click, and fixes that the engineer pre-approved (flush offsets up to 50-75 mm, column
             rotation, size swap with same area) applied directly.
             Money angle (the wow for developers): a 230 x 600 column poking 75 mm into a bedroom on both
             sides loses about 0.09 m2 per location; 40 such points per floor x 12 floors = ~43 m2
             (~465 sq ft) of carpet area. At Rs 12,000-25,000 per sq ft that is Rs 55 lakh to 1.15 crore
             of saleable value hidden inside walls that could have been flush. No BIM report today says
             that number; this one prints it on the first page.
             Quality saved: beams over doors that force a 1950 mm door, windows cut by a drop beam,
             toilets sunk on the wrong panel (leaks, broken slabs on site), shafts drilled through slabs
             after casting, furniture that does not fit because a column face is in the wardrobe, and
             drawings that show columns flush when the model says otherwise.
             Trending angle: "BIM coordination saves money", "clash detection before site" and "carpet
             area loss" are constant topics on LinkedIn among Indian developers and BIM managers; before /
             after reels of a red-clash model turning green are popular on Instagram BIM pages. The reel:
             ghost walls fade in over the structure -> 312 red bumps appear -> counter "Carpet lost:
             486 sq ft = Rs 72 lakh" -> press Flush -> columns slide into the walls one after another ->
             counter drops to "38 sq ft, 14 RFIs" -> RFI pack PDF flips open.
             Market: Navisworks Clash Detective (paid, hard clashes only, no "flush" or headroom
             semantics, no carpet area, no fix); Solibri (paid, expensive, rule-based but no write-back
             to Revit and no carpet costing); Revible / BIMcollab / Revizto (paid, issue tracking, not
             the domain check); Revit Interference Check (free, crude, solid overlap only, hundreds of
             false positives between walls and columns that are meant to touch). To our knowledge no
             free tool, and no paid tool, checks "column flush with wall face", "beam soffit vs. door /
             window head", "sunken slab vs. structural panel" and "shaft vs. slab opening" together,
             prices the loss in carpet area, and writes approved fixes back.
```

### Description

**The pain (daily life of a BIM modeller in an RCC office).**
- The architect draws walls, doors and windows; the structural engineer places columns and beams from
  analysis, often on a grid that ignores wall faces. Nobody owns the overlay. The BIM modeller finds the
  mismatches one by one while modelling, or the site finds them later.
- The classic issues, every project: column faces 50-150 mm out of the wall into a room or corridor; a
  column inside a door swing or a wardrobe niche; a beam 600 deep over a door that needs 2100 clear; a
  drop beam crossing a living-room window head or a French window; toilets marked sunken by the
  architect but the structural slab panel is not lowered, or the sunken area crosses a beam; a duct shown
  on the arch plan with no slab opening in the structure; a staircase with headroom under 2.1 m below a
  landing beam.
- Revit's interference check reports every column touching every wall as a clash; real problems drown
  in noise, so nobody runs it.
- RFIs are written by hand with screenshots in Word; replies come back as a marked PDF; nobody tracks
  which revision closed which issue.

**The idea, in seven parts.**
1. **Read both sides.** Arch: Revit model or link (walls with finish faces, doors, windows with head and
   sill, rooms with names, floors with levels and sunken areas, shafts / openings), or a DXF through C2B /
   Plan Tracer (wall lines by layer, door / window blocks with tags, a level table). Structure: Revit
   structural model, cad2bim / Structura IFC, or ETABS through etabs-mcp (columns, beams with depth,
   slabs with level and drops, openings). Levels matched by elevation with a tolerance.
2. **Fit rules, not solid clashes.** A rule pack (YAML, office editable) checks meaning, not overlap:
   - `COL_PROTRUDE`: column face outside the finished wall face by more than t (default 10 mm) on a room
     side; measures protrusion depth and length per room, ignores walls meant to be thicker (pilasters
     the architect drew).
   - `COL_IN_SWING` / `COL_IN_NICHE`: column in a door swing, wardrobe, kitchen counter or lift lobby
     clear width.
   - `BEAM_HEAD`: beam soffit below door head + frame + 50 mm, or below window head / lintel band;
     reports clear height and needed height.
   - `BEAM_VISIBLE`: drop beam crossing a room away from walls (living, master bedroom), with the
     option "make it a flat / hidden beam?" flagged to the engineer.
   - `SUNKEN_MISMATCH`: arch sunken zone not matching a lowered slab panel, crossing a beam, or depth
     different from the architect's note (e.g. 300 vs. 450).
   - `SHAFT_NO_OPENING` / `OPENING_NO_SHAFT`: duct, lift, stair, OHT and plumbing shafts vs. slab
     openings, both ways, with size difference.
   - `STAIR_HEADROOM`: clear headroom under beams and landings along the stair flight below 2.1 m.
   - `GRID_DRIFT`: column centre vs. arch grid off by more than t (drawing set mismatch, often a
     revision issue rather than design).
3. **Carpet cost.** For each protrusion inside a room, the lost area is the polygon between the wall
   face and the column face; summed per room, flat and floor using the room boundaries; converted to sq
   ft and rupees with the project rate (Area Statement Pro rule set for carpet area per RERA). The sheet
   ranks flats by loss and columns by cost.
4. **Fixes the engineer pre-approves.** A small "allowed fixes" table signed off once per project by the
   structural engineer: shift column to flush up to X mm (default 50), rotate a rectangular column 90°,
   swap to a same-area section (230 x 750 to 300 x 600), or change orientation of eccentric columns at
   the edge. Flush Fit proposes the fix per issue, previews it (ghost of the new position), checks it
   does not create a new issue (beam bearing, cover to grid, other walls, lift clearances), and applies
   it in one `TransactionGroup` with a single undo. Anything outside the allowed table goes to RFIs.
5. **RFI pack.** One PDF + Excel: per issue type, a cropped plan snip and a 3D snip with dimensions,
   current vs. proposed, carpet loss, issue ID; the same issues as BCF for Revizto / BIMcollab / cad2bim.
   Replies (accept / reject / alternate) are typed into the Excel and read back; accepted ones are
   applied on the next run.
6. **Fit score and revision tracking.** Each run stores issues by stable key (rule + element ids +
   location); the next revision shows closed, new and still open; a fit score per floor (100 = no
   issues) is plotted over revisions. Change Radar can show it next to the drawing revision diff.
7. **Ghost view.** In the cad2bim viewer and in Revit: arch walls translucent, columns and beams solid,
   protrusions red with the depth as a label, headroom clashes amber with the clear height, sunken
   mismatches blue. One link to send the architect.

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)**: StrucBIM tools, **Rotate Column**, **Join Priority**, **Unjoin by
  Category** and **Filter Parameter** already work on columns, beams and walls; Flush Fit adds the
  arch-vs-structure brain on top, and its "apply fix" uses the same column editing code.
- **C2B**: already extracts columns, beams and slab panels from client DXFs into a canonical JSON; the
  arch side from a DXF is the same extraction with wall / door / window layers.
- **cad2bim**: the IFC viewer, actionable QA (`docs/design/actionable-qa.md`) and the BCF path from the
  IDS QA idea; the ghost view is a viewer mode.
- **etabs-mcp**: structural columns and beams straight from the ETABS model, so the check can run
  before Revit modelling even starts ("is this ETABS layout architect-friendly?").
- Idea links: **Area Statement Pro** (carpet area rules), **Change Radar** (revision diff), **Wall
  Wizard** (once columns are flush, block layout and plaster are cleaner), **Drain Line** (sunken slabs),
  **Twin Check** (design vs. BIM), **Markup Mirror** (RFI replies as PDF redlines).

**Wow moments.**
- The first page of the report: "This floor hides 486 sq ft of carpet inside walls = Rs 72 lakh".
- 312 issues with Revit's interference check become 41 real ones with Flush Fit, each with a fix.
- Press Flush: 27 columns slide into the walls, 14 RFIs go out, the score jumps from 61 to 94.
- The architect opens a browser link and sees exactly which window is cut by which beam, with the clear
  height in mm.

### Implementation

**Phase 1 — Read and check (3 weeks).**
- Arch reader (Revit API, Python 3 via the existing bridge): walls with location line and both finish
  faces per level, door and window instances with host, width, head and sill heights, rooms with
  boundaries and names, floors with level offsets for sunken areas, shaft openings.
- Structure reader: structural columns (section outline as a 2D polygon per level), beams (axis,
  width, depth, top level, soffit = top - depth), slabs and openings; IFC path through cad2bim's engine
  for non-Revit models.
- 2D geometry per level with Shapely: room polygons, wall face lines, column polygons; protrusion
  polygon = column polygon minus wall band, intersected with room polygon.
- Rules `COL_PROTRUDE`, `BEAM_HEAD`, `SUNKEN_MISMATCH`, `SHAFT_NO_OPENING` first; rule pack in YAML
  with tolerances.
- Output: Excel issue list + Revit 3D view with filters and colour overrides.
- Test fixtures: a sample G+4 with known issues (protruding columns, a beam over a door, a toilet on the
  wrong panel, a missing shaft opening); target zero false positives on columns that are flush.

**Phase 2 — Carpet cost, fixes, RFI pack (3 weeks).**
- Carpet loss per room / flat / floor using RERA carpet rules from Area Statement Pro; project rate in
  the settings.
- Allowed-fix table and fix engine: column shift (move by vector to flush with the wall face), rotate,
  same-area swap; post-check against beam bearing (beam end still on the column by min 75% width),
  cover to grid, other walls, door swings; preview with DirectShape ghosts; apply in one transaction
  group.
- RFI pack: plan / 3D snips via `ImageExportOptions` on temporary cropped views, PDF via reportlab,
  BCF 2.1 / 3.0 zip with viewpoints; Excel reply columns read back.

**Phase 3 — DXF and ETABS inputs, viewer, revisions (2-3 weeks).**
- Arch from DXF through C2B extraction (wall layer pairs to wall bands, door / window blocks to openings
  with head heights from a type table).
- Structure from ETABS through etabs-mcp, so the check runs on the analysis model.
- cad2bim ghost view mode, BCF import, link sharing.
- Stable issue keys and revision tracking, fit score chart, Change Radar hook.

**Tech notes / risks.**
- Wall faces: core vs. finish faces; default to finish face for carpet, core face for structural
  checks; configurable.
- Pilasters and intentionally thick walls: rule ignores protrusions where the architect drew the wall
  around the column (wall band contains the column); a per-room "accept as is" flag stops repeat noise.
- Column moves change the structure; the tool never moves outside the engineer's signed allowed-fix
  table, records who approved, and writes the change to an audit sheet; ETABS update is reported (not
  pushed) in Phase 1-3.
- Linked models with different shared coordinates: use the link transform; warn if levels do not
  match within 25 mm.
- Large projects: index columns and walls per level with an STRtree; a G+30 tower should check in
  under a minute.

### Future update
- Push accepted column offsets back to ETABS through etabs-mcp for re-analysis, and pull the new design
  sizes back (closed loop with Detail Drop and Twin Check).
- Furniture fit: check standard furniture blocks (bed, wardrobe, counter, car parking in Park Pilot)
  against columns and beams, "the king bed no longer fits after the column change".
- MEP layer: duct and pipe routes vs. beams (headroom under false ceiling, with Ceiling Composer) and
  sleeves (Sleeve Punch).
- AI mode: "Make all columns flush on the bedroom side within 50 mm and send the rest to the
  engineer" as one prompt through the cad2bim MCP agent.
- Early-stage mode: run on the architect's concept plan and the structural grid before modelling and
  propose a column layout that keeps every column inside walls (with Tower Stacker for the full tower).
- Developer dashboard: carpet saved per project, per revision, per wing; a number the BIM team can show
  the management every month.
