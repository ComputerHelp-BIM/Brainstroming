# 20261024 Idea: Cool Load — room-by-room heat load straight from the Revit/IFC model, auto TR sizing, auto VRF / split / ducted indoor-unit placement, refnet and copper-pipe routing with manufacturer length-limit checks, condensate drain fall, outdoor-unit terrace layout with slab-load check, and a priced HVAC BOQ with a 3D "thermal heat-map" wow view

## Entry 1

```
date-time:   2026-10-24 (scheduled run)
idea name:   Cool Load — "Click once and every room in the model turns into a coloured heat-map
             (blue = 80 sqft/TR, red = 150 sqft/TR west-facing glass box), the AC tonnage appears
             on each room tag, indoor units drop onto the ceiling, copper pipes and refnets route
             themselves to the outdoor units on the terrace, and a priced HVAC BOQ comes out —
             before the MEP consultant has even opened the drawings."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "MEP" stack in Essential.panel next to
                   StrucBIM): Heat Load, Place Units, Route Refrigerant, Drain Check, HVAC BOQ,
               (2) cad2bim web viewer: 3D thermal heat-map with a sun/time slider, click a room to
                   see the load breakdown (glass / wall / roof / people / lights / equipment / fresh
                   air), share link for the client and the HVAC vendor,
               (3) etabs-mcp bridge: slab capacity check for the outdoor-unit (ODU) bank on the
                   terrace and the AHU plinth; beam depths for duct/pipe crossing,
               (4) earlier engines reused: Sun Compass (orientation, shading, glass SHGC), Ceiling
                   Composer (ceiling void height, diffuser grid), Hanger Hero (pipe and duct
                   supports), Sleeve Punch (wall/beam sleeves for copper and drain),
                   Live Wire (AC electrical load to DB and SLD), Drain Line (condensate tie-in).
             Outputs: room heat-load sheet (ISHRAE/ASHRAE CLTD method, Excel), TR schedule per
             room/flat/floor, indoor-unit schedule, VRF system grouping (ODU HP, connection ratio),
             refrigerant pipe-size schedule + refnet list, length/height-limit compliance report,
             condensate drain profile with slope check, ODU terrace layout DXF with service
             clearances, HVAC BOQ (units, copper by size, insulation, refnets, drain PVC, cable,
             supports, refrigerant top-up kg) and a client one-pager.
potential:   Very high, and almost every project needs it. India and the Gulf are the fastest-growing
             AC markets in the world; every residential tower, office and hospital has VRF or split
             AC, yet the BIM team usually waits 2-4 weeks for an HVAC consultant's heat load and
             layout, and then re-models it by hand. Heat-wave summers (2024-2026 records) and the new
             ECBC / Eco-Niwas Samhita energy rules mean clients now ask "how many TR, what will it cost
             to run?" at concept stage.
             Time today (one G+12 residential tower, 4 flats/floor, ~260 rooms, or a 60,000 sqft
             office floor plate):
               - heat load in Excel / HAP room by room by the consultant: 3-6 days, re-done when the
                 architect changes a window or a glass type,
               - indoor-unit placement + copper routing in Revit by the BIM modeler from the
                 consultant's PDF: 4-8 days per typical floor set,
               - VRF pipe sizing in the manufacturer tool (Daikin VRV Xpress, Mitsubishi Design Tool,
                 LG LATS) — a separate, non-BIM model drawn again, 1-2 days per system,
               - manual check of max pipe length / height difference / first-branch length / drain
                 fall: mostly NOT done, found on site as pump-down failures and leaking ceilings,
               - HVAC BOQ counted by hand: 1-2 days, typical copper over-ordering 15-25 %.
             With Cool Load: first heat load + unit placement + routing + BOQ in 30-60 minutes from the
             existing architectural model; re-run on every revision in 2-5 minutes.
             Expected saving: 10-18 working days per project for the BIM team, 15-25 % less copper and
             insulation through routed (not estimated) lengths, and zero "pipe too long for this ODU"
             surprises on site.
             Wow factor: the thermal heat-map. Drag the sun slider from 9 am to 4 pm and watch the west
             rooms go from blue to red and the TR tags tick up; press "Place" and indoor units snap
             into ceilings, copper lines draw themselves up the shaft to the terrace. A 30-second clip
             is a perfect LinkedIn / Instagram reel (#MEP #BIM #HVAC #VRF #EnergyEfficiency
             #ClimateTech), and it is the slide that wins the client meeting: "Your west glass costs you
             3.5 TR and ₹42,000 per year — switch to SHGC 0.25 glass and save 1.8 TR."
             Market check: Carrier HAP, Trane TRACE 3D Plus, Elite CHVAC and IES-VE are paid, are not
             inside Revit, and do not place equipment. Revit's own Systems Analysis needs Insight /
             EnergyPlus setup and still does not size VRF refrigerant piping or place units. Manufacturer
             tools (VRV Xpress, LATS) are free but brand-locked, are their own 2D sketch, and do not read or
             write the BIM model. MagiCAD / Trimble / Stabiplan have VRF modules but are expensive per-seat
             licences and Europe-centric. A free, BIM-native, brand-neutral, ISHRAE-climate-aware heat load
             → placement → refrigerant routing → BOQ chain does not exist.
```

## Description

**The problem.** The architectural BIM model already contains everything a heat load needs: room areas and
volumes, wall/roof/glass areas per orientation, window types, ceiling heights, occupancy (from room names), and
the site location. Yet the HVAC heat load is still typed into a separate Excel or HAP file, the indoor-unit layout
is drawn again from a PDF, the VRF piping is sized in a third (manufacturer) tool, and the BOQ is counted by hand.
Four disconnected copies of the same building, and every architectural change breaks all of them.

**What Cool Load does.**

1. **Heat load from the model (room by room).**
   - Reads Rooms/Spaces (Revit) or IfcSpace (IFC via cad2bim): area, volume, height, name, level.
   - Finds bounding walls, windows, curtain walls, roofs and exposed floors per room with orientation (N/NE/E…),
     using Revit room boundary + wall normals (or IFC space boundaries).
   - Applies the CLTD/CLF method (ISHRAE handbook / ASHRAE Fundamentals) with city design data from an
     ISHRAE weather table (Mumbai, Pune, Delhi, Chennai, Ahmedabad, Dubai, Riyadh… editable):
     wall & roof conduction (U-value from the Revit type's thermal properties or a default library), glass
     solar gain (SHGC × area × solar factor by orientation and hour, with shading factor from chajjas/fins found
     by Sun Compass), people (from room type rule: bedroom 2, living 4, office 1 per 10 m²), lights and
     equipment (W/m² per room type), fresh air (ASHRAE 62.1 / NBC Part 8) and a safety factor.
   - Outputs sensible / latent / total load in W and TR, plus sqft/TR, per room, flat, floor and building.
   - Peak hour is found per room and per system (diversity), so ODU sizing is not over-sized.

2. **Auto unit selection and placement.**
   - Rule picks unit type per room: high-wall split (bedroom), cassette (living / office), ducted (lobby, shop),
     with standard capacities (0.8 / 1.0 / 1.5 / 2.0 / 2.5 / 3.0 TR or VRF kW model ladders, editable catalogue).
   - Places native Revit mechanical equipment families: high-wall on the wall opposite the bed or longest
     exterior-free wall at 2.4 m; cassettes centred on the ceiling grid (from Ceiling Composer) avoiding lights,
     beams and sprinklers; ducted units in the false-ceiling void with a clearance check to the beam bottom.
   - Places supply/return grilles for ducted units and the wired remote / thermostat at 1.2 m by the door.

3. **VRF system grouping and refrigerant routing.**
   - Groups indoor units into systems per flat / per floor / per zone (rule) and picks the ODU HP so the
     connection ratio is within 50-130 % (editable per brand).
   - Routes liquid + gas pipe pairs from each indoor unit along the ceiling void to the nearest shaft
     (or façade ledge) and up/down to the ODU location, avoiding beams (use beam depth from etabs-mcp/Revit,
     drop or sleeve), columns and other MEP (A* on a 3D grid of the ceiling void, same router as Drain Line).
   - Sizes each segment by downstream capacity (brand-neutral table, with Daikin / Mitsubishi / LG / Blue Star /
     Voltas profiles), inserts refnet Y-joints / headers, and checks the manufacturer limits:
     max actual pipe length, equivalent length, first-branch-to-farthest-unit length, ODU-to-IDU height
     difference, IDU-to-IDU height difference. Violations are flagged red with a "move ODU / split system"
     suggestion.
   - Computes additional refrigerant charge (kg) from the liquid line lengths per size.

4. **Condensate drain check.**
   - Routes the drain from each IDU to the nearest floor trap / toilet / balcony drain (Drain Line engine) with
     a minimum 1:100 fall; flags any run where the ceiling void cannot give the fall (classic leaking-ceiling
     cause) and proposes a drain pump or a re-route.

5. **ODU terrace / ledge layout.**
   - Packs ODUs on the terrace or service ledge with service clearances (front / back / side / top discharge),
     air short-circuit spacing, maintenance path; checks weight on slab (kN/m²) and plinth against slab capacity
     from etabs-mcp; outputs a DXF + 3D layout and a plinth schedule for the structural team.

6. **BOQ, cost and running cost.**
   - Priced BOQ: units, copper by size (m, kg), insulation by size, refnets, drain PVC, communication cable,
     power cable to Live Wire DBs, hangers (Hanger Hero), sleeves (Sleeve Punch), refrigerant top-up kg.
   - Running cost estimate per flat (kWh/year × tariff) and a "what-if" panel: change glass SHGC, add chajja,
     change wall insulation → see TR, capex and ₹/year change live. This is the client-meeting killer feature.

**Who uses it every day.** BIM modeler (one click layout and routing instead of days of drawing), MEP
coordinator (limit checks, clashes), architect (what-if glass and shading), QS (BOQ), sales / client team
(heat-map reel and running-cost slide), site team (drain fall and ODU plinth drawings).

## Implementation

**Data model (shared JSON, same pattern as the other engines).**
```
room      { id, name, type, level, area_m2, volume_m3, height_m,
            envelope[{kind: WALL|GLASS|ROOF|FLOOR, orient, area_m2, U, SHGC, shade_factor}],
            people, light_w, equip_w, fresh_air_lps,
            load{sensible_w, latent_w, total_w, tr, sqft_per_tr, peak_hour} }
unit      { id, room_id, kind: HIGHWALL|CASSETTE|DUCTED|ODU, model, capacity_kw, location, system_id }
system    { id, brand_profile, odu_ids[], idu_ids[], connection_ratio,
            pipes[{seg_id, from, to, liquid_mm, gas_mm, length_m, rise_m}], refnets[],
            limits_report[{rule, value, limit, ok}], extra_charge_kg }
drain     { idu_id, path[], length_m, min_fall, ok, fix }
catalogue { unit_types[], brand_profiles[{name, pipe_size_table, limits, refnet_codes}],
            climate[{city, db_c, wb_c, daily_range, latitude}], materials[{name, U}], rates[] }
rules     { room_type_map, people_per_room, w_per_m2, safety_factor: 1.1, sqft_per_tr_warn: [80, 150],
            ceiling_void_min_mm, drain_fall: 0.01, connection_ratio: [0.5, 1.3] }
```

**Heat-load core (pure Python, unit-tested, no Revit dependency).**
- `coolload/core/cltd.py`: CLTD/CLF/SCL tables (ISHRAE/ASHRAE), corrections for latitude, month, colour,
  indoor/outdoor design temperature; hourly loop 8:00-18:00 for peak finding.
- `coolload/core/envelope.py`: from boundary segments + wall normals → orientation bins; window/glass areas from
  hosted openings and curtain panels; shade factor from overhang depth/height ratio (Sun Compass shared code).
- `coolload/core/psychro.py`: fresh-air sensible/latent load from DB/WB.
- Validated against 3-5 hand-calculated rooms and a HAP output so engineers trust the numbers.

**Revit side (pyRevit-addons).**
- New `MEP.stack` in `Essential.panel` (AnonGee tab) with buttons *Heat Load*, *Place Units*, *Route Refrigerant*,
  *Drain Check*, *HVAC BOQ*; engine in `lib/py3/coolload/`; WPF UI with the AnonGee theme XAML.
- Reads Rooms/Spaces with `SpatialElementGeometryCalculator` for boundary faces → walls/windows with normals;
  thermal properties from `ThermalAsset` / type parameters with a fallback library.
- Writes results back to Space/Room shared parameters (CL_Total_W, CL_TR, CL_SqftPerTR) so normal Revit schedules
  and colour-fill legends work; a "Thermal Heat-Map" 3D view with filters by CL_SqftPerTR.
- Places mechanical equipment families and refrigerant pipes (Pipe type "Copper – ACR", system "Refrigerant
  Liquid/Gas") with fittings; refnets as pipe accessories.

**Web side (cad2bim).**
- "Cool Load" panel in `apps/web` using `@cad2bim/engine` for IFC spaces and geometry; heat-map shader on
  IfcSpace by sqft/TR; sun/time slider re-runs the hourly glass gain in the browser (core ported to TS or run via
  the bridge).
- MCP tools in `packages/mcp`: `coolload.heatload`, `coolload.place`, `coolload.route`, `coolload.boq`, so an AI
  agent can answer "how many TR is flat 1203 and what if we use double-glazing?".

**Structural link (etabs-mcp).** Slab and beam data for ODU plinth load checks and beam-depth aware routing.

**Outputs.** openpyxl (heat-load sheet, schedules, BOQ, limits report), ezdxf (ODU layout, refrigerant schematic
riser), HTML-to-PDF (client one-pager with heat-map image and running cost), IFC export of placed HVAC
(IfcUnitaryEquipment, IfcPipeSegment, Pset_CoolLoad).

**MVP (2-3 weeks).**
1. Room heat load (CLTD) from Revit Rooms with orientation and glass, 6 Indian cities + Dubai.
2. Write TR to rooms + heat-map colour-fill 3D/plan view.
3. Excel heat-load sheet + TR schedule.
4. Auto-place high-wall / cassette units by rule.

**V1 (next 4-6 weeks).** VRF grouping, refrigerant routing + sizing with 2 brand profiles, length/height limit
report, drain fall check, ODU terrace layout + slab check, priced BOQ, cad2bim heat-map with sun slider,
what-if glass/shading panel, MP4 reel via Model Reel.

## Future update

- **AI room-type reader:** when room names are messy ("BR-2", "Mstr Bed", "Pooja"), an LLM maps them to load
  profiles; also reads the architect's furniture to estimate people and equipment.
- **Chiller / AHU path:** for offices and hospitals, extend from VRF to chilled-water: AHU/FCU selection, CHW pipe
  sizing, duct sizing by equal-friction with auto duct routing in the ceiling void (Ceiling Composer link).
- **Energy + carbon:** annual kWh with a simple bin method (later EnergyPlus via OpenStudio), operational carbon
  per flat, ECBC / Eco-Niwas Samhita RETV and EPI compliance sheet (Sun Compass link) — a green-building
  (IGBC / GRIHA / LEED) points estimate.
- **Brand comparison sheet:** same layout priced by Daikin, Mitsubishi, LG, Blue Star, Voltas profiles side by
  side — the vendor negotiation sheet clients love.
- **Site QR for each IDU:** sticker on each unit / refnet opens its pipe sizes, charge, test pressure and
  commissioning checklist (Site QR Pass); pressure-test and vacuum log captured on phone.
- **Retro-fit mode:** from a point cloud / photo of an existing flat (Scan Check / Plan Tracer) produce heat load
  and AC proposal for interior-fit-out and AC dealer teams — a whole new customer group.
- **Comfort replay:** hourly room-temperature replay in the viewer for "AC off vs on" and "glass A vs glass B" —
  a strong Instagram/LinkedIn visual.
- **Improvements to existing tools:** (a) add a "Room / Space health" check to Warning Doctor (unbounded,
  redundant, not enclosed rooms) because every MEP and area engine depends on clean rooms; (b) let Area
  Statement Pro and Cool Load share one room-classification table so room types are set once for both;
  (c) add a shared "MEP router" library (used by Drain Line, Live Wire and Cool Load) to the pyRevit
  `lib/py3` folder instead of each tool having its own path-finder.
