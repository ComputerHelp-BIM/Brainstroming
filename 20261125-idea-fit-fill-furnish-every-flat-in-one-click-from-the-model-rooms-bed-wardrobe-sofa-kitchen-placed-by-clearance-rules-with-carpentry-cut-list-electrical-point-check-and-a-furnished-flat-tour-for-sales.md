# 20261125 Idea: Fit-Fill — furnish every flat in one click from the model's rooms; bed, wardrobe, sofa and kitchen placed by clearance rules, with a carpentry cut list, a switch and socket check, and a furnished-flat tour for sales

## Entry 1

```
date-time:   2026-11-25 (scheduled run)
idea name:   Fit-Fill (Rule-Based Furniture Layout + Clearance Check + Carpentry Cut List + Furnished Tour)
potential:   High for residential, hostel, hotel and office teams. Every sales brochure, client meeting
             and interior hand-over needs "the furnished flat", and today a modeller places furniture
             by hand, flat by flat, only to find at the interior stage that the bed blocks the door
             swing or the wardrobe covers the only socket. Fit-Fill reads rooms, doors, windows and
             points from the model, proposes 2 to 3 layouts per room type, checks clearances, and
             repeats the chosen layout across all repeat flats. Estimated saving (an estimate, not
             measured): half a day to 2 days per typical flat type, and fewer "furniture does not fit"
             calls from the client. Wow: "36 flats furnished in 20 seconds", then a walk-through of
             the furnished flat that works as a ready Instagram or LinkedIn reel.
```

### Description

**The pain (weekly in any residential or hospitality project).**
- The client asks "will my king bed and the wardrobe fit in the master bedroom?" The answer today is a
  hand-drawn 2D furniture plan, or a long wait for the interior team.
- Furniture is placed by hand in the 3D model for sales visuals. It looks right but is never checked:
  door swings, 600 mm walk space beside the bed, 900 mm in front of a wardrobe, the AC throw, the
  window sill height behind a sofa, the only socket behind a bed head.
- A change to one wall means every flat has to be re-furnished by hand.
- The carpenter and the modular-kitchen vendor need sizes. Those come from a second, separate sheet
  that nobody keeps in step with the model.

**What exists today (a short search on 2026-11-25; re-verify before saying "nothing like it").**
- The search found no Revit plugin that furnishes whole apartments by rule as native BIM objects.
  Closest: Morphis (generative layouts of spaces, not furniture), Spazio (copies furnished rooms
  from one Revit model to another, $39), usBIM.codesign AI "Virtual Staging" from ACCA (furniture for
  renderings, not rule-checked placement), and Planner 5D (AI furniture placement, not BIM)
  ([AEC Magazine AI directory: Morphis](https://aidirectory.aecmag.com/entry/morphis/),
  [Autodesk App Store listing](https://marketplace.autodesk.com/apps/81708ba0-12b9-4668-91e0-82a05c009fa6),
  [ACCA Revit AI](https://www.accasoftware.com/en/revit-ai),
  [Planner 5D](https://planner5d.com/use/ai-furniture-placement)). One search is not proof; check again.
- Free web room planners exist but they start from a blank 2D room, not from your BIM model, and do not
  know the door swing, window, socket or beam position.
- In this repo: Same-Same repeats edits across typical flats; Tile Tale does tile layout; Window Wright does
  window joinery; Live Wire places electrical points; Cool Load places AC. None places furniture or checks
  the clearance of use. Fit-Fill reuses their outputs (points, AC, door swings) as inputs.
- Trend (the theme in the feeds, from trade press, not LinkedIn data, which I cannot read from here): AI
  staging, home-tour reels and "furnish it with a prompt" posts. Fit-Fill is the plain, checkable,
  no-key version, and is native to the BIM model.

**The idea.** A **Fit-Fill** panel in the cad2bim web viewer (and the same through the agent helper).

1. **Read the room.** For each IfcSpace (or a closed loop of walls) the engine reads the polygon, door
   positions and swing arcs, window sills and widths, columns and beam drops, and any existing electrical
   points and AC units.
2. **Pick a room type, get layouts.** The room type comes from its name ("Master Bed", "Bed 2", "Living",
   "Kitchen") or is guessed from area and doors. For each type the engine proposes 2 to 3 layouts from a
   small, readable rule set, for example: bed head on the wall with no window, 600 mm free on both long
   sides, 900 mm in front of the wardrobe, wardrobe not across a door swing, TV wall facing the sofa,
   kitchen work triangle between 3.6 and 6.6 m. A layout that cannot meet a rule says which rule and by how
   many mm.
3. **Sizes from a catalogue you own.** A plain CSV or Excel of furniture (name, width, depth, height,
   clearances, price) is the catalogue. A default Indian-size catalogue is provided (single, double, queen,
   king beds; 3-door and 4-door wardrobes; 2-, 3-seat sofas; 4- and 6-seat dining). Swap the catalogue and
   every layout re-runs.
4. **Slider, not a wizard.** One slider for "bed size" and one for "storage priority" re-run in under a
   second. A 3D preview shows the clearance zones as soft green and red footprints.
5. **Apply to all repeat flats.** Pick a layout in one flat; Fit-Fill uses the typical-unit match
   (Same-Same) to place it in every identical or mirrored flat, mirroring the furniture too. Flats whose
   room differs get their own proposal and a note.
6. **Checks that matter.** Door swing hit, wardrobe over a socket or switch, bed head under a window,
   furniture inside the AC throw zone, furniture crossing a beam drop that blocks a loft or wardrobe top,
   walk path from door to window less than 900 mm. Each is a finding pinned to the 3D object.
7. **Outputs.** (a) Furniture as IFC proxy objects with a type name, to Revit as generic models.
   (b) A furniture plan DXF per flat type. (c) A carpentry sheet: wardrobe, loft, bed back and TV unit
   sizes per flat type with panel counts (sizes only, no manufacturing claims). (d) A BOQ line per flat
   type for the interior budget, with rates from the catalogue.
8. **Sales tour.** A one-click furnished walk-through of one flat as a self-contained HTML (see Model
   Capsule) or a short recorded reel (see Model Reel).

**The wow 1: all flats, one click.** A tower of 36 flats goes from empty shells to furnished in about 20
seconds, mirrored correctly.

**The wow 2: the "does my king bed fit?" answer.** Tap a bedroom, pick King, and the answer is on screen
in a second with the exact spare millimetres per side.

**The wow 3: the quiet catch.** "Flat 14 master wardrobe covers the only socket" found before the interior
team starts.

### Implementation

How this fits the repo today (read from `CLAUDE.md` and `docs/`, not from code, so check the names first):

- **Where it lives.** A new module in `@cad2bim/engine` for room reading and the layout solver; a panel in
  `apps/web` using the design system (tokens only, no raw values). The catalogue is a project file
  (`.c2b`) section and a CSV import. No new package and nothing uploaded; everything runs on the device,
  as the rules in `CLAUDE.md` require. Each package that changes gets its own version bump with a
  changelog, README and `guide.ts` entry (`docs/workflow.md#versions`).
- **Room reading.** From IfcSpace boundaries when present, otherwise from walls in the model. Doors give
  swing arcs (door width and hand), windows give sill and head height.
- **Solver.** Rooms are small, so a simple search works: for each piece, in priority order, try wall-aligned
  positions at 0/90/180/270 degrees along each wall in 50 mm steps; keep positions that pass the hard
  rules (no overlap, door swing, clearances); score by soft rules (bed head wall, TV wall, spare space);
  keep the top 3 distinct layouts. No AI is needed, and the same input always gives the same answer.
- **Repeat flats.** Reuse the transform and mirror logic of the typical-unit matcher (Same-Same), so the
  chosen layout is carried across with its frame.
- **Checks** run on the same geometry as the model QA (the IDS pipeline), so findings can leave as BCF.
- **Export.** Furniture is written as IfcFurnishingElement with a Pset (catalogue code, size, price); the DXF
  uses the existing DXF writer.
- **Phase 1 (about 2 to 3 weeks of work, an estimate):** bedroom and living layouts, door swing and clearance
  checks, one flat. **Phase 2:** kitchen work triangle, apply to all flats, catalogue CSV, carpentry sheet.
  **Phase 3:** socket and AC checks, furnished tour.
- **Risks.** Layout taste is subjective, so the tool offers options and shows rule results rather than one
  "best" answer. The catalogue sizes must be the user's own; the defaults are generic and are labelled so.

### Future update

- Vastu mode: use the zones from Sun Compass so the bed head and kitchen face the preferred direction.
- Accessible rooms: wheelchair turning circle and bed-side clearance from Access Pass.
- Hostel, hotel and hospital room kits (4-bed dorm, hotel key, patient room with headwall).
- Link the catalogue to a vendor price list so the furnished flat has a live interior budget (see Rate Rocket).
- Natural language: "make Bed 2 a study with a single bed" using the plain-English command layer.
- Client mode: the buyer drags furniture in the shared tour, and the checks keep running.
