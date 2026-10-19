# 20261019 Idea: Dig Deep — excavation, working space, shoring, backfill and truck trips straight from the foundation model and the survey levels

## Entry 1

```
date-time:   2026-10-19 (scheduled run, second entry of the day)
idea name:   Dig Deep — "Footings modelled? Now see the pit, the soil, the water and the trucks — in one click."
             Feed it the foundation model (footings / pile caps / raft / basement from cad2bim, Structura or any IFC),
             the survey (contour DXF, spot-level CSV or total-station points), the formation / FFL levels and the
             bore-log table (layers, depth, water table) as Excel. The tool builds a ground surface (TIN), cuts a
             real excavation solid around every footing (plan size + working space + side slope or shoring), merges
             neighbouring pits into trenches/bulk-dig, intersects it with the soil strata and the water table, and
             returns: cut volume by depth band and by soil type, PCC / blinding, backfill, surplus soil to cart away
             (with swell and shrink factors), tipper-truck trips per day, shoring / sheet-pile area, dewatering
             exposure, and a 3D "dig animation" (ground peels away, trucks counter ticks up). Outputs: Excel
             earthwork BOQ, DXF excavation plan with levels + sections, IFC 4.3 IfcEarthworksCut / IfcEarthworksFill
             objects, and a one-page "pit card" for the site engineer.
potential:   High, and the cheapest gap in the toolkit to fill. Of the ideas already in this repo, none touches
             earthwork — yet excavation is the first activity on every job, is paid in cubic metres by depth band
             and soil type, and is routinely measured by hand as "L x B x D" per footing, pasted into Excel,
             with working space, slopes and strata forgotten or guessed. Typical miss: 10-25 % on volume, wrong depth
             band (rates jump at 1.5 m / 3 m / 4.5 m), and no truck-trip plan until the road is already blocked.
             Basements make it worse: shoring, dewatering and haul-away are often the biggest early cost and the
             biggest early argument. A modeller or QS spends half a day to two days per block on this; Dig Deep
             turns it into minutes and, more importantly, makes "what if we lift the raft by 300 mm?" a slider
             instead of a re-measure. Direct saving: fewer disputes on RA bill #1, right trucks on the right days,
             earlier warning on over-deep / near-boundary pits.
             Trending angle: excavator, tipper and "ground opening up" visuals are among the highest-engagement
             construction content on short-video platforms, and "cut & fill in 30 seconds" /
             "see how many truck trips your basement needs" is a natural reel hook that also happens to be
             a useful tool, not just decoration. (Engagement claim is my understanding, not verified live this run.)
             Market: Civil 3D / Revit toposurface and Navisworks do cut-fill on a surface, and Civil 3D is paid per
             seat; to my knowledge none of the common tools take a *structural foundation model + bore-log +
             Indian-style depth-band BOQ + truck-trip schedule* and give it as one browser-based result. Open-source
             options (QGIS plugins, scripts) need GIS skill and do not know footings or BOQ bands. Not verified
             exhaustively; worth a 30-minute market check before building.
description: Daily use cases, in the order a BIM modeller / QS meets them:
             1. Foundation excavation BOQ (the bread and butter). For every footing, pile cap, plinth beam,
                lift pit and raft: excavation solid = footprint + working space each side (default 300 mm,
                editable per element or per soil) + side slope (angle of repose by soil) down from the
                ground surface to the founding level. Overlapping pits are merged, so the volume is not
                double counted. Result is split by depth band (0-1.5 / 1.5-3 / 3-4.5 m ... editable to match
                the contract) and by soil stratum from the bore log (soft soil / murrum / hard strata / rock).
             2. Ground from the survey, not from a flat assumption. Contour DXF, spot levels or total-station
                points become a TIN. Sloping sites show their real cut on the uphill side and fill on the
                downhill side instead of an average depth.
             3. Backfill and surplus. Backfill = excavation - (footing + column stub + plinth + PCC volume
                below the backfill line); apply compaction / shrink and swell factors, and the tool tells you
                how much soil leaves the site, how much comes back later, and how much borrow fill is needed
                for plinth filling / sunk-slab filling.
             4. Truck-trip schedule. Cubic metres to trips for the chosen tipper (default 10 m3 loose), cycle
                time from the haul distance, trips per day, days of excavation, with a day-by-day plan that
                can be pulled from the Pour Planner start dates (the pit is open for the minimum time).
             5. Basement and near-boundary pits. Below a set depth or inside a set distance from a boundary /
                existing building the tool switches from open slope to *shoring*: vertical wall with sheet
                pile / soldier pile and lagging area (m2), strut/anchor levels as inputs, and shows a red flag
                where the slope would cross the plot line. It does not design the shoring — it measures it.
             6. Water table and dewatering. Intersects pit depth with the water table from the bore log or a
                pre-monsoon / post-monsoon pair; pit area below water, sump count suggestion, pump-days
                exposure, and an "open in monsoon?" warning when the dig is planned in the rains.
             7. What-if slider. Change founding level, working space, slope or soil assumption, and watch cut,
                backfill, truck trips and cost move. "Lift the raft 300 mm: -420 m3, -42 trips, -Rs 1.1 lakh".
             8. Wow layer. A 3D scene with the ground surface peeling away pit by pit in the Pour Planner
                sequence, a heatmap of depth, trucks driving out with the live counter, strata colours on the
                pit walls. Exported as a 20-second video through Model Reel for LinkedIn / WhatsApp, and as a
                shareable viewer link.
implementation:
             Fits the existing stack (browser-first, Three.js, SheetJS, jsPDF, IFC import) and reuses much of it:
             1. Inputs. Foundation elements from IFC / cad2bim / Structura (reuse viewer.js geometry + parser.js).
                Survey as DXF contours (POLYLINE/LWPOLYLINE with elevation, TEXT spot levels), CSV (x,y,z), or
                Set-Out Pack points. Bore-log as an Excel template (layer, from-depth, to-depth, soil type, SBC,
                angle of repose, rate code) plus water-table cells. Boundary and neighbouring buildings as DXF
                polylines on named layers.
             2. Surface and strata. Delaunay TIN (small JS implementation or d3-delaunay, MIT) for the ground;
                horizontal strata planes interpolated between bore-hole points (inverse distance) for the soil
                layers. Everything in local metres, millimetre rounding at report time only.
             3. Excavation solids. Per element: plan offset polygon (clipper-style offset, MIT-licensed JS
                library) -> extrude to founding level -> side-slope frustum. Union by polygon boolean on plan
                per level slice (slice every 100 mm, sum areas x thickness) — robust, no heavy 3D boolean
                needed, and gives depth-band and strata splits for free. Prismoidal correction only where slopes
                are shown.
             4. Quantities. Slice volumes grouped by band and stratum; backfill from element volumes already
                available in the model (concrete takeoff in boq.js); swell/shrink factors in a rule pack;
                trips = ceil(loose volume / truck capacity); days from cycle time; shoring area = perimeter x
                depth where shoring mode applies.
             5. Rule packs. YAML/JSON per client or region: depth bands, working-space defaults, angle of
                repose by soil, swell/shrink factors, truck capacity, rate codes. A human-reviewed rule pack
                is the contract; the tool never invents rates.
             6. Outputs. Excel BOQ (SheetJS, one sheet per band/stratum + summary + truck plan), DXF plan
                (excavation outline, levels, sections through chosen grid lines; reuse Drawing Autopilot's
                DXF writer), IFC 4.3 IfcEarthworksCut / IfcEarthworksFill with quantities (Qto) and a
                PDF pit card (jsPDF). 3D scene reuses the Structura viewer; video through Model Reel.
             7. Phases (each shippable alone):
                P1 — footing excavation with working space + depth bands + Excel BOQ (1-2 weeks);
                P2 — survey TIN + strata + backfill/surplus + truck trips;
                P3 — shoring / boundary flags + water table + what-if slider;
                P4 — dig animation, Pour Planner sync, IFC 4.3 export, share link.
             8. Validation. Compare against hand-measured earthwork on at least three real foundation sets
                (isolated footings, strip + raft, basement); target within 3 % of the careful hand figure;
                keep those jobs as regression fixtures. Every output carries "Computed from model <hash>, rule
                pack <name/version> - verify before ordering or billing".
             9. Safeguards. Dig Deep *measures*; it does not do slope-stability, shoring design, dewatering design
                or bearing checks. Strata are interpolated between bore holes and are only as good as the bore
                log: show the interpolation distance and warn when a pit is far from any bore hole. Boundaries
                and neighbouring-building offsets are inputs from the drawing, flagged "to be confirmed on site".
future update:
             - Change Radar link: "Rev D deepened FTG-12 by 450 mm: +18 m3 cut, +2 trips, one band change".
             - Bill Painter link: measure the *actual* excavation from the total-station / drone survey after the
               dig and pay RA bill on measured-vs-modelled (Scan Check can compare the as-dug surface to the model).
             - Set-Out Pack link: export pit corner points and formation-level stakes for the excavator operator.
             - Machine-guidance export (LandXML / simple DXF surface) for excavator 3D guidance systems.
             - Haul-route and site-logistics layer: one-way gate, truck queue, road-closure windows, dump-yard
               capacity, and disposal receipts (weighbridge/truck-slip photo -> reconciliation).
             - Soil re-use planner: which excavated soil can be used for backfill vs which must leave site.
             - Weather-aware planning: monsoon / rainfall forecast against open pits and dewatering days.
             - Pile and retaining-wall module: pile cut-off and pile-cap trimming, secant / contiguous pile
               wall lengths, capping beam; the same engine as the pit.
             - Carbon: diesel litres and kg CO2e of the haul plan beside Bar Optimizer's steel carbon number.
             - Community rule-pack library (regional depth bands, soil factors, tipper sizes), reviewed per pack.
```
