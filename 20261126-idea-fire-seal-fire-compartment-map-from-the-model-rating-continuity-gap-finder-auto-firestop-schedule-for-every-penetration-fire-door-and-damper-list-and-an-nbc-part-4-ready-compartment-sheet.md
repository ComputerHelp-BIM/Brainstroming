# 20261126 Idea: Fire Seal — fire compartment map from the model, rating-continuity gap finder, auto firestop schedule for every penetration, and a fire-door / damper list ready for the fire NOC

## Entry 1

```
date-time:   2026-11-26 (scheduled run)
idea name:   Fire Seal — "Show me where the fire wall stops being a fire wall."
             Drop the IFC (or the DXF-built model). Fire Seal colours the building by fire compartment, traces every
             fire-rated wall, slab, shaft and door as a closed envelope, and shows the holes in it: a 2-hour
             wall that stops at a false ceiling, a duct that crosses a rated wall with no damper, a rated shaft
             with a plain door, a slab opening left open between two compartments. Every hole is a pin in 3D
             with a one-click fix suggestion. Then it writes the firestop schedule (one row per penetration:
             location, rating needed, seal type, size), the fire-door and fire-damper list, and the compartment
             sheet the fire consultant and the authority ask for.
potential:   High. Fire compartmentation is checked late and by eye. In most teams I know of, nobody owns it
             between architect, structure and MEP; each trade draws its own part and the gaps appear on site as
             a fire-stop RFI, or at the fire NOC inspection as a rejection. The numbers below are my estimates,
             not measured:
             - Today: 1 to 3 days per building to mark compartments on plans by hand, plus a firestop count taken
               from a sleeve list that is already out of date.
             - With Fire Seal: first compartment map in minutes, the firestop schedule in seconds after every
               revision, and the gap list before the model is issued.
             Why it wins: it is the third party-proof check people forget until the end, and the gap list
             is a visual hook ("the red pins in the fire wall") that a modeller can screenshot into a coordination
             meeting.
             Market: to my knowledge (not verified for this run), firestop BIM content comes from product vendors
             and is tied to their products; compartment checking in a rule checker (Solibri and similar) is paid and
             needs a rule set written by an expert; Revit does not trace a rated envelope or count penetrations by
             itself. A free, browser-based tracer that works on IFC and that keeps files on the device is not one I
             know of.
             Trending: post-fire safety audits, stricter fire NOC checks in Indian cities, and "AI + code
             compliance" posts on LinkedIn. A before/after reel (red gap pins turning green as fixes are accepted)
             fits the "digital compliance" feed. Treat this as a signal of interest, not a measured trend.
```

### Description

**The pain (daily life of a BIM modeller and coordinator).**
- The architect sets the fire rating on walls and doors in one file. Structure sets slab and column ratings in
  another. MEP draws ducts, pipes and cable trays through all of them in a third. Nobody checks the *whole* envelope.
- The firestop list is made late, from a sleeve list, by hand. It misses penetrations that were added after the list.
- Common gaps found only on site: a rated wall that stops under the slab but the rating is lost at the false ceiling
  void; a rated shaft with unrated inspection door; a duct through a rated wall with no fire damper in the model; a
  shaft opening in the slab left open between two compartments; a lift lobby door without a rating.
- The authority wants a compartment drawing, a fire-door schedule and a firestop statement. These are typed again
  at the end of every project.

**What Fire Seal does, in four screens.**

1. **Compartments.** Press *Find compartments*. Fire Seal treats rated walls, rated slabs and rated shafts as
   barriers and flood-fills the spaces between them, level by level and through shafts. Each compartment gets a
   colour, a name (`L03-C2`), an area, and the rating of its enclosure. The user can drag a boundary or mark a wall
   as rated; the map updates live. A compartment area or length over the limit is flagged.
2. **Continuity (the gap finder).** For each rated barrier Fire Seal checks the envelope is closed:
   - wall top meets the underside of the slab above (not a false ceiling, not a beam pocket, not a gap);
   - wall ends meet a column, a wall or another rated element, with no unrated gap;
   - the wall rating is not lower than the rating needed by the rule pack for that space pair;
   - openings in rated barriers carry a rated door, glazing or damper;
   - slab openings between compartments are closed or inside a rated shaft.
   Each failure is a pin with the reason in plain words and the element highlighted in 3D.
3. **Penetrations.** For every ductwork, pipe, tray or cable element that crosses a rated barrier (geometric
   intersection of the element and the barrier), Fire Seal creates one penetration row: where, which barrier,
   rating needed, element type and size, suggested seal class (collar, wrap, mortar, damper) from an editable
   table. It merges with the sleeve list from the existing *Sleeve Punch* idea so there is one list, not two.
4. **Outputs.**
   - **Firestop schedule** (Excel / CSV / PDF), with a stable ID per penetration, so the site can tick them off.
   - **Fire door and damper schedule**, with rating, size, level, compartment pair and swing.
   - **Compartment sheet**: a plan per level with compartments coloured, rated lines drawn heavy, doors and dampers
     marked, a legend and a table. Output as DXF (opens in AutoCAD / Revit) and PDF.
   - **BCF** issues for the remaining gaps, each with a viewpoint, so they land in the usual issue tracker.
   - A **site check-off view** (phone): scan the QR on a firestop, mark *sealed*, add a photo. This feeds the
     handover record, and the model shows sealed / open penetrations in green / red.

**Rule packs, not hard-coded law.** The ratings required (for example, stair enclosure, lift shaft, riser,
refuge area, basement parking, high-rise separation) differ by code, occupancy and height. Ship an editable
pack, such as *NBC 2016 Part 4 starter*, marked clearly "starter values: confirm with the project's fire
consultant and the local authority". Users keep their own packs per city or client. Fire Seal checks the model
against the pack; it does not certify the building.

### Implementation

Fits the existing stack: browser only, IFC read by web-ifc in a worker, three.js viewer, React panels, files stay on
the device. No new server. No GPL code. All of the following uses plain geometry and graph code.

1. **Data reading (`packages/engine`, new `fire/` module).**
   - Pull ratings from `Pset_WallCommon.FireRating`, `Pset_SlabCommon.FireRating`, `Pset_DoorCommon.FireRating`,
     `Pset_CurtainWallCommon`, and the model's own parameters when the IFC came from Revit (mapped by name).
   - Spaces from `IfcSpace` and `IfcRelSpaceBoundary` when present. When they are absent (common in
     structural-only or DXF-built models), build rooms from the walls with a 2D polygon flood fill per level.
   - Elements that carry no rating are listed as *rating unknown* with a bulk-set tool (select, set 2 h), because
     most models come with the property blank.
2. **Compartment graph.** Build a graph: nodes are rooms and shafts, edges are doors, openings and unrated
   boundaries. Rated barriers cut edges. Connected components are compartments. Shafts connect levels vertically.
   Pure TypeScript, a few hundred lines, tested on small synthetic IFCs with known answers.
3. **Gap rules.** Each check is a small function over the graph and geometry (wall-top gap measured against the
   slab / ceiling level, end gap measured to neighbours, opening hosted in barrier without a rated door). Results
   are `{ruleId, guid, level, reason, fix}` in the same shape as the existing QA panel, so they appear in the QA tab
   and in the BCF export without new UI.
4. **Penetrations.** Use a bounding-volume pass (the engine's spatial index) to find MEP elements that intersect
   a rated barrier, then an exact test on the mesh pair for the survivors. Group by barrier and run.
   Cap work per barrier so a 1 million element model stays responsive (see `docs/design/million-elements.md`).
5. **Outputs.** XLSX and CSV writers already exist for BOQ and BBS; reuse them. Compartment sheet reuses the DXF
   writer and the CH v2 layer conventions so lines come out on named layers. PDF reuses the print path.
6. **Phone check-off.** Small static page with the penetration list per level, QR deep link per row, photo stored
   on the device and added to a handover file the user exports. Nothing leaves the device unless the user sends it.
7. **Testing.** Unit tests on synthetic IFCs (closed envelope, gap at ceiling, duct without damper, slab opening);
   a golden file of firestop rows. Use two or three real models in a scratch folder, never in git.
8. **Delivery order (small steps).**
   - Step 1: ratings reader + *rating unknown* list + bulk set. About a week.
   - Step 2: compartments by flood fill + colour map. About a week.
   - Step 3: gap rules + QA pins. About a week.
   - Step 4: penetrations + firestop and door schedules. About a week.
   - Step 5: compartment sheet (DXF / PDF) + BCF. A few days.
   These are my estimates of effort, not measured.

**Risks and honest limits.**
- Models with no ratings or no rooms give weaker results; the tool says so and leads the user to fix it first.
- Required ratings come from the rule pack; wrong pack means wrong result. Always show the pack name on every output.
- Not a code certification. The sheet header must say "checked against pack X, to be confirmed by the fire consultant".
- Curved or sloped barriers and very dense MEP need more geometry testing.

### Future update

- **Smoke and pressurisation view:** mark smoke-stop doors and lobbies; check lift lobby and stair pressurisation paths.
- **Link to *Exit Map*:** compartment-aware egress, with travel distance counted within and across compartments.
- **Link to *Hanger Hero* and *Sleeve Punch*:** one shared penetration list, with hanger and sleeve data on each row.
- **Auto fix:** extend a wall up to the slab above, swap a door type for a rated one, insert a damper family
  (written back through the Revit bridge, always with a person's confirmation).
- **Revision watch:** after every new IFC, show *new gaps*, *closed gaps*, and *new penetrations since last issue*.
- **Product catalogue:** the user's own firestop products (collar, wrap, mortar, damper) with rating and size range,
  so the schedule names real products. Kept as the user's own file, no vendor tie.
- **Fire strategy report:** compartment table, rating table, and open gaps as the appendix of the consultant's report.
- **Sprinkler and detector zones** inherited from compartments (links to *Ceiling Composer*).
- **Agent tool:** expose `find_fire_gaps` and `firestop_schedule` through the cad2bim MCP helper so an AI agent can
  answer "which rated walls are open at level 5?" with a 3D proof.
