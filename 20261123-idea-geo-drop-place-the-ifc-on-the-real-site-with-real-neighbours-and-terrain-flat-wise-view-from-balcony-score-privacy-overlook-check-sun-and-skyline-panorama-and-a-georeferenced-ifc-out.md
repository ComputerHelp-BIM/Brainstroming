# 20261123 Idea: Geo Drop — Type the plot's address or drop a pin and the model lands on the real site, with the real neighbouring buildings and the ground. Then it shows the view from every flat's balcony, checks which windows look into the neighbour's rooms, and writes a correctly georeferenced IFC. All in the browser.

## Entry 1

```
date-time:   2026-11-23 (scheduled run, UTC)
idea name:   Geo Drop (IFC or DXF-built model placed on a real map: real neighbours and terrain,
             flat-wise "view from my balcony" panorama and score, window-to-window privacy check,
             sun and skyline on the real context, and a clean IfcMapConversion written back)
potential:   High for Indian residential teams, developers' sales teams and BIM leads who send
             models to authorities, consultants and clients. Very high "wow": on a phone or in the
             browser, tap "Flat 1204, east balcony" and a 360-degree panorama opens from that exact
             spot, showing the real towers, road and hill around the plot, with the sun path on top.
             Sales teams already post "view from your balcony" reels; today those are made by hand
             in Twinmotion, Lumion or by a render studio, per flat, in days. Geo Drop makes them
             from the model in minutes.
             Time saved: NOT measured. The pain below comes from public sources and team
             experience, not from a timed study.
             First test: take one finished tower model and its plot (kept out of git). Time a
             modeller doing the manual route (place the model on a map, find neighbours' heights,
             render two views). Then run Geo Drop on the same model and compare time, height
             errors of neighbours (check 5 buildings on site or with the survey) and view
             direction. Do not quote savings or accuracy until this is done.
             Market check (2026-11-23). Two standard web searches. Found: IFC Georeferencer
             (browser-based, runs on the device, Apache-2.0, reads and writes IfcMapConversion and
             IfcProjectedCRS; built for buildingSMART Nederland), HTW Dresden BuildingLocator
             (web app that finds an address with OpenStreetMap's Nominatim; it cannot write the
             result back to the IFC), a 2018 TU Delft IFC viewer on a Cesium globe, and research
             papers on web digital twins. Autodesk Forma exists; its public price pages mix the old
             Construction Cloud product with it, with prices that conflict (G2 vs TrustRadius), so
             I do not quote a price.
             What I did NOT find: one tool that does all of: georeference + real neighbours + a
             per-flat balcony view with a score + a window-to-window overlook check + a georeferenced
             IFC out, free and on the device. "I did not find one", not "none exists". The search
             tool was thin on the urban-design side (view and overlook analysis), so check Forma,
             Twinmotion and Indian sales-visual studios' feature lists before building.
             Trend: "view from your flat" and "digital twin of the city" posts are regular on
             Instagram real-estate pages and LinkedIn. I could not count them, so this is not a
             measured claim.
             Extends tools and ideas already here (does not repeat them):
               - cad2bim's own gap: docs/known-issues.md and docs/research/sgvue-comparison.md say
                 the app opens files with COORDINATE_TO_ORIGIN and has no georeferencing readout
                 ("cad2bim missing"). Geo Drop's first step is that readout, then goes further.
               - Sun Compass: sun hours and daylight on one's own building. Geo Drop adds the REAL
                 neighbours that cast shadows on it and on the terrace.
               - Neighbour Shield: pre-dig condition record of adjacent buildings. Geo Drop gives
                 their position, height and distance from open data in one click, as a starting list.
               - Plot Pilot: massing from bylaws. Geo Drop is the next step: context and views for
                 the chosen massing, and the real plot lines.
               - Model Reel / Mood Swap: videos and photoreal finishes of the model. Geo Drop is the
                 outside-world part: what the flat sees.
               - Set-Out Pack: total-station points. Geo Drop gives the site origin in a real CRS
                 that Set-Out Pack can use.
               - Grid Lock: aligns consultants' models by grids. Geo Drop aligns by real
                 coordinates when grids are missing.
             The wow: type "Plot 14, Sector 21, Navi Mumbai" (or paste a lat-long, or draw the
             plot). The tower stands on the map in 3 seconds. A slider moves a camera from the
             ground floor to floor 24 on the east balcony; the pixels show a hill, a road, two
             other towers. A table lists all 96 flats: view score, "sea sliver", "facing a wall 9 m
             away", "overlooks the neighbour's bedroom window". One button exports the reel.
```

### Description

**The pain.**
- Revit and IFC models often sit at the origin or far from it. Sharing them with a consultant
  or an authority means someone fixes the origin, rotation and units by hand, and each file
  lands in a different place.
- The architect and the sales team want to know "what will the buyer see?" and "who sees into
  whom?". Today a visualiser builds the neighbouring buildings by hand for each view.
- Neighbours' heights are guessed for the shadow, view and privacy talk with the client or the
  neighbour. A wrong guess shows up as a complaint or a redesign.
- Real-estate sales stories live on views and sunlight, and a plain model view says nothing about
  them.

**What Geo Drop does.**
1. **Find and place.** The person types an address, drops a pin, or loads a survey point. The model
   (the IFC, or the 3D built from the DXF) is placed with the site's north, the plot outline and
   ground levels. The readout shows the coordinates, the CRS and the true north, and says what is
   missing or looks wrong (a site 1,200 km off the pin, a rotation of 180 degrees, millimetres vs
   metres).
2. **Real context.** The neighbouring buildings come from open map data (footprint and a height if
   the data has one; the app marks "height guessed" in amber and lets the person type the real
   one). Ground comes from an open terrain set. Roads and water are drawn flat. Each item says
   where it came from and when.
3. **Flat-wise view.** For each flat (an IfcSpace or a marked balcony/window point), render
   a 360-degree panorama from the balcony or window centre at eye height. Show: the view score
   (open sky, distance to the nearest facing wall, share of green or water in the view, how much
   of the view is the project's own other tower), a one-line label, and the direction.
4. **Overlook check.** From every window, test the straight sight lines to neighbours' windows
   inside a distance the person sets (the local rule, or a team limit). It lists "window A of Flat
   804 looks at the neighbour's building, 11.2 m away, same height band". It does not claim
   anyone's legal rights: it lists where privacy may be an issue so the architect can act.
5. **Sun and skyline.** Put the sun path of any date over the real context. Per flat and per
   terrace: sun hours with the real neighbours' shadows. Compare today's neighbours with the
   permitted heights of plots that are still empty ("what if the plot next door goes to 70 m?").
6. **Output.**
   - A georeferenced IFC (IfcMapConversion, IfcProjectedCRS) that other tools read correctly.
   - A flat-wise view sheet (PDF) and an Excel table for the sales team.
   - A short MP4 or a link-free HTML panorama pack per flat (works without internet, in the
     spirit of Model Capsule).
   - A glTF or KMZ of the model in place for other viewers.

**What the person sees.**
- A *Geo* tab in the viewer: map and 3D together, an address box, a pin, a plot outline.
- A slider for floor height and a compass ring for direction. A flat table beside it.
- Amber labels on anything guessed; green on anything checked against a survey.
- Buttons: Place model, Flat views (PDF + Excel), Panorama pack (HTML), Georeferenced IFC.

**What it is not.** It is not a survey. Open-map heights and terrain have errors, so the result
shows "context for decisions", not a legal boundary or a sanction drawing. It does not replace a
daylight or privacy rule of the local authority.

### Implementation

- **Where it lives.** The georeferencing reader and writer go into the IFC engine
  (`@cad2bim/engine`), as the SGVue comparison note already recommends (a small `georef` module
  that reads `IfcMapConversion` and `IfcProjectedCRS`, and the `IfcSite` placement). The map and
  context view are a panel in the web app (three.js, React, the design system). This repo holds
  ideas only; nothing is built here.
- **Georeferencing maths.** Coordinate conversions use a small, permissively licensed projection
  library (MIT or BSD, licence checked first; no GPL). EPSG codes come from the person's pick or
  a local list for India (UTM zones 43 to 46, and the state grids if the team uses them). Scale
  direction of `IfcMapConversion` differs between Revit versions, so test with files from at least
  two Revit versions, as the SGVue note warns.
- **Opening files today.** The app opens IFCs with `COORDINATE_TO_ORIGIN`. First version keeps that
  for fast viewing and shows the removed offset in the readout; placing on the map applies the
  stored offset again.
- **Open data and the device rule.** The app's rule is "nothing leaves the device unasked". So
  map and terrain fetches are an explicit button ("Load context for this area"), say which
  server will see the plot's area, and send only the area box, never the model, its name or its
  project. Also offer a local file path: the person loads a context file (GeoJSON of
  footprints, a terrain GeoTIFF) and no request is made. Licences of the data (for example
  OpenStreetMap's ODbL needs attribution) are shown on the sheet and in the credits; a data
  licence check is a step before shipping.
- **Context building.** Footprints are extruded to the given height, or to `levels x 3 m` marked
  as guessed. A roof-less box is enough for view and shadow; a person can replace one building
  with their own IFC.
- **Views.** Render a cube map from the flat point with three.js, stitch to a panorama on the
  GPU, read the pixels back (a downsized copy) for the score: sky fraction, nearest hit distance,
  share per class (building, ground, water, vegetation, own project). Scoring rules live in a
  small rules file that a firm can edit; the app sets no standard.
- **Overlook rule.** Ray casts between window centres in a distance and angle band; the result is
  a pair list (window, neighbour window or wall), with distance and height difference. Cheap
  because windows are few (hundreds, not millions).
- **Sun.** Sun position from a standard solar formula (public domain maths); shadows from the real
  context via shadow maps for views and a ray test per point for the hour tables. Cross-check one
  date against a known source before trusting it.
- **Agent helper.** Expose `geo_place(address_or_latlong)`, `flat_views(model)` and
  `overlook_pairs(model, limit)` through the MCP helper so an agent can answer "which flats see
  the neighbour's windows within 12 m?". This follows `docs/agent/protocol.md`.
- **First version (smallest useful).** Georeferencing readout and write-back (reader/writer, one
  CRS pick, a pin on a map), plus the real neighbours as extruded boxes and the sun path. Then:
  flat-wise panorama and score, then the overlook check, then the sheets and the reel.
- **Test.** One tower and one low-rise plot where the neighbours' heights are known from a survey;
  check the placed position against the surveyor's point (error in metres), neighbours' heights
  against the survey, one panorama against a drone or phone photo from the same floor, and shadows
  against a real photo at a known time. Files kept out of git.
- **Version.** A new minor version of the engine and the app when built; changelog, README and
  guide entry as CLAUDE.md asks.

### Future update

- Terrain and building data from the authority's own GIS layers or the client's survey, with the
  source named on every sheet.
- Photo-match: stand on the future balcony or the empty plot, take a phone photo, and Geo Drop
  lines it up with the model's view to check or correct the heights of neighbours.
- "Price the view": a table of view score by floor and direction that the sales team can use for
  floor-rise pricing, from the team's own sales history rather than a built-in number.
- Noise and wind context: major roads and the prevailing wind from open data, for a quick
  early-design screen (an extension, not a calculation of record).
- Future-neighbour what-if: use the master plan's permitted heights on empty plots to show the
  worst case for views and sun in ten years.
- Link to Plot Pilot and Sun Compass: massing, sun and views change together as one slider.
- Link to Neighbour Shield: the neighbour list becomes the condition-survey list, with distances.
- Handover: put the georeferenced model on the facility twin so the building's location is right
  for the owner's GIS from day one.
