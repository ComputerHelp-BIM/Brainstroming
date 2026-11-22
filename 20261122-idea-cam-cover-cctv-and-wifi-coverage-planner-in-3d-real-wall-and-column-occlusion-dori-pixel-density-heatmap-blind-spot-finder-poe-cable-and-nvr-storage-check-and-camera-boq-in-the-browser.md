# 20261122 Idea: Cam Cover — Place CCTV cameras and Wi-Fi access points in the 3D model and see, live, what each one really sees: walls, columns and beams block the view, a colour map shows who can be identified and who is only detected, and blind spots, cable runs, NVR storage and the camera list come out ready for the vendor. In the browser, nothing uploaded.

## Entry 1

```
date-time:   2026-11-22 (scheduled run, UTC)
idea name:   Cam Cover (IFC / .c2b / drawing-built model to CCTV and Wi-Fi coverage plan with
             true line-of-sight, DORI pixel-density heatmap, blind-spot list, PoE cable and
             storage check, and a camera schedule; on the device)
potential:   Medium-high. Every building (residential tower, office, school, hospital, mall)
             needs CCTV, and most also Wi-Fi and access control. Today the camera layout is
             drawn late, on a 2D plan, by the vendor or the electrical modeller, with cone
             symbols that ignore walls, columns and beams. Blind spots are found after
             installation, when moving a camera costs a ceiling opening and a cable re-pull.
             A structural-first toolkit already holds the exact thing this needs: true
             geometry of every wall, column, beam and slab. High "wow" per minute: drag a
             camera and watch the coverage cone clip around a column in real time.
             Time saved: NOT measured. Do not quote hours until the first test below is done.
             First test: take 3 finished models kept out of git (one residential tower
             podium + lobby + parking, one office floor plate, one school or hospital wing).
             Ask a vendor or a modeller for the camera layout they would draw today and time
             it. Run Cam Cover on the same floors. Then compare: cameras needed, blind spots
             found by each method, and (best test) walk the real building after install and
             count the spots that were not covered.
             Market check (2026-11-22). Two standard web searches. Found, not verified in
             depth:
               - Avigilon / Motorola Revit plugin: simulates 3D fields of view, checks the
                 model for walls and pillars that block the view, shows DORI ranges per
                 IEC EN-62676-4. Needs Revit; I did not find its price.
               - IPICA "IP Video System Design Tool": standalone Windows app; 2D and 3D
                 camera placement, field of view, pixel density, DORI zones, bandwidth and
                 storage estimate. Not BIM / IFC based.
               - Axis Site Designer: free, in the browser; coverage on maps, bandwidth and
                 storage estimates, a bill of materials. The results I saw did not describe a
                 3D field of view.
               - Research papers: coverage from BIM (Albahri and Hammad, ITcon 2017; Aziz et
                 al., ITcon 2024, camera placement optimised on a hospital model with higher
                 coverage and less overlap than an experience-based layout; a voxel method
                 for construction sites). Papers, not products.
             Wi-Fi planners such as Ekahau exist; I did not check their price or features.
             What I did NOT find: one free tool that reads an IFC or a drawing-built model in
             the browser and gives line-of-sight, DORI, cabling, storage and BOQ together,
             next to the structure. "I did not find one", not "none exists". Vendor tools
             mostly cover only their own camera ranges.
             Trend: AI video analytics and smart-building / security-by-design posts are
             common on LinkedIn. I could not measure counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Live Wire (room to electrical points, conduit, DB): power points. Cam Cover
                 is the low-current half (cameras, access points, readers) with its own
                 rules (PoE 100 m limit, storage, pixel density), and can reuse Live Wire's
                 conduit routes.
               - Ceiling Composer (RCP: lights, sprinklers, detectors): shares the ceiling
                 grid and height; Cam Cover places on that grid and checks beam and duct
                 drops that hide a lens.
               - Sleeve Punch / Cut Check: a camera cable needs a sleeve or a core; Cam Cover
                 lists them so they are cast in, not drilled later.
               - Handover Hub: the camera and AP schedule becomes asset tags and an AMC
                 list.
               - Model Capsule / Room Link: share the coverage view with the client or
                 security consultant without a server.
             The wow: open a tower model, go to the ground floor lobby and parking, press
             "Auto-place". Cameras appear; each throws a coloured cone that stops at the
             columns. Red patches are blind. Press "Intruder walk": a figure walks the
             shortest route from the gate to the lift, and the screen says "unseen for 14 m
             behind the ramp wall". Drag one camera 2 m, and the red patch closes while the
             camera count in the corner drops from 11 to 9.
```

### Description

**The pain.**
- The camera layout is drawn on a 2D plan with cone symbols. The cone passes through walls,
  columns, beams and half-height partitions as if they were glass.
- Mounting height, lens angle and the ceiling are guessed. A camera under a deep beam sees the
  beam.
- "Cover everything" is not a spec. The real question is "can we identify a face at this door"
  (high pixel density) versus "can we see that a person is there" (low), and a 2D cone cannot
  answer it.
- Cable runs are found on site: a switch too far (over the usual 100 m copper limit) or a
  riser with no room for the extra trunk.
- Storage is a vendor spreadsheet, redone after every change in camera count.
- Wi-Fi access points are placed by rule of thumb, then a survey finds dead rooms behind
  shear walls and lift cores.
- Late changes cost a core, a ceiling opening and a re-pull.

**The idea.** Six moves.
1. **Pick the model and the floors.** The structure and walls come from the open model. Doors,
   glass, and partitions are read with their types, so a glass wall is see-through and a
   brick wall is not.
2. **Place devices.** By hand (click on a wall or ceiling) or "Auto-place" from a rule:
   entry doors, lobbies, lift lobbies, staircase landings, parking aisles, perimeter, and
   cash or server rooms. Camera type is a short list (dome, bullet, PTZ, fisheye) with
   lens field of view, resolution and mounting height, from a library the team edits.
3. **True line-of-sight.** For each camera, rays are cast against the real geometry (walls,
   columns, beams, slab edges, door leaves), so the cone clips around a column and a
   beam. Output is a floor coverage map on a grid (say 0.25 m).
4. **DORI by colour.** Each grid cell is coloured by the best pixel density it gets,
   against the IEC 62676-4 levels (Detect, Observe, Recognise, Identify). The person
   chooses the goal per zone ("Identify at the main door; Observe in the car park").
   Blind cells and cells below their goal are listed.
5. **Cable, power and storage.** Routes follow ceiling and riser paths to the nearest
   switch or rack room; the tool flags a run over the copper limit, a switch with no free
   ports and a riser with no room. Storage in TB comes from bitrate, frame rate, codec,
   days kept and motion share, all shown as inputs the person can change.
6. **Wi-Fi in the same screen.** Access points use the same model with wall materials
   carrying a simple loss value (brick, RC, glass, drywall, lift core). Output is a signal
   map and a "needs another AP here" list. Estimated, not a survey, and labelled so.

**What the person sees.**
- *Coverage* tab: floor selector, device library, goal per zone.
- 3D view: cones, and a floor heatmap; click a camera for its view as a picture ("camera
  POV") rendered from the model.
- A blind-spot list with a "fix" suggestion (move 1.5 m, change lens, add one camera).
- *Intruder walk* and *Escort route* replays on any path.
- Buttons: Camera schedule (XLSX), Layout drawing (DXF per floor), Coverage report (PDF),
  Sleeve and core list, Share view (single file).

### Implementation

- **Where it lives.** A new module in `@cad2bim/engine`, reading the geometry the viewer
  already holds, and a *Coverage* panel in the app. No server. Nothing leaves the device.
- **Line of sight.** Use the existing three.js scene and a BVH raycaster (check the licence of
  any BVH library: MIT, Apache-2.0, BSD or ISC only) to test cell-to-camera visibility.
  Start with a 2D grid at eye height (1.6 m for faces, 1.0 m for body) and one ray per cell
  inside the cone. Run in a worker so the page stays responsive; reuse results per camera so
  moving one camera recomputes only its cells.
- **Pixel density.** Pixels on target = horizontal resolution / horizontal width covered at
  that distance. Compare with the DORI figures (published as pixels per metre) from
  IEC 62676-4. Print the exact thresholds in the app and in the report, with the standard
  edition, rather than hide them.
- **Auto-place.** First version: rules plus a greedy cover. Place on each rule point, then
  add the camera that covers the most blind cells until the goal is met or a camera cap is
  reached. Better optimisation later (the 2024 paper's genetic-style method is a guide,
  not a dependency).
- **Wi-Fi.** Free-space path loss plus a wall loss per crossed material (a table the team
  edits, shown on screen). Say clearly it is an estimate. A later version can read a survey
  CSV to calibrate the table per project.
- **Cables.** Reuse the routing a project already has from Live Wire / Ceiling Composer if
  present; otherwise a plain shortest path along the ceiling plus a riser, with a stated
  slack factor.
- **Library.** A JSON file of camera and AP models the team owns (resolution, sensor
  size, lens range, bitrate). No vendor data is shipped from here; the team adds its own
  models from datasheets.
- **First version (smallest useful).** One floor, cameras placed by hand, line-of-sight and
  DORI heatmap, blind-spot list, camera schedule XLSX. Ship that, then auto-place, cables
  and storage, then Wi-Fi.
- **Test.** A hand-built room with one column and one camera, where the shadow is computed on
  paper and compared; the same cell counts with the rays run at two grid sizes; one real
  site, measured after install with a person walking the blind list.
- **Version.** New minor version of the engine and the app; changelog, README and guide entry
  as CLAUDE.md asks. Not done here; this repo holds ideas only.

### Future update

- Live view: send the layout to a vendor VMS format when the vendor publishes one.
- Camera POV snapshots stored per camera, so the installer sees the target view on the
  tablet and ticks "aimed" on site with a photo.
- Access control: readers, locks and door schedules from the same door list, with
  fire-exit checks (door must free-release on alarm).
- Fire alarm and PA speakers on the same panel, once Ceiling Composer's detector layout is
  shared.
- AI-assisted analytics zones (line-cross, loiter, count) drawn on the floor and checked for
  enough pixel density.
- Privacy zones: mark windows and toilets as "must not be seen" and flag any camera whose
  view reaches them, a point that comes up in authority and resident disputes.
- Construction-phase version: temporary site cameras on tower crane and hoarding, linked to
  the 4D stage, with a blind-spot check for the yard.
- Post-install loop: import the as-built camera positions and compare with plan (Scan
  Check style).
