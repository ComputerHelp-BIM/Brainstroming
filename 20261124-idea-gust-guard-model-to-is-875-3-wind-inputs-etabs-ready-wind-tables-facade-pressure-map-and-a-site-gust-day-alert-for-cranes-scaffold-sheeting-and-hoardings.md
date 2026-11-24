# 20261124 Idea: Gust Guard — model to IS 875-3 wind inputs, ETABS-ready wind tables, a facade pressure map, and a site "gust day" alert for cranes, scaffold sheeting and hoardings

## Entry 1

```
date-time:   2026-11-24 (scheduled run)
idea name:   Gust Guard (Model-Read Wind Inputs + Facade Pressure Map + Construction-Stage Gust Alert)
potential:   High. Every building needs wind load, and every wind sheet starts with the same slow step: someone
             reads width, depth, height, openings and neighbours off drawings and types them in. The model already
             holds these. The second half is new for most teams: wind on the temporary works (tower crane, scaffold
             sheeting, hoardings, aluminium formwork panels, parapet shuttering) is rarely checked in the BIM, yet
             it is when things fall. Estimated saving (an estimate, not measured): 1 to 3 hours per wind sheet per
             revision, and one fewer "which floor is the crane tied to and what wind speed stops lifts" argument
             per project. Wow: press one button, the building's windward and leeward faces light up with pressure,
             arrows stream past the model, and a banner says "Tuesday: gusts 58 km/h, stop lifts above floor 14".
```

### Description

**The pain (a few times a project, every project).**
- Structural engineer asks the modeller for "building dimensions for wind": width and depth per direction, height
  per level, opening percentage, whether a podium or a taller neighbour shields the face. The modeller measures
  by hand, the revision changes the plan, and the wind sheet is out of date.
- Facade, ACP and glass teams need local pressure (corners, edges, parapets, canopies). Nobody has it on the model.
- Site asks: "At what wind speed do we stop crane lifts, and is the scaffold sheeting safe?" The answer lives in
  a PDF the safety officer found last year.
- After revision 5 nobody knows if the wind inputs match the model.

**What exists today (checked once; re-verify before claiming "nothing like it").**
- Wind generators exist inside analysis tools: [STAAD.Pro's IS 875 (Part 3): 2015 generator](https://docs.bentley.com/LiveContent/web/STAAD.Pro%20Help-v2024/en/topics/User_Interface/r-stpst-dbox_Wind_Load_Gen_IS_875_P3.html)
  (paid), [Tekla Structural Designer's IS 875 wind wizard](https://support.tekla.com/doc/tekla-structural-designer/2025/is_wind_wizard)
  (paid) and [SkyCiv's IS 875-3 load generator](https://skyciv.com/docs/load-generator/wind/is-875-wind-load-calculations/)
  (detailed reports need a paid plan). In all of them the geometry is typed in or drawn again; none reads an
  existing Revit or IFC model.
- A web search did not find an IS 875-3 wind automation for Revit or Autodesk Forma, and I could not confirm what ETABS
  offers. Treat as "not found", not "does not exist".
- In this repo, wind is only a side note: Skin Cutter (bracket load), Load Loader, Load Path X-Ray, Sun Roof
  (PV mounting) and Basis Book mention IS 875-3 inside their own scope. None reads wind inputs from the model, maps
  facade pressure, or checks temporary works.
- Trend: the feeds in late 2026 carry AI and agent stories in BIM, plus fast "wind and microclimate in the early
  model" demos (Forma-style). Trade press, not LinkedIn data; I could not read LinkedIn from here. A browser tool
  that does the Indian code and shows the answer on your own IFC, with no upload, is a different, practical angle.

**The idea.** A **Gust Guard** panel in the cad2bim web viewer; the same calls through the agent helper and Python
API.

1. **Read the model.** For each wind direction (0, 90, 180, 270 or any angle): projected width per level, building
   height, h/w and l/w ratios, plan shape, roof type, opening percentage per face (windows and doors from the IFC),
   parapet height. Shown in a table with every number clickable to its place in 3D.
2. **Site inputs.** Pick the city or type basic wind speed Vb; terrain category, topography, importance and design
   life factors. The code edition (1987 or 2015) is a visible switch, never assumed.
3. **Neighbours.** From the model's context (Geo Drop, Yard Smith) or drawn boxes: mark shielding by taller
   neighbours, shown as a warning only; the engineer decides.
4. **Wind table per level.** Design pressure and force per level per direction, as an Excel sheet and an
   ETABS-ready load table, with the formulas shown beside each number. Gust Guard prepares inputs and a check
   sheet; **the engineer signs the wind design**. It never claims to replace analysis.
5. **Facade pressure map.** External and internal pressure coefficients by zone (windward, leeward, side, corner,
   edge, roof) become colours on the facade. Click a panel: its design pressure and uplift. Feeds Skin Cutter's
   bracket check.
6. **Temporary-works gust table.** For crane, scaffold with or without sheeting, hoardings, formwork panels and
   parapet shuttering, the person enters the stop-work wind speed from the maker's manual (never guessed) and
   Gust Guard shows, for each item, the level and zone where it is most exposed.
7. **Gust-day alert.** With a place and the 4D plan, the person may fetch a wind forecast on a button (only the
   location goes out, never the model). A banner lists days where gusts pass a stop-work speed, and which 4D
   tasks that hits ("Day 12: crane lift of reinforcement cages, floor 14").
8. **Wind replay.** Animated flow arrows over the model, with pressure going red on windward faces. It is a
   teaching picture from code coefficients, **not CFD**, and is labelled so.

**Wow 1: the facade lights up.** Tower in, pressure colours on every face in seconds.
**Wow 2: the revision follows.** A floor is added, the sheet and the map re-run, and "wind table changed: +2 levels"
shows in the change note.
**Wow 3: the Monday banner.** "Gusts 58 km/h Tuesday: lifts above floor 14 stop" over the 4D, shareable as a card.

### Implementation

How this fits the repo today (read from `CLAUDE.md` and `docs/`, not from code, so check names before starting):

- **Where it lives.** A new `wind` module in `@cad2bim/engine` (IFC side, beside BOQ, BBS, QA), pure functions on
  the elements already read. The panel is in `apps/web` using the design system and tokens. No new package. Each
  bump gets a changelog entry, README line and `guide.ts` entry (`docs/workflow.md#versions`).
- **Geometry read.** Projected width per level per direction from element bounding extents (convex hull of the slab
  edge per storey); opening percentage from IfcDoor and IfcWindow areas over IfcCurtainWall / IfcWall face area.
- **Code engine.** IS 875-3 formulas as data tables (k1, k2, k3, k4, Cpe, Cpi, Kd, Ka, Kc, gust factor) in a JSON
  file with the clause number beside each value. The standard is freely published by Public.Resource.Org; **read
  the tables from the standard and check against a hand-worked example before use** (the same caution as the BBS
  design note about scanned figures). Unit tests: hand-worked sheets from a textbook example, kept beside the
  code with the source named.
- **Output.** Excel via the existing writer; ETABS-ready tables through the ETABS path in `docs/design/etabs.md`.
- **Gust alert.** A single optional fetch to a public forecast API, sending only latitude and longitude, on a
  button, off by default (rule: nothing leaves the device unasked). Joins tasks through the 4D timeline already
  planned in the roadmap.
- **Agent helper.** `wind_inputs()`, `wind_table(direction)`, `wind_map()` in the MCP helper (`docs/agent/protocol.md`).
- **Decisions to record** in `docs/decisions.md`: code editions supported first; "inputs and check, not design".
- **First version (smallest useful).** Model-read table (width, height, ratios, openings) per direction, the IS 875-3
  level table to Excel, and the facade colour map for a rectangular tower. The rest second.
- **Test.** Two real towers' IFCs kept out of git, plus a synthetic box in the repo tests. Measure: table against the
  engineer's own sheet (difference per level), time to fill by hand vs by Gust Guard. Report only what was measured.
- **Rules.** On device, no GPL code, no key needed, short labels, no helper text.

### Future update

- **Pedestrian level wind.** Simple massing-based comfort flags around podium edges (clearly marked screening).
- **Wind tunnel handoff.** Export a clean massing and neighbours as STL / IFC for a wind-tunnel lab, and bring their
  pressure coefficients back onto the facade map.
- **Cladding fixings.** Pressure per panel to fixing count and spacing, with Skin Cutter and Hanger Hero.
- **Cyclone and coastal packs.** Region switches (and other codes) beyond India, each from a published standard.
- **Seasonal pour and lift calendar.** Plan lifts and facade work on the lowest-wind weeks, with Sky Slot and Crane Canvas.
- **Honest scoreboard.** After five projects, publish the measured difference to the engineers' sheets.
