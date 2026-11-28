# 20261128 Idea: Culvert Cutter — a road or drain alignment drawn in DXF (plan + L-section + cross-sections) becomes a 3D corridor with box culverts, U-drains and retaining walls, rebar, BBS and BOQ in the browser, and a "drive the corridor" flythrough for the client

## Entry 1

```
date-time:   2026-11-28 (scheduled run)
idea name:   Culvert Cutter — "Our drawings are building-only. The same DXF-to-3D engine can build the culverts, drains and
             retaining walls of a road or township job, with every bar counted."
             Infrastructure packages (internal roads, nala / storm-drain crossings, box culverts, U-drains, retaining
             walls along a cut or fill) reach the BIM team as a plan DXF, a longitudinal section (L-section) with
             chainages and levels, and a few typical cross-sections. Nothing in the toolkit reads them. Culvert Cutter
             reads the alignment from the plan, the ground and finished levels from the L-section, picks up the culvert
             and drain marks and sizes from the labels, and builds: the road corridor surface, each box culvert (barrel,
             wing walls, return walls, apron, cut-off walls) at its chainage and skew, each U-drain run with its falls,
             and each retaining wall by chainage and height band, with rebar from the standard sheet for that type.
             It then gives the quantity (concrete, steel, excavation, backfill, PCC), the BBS, a chainage-wise BOQ and
             a "drive the corridor" flythrough.
potential:   High, and unusual: it opens a second market for the same engine.
             Day-to-day pain: infrastructure teams model in Civil 3D / OpenRoads for the corridor, but the RCC
             structures on it (box culverts, drains, walls) are drawn as repeated 2D standards and counted by hand into
             Excel, chainage by chainage. A 12 km road with 30 culverts and 6 km of drain has hundreds of repeated
             structures that differ only in span, height, skew, fill and length. The modeller redraws or re-counts them;
             a change in the L-section means redoing the list.
             My estimate (not measured): a drawing-driven build turns "one culvert a day by hand" into "a full
             package in an hour, then minutes per revision". Test it on one real package and time it before claiming
             this.
             Market, to my knowledge (not verified for this run): Civil 3D, OpenRoads and InfraWorks do corridors and
             are paid and heavy; Revit is weak at linear infrastructure; culvert and drain rebar is usually done in 2D
             CAD or Excel. A free, in-browser tool that goes from the contractor's own DXF to a counted 3D RCC model
             is not one I know of.
             Trending: Indian infrastructure spending and BIM-for-infrastructure mandates are a steady LinkedIn topic
             (reported, not checked for this run); a short clip of a road "building itself" chainage by chainage with
             culverts popping in is the kind of post infra and site teams share.
```

### Description

**The pain (a modeller's day on an infrastructure package).**
- The inputs are a plan DXF with the alignment and chainage ticks, an L-section with the ground line and the
  design (finished) level, a culvert and drain schedule ("Ch 3+450, 2 x 3 m x 3 m box, skew 15 deg, fill 1.2 m"),
  and the owner's standard drawings.
- Today each structure is hand-drawn or hand-counted. Eight culverts that "are all the same" are never quite the
  same: span, height, skew, wing length and fill all differ.
- When the designer lowers the road 300 mm at one chainage, the fill over the culvert, the wall heights, the
  excavation and the steel all change. Nobody knows what else moved.
- The client and the authority want a picture and a BOQ they can trust. The modeller has Excel and a
  screenshot.
- The same engine that turns a building plan into 3D already does the hard parts: reading a drawing, finding
  labels, placing parametric RCC, and counting bars. It has simply never been pointed at a corridor.

**What Culvert Cutter does.**

1. **Read the corridor** from the drawings:
   - *Alignment* from the plan polyline (straights and arcs), with chainage ticks or a start chainage and a
     scale. Clothoids are treated as arcs or polylines with a stated tolerance.
   - *Levels* from the L-section: ground line, design level, and the culvert marks written on it. A text like
     "Ch 3+450  2/3x3 box" is parsed into chainage and size.
   - *Cross-sections* (typical): carriageway, shoulders, side slopes, drain and wall positions, as the sections
     are drawn.
   - A "no guesses" rule: anything that cannot be read (an unclear unit, two chainage scales) is shown in a list
     to confirm, never guessed.
2. **Build the structures as parametric RCC** from a small, editable library of standard types:
   - *Box culvert:* single or multi-cell, with a top slab, bottom slab, walls, haunches, wing / return walls at
     skew, an apron, and cut-off walls; sizes from the label.
   - *U-drain / covered drain:* lined with a precast or cast-in-situ cross-section, with falls taken from the
     L-section and cover slabs where a crossing needs one.
   - *Retaining wall:* a cantilever or gravity wall by chainage and height band, with stepped footings where the
     ground changes.
3. **Put them on the corridor** at their chainage and skew, on the ground and design levels, with excavation and
   backfill volumes against the ground line.
4. **Rebar**: each type carries the owner's standard bar layout (a table the team edits and signs off once),
   scaled to the size. The bars are real, so the BBS, bar optimiser and weights from the existing tools work
   without change. The standard sheet's number is always the source, shown beside the bar.
5. **Check, do not design.** A short rules file flags the obvious: fill height outside the standard's range, a
   drain with no fall or a back-fall, a wall taller than its standard sheet allows, a culvert whose size is
   smaller than the one upstream, a wing wall that runs into the road. It does not size anything. Where a
   design is needed, it says "needs the designer".
6. **Outputs**:
   - 3D corridor in the viewer, coloured by structure type and by status (new, existing, to be removed).
   - *Chainage-wise BOQ:* concrete by grade, steel by diameter, PCC, excavation, backfill, formwork area, per
     structure and per kilometre, to Excel with live formulas (the same BOQ window).
   - *BBS* per structure and a typical-bar summary, to Excel.
   - *Standard-sheet set:* plan and section per culvert as DXF from the model (the model-to-DXF route that already
     exists), so the site gets drawings that match the 3D.
   - **Drive the corridor:** a camera runs along the alignment at eye height while structures build up in
     chainage order, with the chainage in the corner. It exports as a short video or a self-contained web link.
7. **Revision-proof.** A new L-section or schedule is read again and only the structures that changed are
   rebuilt, keeping their ids (the same revision behaviour the building pipeline has); a "what changed" list
   shows chainage, size and quantity delta.

### Implementation

How it fits what already exists in cad2bim (read from `docs/roadmap.md` and `docs/design/` for this run; nothing in
the source repository was changed):

- **Reading and labels:** reuse the CAD core's drawing reading and the label and mark logic of the CH template
  (`docs/design/ch-template-v2.md`), with a small "infra template": a layer name pattern for the alignment, the
  L-section frame and the culvert / drain / wall labels. A reader module sits beside the building one in
  `packages/pipeline` (a new phase set, as the CH v2 reader sits beside v1), without touching v1's `Entity` or
  `analyze`.
- **Geometry:** the structures are parametric solids placed along a polyline by chainage; the library lives in
  data files so an owner's standard changes it without code. The IFC classes are the building ones where they
  fit (IfcWall, IfcSlab, IfcFooting) and IFC 4.3's infrastructure entities where the reader supports them
  (IfcRoad, IfcAlignment); where it does not, it falls back to proxies with a clear property set, and says so in
  the QA window.
- **Rebar and BBS:** the standard sheets are stored as bar tables in the BBS design (`docs/design/bbs.md`): shape
  code, diameter, spacing, and a formula in the structure's dimensions. Bars come from the same bar generator as
  the building frame, so cut length, weight and the Excel export are the existing ones.
- **BOQ and 4D:** quantities feed the BOQ window; the build-up order (by chainage) is a simple 4D list, so the
  flythrough is the 4D player with the camera on the alignment.
- **Corridor surface:** a rough surface built from the plan, the design levels and the typical section. Not a
  replacement for Civil 3D's corridor; it is there so the structures sit in the right place, and the client sees a
  road.
- **Agent and Python:** exposed through the same API as the other tools (a tool such as `culverts()` and
  `chainage_boq()`), so an AI agent or a script can ask "list every culvert with fill over 3 m".
- **Privacy and licences:** everything runs on the device; the standards the owner provides stay in their project
  file. No GPL code; standards are the team's own data, with the owner's sign-off, not bundled.

**Build order (smallest useful first).**
1. Read the alignment and the L-section from one DXF; list the chainages and levels and show them as a plan and a
   profile in the viewer.
2. One structure type end to end: a single-cell box culvert, with rebar from one standard table, a BOQ and a BBS.
3. U-drain along an alignment with falls and a back-fall check.
4. Retaining wall by height band.
5. Revision rebuild and the "what changed" list.
6. Drive-the-corridor flythrough and standard-sheet DXF out.

**How to test it honestly.**
- Take one finished package, with the contractor's hand-counted quantities. Run Culvert Cutter. Compare the
  concrete and steel per structure and report the difference as a number, not "close".
- Time a real modeller doing the same package by hand and with the tool. Report both.
- Show two infrastructure engineers the flythrough and the BOQ and ask only: "would you use this on your next job,
  and what is missing?". Keep the negative answers.
- Treat the standard sheets as the main risk: the tool is only as right as the table it is given. Say that on the
  screen.

### Future update

- **More structures:** box and slab culverts with a pier, pipe culverts (Hume pipe) with headwalls, minor
  bridges' abutments and piers (with the existing pile and pad tools for foundations), crash-barrier and kerb
  runs, manholes and catch pits.
- **Read the corridor from LandXML or a Civil 3D export** when the team has one, instead of drawing it in DXF, and
  write the structures back as IFC for the highway consultant.
- **Utilities crossing:** read the services' plan and flag every crossing of a culvert or drain, with the
  clearance.
- **Weather and drains:** the rain-flow replay of the building idea (*Rain Run*) applied to the side drains: where
  does the water go, does it reach the outfall, and where does it pond.
- **Quantity by stage:** link the structure to the bill and progress ideas (*Bill Painter*, *Stage Stamp*) so a
  running bill for a road package is painted on the corridor chainage by chainage.
- **Site tagging:** QR tags per structure with a pour card and rebar manual (*Site QR Pass*, *Cage Manual*).
- **Drone or phone survey compare:** an as-built level survey set against the design levels, with a colour per
  chainage (*Plumb Line* applied to the corridor).

*References read for this run: the repository's `docs/roadmap.md` and `docs/design/` folder listing (cad2bim), and
the existing idea files in this folder, to avoid repeating them. A search of the 164 existing files found no
mention of culverts, highways, box girders, underpasses or MoRTH; retaining walls appear only inside the basement
and "deja-model" ideas. Nothing in the source repository was changed.*
