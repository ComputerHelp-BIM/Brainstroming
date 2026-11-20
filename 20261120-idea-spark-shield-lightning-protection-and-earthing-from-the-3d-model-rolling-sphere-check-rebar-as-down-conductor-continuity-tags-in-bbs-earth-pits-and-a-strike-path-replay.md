# 20261120 Idea: Spark Shield — Lightning protection and earthing from the 3D model: a rolling-sphere check on the terrace and projections, air terminals and down conductors laid out, the column and footing rebar tagged as the conductor path (with lap and weld continuity checked in the BBS), earth pits placed, a BOQ, and a "strike path" replay that shows the current going down the rebar to the ground.

## Entry 4

```
date-time:   2026-11-20 (scheduled run, UTC)
idea name:   Spark Shield (lightning protection and earthing from the 3D model, rebar continuity as the
             down conductor, rolling-sphere check, strike-path replay; in the browser, nothing uploaded)
potential:   Medium-high. Every building over a certain height or type needs a lightning protection
             and earthing scheme, and it is usually drawn late, by hand, on 2D plans, after the
             structure is fixed. Indian high-rises mostly use the RCC columns as down conductors
             (see sources), but the scheme is rarely checked against the real rebar in the model: are
             the chosen columns' bars lapped or welded, do they reach the footing, is the path
             broken by a transfer beam or a movement joint? That is the gap. It is also a clean
             extension of the rebar tools already in this repo.
             Time saved: NOT measured.
             First test: take 2 real projects (kept out of git) with an existing LPS drawing by the
             electrical consultant. Lay out the scheme by hand with a stopwatch, then with Spark
             Shield. Compare (a) minutes, (b) items the consultant's drawing missed that the tool
             flagged (broken rebar path, projection outside the sphere), (c) flags the consultant
             says are wrong (false positives). Do not quote hours or accuracy until this is done.
             The tool is a drafting and checking aid. It is NOT the lightning risk assessment and
             not a signed design; the electrical designer decides and signs. It must say so.
             Market check (2026-11-20, two standard web searches; not exhaustive, so "I did not find
             one", not "none exists"). Found:
               - DEHNplan (DEHN): web-based 3D lightning protection and earthing planning under
                 IEC 62305, shows the protected volume (rolling sphere), imports/exports IFC, DWG,
                 DXF. A DEHN brochure says Revit compatibility is "coming soon". It is a vendor tool
                 tied to a vendor's product range; I did not check its price or licence.
               - ZANDZ plugin for Revit: a Revit add-in for lightning protection and grounding
                 (Revit 2022-2025, beta in late 2024). Its pages cite a Russian standard, not
                 IEC 62305. Not checked: price, and whether it has a rolling sphere.
               - CYPE (Open BIM): a lightning protection module with a rolling sphere shown in the
                 model. Standard followed not confirmed. Paid suite; not checked further.
               - Axis India: a free lightning risk assessment tool based on IEC 62305-2, standalone,
                 not linked to a model.
             What I did not find: a free tool that starts from the structural model's rebar and
             checks the down-conductor path through the real bars (laps, welds, breaks at joints),
             then writes the result into the BBS and BOQ. "I did not find one", not "none".
             Trend: I could not measure LinkedIn or Instagram counts. What I can say from the search:
             an academic paper says Indian practice moved to RCC columns as down conductors after
             the National Building Code of India 2016, and that nearly all high-rises now use them.
             That is why the rebar side matters. The rolling-sphere and replay visuals are the kind
             of thing that suits a short clip, but that is my judgement, not a measured trend.
             Extends tools already in this repo (does not repeat them):
               - Joint Congestion Doctor / Bar Optimizer / BBS Weaver: the bars exist in 3D with laps.
                 Spark Shield reads them as an electrical path, not a structural one.
               - Live Wire: room to electrical points, conduit, DB. Spark Shield adds the roof and
                 earthing side that Live Wire does not cover (I found no "earthing" or "lightning"
                 idea in the repo except passing mentions in Sun-Roof and Demo-Map).
               - Sun-Roof: rooftop solar layout. PV frames and rails must be bonded and sit inside the
                 protected volume; Spark Shield checks that on the same terrace.
               - Gap Guard: movement joints break a rebar path; Spark Shield asks for a bonding strap.
               - Cage Manual / Cage Ghost: the "tagged bars" are shown in the fixing manual and in the
                 AR pre-pour check, so the site knows which bars must be lapped 20 d or welded.
               - Hanger Hero / Sleeve Punch: earth strips and pits near MEP routes are checked.
             The wow: press "Strike" on a building. A translucent sphere rolls over the terrace, lift
             machine room, water tank and chajja edges; the spots it touches glow red ("not covered").
             Add three air terminals and the red goes away. Press play: a bolt hits a terminal and the
             current runs down the tagged column bars, bar by bar, to the footing and the earth pit,
             then freezes at the first lap that is too short: "C14, level 9: lap 300 mm, needs 20 d
             = 480 mm for 24 mm bars".
```

### Description

**The pain.**
- The lightning scheme is drawn on 2D plans after the structure is fixed; the rebar is never part
  of the check.
- "Use the columns as down conductors" is written on the drawing, but nobody checks that those
  columns' bars are continuous to the footing, or that a transfer beam or expansion joint does not
  break the path.
- The site finds out at pour time (or never) which bars needed welding, bonding or a copper strap
  cast in. Adding it after the pour means drilling and external conductors.
- Terrace projections (water tank, lift machine room, antenna, parapet corners, solar frames) are
  missed because the 2D plan does not show height.
- Earth pits, test pits and the foundation earth ring are placed by rule of thumb with no link to
  the footing layout, the basement and the pile caps.

**The idea.** Five moves.
1. **Read the roof as 3D.** From the IFC model: roof levels, parapets, tanks, machine rooms,
   projections and their heights. The tool lists them as "things lightning can hit".
2. **Rolling-sphere check.** A sphere of the radius for the chosen protection level rolls over the
   model. Any surface it touches is "exposed"; air terminals (rods, strips) are placed to fix it.
   Auto-suggest places the fewest terminals; the user can drag them. The radius table is a
   plain, editable data file with the standard's values, shown with the clause so it can be checked.
3. **Pick the down conductors and prove the path.** Choose columns or let the tool suggest them
   (spacing around the perimeter, near the corners). For each, the tool walks the rebar in the
   model from the roof to the footing and checks: continuous bars, lap length at least the
   limit the user sets (the guidance in the sources says 20 times the bar diameter, or welded or
   clamped), no gap at a construction joint or movement joint, reaches the footing steel.
   Result per column: green, or a list of breaks with the exact bar and level.
4. **Earthing.** Foundation earth from the footing and raft steel, plus earth pits placed on the
   plot plan with spacing and clear of the basement walls and the services; bonding to the main
   electrical earth; test points at set heights. Each pit gets a resistance target and a place for
   the measurement entry later (not a calculation of resistance in v1).
5. **Write it back.** Tagged bars go into the BBS as "continuity bar: weld or lap 20 d" (extra
   length and weld counts), and into the 3D as a property so Cage Manual shows them. A BOQ of rods,
   strips, clamps, test links, pits and copper strap is produced, and a one-sheet LPS layout.

**What a user sees.**
- A "Spark Shield" panel with the protection level, the sphere radius, a list of exposed spots and
  a list of down-conductor columns with a green, amber or red path state.
- The "Strike" replay described above, and a PNG or short clip of it for a client or a site talk.
- A list of "needs a decision": a column path broken at a joint, a projection with no terminal,
  an earth pit where the basement wall is.

### Implementation

Inside the cad2bim app (packages/engine for the checks, apps/web for the panel). Nothing leaves the
device unasked. Runs on the open model in the browser.
- **Reading the model:** the existing IFC reading for levels, roof geometry, projections,
  reinforcing bars and their placement. The rebar path is a graph: bars are nodes, a lap or weld or
  coupler is an edge if the overlap or connector meets the rule; a break is a missing edge. The graph
  walk is a pure function that can be tested with small hand-made cases in Vitest (one column, one
  short lap, one expansion joint).
- **Rolling sphere:** a plain geometry check against the roof surfaces (distance field or sphere
  contact test over a mesh); no new renderer. The sphere and terminals are drawn with the existing
  viewer overlay code. Suggested terminal places come from a simple greedy cover over the exposed
  points; the user's edits always win.
- **Rules as data:** protection levels, sphere radii, mesh sizes, down-conductor spacing, minimum
  conductor sizes and the lap rule are in an editable table with the source clause next to each
  number. I did NOT verify the exact numbers or the current IS/IEC 62305 clauses in this search; the
  first build must be checked against the published parts and the National Building Code of India by
  an electrical engineer before a release. The table ships empty of claims until that is done.
- **BBS write-back:** reuse the BBS writer; add a "continuity" tag and extra length per weld or lap
  so the weight stays right. The tag is stored as an IFC property on the bar, so Revit and other
  tools can read it.
- **Replay:** an animation along the walked path (bars in order, then the footing, then the pit).
  The replay is a picture of the path check, NOT a lightning simulation, and says so on screen.
- **Agent access:** MCP tools in packages/mcp: "exposed spots", "down conductor path for column X",
  "continuity breaks". An agent answers "which columns fail as down conductors?" from the same
  engine.
- **Build order:** (1) rolling sphere + terminals on the roof; (2) rebar path walk + report for one
  column; (3) all down conductors + joint breaks; (4) BBS and BOQ write-back; (5) earth pits and the
  plot plan; (6) replay and clip; (7) MCP.
- **Risks:** this is a safety item, so a wrong green is the worst failure; the tool must show why
  each result is green or red and must never print "compliant". A model with thin rebar data (bars
  not modelled in columns) gives a thin result, and the panel says "bars not modelled, cannot check
  the path". Resistance of an earth system depends on soil and is NOT calculated here.
- **Check before release:** 2 real projects, the stopwatch test above, and review by a qualified
  electrical engineer who signs off the rule table.

### Future update

- **Risk assessment first:** a small IEC 62305-2 style risk questionnaire (size, height, use, flash
  density, services in) that suggests the protection level, with the sheet it came from; the free
  stand-alone tools in the market check do this only outside the model.
- **Earth resistance estimate:** soil layers from Pile Pilot's bore-log reading feed a first estimate
  of the earth electrode resistance, clearly marked as an estimate until measured on site.
- **Site test log:** the pit resistance and continuity readings entered on a phone (via Site QR
  Pass) and shown as a status light on each down conductor in 3D; handover into Handover Hub.
- **Surge and bonding map:** where services enter (power, water, gas, data), mark bonding bars and
  surge devices; check them against the Live Wire panel layout.
- **Retrofit mode:** use Scan-to-Frame or Scan Check to find where an existing building's path is
  doubtful and propose an external conductor route, as the guidance recommends when continuity is
  unknown.
- **Solar and antenna check:** with Sun-Roof, flag PV frames, railings and antennas outside the
  protected volume or unbonded.

Sources used for the market check and the trend note (vendor and industry material, treat as
claims; clause numbers not verified):
[DEHNplan](https://dehn-international.com/products/services/dehnplan),
[DEHNplan brochure DS132](https://www.dehn-international.com/sites/default/files/media/files/dehnplan-digitally-planning-lightning-protection-ds132.pdf),
[ZANDZ plugin for Revit](https://zandz.com/plagin-zandz-dlya-revit/),
[CYPE lightning protection](https://info.cype.com/en/subject/cypelec-distribution-lightning-protection/),
[Axis India, rolling sphere method](https://axis-india.com/rolling-sphere-method/),
[Nepal paper on lightning protection practice with RCC down conductors](https://nepjol.info/index.php/arj/article/download/65541/49705/192704),
[Earthing and lightning protection handbook](https://www.powerandcables.com/wp-content/uploads/2017/04/Earthing-Lightning-Protection-Systems-AN-Wallis-Handbook.pdf).
