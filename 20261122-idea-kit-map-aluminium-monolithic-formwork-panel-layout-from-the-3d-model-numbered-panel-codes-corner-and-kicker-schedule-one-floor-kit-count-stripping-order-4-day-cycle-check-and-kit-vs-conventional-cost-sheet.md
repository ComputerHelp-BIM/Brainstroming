# 20261122 Idea: Kit Map — Aluminium (Mivan-type) monolithic formwork planned from the 3D model: every wall, slab and beam face cut into catalogue panels with a code, corners and kickers placed, one-floor kit counted, stripping order and 4-day cycle checked, and a kit-vs-conventional cost sheet.

## Entry 1

```
date-time:   2026-11-22 (scheduled run, UTC)
idea name:   Kit Map (aluminium monolithic formwork layout from the typical floor: panel codes,
             corners, kickers, kit count, stripping order, cycle check, cost; in the browser)
potential:   High for contractors, formwork vendors' design desks and BIM teams on shear-wall
             residential towers in India (the aluminium system is used on many high-rise
             housing towers; one supplier claims about 99 % of Mumbai high-rises, which is a
             marketing claim, not checked). Medium-high for the toolkit: it uses the walls and
             slabs the DXF -> 3D pipeline already makes, and gives the "typical floor" a
             second life after rebar.
             The pain: the panel layout of the kit is drawn by the vendor's design desk
             (often a proprietary tool, often days per typical floor). Every revision of the
             architect's or consultant's plan sends the BIM modeller and the vendor into a
             slow email loop: "wall W14 moved 150 mm, does the kit still fit?" Wrong panel
             counts are found on site, at the first floor, when it is costly.
             Time saved: NOT measured.
             First test: take 2 finished towers (kept out of git) with a vendor's kit drawing
             for the typical floor. Run Kit Map on the same plan. Compare panel count by type
             and total kit weight with the vendor's list. Quote no accuracy before this.
             Market check (2026-11-22), two standard web searches:
               - Found: market news (a new Indian aluminium monolithic formwork brand launched
                 at Bauma CONEXPO India 2026); papers that say BIM-automated drawings for
                 aluminium formwork cut cost (a 2026 academic claim of 31-53 % vs timber,
                 method not checked); Doka 360, a browser platform for formwork planning,
                 orders and AI-assisted material counting (seen in a LinkedIn post, US launch).
               - NOT found: a named, open tool that lays out Indian-style aluminium monolithic
                 panels from an IFC or DXF-made model in the browser. Vendors' design tools
                 are proprietary or paid, and I did not check them feature by feature.
                 "I did not find one", not "none exists".
             Trend: formwork and AI-assisted BIM are in 2026 posts and trade news (above). I
             could not measure LinkedIn or Instagram counts, so no trend is claimed beyond that.
             Extends tools already in this repo (does not repeat them):
               - Shutter Plan (20261017): conventional ply, props and beams cut list. Kit Map is
                 the other system: a fixed catalogue of reusable panels, no ply cutting.
               - Tower Stacker: makes the full tower from one typical floor. Kit Map plans the
                 kit that is reused on every one of those floors.
               - Pour Planner, Crew Curve, Order Clock: they take the cycle days as input. Kit
                 Map says whether a 4-day (or 5, 7) cycle is possible with this kit.
               - Core Cage: the shear-wall rebar. Kit Map is the shuttering around that rebar,
                 and checks the wall tie and rebar cover do not collide.
               - Sleeve Punch: sleeves and openings. Kit Map turns each into a panel cut-out.
             The wow: drop the typical-floor model; in seconds every wall and slab face turns
             into coloured panels with codes (W-1200x2400, SL-600x1200...), a "kit explosion"
             animation lays them out wall by wall in the order the gang will strip them, and the
             counter at the side shows "kit: 1,842 panels, 96 t". Move a wall 150 mm: only the
             changed panels flash and the count updates.
```

### Description

**The pain.**
- The kit is a few thousand panels of fixed sizes. One wrong corner or kicker stops a pour.
- The vendor's design desk is the bottleneck; changes in the architect's plan wait in a queue.
- The contractor cannot easily test options: "what if the floor-to-floor goes from 3.0 to 3.1 m?",
  "what if the lift core is cast ahead?" The answer is an email.
- Clash with MEP sleeves, ducts and openings is found when the panels arrive.

**The idea.** Five moves.
1. **Read the typical floor.** Walls, slabs, beams, columns and openings from the model the app already
   builds (DXF -> 3D) or from an IFC. The user picks the floor-to-floor height and slab thickness.
2. **A panel catalogue (editable).** Panel widths and heights, corner types, kicker, soffit, beam bottoms
   and prop heads, in a small JSON pack. Starter values are copied from public vendor sheets and marked
   "check with your vendor"; each output prints the catalogue and its version.
3. **Lay out and code.** For each wall face and slab, fill with the fewest catalogue panels, put
   the fillers last, place internal and external corners, kickers, soffit and prop heads, and give every
   piece a code and a mark that repeats floor after floor.
4. **Check.** Opening and sleeve cut-outs, rebar cover and tie spacing against the wall, piece weights
   against the one-person-lift limit, and the pour order of wall and slab for a given cycle
   (4 / 5 / 7 days) with the strike-ready times of the walls and the slab.
5. **Output.** A panel list by code and count, the kit weight, a stripping order sheet, panel
   marking drawings (DXF) and a kit-vs-conventional cost sheet (kit rent or purchase, reuse count, labour,
   plus the conventional rate set from Shutter Plan).

**What a user sees.**
- A "Kit Map" panel: floor-to-floor, cycle days, catalogue and reuse count.
- 3D panels coloured by type, a counter of the kit, and a play button for the stripping order.
- Click a panel: code, size, weight, which wall it faces, and which clash it avoids.
- Outputs: Excel (panel list, kit weight, cost), DXF per wall elevation with codes, a PDF
  stripping order for the site.

### Implementation

Inside the cad2bim app; the model never leaves the device.
- **Base:** wall and slab faces from the engine (packages/engine); no new reader needed. The plan comes
  from the Rust pipeline (packages/pipeline) like the other tools.
- **Core:** a pure function (faces, openings, catalogue) -> panels, with a small packing step per face:
  start at a corner, take the widest panel that fits, end with the smallest filler. Test with Vitest on
  hand-drawn walls with known answers (an L-shape, a wall with a door cut-out, a slab with a shaft).
- **Catalogue:** JSON pack with a source line and date; editable in the panel; a vendor may supply
  their own pack.
- **Cycle check:** a simple time model of wall pour -> strike -> slab pour -> strike, using
  concrete strength-time rules the user sets (links later to Cube Clock).
- **3D:** colour overrides on the faces; a play step for the stripping order using the planned
  camera-moves player.
- **Output:** DXF per elevation using the existing CAD writer; Excel via the existing BOQ writer; PDF via
  the existing documents core.
- **Agent access:** MCP tools in packages/mcp: "how many panels for tower B typical floor", "what changes
  if W14 moves 150 mm", "which walls need a custom panel".
- **Build order:** (1) catalogue and a one-wall packer with a table output; (2) all walls, corners and
  kickers; (3) slab and beam soffit; (4) openings and sleeves; (5) kit count, weight and cost sheet;
  (6) cycle check and stripping order; (7) DXF elevations; (8) MCP.
- **Risks:** real kits have vendor rules that this tool cannot know (special corners, tie patterns,
  safe lift weights): the output is a first layout, and the vendor or formwork engineer signs it; the
  catalogue is not a vendor's product and does not claim to be; irregular plans need many custom
  panels, and the tool lists them rather than hiding them; panel names are the vendor's own, so the
  pack lets users rename them.
- **Check before release:** the 2 towers above with the vendor's lists, and a site engineer's review of
  the default rules and cycle times.

### Future update

- **Vendor packs:** a vendor loads their catalogue and rules; the contractor sees the vendor's real panel
  codes in the model, and sends the list back as an order.
- **Live kit on site:** QR on each panel (as in Site QR Pass); the gang scans, the 3D shows which
  floor and which wall each panel is on, and lost or damaged panels are counted.
- **Reuse and wear:** count uses per panel, flag panels near the end of life, and plan the kit shift to
  the next tower.
- **Design-change diff:** when Change Radar finds a new revision, Kit Map lists only the new, removed
  and re-cut panels, with a cost delta.
