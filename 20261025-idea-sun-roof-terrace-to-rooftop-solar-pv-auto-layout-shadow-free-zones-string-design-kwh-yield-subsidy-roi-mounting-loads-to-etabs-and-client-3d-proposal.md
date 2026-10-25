# 20261025 Idea: Sun Roof — turn the terrace in the Revit / IFC / DXF model into a ready rooftop solar PV design in one click: shadow-free zones from the real tanks, mumty, parapets and neighbours, auto panel layout with walkways, string and inverter sizing, kWh yield month by month, subsidy and payback, mounting dead load and wind uplift pushed to ETABS, a PV BOQ, and a 3D client proposal where the sun sweeps the roof and the "₹ saved" counter climbs

## Entry 1

```
date-time:   2026-10-25 (scheduled run)
idea name:   Sun Roof — "Open the terrace. Press Solar. The tool runs the sun over the roof for a whole
             year, finds where the water tanks, the staircase mumty, the lift machine room, the parapet and
             the next building throw shadows, and paints the roof green / amber / red. Panels snap into the
             green zones with cleaning walkways, rows are spaced so they do not shade each other, strings
             are matched to the inverter, and the screen shows: 18.4 kWp, 26,900 kWh a year, ₹1.9 lakh
             saved a year, payback ~4 years after subsidy, ~19 t CO2 saved a year. The structure dead load and
             wind uplift go to ETABS as area loads; the BOQ, layout sheet and a client 3D proposal come out
             with it."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Solar" stack in the Advance panel): pick the roof
                   (floor / roof element or room on the terrace), detect obstacles from the model,
                   run shading, place panels as a Revit family, number strings, make the layout sheet,
               (2) C2B / DXF path: the terrace outline, OTS, tanks and mumty read from the client's
                   architectural DXF (closed polylines + heights from text / a small table) — so it works
                   even before a Revit model exists, and for quick proposals,
               (3) Structura-style static web viewer ("Sun" mode): the roof in 3D with a sun slider
                   (date + hour), live shadows, shade heatmap, panels, yield and ROI cards — the client
                   proposal, shareable as a link or opened from a folder,
               (4) etabs-mcp / PlanWin bridge: panel + structure dead load (kN/m²) and IS 875-3 wind
                   uplift / downward pressure per zone written as load patterns on the roof slab or
                   as point loads at the leg / ballast positions, so the structural check is done on the
                   real layout, not a guessed 1.5 kN/m².
             Outputs: shade heatmap (annual % unshaded, 9 am-3 pm winter design window), panel layout in
             Revit (family instances with string / MPPT ID parameters) and DXF, layout drawing sheet with
             dimensions and walkways, DC string schedule + simple SLD (strings -> MPPT -> inverter ->
             ACDB -> net meter), monthly and yearly kWh yield, bill saving, subsidy, payback, 25-year cash
             flow and CO2, PV BOQ (modules, inverter, GI / aluminium structure kg, DC / AC cable m, MC4
             pairs, earthing pits, lightning arrestor, ACDB / DCDB), structural load sheet for ETABS /
             PlanWin, and a client proposal PDF + 3D web link.
potential:   Very high, and trending right now. In India the rooftop solar subsidy scheme (PM Surya Ghar:
             Muft Bijli Yojana, target 1 crore homes) plus net metering has turned "put solar on the roof"
             into a question almost every residential, society, school, hospital and factory client asks.
             Architects and BIM teams are now asked to show the solar area in the design, keep the
             terrace free of conflicts, and give the structural engineer a real panel load — usually with
             no tool for it inside Revit.
             Time today (one G+4 to G+12 building, 150-600 m² terrace):
               - solar vendor visits, measures the roof by tape, sketches a layout by hand or in SketchUp:
                 2-5 days (and the layout ignores the MEP / water tank changes made later),
               - BIM team redraws the vendor layout in Revit for coordination: 1-2 days,
               - shading guessed ("keep 1 m away from the tank"), rows spaced by thumb rule,
               - structural engineer assumes a load, or re-checks after the vendor picks an elevated
                 structure: 0.5-1 day, often after the slab is cast,
               - yield, subsidy and payback calculated by the vendor in their own sheet, which the client
                 cannot compare across vendors.
             With Sun Roof: shade map + layout + strings + yield + ROI + BOQ + loads in about 15-30 minutes
             from the model; re-run in 2 minutes when the tank or the mumty moves.
             Expected saving: 3-7 days per project, a design-stage solar capacity the client can take to
             tender (vendors quote on the same layout, so quotes are comparable), and no late slab re-checks.
             Wow effect: the terrace turns into a heatmap as the sun sweeps across it from morning to
             evening and from June to December — the long winter shadow of the water tank crawls over the
             roof; then the panels drop in row by row only where it is green, the string wires light up in
             colours, and the yield counter and "₹ saved" counter climb while the payback bar fills. The
             before / after reel ("this roof was earning ₹0, now it earns ₹1.9 lakh a year") is exactly
             the kind of solar + BIM + sustainability content that is trending on LinkedIn and Instagram.
             Market: Aurora Solar, HelioScope, OpenSolar (free for installers, but tied to its own sales
             platform and satellite / map imagery, not to the BIM model), PVsyst (paid, detailed yield,
             no 3D roof modelling from BIM), PVcase Roof Mount (paid AutoCAD plug-in), Skelion (paid
             SketchUp plug-in), Revit's own solar analysis (Insight: irradiance on surfaces only, no
             panel layout, strings, BOQ or ROI), PVWatts / PVGIS (free yield calculators, no geometry).
             To our knowledge nothing free goes from the BIM model (with its real tanks, mumty and
             parapets) -> shading -> panel layout -> strings -> yield -> subsidy / ROI -> BOQ ->
             structural loads into ETABS -> 3D client proposal in one tool, inside Revit.
```

### Description

**The pain (daily life of a BIM modeller and the design team).**
- The client says "we want solar on the terrace" at the last design meeting. Nobody knows how many kW fit,
  because the terrace has an OTS, two water tanks, a staircase mumty, a lift machine room, a parapet, rain
  water outlets and an HVAC ODU zone (Cool Load, Rain Run — earlier ideas) all competing for the same roof.
- The solar vendor comes after construction, measures the roof by tape and sketches a layout that ignores
  the drawings. Panels end up in the winter shadow of the tank, over the rain outlets, or blocking the
  cleaning path.
- The structural engineer is asked "can the slab take an elevated structure?" with no panel weight, no leg
  positions and no wind uplift numbers.
- Every vendor gives a different kWh and payback number. The client cannot compare them.

**What Sun Roof does.** It treats the roof as one record — *usable area, obstacles with heights, shade per
point through the year, panel positions, strings, structure, loads, yield and money* — kept in the model.
Every output (layout sheet, BOQ, SLD, loads, proposal) is generated from that record, so a change to the tank
or the mumty re-flows to all of them.

**Daily use cases.**
1. *Residential tower / society*: common-area solar sized at design stage; the architect shows the panel
   zone on the terrace plan; the subsidy and payback go in the sales brochure.
2. *Individual bungalow / G+2 house*: 3-10 kWp layout from the DXF in minutes, with a WhatsApp-friendly
   proposal PDF for the owner — great for small offices that design hundreds of houses a year.
3. *School / hospital / factory / warehouse*: large flat or metal-sheet roofs, elevated or ballasted
   structure; the load sheet goes straight to ETABS / PlanWin.
4. *Coordination*: panel zone is a Revit element, so clash checks with MEP (ODUs, exhaust fans, vents,
   outlets) and the walkway / fire access path happen in the normal BIM workflow.
5. *Retrofit*: existing building traced from a scan / PDF (Plan Tracer, Scan Check — earlier ideas), roof
   obstacles added by height, solar sized for the owner before calling vendors.
6. *Carport / stilt parking canopy*: the same engine on a canopy surface over parking bays (Park Pilot —
   earlier idea).

### Implementation

**1. Read the roof and the obstacles.**
- Revit: pick a roof / floor / terrace room; usable boundary = slab edge minus parapet setback (office rule,
  e.g. 600 mm), minus OTS and shafts, minus keep-out zones (rain outlets, ODU zone, access door swing,
  fire / maintenance access paths).
- Obstacles with height: all model elements above the roof level within the boundary and a buffer around it
  (tanks, mumty, LMR, parapet, chimneys, ODUs, lightning masts) — bounding boxes or meshes.
- Neighbours: optional massing of adjacent buildings (simple boxes with height, or OpenStreetMap buildings
  with storey count) and big trees.
- DXF path (C2B): closed polylines on named layers (roof outline, obstacle outlines) + heights from a text
  tag or a small Excel, extruded into the same scene.

**2. Sun and shade.**
- Sun position from site latitude / longitude / time zone (NOAA / SPA algorithm, as in `pvlib`) — the same
  engine as the Sun Compass idea (sun path, sun hours) so it is written once.
- Grid of sample points on the usable roof (e.g. 250 mm); for each hour of a typical year (or a reduced set:
  21st of each month, every 30 minutes), cast a ray towards the sun and test against obstacles
  (Revit `ReferenceIntersector`, or an exported mesh with a BVH ray caster in Python / three.js).
- Two maps: annual unshaded % (weighted by irradiance) and the design-window check (no shade 9 am-3 pm solar
  time on the winter solstice, the usual industry rule). Green >= 95 %, amber 85-95 %, red < 85 % (office
  thresholds).

**3. Auto layout.**
- Module library (Excel / JSON: size, Wp, Voc, Vmp, Isc, temperature coefficients, weight), e.g. a
  2278 x 1134 mm, 575 Wp TOPCon module; structure types: low-tilt fixed, elevated GI structure (legs on
  pedestals, clears the terrace for use), ballasted, metal-sheet clamp.
- Tilt and azimuth: default tilt ~ site latitude (or the low tilt the vendor prefers), facing true south in
  the northern hemisphere; azimuth snapped to the roof edge if within a few degrees, to keep rows parallel.
- Row pitch so a row does not shade the next row in the design window:
  `d = h x cos(sun azimuth - array azimuth) / tan(sun altitude)` at the worst hour, plus the row depth.
- Fill algorithm: place rows in green zones, try portrait / landscape and a few offsets, keep walkways every
  N rows and around the edge (for cleaning and fire / maintenance access), score by total kWp x shade
  factor; the user can drag or delete panels and everything re-computes.

**4. Electrical (simple, design-stage).**
- Strings sized from the inverter MPPT window: modules per string between `ceil(Vmppt_min / Vmp_hot)` and
  `floor(Vdc_max / Voc_cold)` using the site's minimum and maximum cell temperature; strings grouped by
  orientation / shade so one shaded row does not drag a whole string down.
- DC / AC ratio check (e.g. 1.1-1.3), inverter count, DCDB / ACDB, cable length from each string to the
  inverter location picked on the roof or in the electrical room (Live Wire — earlier idea — owns the
  building side: ACDB, net meter and the main panel).
- Earthing pits and lightning arrestor position suggested, cable size from current and drop limit.

**5. Yield and money.**
- Irradiance and temperature from free sources (PVGIS / NASA POWER typical-year data for the site), modelled
  with `pvlib`: plane-of-array irradiance, shade factor per row from step 2, temperature loss, inverter and
  cable losses, soiling, degradation per year.
- Output: kWh per month and per year, specific yield (kWh/kWp), performance ratio.
- Money: tariff slabs from the office / DISCOM table, net metering credit, current central subsidy rules
  (kept in an editable table, since rates and caps change), state subsidy if any, system cost per kWp from
  the office rate list -> payback, 25-year cash flow, IRR, CO2 saved (grid emission factor table).
- Disclaimer on every sheet: design-stage estimate; final yield and electrical design by the licensed vendor.

**6. Structure loads (etabs-mcp / PlanWin).**
- Dead load: module + structure weight per m² from the library (typically low tens of kg/m²), ballast if used.
- Wind: IS 875-3 basic wind speed for the site, terrain and height factors; uplift and downward pressure
  coefficients per roof zone (edge / corner / interior) — clearly marked as a simplified design-stage
  estimate for the structural engineer to confirm.
- Written as area load patterns on the roof slab, or as point loads at elevated-structure legs, through the
  etabs-mcp REST API (load patterns + assignments); for PlanWin, as an extra load on the terrace panels in
  the load take-down.

**7. Outputs.**
- Revit: panel family instances (with string ID, MPPT, row, kWp parameters), structure lines / legs, shade
  heatmap as a filled region / AVF display, layout sheet with dimensions and walkways, string schedule, BOQ
  schedule.
- DXF layout (for vendors who work in AutoCAD), Excel BOQ and string schedule, client proposal PDF (cover
  render, shade map, layout, yield chart, payback, subsidy, CO2), JSON for the web viewer.

**Tech stack.** Python core (`pvlib` for sun position and yield, `numpy` for the shade grid, `ezdxf` for DXF,
`openpyxl` for Excel) shared by the pyRevit button and a CLI; Revit side in pyRevit (geometry read, ray
casting with `ReferenceIntersector`, family placement, AVF heatmap); web viewer in the Structura style
(three.js shadow maps for the sun slider, xlsx.js, jsPDF already vendored there); etabs-mcp REST API for the
loads.

**MVP (4 weeks).**
1. Roof + obstacle extraction from Revit (flat RCC terrace) and from DXF (closed polylines + heights).
2. Sun position + shade grid + green / amber / red heatmap (21st of each month, 30-minute steps).
3. Auto layout for one module and one fixed-tilt structure, with row pitch, walkways and manual edit.
4. String sizing for one inverter type, yield with `pvlib` + PVGIS / NASA POWER data, payback with subsidy
   table.
5. BOQ + layout sheet + proposal PDF + static web viewer with the sun slider. Demo on a sample residential
   tower and record the "sun sweeps the roof, the counter climbs" reel.

### Future update

- **Load push to ETABS** as a one-click step (MVP gives the load sheet; v2 writes the load patterns and
  re-runs the slab / beam check through etabs-mcp, showing utilisation before / after solar).
- **Elevated structure designer:** GI / aluminium member sizing, leg positions snapped to beams and columns
  (not the middle of the slab), bolt / anchor schedule and fabrication cut list — shares the Bolt Up idea's
  steel engine.
- **Pitched / metal-sheet roofs and facades:** sloped roof planes, BIPV on facades and balcony railings
  (links to Skin Cutter — facade panelization).
- **Vendor tender pack:** same layout and specs to 3-5 vendors, their quotes imported back into one
  comparison sheet (₹/Wp, warranty, inverter brand, yield guarantee).
- **Battery and EV charger option:** size a battery for backup hours and EV chargers in the parking (Park
  Pilot), with the effect on payback.
- **Live monitoring in the twin:** after installation, the inverter's cloud / Modbus data shows actual vs
  predicted kWh per string on the 3D roof (Handover Hub web twin).
- **Society / developer portfolio view:** run Sun Roof on every project in the office and show the total
  solar capacity and CO2 saved — a sustainability slide for the firm.
- **Green rating support:** export the solar share for IGBC / GRIHA / LEED renewable-energy credits.

### Improvement suggestions for existing tools in the repos

- **pyRevit-addons (AnonGee):** a shared "sun engine" module (sun position, sun vectors, ray-cast shade grid,
  AVF heatmap display) would serve Sun Roof, Sun Compass (chajja sizing, daylight) and Rain Run (wet / dry
  terrace zones) alike; and a "roof obstacle collector" (everything above a level inside a boundary, with
  heights) is reusable for Cool Load ODU placement and Edge Guard fall-hazard edges.
- **etabs-mcp:** add a small high-level endpoint / MCP tool "apply area load to slab(s) at story X by load
  pattern" (and "apply point loads at XY list"), so tools like Sun Roof, Cool Load (ODU weight) and Rain Run
  (screed) can push design-stage loads without driving the generic CSI API calls themselves.
- **PlanWin AI Pro:** let the AI assistant accept "add solar on the terrace, 18 kWp elevated" and translate it
  into a terrace superimposed load (with the source noted), so the load take-down includes it from day one.
- **Structura:** a "sun slider" (date + hour -> directional light + shadow map in three.js) and a heatmap
  overlay on any surface would make its viewer the base for the solar proposal and for other environmental
  views.
- **C2B:** a "roof / terrace profile" in the classifier (roof outline, OTS, tank and mumty outlines, height
  tags) so the architectural terrace plan can be read the same way the structural plan is read today.
- **cad2bim:** a "Solar" tab in the web app that reads the terrace from the uploaded DXF / model and gives
  the quick kWp + payback estimate in the browser — a strong free lead magnet for the website.
