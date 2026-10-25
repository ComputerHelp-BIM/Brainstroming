# 20261025 Idea: Plot Pilot — drop the plot (survey DXF, 7/12 sketch, PDF or a map pin), pick the city's development rules, and get 10-20 buildable massing options in under a minute: setbacks, FSI with premium and TDR, ground coverage, fire-height bands, unit mix, parking count, sun hours and a sales-area figure for each, side by side, then one click turns the chosen option into a Revit start model (levels, grids, masses, floors, area scheme) that feeds Tower Stacker, Park Pilot and Area Statement Pro

## Entry 1

```
date-time:   2026-10-25 (scheduled run)
idea name:   Plot Pilot — "The developer sends a 7/12 extract and a road width on WhatsApp at 6 pm
             and wants to know by tomorrow how many flats fit. Open Plot Pilot, trace or import the
             plot, pick 'UDCPR — residential — 18 m road'. The tool draws the setback envelope,
             checks FSI, premium FSI, TDR, ground coverage and the height bands where fire norms change,
             and grows 14 massing options on the plot while you watch: 1 tower, 2 towers, L-shape,
             podium + tower, row of bungalows. Each card shows: 52 flats (2 BHK), 5,000 m² FSI area used of
             5,000, 42.6 m high (stays under the next fire band), 57 car spaces (stilt + 1 basement),
             winter sun 3.5 h on the lowest flat. Drag the tower and every number updates. Pick one, press
             'Send to Revit', and the start model is there: levels, grids, tower masses, floors and an
             area scheme, ready for the modellers the next morning."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Feasibility" stack in the Advance panel): pick a
                   site boundary (property line, topography, or a linked DXF), pick a rule set, run the
                   option generator in a modeless window, write the chosen option as native Revit
                   elements (levels, grids, mass families or in-place masses, floors, area scheme +
                   areas, project parameters for rule-set and option ID),
               (2) C2B / DXF path: plot boundary, road edges, road widening line, existing trees and
                   reservations read from the survey / approval DXF (named layers or the classifier),
                   so it works before any Revit file exists,
               (3) Structura-style static web viewer ("Feasibility" mode): the plot in 3D with the
                   envelope as a glass box, options as cards, sliders (floors, floor plate, tower
                   count, unit mix), live score table — a link the developer opens on a phone,
               (4) Tower Stacker / Park Pilot / Area Statement Pro / PlanWin hand-off: the chosen
                   option becomes the typical floor seed + floor matrix (Tower Stacker), the parking
                   brief (Park Pilot), the approval area statement draft (Area Statement Pro) and a
                   first column grid + storey list for PlanWin / ETABS.
             Outputs: buildable envelope (setbacks by height and road width, marginal distances between
             towers, height cap from airport / heritage / local limits if entered), 10-20 options with a
             comparison table (FSI used, ground coverage, height and fire band, flats by type, RERA carpet
             and saleable area, parking ECS required vs provided, open space %, sun hours, cost and
             revenue estimate), 3D option cards and a short "options reel" video, a 2-page feasibility
             PDF for the client, and the Revit start model for the chosen option.
potential:   Very high, and it sits at the very start of every project, so everything downstream (Tower
             Stacker, Park Pilot, Area Statement Pro, Sun Compass, Exit Map — earlier ideas) gets a clean
             seed instead of a hand sketch.
             Time today (one plot of 1,000-10,000 m²):
               - the architect reads the DCPR / UDCPR tables by hand for road width, zone and plot area,
                 works out setbacks, FSI, premium, TDR and ground coverage in Excel: 0.5-1 day,
               - 2-3 massing options sketched in AutoCAD or SketchUp, flats counted by hand: 1-3 days,
               - every time the developer asks "what if we buy TDR / add a floor / go for 3 BHK",
                 the Excel and the sketch are redone: 0.5 day per round, typically 3-6 rounds,
               - once one option is picked, the BIM team rebuilds it in Revit from the sketch:
                 levels, grids, floor plates: 1-2 days,
               - mistakes found later: the building crosses a fire-height band (extra staircase, refuge
                 area, fire lift), parking cannot fit, or the marginal distance between towers is short.
             With Plot Pilot: rules + envelope + 10-20 scored options in about 1 minute after the plot is
             in; a "what if" round in seconds; the Revit start model in a couple of minutes.
             Expected saving: 3-7 days per plot at feasibility stage, 1-2 days of Revit set-up per project,
             and fewer late redesigns because the fire band, parking and marginal distances are checked
             on day one. For an office that looks at 3-5 plots a month for developers, this is the
             difference between replying in a week and replying the next morning — which is how
             developer work is won.
             Wow effect: the plot outline appears, the setback lines slide inward, the glass envelope
             rises to the height cap, then towers grow out of the ground one floor at a time while
             the FSI bar fills up — and stops exactly at 100 %. 14 option cards flip in beside it like a
             deck. Drag a tower 2 m and the parking count, sun hours and flat count tick up and down live.
             Press "Send to Revit" and the same building appears in Revit with levels and grids. A
             30-second "plot to 14 options to Revit" reel is exactly the AI generative design content
             that is trending on LinkedIn and Instagram (Autodesk Forma, TestFit and "AI feasibility"
             posts) — but done on Indian bylaws, which those tools do not ship.
             Market: Autodesk Forma (paid subscription, site and environmental analysis, no Indian
             DCPR / UDCPR rule sets out of the box), TestFit (paid, US-focused site solver), Archistar
             (paid, Australia / US planning rules), Giraffe, Hypar, Snaptrude (paid / freemium, generic
             massing), Revit's own generative design (Revit 2021+, needs a Dynamo study written per
             project, no bylaw tables). To our knowledge nothing free takes a plot + an Indian city's
             development rules -> envelope -> scored generative options -> unit mix + parking + sun ->
             a native Revit start model in one tool.
```

### Description

**The pain (daily life of a BIM modeller and the design team).**
- Feasibility work arrives as a 7/12 extract, a hand-drawn plot sketch, a city survey PDF or a Google Maps pin,
  with a one-line brief: "residential, max flats, 2 BHK mostly, tell me tomorrow."
- The rules are spread over the city's DCPR / UDCPR / local bylaw PDF: FSI by zone and road width, premium FSI,
  TDR, setbacks by height and road width, ground coverage, marginal distance between buildings, parking ECS per
  flat by carpet area, recreational open space, and fire-norm changes at certain heights. One wrong table row
  and the whole option is wrong.
- Options are sketched in AutoCAD / SketchUp and counted in Excel; every "what if" from the developer means
  redoing both. Only 2-3 options are ever explored, so the best one is often missed.
- When an option is approved, the BIM team starts the Revit model from scratch, from the sketch, and the
  numbers in Revit no longer match the numbers that were promised to the developer.

**What Plot Pilot does.** It treats feasibility as one record — *plot, roads, rule set, envelope, options,
scores, chosen option* — and keeps the rule set in an editable table (one sheet per city / zone, with
the clause reference beside each row), not in code. Every option is generated from the record and every
number on every card traces back to a rule row, so a reviewer can see why the tool says "FSI 2.5" or
"setback 6 m".

**Daily use cases.**
1. *Developer enquiry*: plot + road width -> 10-20 options and a 2-page PDF the next morning, with the flat
   count and saleable area the developer cares about.
2. *Land purchase decision*: compare two or three plots on the same brief (flats, saleable area, parking cost
   per flat) before the developer buys.
3. *TDR / premium FSI decision*: the tool shows options with base FSI, with premium, and with premium + TDR,
   so the developer sees the extra flats against the extra cost.
4. *Project start in Revit*: the chosen option becomes levels, grids, masses and floors in the office template
   — no redraw from a sketch.
5. *Redevelopment (society buildings)*: existing members' carpet area entered as a constraint ("rehab
   flats first"), the rest of the FSI goes to sale flats — a very common Mumbai / Pune brief.
6. *Bungalow plots and row houses*: the same engine on small plots gives setback, ground coverage and
   floor count for G+1 / G+2 houses in seconds.

### Implementation

**1. Get the plot in.**
- DXF / DWG (via C2B's reader): closed polyline on a named layer = plot; lines on road layers = road edges with
  width tag; road widening line; reservation and existing tree layers.
- PDF / scan: Plan Tracer (earlier idea) to trace the boundary, then scale from two known dimensions.
- Map pin: draw the boundary on an OpenStreetMap / satellite tile in the web viewer; area checked against the
  7/12 area typed by the user (warns if they differ by more than a few %).
- Revit: property line or topography boundary in the active project.

**2. Rule sets (data, not code).**
- One Excel / JSON per city and zone: FSI by road width and plot area, premium FSI and TDR caps, ancillary
  FSI, setbacks as a function of building height and road width, ground coverage, marginal distance between
  buildings, recreational open space %, parking ECS per flat by carpet band + visitor %, height bands where
  fire norms change (staircase count, refuge area, fire lift), and any height cap (airport NOC, heritage).
- Every row carries the clause number and the date of the rule version. A "rules changed" log keeps old
  versions so past feasibility reports can be re-run as they were.
- First rule sets: UDCPR (Maharashtra, excluding Mumbai) and DCPR 2034 (Mumbai) residential, since that is
  where most of the office's work is; others added by filling the same template. Rules change, so every
  output says "design-stage estimate, verify with the authority's current rules".

**3. Buildable envelope.**
- Offset the plot boundary inward by the setback for each side (front setback from road width; side and rear
  from height), with road widening removed from the buildable area but kept for FSI credit where the rule
  allows it.
- Because setbacks grow with height, the envelope is a stepped solid: one footprint per height band. The
  generator keeps each tower inside the footprint for its own height.
- Marginal distance between towers enforced as a minimum gap that also depends on height.

**4. Option generator.**
- Building typologies as parametric templates: point tower (4-8 flats around a core), slab block (double-loaded
  corridor), L-shape, podium + tower(s), row houses / bungalows. Each template knows its core (staircase,
  lifts, lobby — FSI-free where the rule allows), its flat modules (1 / 2 / 3 BHK with RERA carpet, balcony
  and wall thickness from the office's unit library), and its floor-to-floor height.
- Search: for each typology, vary tower count, position, rotation (snapped to the plot edges and to true
  north), floor plate size, floor count and unit mix; reject anything outside the envelope or over FSI /
  ground coverage / height cap; score the rest. A simple grid search plus a genetic algorithm (e.g. NSGA-II
  via `pymoo`) for multi-objective trade-offs is enough at this scale and runs in seconds.
- Scores (user picks the weights): FSI used %, flats count, saleable area, parking feasibility, open space %,
  winter sun hours on the lowest flat (Sun Compass engine), view / privacy distance between towers, fire band
  penalty (crossing the next band costs a staircase / refuge area), structural regularity (towers on a
  simple column grid).
- Parking: ECS required from the rule set; ECS that fit estimated from stilt / podium / basement areas with
  the Park Pilot packing engine (quick mode); option flagged red if they do not fit.

**5. Money (simple and editable).**
- Construction cost per m² by typology and height band (office rate table), basement cost per m², TDR /
  premium cost from the user, sale rate per m² of carpet -> revenue, cost and a rough margin per option.
  Clearly marked as a feasibility estimate, not a valuation.

**6. Send to Revit.**
- Levels from the floor matrix (stilt / podium / typical / terrace / mumty), grids from the typology's column
  grid, one mass or floor per level per tower, an area scheme with the flat modules as areas (carpet and
  balcony), project parameters for option ID and rule-set version.
- The same JSON seeds Tower Stacker (typical floor + floor matrix), Park Pilot (parking brief), Area Statement
  Pro (approval statement draft) and a first PlanWin / ETABS storey list and column grid.

**7. Outputs.**
- Comparison table (Excel), option cards (PNG), feasibility PDF (plot, rules applied with clause numbers,
  envelope, top 3 options, table), 3D web link, a short options reel (reuses the Model Reel idea's renderer),
  and the Revit start model.

**Tech stack.** Python core (`shapely` for offsets and polygon checks, `numpy`, `pymoo` for the
multi-objective search, `ezdxf` for DXF, `openpyxl` for rule sets and reports) shared by the pyRevit button
and a CLI; Revit side in pyRevit (levels, grids, masses, floors, area scheme via the Revit API); web viewer
in the Structura style (three.js, xlsx.js and jsPDF already vendored there; the generator runs as a Pyodide
or a small JS port in a Web Worker so the browser version needs no server).

**MVP (4 weeks).**
1. Plot from DXF and from a drawn polygon; UDCPR residential rule set (FSI, premium, setbacks by height and
   road width, ground coverage, parking ECS, two fire-height bands).
2. Stepped envelope + point tower and slab block typologies with a 2 BHK / 3 BHK unit library.
3. Grid search with 5 scores; comparison table and option cards.
4. Web viewer with sliders and live scores; 2-page PDF.
5. "Send to Revit": levels, grids, masses, floors, area scheme. Demo on a real past plot (compare with the
   option the office actually drew) and record the "plot to 14 options to Revit" reel.

### Future update

- **More cities and uses:** DCPR Mumbai redevelopment schemes, Bengaluru / Hyderabad / Delhi bylaws,
  commercial and mixed use, plotted layouts with internal roads and amenity plots.
- **AI rule reader:** paste a new bylaw PDF and the assistant drafts the rule-set rows with clause references
  for a human to approve — the rule table stays the source of truth.
- **Floor plan fill:** after massing, auto-fill the typical floor with flat layouts from the office's library
  (links to Tower Stacker) and show the RERA carpet per flat.
- **Structure early:** a first column grid and storey stiffness check through PlanWin / etabs-mcp so options
  with very long spans or transfer floors get flagged at feasibility.
- **Environment scores:** wind comfort at ground (simple CFD or a pretrained surrogate), daylight on facades,
  noise from the main road; links to Sun Compass and Sun Roof (solar kWp per option).
- **Sales team view:** the chosen option becomes an interactive 3D inventory (flat by flat, sold / unsold) for
  the developer's sales office.
- **Approval package:** the chosen option flows to Area Statement Pro, Exit Map (fire egress) and Park Pilot
  so the approval drawings start from the feasibility, not from a redraw.

### Improvement suggestions for existing tools in the repos

- **pyRevit-addons (AnonGee):** "Auto Level Manager" could accept a floor matrix (JSON / Excel: name,
  elevation, type) so Plot Pilot, Tower Stacker and PlanWin can create the same level set; a small
  "grid from spacing list" helper would serve the same three tools.
- **C2B:** add a "site / survey profile" to the classifier (plot boundary, road edges with width tags, road
  widening, reservations, trees) next to the structural profile, so survey DXFs are read the same way
  structural drawings are today.
- **Structura:** a generic "option cards + sliders" panel (parameters in, scores out, 3D swap) would serve Plot
  Pilot and any other generative tool, and a glass-envelope / ghost-solid display mode in three.js.
- **PlanWin AI Pro:** let the AI assistant accept "start from Plot Pilot option 7" (storey list + column grid
  JSON) so the structural model starts the same day the developer picks the option.
- **etabs-mcp:** a "create stories and grid from JSON" endpoint / MCP tool, so the same option JSON can start
  an ETABS model as well.
- **cad2bim:** a free "How many flats fit on my plot?" page (draw the plot, pick the city, see 3 options) —
  a strong lead magnet for the website, with the full option set and "Send to Revit" in the paid plan.
