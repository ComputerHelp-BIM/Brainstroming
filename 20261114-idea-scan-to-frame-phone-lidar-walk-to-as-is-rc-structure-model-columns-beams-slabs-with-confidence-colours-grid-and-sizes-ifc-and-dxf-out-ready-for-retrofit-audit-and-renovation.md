# 20261114 Idea: Scan-to-Frame — a phone LiDAR walk becomes an as-is RC structure model (grid, columns, beams, slabs) with a confidence colour on every member, IFC + DXF out, ready for retrofit, audit and renovation

## Entry 1

```
date-time:   2026-11-14 (scheduled run)
idea name:   Scan-to-Frame (drop in the scan an ordinary phone made — PLY / OBJ / GLB / LAS / E57 from Polycam,
             Scaniverse, 3D Scanner App, or the walls-and-doors room file from an iPhone RoomPlan app — and the
             browser rebuilds the RC *structure* behind the plaster: grid lines, columns, beams, slab levels and
             thickness, each member coloured by how sure the fit is; the engineer fixes the amber ones in a few
             clicks; out comes an as-is IFC + a plan DXF + a member list that Demo Map, Audit Eye, Gap Guard and
             Load Path X-Ray can start from)
potential:   High and daily-life real in India and any market with old buildings. The most common job that has
             no drawings: society structural audit, floor addition, change of use, a shop turned into a clinic,
             a mall unit fit-out, a factory shed extension. Today the "as-is model" is a surveyor with a tape for
             two days plus a junior drawing it, or a paid scan-to-BIM service (days of turnaround, priced per
             square metre — my estimate of how this is billed, not a measured quote). Saves roughly 1–3 days per
             floor of site measuring and redrawing: an ESTIMATE from how this work usually runs; calibrate on one
             real building before quoting it to anyone.
             Why it is a real gap (checked 2026-11-14, one standard search each, NOT exhaustive):
               - Paid desktop scan-to-BIM: Leica Cyclone 3DR "Scan to Steel" (steel members), rmDATA Reality3D
                 (a Revit add-on, listed at about EUR 90 per user per month plus a EUR 290 one-time setup on its
                 page — confirm before relying on it), EdgeWise (AI to Revit, price not found). Heavy software,
                 a surveyor-grade cloud, a Revit seat.
               - Phone apps built on Apple RoomPlan (RoomScan Pro, Metaroom, BeaverPlot) export IFC/DXF, but they
                 model *rooms*: walls, doors, windows, furniture. They do not give you the concrete column at C-4
                 or the beam drop under the slab. Some IFC exports sit behind a subscription (a reseller guide's
                 claim; may be outdated).
               - Free/open: Cloud2BIM (open-source Python, point cloud to IFC) and the ODA Scan-to-BIM SDK (beta
                 in April 2026) are pipelines for developers; neither is a browser tool aimed at RC columns,
                 beams and slab thickness with a "how sure" colour.
               - In this repo: Scan Check compares a scan to a model that **already exists**; Demo Map, Gap Guard
                 and Audit Eye all **start from** an existing model or an old drawing. Nobody makes the model
                 when there is no drawing. This idea is the missing first step for all four.
             Trend honesty: phone LiDAR to BIM is shipping in several apps, but my search found no 2026 proof
             that it is trending on LinkedIn or Instagram; Scan Check's note calls phone scanning a standing
             reel topic. Check LinkedIn before posting a claim. Accuracy claims of "1–3 cm indoors" are second-
             hand (reviews citing a vendor-adjacent test); measure it on your own phone before promising anything.
             The wow: scan a flat on a phone (3 minutes), drop the file on cad2bim. The point cloud fades to
             ghost dots; a grid snaps into place; columns grow out of the dots one by one in green; beams drop
             in under the slab; a counter reads "38 columns, 71 beams, 4 slabs; 9 members to check". Tap an amber
             column: the scan dots behind it light up and a card says "230 x 450, hidden by plaster on two faces,
             sure about one". Press "Plaster off": the model strips back to the structure. 15 seconds, perfect
             for a reel; and nothing was uploaded.
```

### Description

**The pain (a BIM modeller / structural engineer with a building and no drawings).**
- The society says "our building is 30 years old, check if we can add a floor". The architect's plan is lost or
  never matched what was built. Nobody knows the column sizes, the beam depths or the slab thickness.
- The surveyor measures with tape and laser; a junior redraws it. Columns are plastered, so the measured size is
  the plastered size (a 230 mm column reads 265 mm). Errors of this kind go straight into the load check.
- A scan service returns a model in days and charges by area. For a 4-floor society that is a fair cost for a
  job that may end in "no, you cannot add a floor".
- The free phone scan exists in the engineer's pocket, but it is a "pretty 3D toy": nobody turns it into members
  an engineer can use.

**The idea in one line.** Walk with a phone, get the skeleton of the building, and be told honestly where the
model is a guess.

### How it works (user view)

1. **Drop the scan.** PLY / OBJ / GLB / LAS / LAZ / E57 (the CAD and IFC readers and a point-cloud reader are on the
   formats list in `docs/design/formats-and-exchange.md`; read there as a reference format, not yet built). Heavy
   clouds are thinned in a Web Worker and shown as tiles, as Scan Check proposes. Nothing leaves the device.
2. **Find the floors.** Horizontal planes by height histogram (gravity from the scan's up axis): floor slabs and
   soffits show as peaks. Storey heights and slab-to-slab clear height come out; a stair or ramp shows as a tilted
   plane and is left alone.
3. **Find the grid.** Wall and column faces vote for two dominant directions (usually 90 degrees apart, but a skewed
   plot gets its own angle). Parallel lines that repeat are offered as a grid. The engineer drags a label or
   renames. A grid is only a suggestion until accepted.
4. **Grow columns.** Vertical features that repeat between the floor and the soffit, with a rectangular or round
   section by a robust fit, at a grid crossing or near it. Where plaster hides a face, the model keeps the visible
   faces and states the plaster allowance (the engineer enters 12–25 mm; the default is shown as a default).
5. **Drop beams.** Under each slab soffit, a downstand strip with a constant depth between two columns or a wall
   becomes a beam: width, depth below soffit, span. Flush or hidden beams (false ceiling) are the weak spot: the
   model says "ceiling hides this" and offers a ceiling-hatch scan or a hammer-tap thickness entry.
6. **Slabs.** Slab top level from the floor plane; thickness from a measured edge, a stair-opening or sunken-toilet
   cut, or typed in. If no edge is visible, thickness is left blank with an amber flag. It is never invented.
7. **Confidence colour on everything.** Green = measured with support on enough faces; amber = partly hidden or
   inferred from repetition ("the other 11 columns on this line are 230 x 450"); red = guess. The colour is shown
   in 3D and written into the IFC as a property, so the next tool knows what to trust.
8. **Fix in a few clicks.** Click an amber member: the scan dots behind it light up, nudge or retype, and the
   neighbours that were inferred from it update.
9. **Output.** As-is IFC (IfcColumn, IfcBeam, IfcSlab, storeys and grids, with the confidence and the plaster
   allowance as properties), a plan DXF per floor on the CH layer names, and a member list (mark, size, level,
   confidence) for Excel. Hand-off buttons: *Open in Demo Map*, *Start an audit in Audit Eye*, *Check expansion
   joints in Gap Guard*, *Run a quick load takedown in Load Path X-Ray*.

### Why it fits cad2bim

- The CAD core already finds closed loops, repeated blocks and tables in DXF; the *grid-and-repeat* thinking in
  the pipeline is the same kind of logic, applied to scan planes instead of drawing lines.
- The IFC writer and the viewer already draw columns, beams and slabs with properties; the confidence value is one
  more property set.
- Scan alignment, voxel thinning and tiling are proposed for Scan Check, so they are shared work, not new.
- Files stay on the device. A client's building scan is private data, so "no upload" is a strong selling line, and
  it matches the product rule "nothing leaves the device unasked".
- It feeds four existing ideas at once, so each of them gets a door in.

### Implementation

**Phase 1 (about 2 weeks, estimate).** PLY / OBJ / GLB reader, up-axis and floor-plane finder, storey heights, grid
suggestion from wall and column faces, a plan view of the cloud with the grid. Output: grid DXF and storey list.
**Phase 2 (about 3 weeks).** Column fitting (rectangle and circle) with plaster allowance and confidence; repetition
inference; the amber/green/red view; click-to-fix. Output: as-is IFC of columns and slabs, member list.
**Phase 3 (about 3 weeks).** Beam detection under soffits, hidden-beam flags, slab thickness from openings or typed,
plan DXF per floor. Hand-offs to Demo Map, Audit Eye, Gap Guard, Load Path X-Ray.
**Phase 4.** RoomPlan room file as a second input (walls fix the grid faster); LAS / E57; a *verify* mode that runs
Scan Check against the model it just built to show the residual per member.

Where it goes: the point-cloud reader and plane/RANSAC fitting as a worker beside the CAD engine (`packages/cad-engine`),
member logic in the pipeline crate (`packages/pipeline`) so it is testable without a browser, the panel under the
Structure ribbon. This note only proposes; nothing in the source was changed.

**Risks, said plainly.**
- A phone scan sees the plastered surface. The hidden concrete size is the plaster allowance away; the tool must
  never hide that. Columns inside masonry walls, and beams above a false ceiling, are often invisible. These stay
  amber or red, and the report says "needs a hammer tap, a cover meter or a core cut".
- This produces **geometry for planning, not a structural certificate**. Rebar, concrete grade and condition need
  site tests (cover meter, rebound hammer, cores); Audit Eye plans those. The IFC and the report carry that sentence.
- Phone LiDAR drifts over long walks and is short-range. Scan one floor at a time, close the loop back at the start,
  and use a known distance (a tape between two columns) as a scale check; the tool shows the closing error.
- The first version should work on regular framed buildings. Flat slabs, shear walls, ribbed slabs and steel will
  come later; unsupported cases are listed, never silently modelled as something else.
- Test with the team's own scans kept out of git (client buildings).

### Future update

- Rebar hints from a cover-meter CSV: bar positions along a measured line pinned to the member.
- Compare two scans of the same building: before and after (a crack gauge, a retrofit jacket added).
- Join storeys scanned on different days by shared stair or lift core.
- MEP and equipment as a second layer (pipes, ducts, DB boards) for Hanger Hero and Sleeve Punch.
- AR mode: hold a phone in the room and see the model's hidden columns and beams overlaid (WebXR) for a client
  walkthrough.
- Calibrate against the team's measured buildings: a one-page "scan vs tape" accuracy table per phone model.
- Feeds Scope Scout: a scan gives the member count for a quote before anyone draws a line.

### Sources

- [Leica Cyclone 3DR, Scan to Steel](https://leica-geosystems.com/products/laser-scanners/software/leica-cyclone/leica-cyclone-3dr/scan-to-steel)
- [rmDATA Reality3D pricing page](https://www.rmdatagroup.com/en/software/rmdata-reality3d/pricing-model/)
- [8 Best Scan to BIM Software for AEC Teams, Matterport (mentions EdgeWise)](https://matterport.com/blog/best-scan-to-bim-software)
- [Cloud2BIM: open-source point cloud to IFC pipeline](https://arxiv.org/html/2503.11498v2)
- [ODA ScanToBIM technical deep dive (April 2026)](https://www.opendesign.com/files/2026-04/ODA_ScanToBIM_Technical_Deep_Dive.pdf)
- [Scan-to-BIM on Your Phone, LiDAR News](https://lidarnews.com/smartphone-lidar-scan-to-bim/)
- [RoomScan Pro review](https://illustrarch.com/articles/design-softwares/110334-roomscan-pro-review.html)
- No LinkedIn or Instagram post could be checked from here; do that by hand before using "trending" in a post.
