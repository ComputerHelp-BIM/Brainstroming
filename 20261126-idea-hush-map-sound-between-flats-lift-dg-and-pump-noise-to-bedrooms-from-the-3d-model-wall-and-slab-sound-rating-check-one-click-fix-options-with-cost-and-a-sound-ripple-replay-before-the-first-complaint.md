# 20261126 Idea: Hush Map — sound between flats, lift / DG / pump noise to bedrooms, from the 3D model; wall and slab sound-rating check, one-click fix options with cost, and a sound-ripple replay before the first complaint

## Entry 1

```
date-time:   2026-11-26 (scheduled run)
idea name:   Hush Map — "Will the lift keep my client awake? Know before the wall is built."
             Drop the IFC (or the DXF-built model). Hush Map takes the noisy things (lift machine room and shaft,
             DG set, pumps, STP blower, AHU and VRF outdoor units, the road, the party wall to the next flat) and
             the quiet things (bedrooms, living rooms, classrooms, wards, offices), and walks the sound from one
             to the other through the real walls, slabs, doors and windows in the model. Every quiet room gets a
             score and a colour. Every failure is a pin with the cause in plain words ("bedroom B2 shares a 115 mm
             wall with the lift shaft") and fix options with their cost ("200 mm block + 12 mm plaster: +6 dB,
             +Rs 38,000 for this wall"). Then it plays a sound-ripple replay over the 3D model, so a client can
             SEE why the bedroom next to the lift is a problem.
potential:   High. Noise is the complaint that arrives AFTER handover, when it costs the most to fix, and it is
             almost never checked in a BIM model. In my reading of the existing ideas in this repo, none covers
             sound; fire, light, wind, sun, drainage and egress are all covered. Estimates below are mine, not
             measured:
             - Today: a noise check, if done at all, is a consultant's report on a few rooms, bought late,
               or a rule of thumb ("keep bedrooms away from the lift") that nobody can prove or price.
             - With Hush Map: a first whole-building screening map in minutes, re-run after each revision, and
               a priced fix list to take to the architect on day one of design coordination.
             Why it wins: it answers a question every client and every flat-buyer asks ("is it quiet?"), it
             turns a hidden risk into a picture, and a fix costs a few thousand rupees in design and several
             lakhs after the walls are up and furnished.
             Market: to my knowledge (not verified for this run), proper acoustic prediction is paid desktop
             software (room and environmental noise packages) run by specialist consultants, and most are
             not driven from an IFC in a browser. Revit has no sound analysis. A free, on-device, screening-level
             tool that works on any IFC and on the whole building, not on one room, is not one I know of.
             The honest limit is stated up front: it is a screening tool, not an acoustic certificate.
             Trending: "noisy neighbour" and "lift next to my bedroom" posts are common on LinkedIn and Instagram
             real-estate and homebuyer feeds; "I took the sound out of the building" is a strong reel idea.
             Treat this as a signal of interest, not a measured trend.
```

### Description

**The pain (daily life of a BIM modeller and coordinator).**
- Plans are laid out for area, light and vastu. Sound is nobody's job. The lift shaft ends up against a bedroom
  wall, the DG set sits under a living room, the pump room is below a flat, and the party wall between two flats
  is a single 115 mm wall because that is the default wall type in the template.
- The modeller already holds everything needed: where each room is, which walls and slabs surround it, their
  thickness and material, where the doors and windows are, where each plant item sits. Nobody turns it into an
  answer.
- Clients and buyers ask "will it be quiet?" and the team has only an opinion. After handover, the fix is
  re-building a wall, or adding a floating floor to an occupied flat.
- A consultant's noise report arrives late and covers a few rooms. It cannot be re-run in ten seconds after the
  architect moves the lift.

**What Hush Map does, in four screens.**

1. **Sources and receivers.** Hush Map finds them from the model with a rule and lets the user correct it:
   - *Sources:* lift machine room and shaft, DG set and its room, pump rooms, STP and blower room, AHU, chiller,
     VRF outdoor unit, transformer room, garbage chute, the road edge (a drawn line with a traffic class), a
     neighbouring plot function (drawn polygon, e.g. school, market). Each has an editable sound level that
     the user sets from the vendor datasheet. Starter values are shown in grey, labelled *example only*.
   - *Receivers:* bedrooms, living rooms, study rooms, classrooms, wards, meeting rooms and offices, from
     `IfcSpace` names and the room-type table already used for other checks. Each has a target ambient level
     from an editable table (starter: *NBC / CPCB-style values, confirm with the project's code and consultant*).
2. **Transmission (the screening engine).** For every source and receiver pair that shares a boundary, or
   sits within a set distance, Hush Map walks the path and sums it:
   - **Direct path:** the sound-reduction value of the wall, slab, door or window between them. The value comes
     from an **editable assembly table** (material, thickness, finishes, door and glazing types, with the
     rating in dB), not from one formula. Starter table includes common Indian assemblies (115 / 230 mm brick or
     block, 150 / 200 mm RCC slab, flush door, 5 + 6 + 5 glass); every row says *starter; replace with
     tested values from your supplier*. A simple mass-law estimate is the fallback for assemblies that are not
     in the table, and is marked as an estimate.
   - **Weak links:** the rating is area-weighted, so a hollow-core door in a 230 mm wall, or a 1 m² duct
     opening, lowers the wall's rating, as in real life. The tool shows which element is the weak link.
   - **Flanking and structure-borne:** a flat penalty for rigid connections, and a flag where a vibrating
     machine (lift motor, pump, DG) sits on a slab directly above or beside a receiver with no isolation. This
     is flagged, not computed.
   - **Floors:** for slab-over-slab pairs, an impact-sound flag from slab thickness and floor finish, e.g.
     "bare screed over a bedroom".
   - **Level at the receiver** = source level − reduction + a room-absorption term. All of it is shown as three
     numbers in the pin ("source 85 − wall 38 + room 3 = 50 dB(A)") so the user can check it. No black box.
3. **The map.**
   - **3D colour-by-result:** each receiver room is coloured green / amber / red by margin to its target; the
     path elements (walls, slab, door) are outlined. It uses the same colour-by-property route as other checks.
   - **Plan view:** per level, the sources, the paths and the failing rooms, ready to screenshot.
   - **Sound-ripple replay:** press *Play* on a source; a soft ring expands through the model, dims at each wall
     in proportion to its rating, and the receiver rooms light up by level. It records to a short video using
     the planned camera-moves-and-video feature. This is the shareable hook: ten seconds, no text needed.
4. **Fix options with cost.** For each failing pair Hush Map proposes ranked fixes from an editable list, each
   with the dB gained, the quantity from the model and the cost from the project's rates (the BOQ rate table):
   - thicker or denser wall, or a second leaf with an air gap and mineral wool;
   - swap to an acoustic door or a seal kit; laminated glass for facade windows;
   - spring or pad isolation under pumps and DG; a floating floor on the slab above;
   - and the cheapest fix of all, **move the use**: swap the bedroom and the kitchen, or move the lift lobby
     wall; Hush Map shows the score for the swapped plan before the user commits.
   *Accept* writes the new wall type to the model (through the existing edit path) and re-scores at once;
   *Reject* hides it. The fix list can be exported to BCF so the architect gets pins, not a PDF.

**Outputs.**
- **Noise screening sheet** (PDF / XLSX): every receiver room, target, predicted level, margin, the three-number
  derivation and the weak-link element. States the method and the starter-value status on page one.
- **Assembly schedule** (Excel): the wall, slab, door and window types used, with rating and where used. This is
  what the acoustic consultant asks for first; handing it over saves them a day.
- **Fix list with cost delta** and **before / after score** per room.
- **BCF** issues per failing pair with viewpoint.
- **Client card:** one image per flat ("Master bedroom: quiet, 33 dB(A) from lift") for a sales or handover pack.

### Implementation

Built on what cad2bim has, in small steps (my sizing, not measured). The app keeps files on the device, so nothing
is uploaded; no GPL code; the rating tables and targets are the project's, not ours, and every sheet says so.

1. **Source and receiver finder (1 week).** Read `IfcSpace`, plant equipment and rooms by name, with a
   room-type map reused from other room-based checks. Editable list; unit tests with a synthetic 6-storey block
   (a lift against a bedroom, a pump under a living room): all pairs found.
2. **Boundary graph (1 to 2 weeks).** For each pair, the elements between them (shared wall, slab above or
   below, door, window) from geometry in the engine's model. This is the same adjacency idea the compartment and
   cover checks need, so build it once and share it.
3. **Assembly table and screening engine (1 week).** JSON table of assemblies with rating, area-weighted
   combination, the level-at-receiver formula, the three-number explanation. Unit tests with hand-worked
   examples checked in a spreadsheet. Ship the table as *starter values*.
4. **3D result view (1 week).** Colour by margin via the existing colour-by property path; path outlines; plan
   screenshots with the design-system chart tokens.
5. **Fix engine (1 to 2 weeks).** Fix list with dB gain, quantity and cost from the BOQ rate table; accept /
   reject writes a wall-type swap through the edit path and re-scores. A "swap rooms" what-if on the plan.
6. **Sound-ripple replay (1 week).** A shader ring that fades by the rating of each element it crosses; a
   timeline slider; record through the camera-moves-and-video feature when it ships, a screen capture until then.
7. **Sheet, schedule, BCF (1 week).** Through the existing report path; one sample IFC in `docs/examples/`;
   a design note `docs/design/hush-map.md` in the app repo.

**Rules.** Every number on screen can be traced to a table row or a formula the user can see. Starter values are
labelled *example only*. The tool says plainly on every sheet: *screening, not an acoustic certificate*; where a
project needs one, the acoustic consultant's own rating replaces ours in the table.

### Future update

- **Measured loop:** the user types or imports a phone sound-meter reading from a finished room; Hush Map
  compares it with the prediction and offers to correct the assembly rating for the project ("our 230 mm block
  wall measures 4 dB better than the starter value").
- **Facade and road noise:** import a road-traffic count or a noise contour and score facade windows room by
  room; pair with Sun Compass to compare "more glass for light" against "more noise".
- **Reverberation and speech:** classrooms, offices and halls get a simple reverberation estimate from room
  volume and finishes, with a finish swap that brings it into range (pairs with Tile Tale and Ceiling Composer).
- **Vibration:** link the Hanger Hero hanger list and the pump and DG isolators so the vibration flag becomes a
  checked isolator schedule.
- **Hear it:** on a source and receiver pair, play a short stock sound filtered by the predicted reduction, so the
  client can *hear* a bedroom next to a lift at 50 dB(A) and at 35 dB(A). Clearly labelled an illustration.
- **Rule packs per city and building type:** hospitals, schools, hotels, with the code's clause shown beside each
  target, kept in the user's own packs.
- **Handover link:** the as-built assembly values and any measured readings go into the Handover Hub O&M pack.

### Related ideas in this repo

Complements Sun Compass (light), Fire Seal (fire), Gust Guard (wind), Cool Load (heat) and Fit Fill Furnish (room
use): it adds the missing comfort check, sound. It reuses the room, wall and slab data that Flush Fit and Fire
Seal already need, and the wall-type swap the fix engine uses is the same edit path as Mark Merge. None of them
scores noise between rooms.
