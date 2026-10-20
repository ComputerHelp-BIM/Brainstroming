# 20261020 Idea: Exit Map — fire-egress travel-distance heatmap, NBC exit check and a 3D evacuation replay, from the model

## Entry 1

```
date-time:   2026-10-20 (scheduled run, third entry of the day)
idea name:   Exit Map — "Press one button. Every floor turns into a green-to-red map of the walk to the nearest
             fire exit, every failing room and door is listed with the rule it breaks, the fire-NOC drawings and
             the occupant-load sheet come out ready to print, and the browser plays the building emptying in 3D."
             A pyRevit button (AnonGee tab) that reads rooms, doors, stairs and walls on every level, builds a walk
             graph, measures the real path (around walls, through doors, down stairs) from every point of every
             room to the nearest exit, computes occupant load and exit capacity per floor, checks them against an
             editable rule pack (NBC 2016 Part 4 first, plus project or authority overrides), and exports
             (a) colour-filled heatmap views, (b) fire-NOC plans with exit signs, travel lines and widths
             annotated, (c) an Excel/PDF compliance sheet, and (d) an IFC + path file that the cad2bim web viewer
             plays back as an evacuation replay.
potential:   High, and every project needs it once per design stage, then again after every layout change.
             Fire-NOC drawings, exit-width calculations and "is this flat too far from the stair?" questions are
             done today by hand: a modeller draws polylines through doors on each floor, measures them, counts
             occupants from area tables, divides by the unit-width numbers, and redraws all of it whenever the
             architect moves a wall. On a 15-floor residential or a hospital that is 1-3 days per submission round,
             and the mistakes are found by the fire officer, which costs weeks.
             Time saved: first full check of a 15-floor building drops from ~2 days to ~10 minutes; each re-check
             after a layout change from ~half a day to one click. The rule failures arrive at the modeller's desk
             during design, not as an objection letter from the authority.
             Trending angle: "fire safety by design", "AI code checking" and evacuation simulations (thousands of
             little dots streaming to the stairs) are among the most shared AEC posts on LinkedIn and Instagram
             in 2025-26, and they get attention because they look like a video game but are about life safety.
             The 15-second clip "plan turns into a heatmap -> red room highlighted -> door widened -> map turns
             green -> 3D replay of the floor emptying" is the post.
             Market: Revit has Path of Travel / Route Analysis (free, 2020+), but it draws single paths you place by
             hand; it has no heatmap, no occupant load, no exit-width check and no code rules. Pathfinder (Thunderhead),
             Oasys MassMotion and EXODUS are paid, expert evacuation simulators with their own model import.
             Solibri and UpCodes check code rules but are paid per seat and do not cover NBC India out of the box.
             To my knowledge nothing free goes from a live Revit model to an NBC-checked heatmap, NOC sheets and a
             browser replay in one press.
```

### Description

**The pain (day-to-day of a BIM modeller in an Indian design office).**
- The fire-NOC drawing set is redrawn by hand for every authority submission: travel lines, exit signs, staircase
  widths, door widths, refuge areas, occupant counts per floor. When the plan changes, every number is stale.
- "Travel distance" is measured as a straight line or a rough polyline, so the worst room is often missed: the
  one where the real walk goes round a lift core and a shaft.
- Occupant load and exit capacity are calculated in a side Excel that is never linked back to the rooms, so a
  room changing from "store" to "office" (very different occupant density) does not change the stair width check.
- Architects ask the BIM team "can we delete this second stair?" and nobody can answer in less than a day.
- The toolkit already has the pieces: Revit room and level access in the AnonGee pyRevit tab, the in-Revit bridge
  and command bus (`AnonGee.extension/startup.py`, `bridge/`), `Export Schedule` for Excel output, the cad2bim
  three.js IFC viewer with picking (`packages/engine`), and the `@cad2bim/mcp` helper. What is missing is a
  walk graph, a rule pack and a picture that a non-expert understands in one second.

**The idea, in six parts.**

1. **Walk graph from the model (no new modelling).** Per level: rooms (boundaries from `Room.GetBoundarySegments`),
   walls and columns as obstacles, doors as openings with their clear width, stairs and ramps as vertical links
   with their flight width and riser count, and exits picked automatically (doors with `Function = Exterior`,
   doors into a stair enclosure, or a shared parameter `Exit Type`). The floor is rasterised into a 250 mm grid
   (configurable); free cells become graph nodes, doors are bottleneck edges. Rooms with no boundary, doors not
   hosted in a wall and stairs not reaching a level are reported first, because they silently break the result.

2. **Distance field = the heatmap.** A multi-source Dijkstra (fast-marching option for smooth curves) from every exit
   gives the real walking distance at every cell. Each room gets its worst point, its average and the path that
   produces the worst point. Rooms are colour-filled green / amber / red against the allowed distance for their
   occupancy, and the worst path is drawn as a detail line with its length. Dead-end corridors (cells with only one
   way out beyond the allowed dead-end length) are hatched separately.

3. **Occupant load and exit capacity (linked to rooms).** Each room's occupancy and density come from its `Room
   Name` / `Department` / a shared parameter, mapped through the rule pack (e.g. office, residential, assembly,
   mercantile, hospital ward). Occupants per floor are summed and assigned to exits by nearest walk distance
   (with a "lose the largest exit" check, as codes require). Required width = occupants / persons-per-unit-width
   x unit width, compared with real door clear width and stair clear width. Every number in the sheet links back
   to the room or door ID, so clicking a red row in Excel selects the element in Revit (via the bridge).

4. **Editable rule pack, NBC first.** Rules live in a small YAML file, not in code:
   `max_travel_m` per occupancy (sprinklered / not sprinklered), `max_dead_end_m`, unit exit width and persons per
   unit for doors / stairs / ramps, minimum stair and corridor width per occupancy and building height, number of
   exits by occupant load, refuge-area triggers. Ship `nbc-2016-part-4.yaml` with values typed from the code and a
   "check against the current edition and local authority" header; allow `project-override.yaml` (e.g. a state
   fire service circular or an IBC / UAE Civil Defence pack contributed later). Each failure cites its rule ID and
   clause so the BIM lead can verify it, and the tool never claims "approved", only "passes rule pack vX".

5. **One-click outputs.**
   - Heatmap views per level (filled region overlay on a duplicated plan, legend, so the original views stay clean).
   - Fire-NOC plan sheets: exit signs and arrows placed at exits and turns of the main paths, travel lines with
     length, door / stair widths dimensioned and tagged, occupant count per floor in a title-block table.
   - `ExitMap_<project>_<date>.xlsx` and PDF: per-floor occupant load, required vs provided width, worst rooms,
     dead ends, missing data, plus a front page with the pass / fail count by rule.
   - `exitmap.json` (graph, distances, paths, occupants per exit) next to the IFC export.

6. **Evacuation replay in the browser (the wow).** cad2bim's viewer loads the IFC plus `exitmap.json` and plays a
   simple agent replay: each room spawns its occupants as dots, dots follow the distance field down the gradient,
   doors and stairs throttle flow by their width (persons per metre per second), queues build up visibly at narrow
   doors. A timeline shows "floor clear at 2 min 40 s", the slowest exit glows. This is not a certified evacuation
   simulation (and says so on screen); it is a design-stage picture that makes the bottleneck obvious to an
   architect or a client in 10 seconds, and it can be recorded straight to an MP4 / WhatsApp clip.

### Implementation

- **Where it lives.**
  - `pyRevit-addons/AnonGee.extension/AnonGee.tab/<panel>/ExitMap.pushbutton` — collector, graph build, rule check,
    Revit outputs (views, filled regions, tags, detail lines, sheets). CPython3 engine for the numeric part
    (`numpy` / `scipy.sparse.csgraph.dijkstra` or `scikit-fmm`), IronPython fallback with a plain heap Dijkstra
    on a coarser grid.
  - `pyRevit-addons/AnonGee.extension/lib/exitmap/` — `graph.py` (rasterise, doors, stairs), `field.py` (distance
    field, worst paths), `occupancy.py`, `rules.py` (YAML loader + checks), `report.py` (Excel via the existing
    `Export Schedule` helpers, PDF), `rules/nbc-2016-part-4.yaml`.
  - `cad2bim/packages/engine` — an `ExitReplay` layer: instanced points (one draw call for thousands of agents),
    distance-field texture per storey, timeline control in the bottom panel of `@cad2bim/ui`.
  - `@cad2bim/mcp` tool `exitmap.check` / `exitmap.what_if(door_id, new_width)` so an agent can answer "what if we
    widen D-204 to 1200?" or "can we drop stair S2?" in chat.
- **Steps.** (1) Collector and data-health report for rooms / doors / stairs; (2) single-level raster + Dijkstra +
  room fill; (3) multi-level with stairs as links; (4) occupant load and width checks with the NBC YAML;
  (5) Excel / PDF and NOC sheet generation; (6) `exitmap.json` + replay in cad2bim; (7) MCP what-if tools.
- **Performance target.** 250 mm grid on a 2,000 m² floor is ~32k nodes; Dijkstra in scipy is well under a second
  per floor, so a 30-floor tower checks in under a minute including Revit writes. "Re-check this level only" for
  fast iteration.
- **Testing.** Small hand-calculated fixtures (a corridor with one door, an L-shaped flat round a core, two stairs
  with one blocked) whose distances and widths are checked to ±0.3 m and exact occupant counts; one real project
  compared against the last manually prepared NOC set.
- **Effort.** MVP (steps 1-4, Excel output, heatmap views): ~3-4 weeks for one developer. Sheets + replay: another
  3-4 weeks.

### Future update

- **Live mode.** Dynamic updater on room / door / wall changes re-colours only the affected level, so the heatmap
  updates while the architect drags a wall.
- **Fix suggestions.** "Widen D-204 to 1,050 mm", "add a door here cuts worst distance from 41 m to 27 m",
  "this dead end needs to be 3 m shorter" — computed by trying candidate door positions on the walk graph.
- **More rule packs.** State fire service amendments, IBC / NFPA 101, UAE Civil Defence, Saudi SBC 801, contributed
  as YAML by users; a "rule pack diff" view when the code is revised.
- **Accessibility pack.** Same walk graph for wheelchair routes (ramps instead of stairs, door clear widths,
  turning circles), lift-to-refuge distances and the Harmonised Guidelines for universal accessibility.
- **Fire-fighting layer.** Hose-reel / hydrant / sprinkler coverage as a second distance field (hose length along
  the walk path, not a circle), plus fire-tender access and turning on the site plan.
- **Daily-use spin-offs.** "Nearest toilet / nearest lift" heatmaps for hospitals and offices, cleaning and security
  patrol route lengths, and wayfinding signage placement from the same graph.
- **Connect to existing ideas in this repo.** Change Radar can flag "this revision made room 1204 fail travel
  distance"; Model Reel can append the evacuation replay to the weekly progress video; IDS QA can enforce that every
  room has the occupancy parameter Exit Map needs.
