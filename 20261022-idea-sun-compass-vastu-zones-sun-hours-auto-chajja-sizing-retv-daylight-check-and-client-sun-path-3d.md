# 20261022 Idea: Sun Compass — Vastu zones, sun hours per window, auto-sized chajjas / fins, RETV and daylight check on the model, and a client sun-path 3D link

## Entry 1

```
date-time:   2026-10-22 (scheduled run)
idea name:   Sun Compass — "Set true north once. In under a minute every room gets its Vastu zone and
             score, every window gets its direct-sun hours and heat gain per season, every chajja and fin is
             sized to cut summer sun while letting winter sun in (and drawn in Revit), the flat gets its
             RETV and daylight check against Eco-Niwas Samhita / ECBC and the bye-law, and the client gets a
             phone link where they drag the sun across the day and see the light move through their flat."
             One Python engine (shared by C2B and the AnonGee pyRevit tab) with four front ends:
               (1) pyRevit button (AnonGee tab, Essential panel): reads rooms, doors, windows, walls, project
                   location and true north; writes Vastu-zone colour fills, window sun-hour tags, auto-sized
                   chajja / fin / louvre families and the compliance sheet,
               (2) C2B: on a client architectural DXF with no Revit model yet, rooms and openings come from
                   the plan layers, so a Vastu + sun report can go out on the day the plan arrives,
               (3) cad2bim web viewer: the client sun-path link. A time-of-day and date slider, real shadows,
                   sun patches on the floor, Vastu 16-zone overlay, "which room gets morning sun",
               (4) PlanWin AI Pro: shading loads. Chajja projections, fins and parapets it adds become
                   cantilever members with self-weight, so the structural engineer sees them before the
                   drawings go out.
             Outputs: Vastu report per flat (16 zones / 8 directions, room-by-room score, main door pada,
             kitchen / pooja / master-bed / toilet placement, brahmasthan clear check), sun-hours and
             solar-gain table per window per season, shading device schedule (depth, width, height above
             lintel, fin depth, angle) with BBS-ready cantilever sizes, RETV calculation sheet, daylight /
             ventilation opening ratio per room against the bye-law, overheated-room list, and a sun-path 3D
             link for the client.
potential:   Very high, every week, on every residential project. In India almost every flat buyer and
             bungalow client asks "is it Vastu compliant?" and "which side gets the sun?" before signing.
             Since Eco-Niwas Samhita 2018 (ECBC-R), RETV limits are being written into state bye-laws and
             GRIHA / IGBC ratings ask for daylight and shading evidence. Today these are three separate
             hand jobs done by three people.
             Time today (one typical floor plate of 4-8 flats):
               - Vastu check: architect or consultant overlays a 16-zone grid by hand in AutoCAD per flat,
                 marks each room, writes remarks: 3-6 hours, redone at every plan change,
               - chajja / fin depth: usually a thumb rule (450 / 600 mm everywhere), sometimes a manual
                 sun-angle calc per facade: 2-4 hours, then modelled by hand on every window: 4-8 hours,
               - RETV sheet: window and wall areas per orientation counted by hand into Excel: 4-6 hours,
               - daylight / ventilation 1/10th-floor-area check per room: 2-3 hours,
               - client sun / shadow video in Lumion or Enscape: half a day to a day.
             With Sun Compass: 20-40 minutes per floor plate including review, and every revision is a
             re-run. Over a project with 5-10 plan revisions this saves 2-4 weeks of architect + BIM time,
             and it wins clients in the first meeting.
             Quality saved: the thumb-rule 600 mm chajja that blocks winter sun on the north-east and still
             lets afternoon sun in on the west; west bedrooms that overheat; toilets that fail the opening
             ratio; RETV failing at approval and forcing a facade redesign; Vastu remarks that contradict
             the current plan because the consultant had an old drawing.
             Trending angle: "Vastu reels" are among the most viewed home content on Indian Instagram, and
             "sun path study in 30 seconds", Ladybug / Forma solar analysis and passive-design posts are
             constantly shared on LinkedIn, with heat waves and net-zero homes in the news every summer.
             The reel: a grey flat plan; a compass spins and snaps to true north; the 16 Vastu zones fan
             out in colour and each room gets a tick or a cross; the sun arcs across, yellow sun patches
             slide across the floors; the west bedroom glows red at 4 pm; one click and chajjas and fins
             grow on every window, the red patch shrinks to nothing, and a counter shows "RETV 17.8 -> 13.9
             W/m², PASS". 30 seconds.
             Market: Autodesk Forma / Insight (paid, cloud, massing-level solar, no Vastu, no shading device
             sizing written back to Revit); Ladybug Tools (free, but Rhino / Grasshopper and expert-level,
             no Indian codes, no Vastu); Revit Sun Path and Solar Study (visual only, no numbers per window);
             Enscape / Lumion / D5 (paid render, pretty but no analysis); Vastu apps and software such as
             desktop Vastu plan checkers (paid, 2D image overlay, no BIM, no sun); BEE's ECO-Niwas online
             RETV calculator (free, but the areas are typed in by hand). Nothing free that goes Revit / DXF
             plan -> Vastu score + per-window sun + auto shading in Revit + RETV + daylight + client 3D link
             in one step.
```

### Description

**The pain (daily life of a BIM modeller in an Indian architecture / BIM-services office).**
- The plan goes to the client and the first question is Vastu. Someone draws a 9 x 9 or 16-zone grid over
  each flat by hand, finds the centre (brahmasthan) of an L-shaped flat by eye and writes "kitchen in SE,
  OK; toilet in NE, not OK". When the architect moves one wall, the overlay is stale and nobody redoes it.
- Every window gets the same 450 or 600 mm chajja because that is what the last project had. On the west
  facade it is too short to stop the 3-5 pm sun; on the north it is needless concrete and steel. Fins on
  east / west facades are drawn only when the client complains of heat.
- The Eco-Niwas Samhita RETV sheet needs opaque wall and glazing areas per orientation, U-values, SHGC and
  shading factors. It is filled by hand from the drawings, late, and does not match the model.
- The bye-law asks for openings of at least a set fraction of the floor area (commonly 1/10 for habitable
  rooms) and ventilation for toilets. This is checked room by room with a calculator.
- The client wants to "see the sun". The team renders a sun study video in Lumion for half a day; when the
  window changes, the video is old.

**The idea, in seven parts.**
1. **Site and north once.** Read latitude, longitude, time zone and true north from the Revit project
   location (or a pin on a map / a city list in C2B). Sun position from NOAA / SPA equations, no internet.
   A "check north" step overlays the site plan on satellite tiles so the wrong north (the classic error)
   is caught before any result.
2. **Vastu engine.** Per flat or per bungalow: find the plot or flat polygon (from the Area Statement Pro
   unit boundary), compute its centroid and the brahmasthan zone, split into 8 directions / 16 zones by
   true north, place every room by its area share in each zone, and score with a rule pack (YAML): kitchen
   SE preferred, master bed SW, pooja NE, toilets not in NE / centre, main door pada out of 32, staircase
   and overhead tank zones, slopes and open-space notes. The pack carries its source and is editable, so a
   consultant's own rules or a softer "preference" mode is one file. The report shows remarks and the
   simplest fix ("swap toilet and utility: +2 zones").
3. **Sun hours and heat per window.** For every window and glazed door, ray-trace the sun across a design
   year (hourly, or 21st of each month) against the model: own wall reveals, chajjas, balconies above,
   neighbouring towers and site context. Result per window: direct-sun hours by season, peak and annual
   solar gain (kWh), and a floor "sun patch" map per room.
4. **Auto shading sizer (the unique part).** For each window: target "no direct sun on the glass between
   1 March and 30 September from 10 am to 5 pm, full sun in December mornings" (editable). Solve the
   horizontal chajja depth from the vertical shadow angle, and vertical fin depth / spacing from the
   horizontal shadow angle, per facade orientation. Offer three styles: chajja only, chajja + fins, louvre
   box. Clamp to practical limits (max 900 mm cantilever, set-back and projection limits from the bye-law,
   FSI-free projection limits from the Area Statement Pro rule pack). Write the result into Revit as
   chajja / fin families on each window host, sized by parameters; group identical windows into types.
5. **RETV and daylight.** From the same model: wall and glazing areas per orientation, U-values and SHGC
   from material / glazing types, external shading factor from the device just sized. Compute RETV per
   Eco-Niwas Samhita and show the margin. Per room: opening area / floor area against the bye-law, a simple
   daylight factor estimate (window-to-floor ratio + sky view), and a "dark room" flag.
6. **Client sun-path link.** Export IFC + sun data to the cad2bim viewer: drag the time and date, see real
   shadows and floor sun patches, toggle the Vastu overlay, tap a room for its card ("morning sun 7-10 am,
   no direct sun after 2 pm, Vastu: SW, OK"). Shareable on WhatsApp, works on a phone.
7. **Downstream.** Chajja and fin concrete, shuttering and steel go to the BOQ (structura / Bill Painter)
   and to Wall Wizard's lintel schedule (chajja cast with the lintel). PlanWin AI Pro gets the cantilever
   loads. Drawing Autopilot puts the chajja schedule and a typical section on the facade sheets.

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)** gets a high-visibility button built on the existing WPF UI templates and
  `bridge/`; Revit already holds rooms, openings, walls and the project location.
- **C2B** already reads client DXF plans; rooms and openings from architectural layers let the Vastu and
  sun report go out before a Revit model exists.
- **cad2bim web** already renders IFC with three.js in the browser; a sun light, shadow maps and a time
  slider are a viewer plugin, and the "files stay on your device" model suits client flats.
- **PlanWin AI Pro** and **etabs-mcp** pick up chajja / fin cantilevers as members with loads.
- **structura** BOQ and the Bill Painter / Wall Wizard ideas consume the chajja quantities; Area Statement
  Pro supplies flat boundaries and the projection rules; Model Reel records the 30-second sun-path clip.

**Wow moments.**
- The compass snaps to north and the 16-zone Vastu wheel fans over every flat with a score per room.
- The sun sweeps from 6 am to 7 pm and the sun patches slide across the floor plans; the west bedroom glows
  red at 4 pm.
- One click: chajjas and fins grow on every window in Revit at the right depth for that face, the red patch
  vanishes, the RETV counter flips to PASS, and the concrete saved on the north facade shows as rupees.
- The client opens the link on a phone in the meeting and drags the sun through their own flat.

### Implementation

**Phase 1 — Sun + Vastu engine and report (3 weeks).**
- Python package `c2b.sun` (shared by pyRevit via the bridge): `sun_position(lat, lon, tz, dt)` (NREL SPA
  or pvlib-style equations, no network), `vastu(flat_polygon, rooms, north, rules) -> VastuReport`,
  `opening_ratio(rooms, openings, rules) -> [Finding]`.
- Geometry with Shapely: flat centroid, 16-zone sectors clipped to the flat, room area share per zone,
  32-pada main door position along the entrance wall.
- Rule packs in YAML: `vastu_default.yaml`, `bye_law_openings_<city>.yaml`, each with version + source note.
- Report writer: Excel (openpyxl) + PDF per flat, with a coloured zone plan (matplotlib / ezdxf).
- CLI: `c2b sun plan.dxf --lat 18.52 --lon 73.86 --north 12.5 --rules profiles/vastu_default.yaml`.
- Tests: square, L-shaped and rotated flats with hand-checked zones; sun angles checked against NOAA tables.

**Phase 2 — Ray tracing, shading sizer, RETV (3 weeks).**
- Shadow casting: triangulated model (Revit geometry or IFC via IfcOpenShell) into a BVH (trimesh / embree);
  per-window sample grid on the glass; hourly sun vectors for the design days; cached per model revision.
- Shading sizer: vertical / horizontal shadow angle per orientation; solve depth for the target window;
  clamp by bye-law projection rules; group by size / orientation to keep few chajja types.
- RETV per Eco-Niwas Samhita: areas per orientation, U, SHGC, shading factor tables; per-flat and building.
- Writes a JSON "shading proposal" the pyRevit button applies.

**Phase 3 — Revit write-back, web viewer, downstream (3 weeks).**
- pyRevit button: WPF form (north check, rule packs, target dates / hours, style), places chajja / fin
  families hosted on the window's wall with `Depth`, `Width`, `Offset_Above_Lintel`, `Fin_Depth`
  parameters; Vastu filled regions per room in a dedicated view; window tags showing sun hours; RETV and
  Vastu schedules on a sheet. All changes in one transaction group, with an undo.
- cad2bim viewer plugin: three.js directional light from sun vectors, shadow maps, floor sun-patch decals,
  time / date slider, Vastu overlay layer, room cards; a Model Reel preset for the 30-second clip.
- Export chajja / fin members to PlanWin AI Pro (cantilever with self-weight + live load) and quantities to
  the structura BOQ.

**Tech notes / risks.**
- Vastu is a client preference, not a code. The tool scores against a stated, editable rule pack and never
  calls a design "wrong"; remarks are worded as suggestions, with the pack name on every report.
- Wrong true north ruins every result: the north check against satellite tiles and a site-plan arrow is
  mandatory, and the report prints the north angle used.
- RETV and opening-ratio rules differ by state and change by notification; packs carry a version and the
  sheet says which pack was used. Output is a design aid for the architect to approve.
- Ray tracing large towers with context: limit to the design days, cache per revision, run off the UI
  thread in the bridge and stream progress.
- Revit chajja families vary by office; map by type name in a JSON config and ship a default family set.

### Future update
- Neighbour and future-tower shadow: "what if the plot next door builds G+20" overlay for sales and for
  the right-to-light check on our own lower floors.
- Solar rooftop: PV panel layout on the terrace avoiding shadows from the lift room, tank and parapet, with
  kWp, yearly kWh and net-metering payback; feeds Live Wire's SLD.
- Cross-ventilation wind arrows per flat from wind rose data (CFD-lite), with the Vastu report.
- Glare and heat for commercial glass facades: SHGC choice per facade and ECBC compliance for offices.
- Thermal comfort / cooling load estimate per room (CLTD-lite) to size split ACs, handed to the HVAC team.
- Sales mode: a floor-wise "sunniest flats" heatmap for the builder's pricing and brochure, and a QR on the
  site sample flat (Site QR Pass) that opens the live sun-path view.
- AI mode: "make all bedrooms Vastu OK without losing carpet area": the AI box proposes room swaps and
  re-runs the score.
