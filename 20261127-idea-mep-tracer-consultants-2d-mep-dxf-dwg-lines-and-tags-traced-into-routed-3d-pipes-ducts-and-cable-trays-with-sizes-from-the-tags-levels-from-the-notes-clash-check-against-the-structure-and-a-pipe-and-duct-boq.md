# 20261127 Idea: MEP Tracer — the consultant's 2D MEP DXF/DWG (lines and tags) traced into routed 3D pipes, ducts and cable trays, with sizes from the tags, levels from the notes, a clash check against the structure, and a BOQ

## Entry 1

```
date-time:   2026-11-27 (scheduled run)
idea name:   MEP Tracer — "The MEP consultant sends 2D lines and tags. Get a routed 3D MEP model in minutes, not weeks."
             Drop the MEP consultant's DXF or DWG (plumbing, drainage, HVAC ducts, fire lines, cable trays, conduits).
             MEP Tracer reads the lines on each service layer, reads the nearby size and level tags ("150 dia
             SWR", "600x300 duct, BOD +2.80", "CT 300x50"), joins the lines into runs, adds fittings at bends,
             tees and risers, and builds 3D MEP as IFC (IfcPipeSegment, IfcDuctSegment, IfcCableCarrierSegment,
             with fittings). The model is then checked against the structure already in cad2bim (beams, slabs,
             walls) and a pipe, duct and tray BOQ is written, each quantity clickable to its 3D proof.
potential:   High. Every Indian project still receives MEP as 2D drawings. The BIM team either models MEP by hand in
             Revit (days per floor) or skips it, and then coordination happens on site, with core-cutting and
             "duct hits the beam" as the result. The estimates below are mine, not measured:
             - Today: 2 to 4 days per typical floor to model plumbing, HVAC and electrical trays by hand from
               2D, plus a rework round after every MEP revision.
             - With MEP Tracer: first routed model in minutes, 1 to 2 hours to review the flagged lines, a
               revision re-trace in seconds with the same IFC ids kept.
             Why it gives the "wow": the flat DXF grows pipes, ducts and trays in 3D in front of the user, runs
             clashing with beams turn red, and "duct at +2.80 vs beam soffit +2.65" shows as a number on screen.
             That is easy to understand in a 15-second reel and needs no IFC knowledge.
             Fits the repo: it is the same method as the DXF to 3D pipeline (read layers and lines, read tags,
             build IFC) applied to services instead of structure, and the roadmap item "Every discipline in one
             project" already plans MEP in the same .c2b.
             Market: to my knowledge (not verified for this run), tools that turn 2D MEP into 3D are paid plug-ins
             for Revit or AutoCAD MEP, per seat, and tuned to one drawing standard. A free, in-browser tool that
             keeps files on the device and works with any layer scheme (by mapping) is not one I know of.
             Trending: "AI for drawings to BIM" and "clash detection before site" are repeated LinkedIn
             topics, and scan-to-BIM and takeoff automation are active in 2026 vendor news. Treat this as a
             signal of interest, not a measured trend.
```

### Description

**The pain (day to day of a BIM modeller and MEP coordinator).**
- The architect and structure are in the model. MEP arrives as DXF/DWG: lines on layers like `P-SWR`, `HVAC-SUPPLY`,
  `E-TRAY`, with text next to them. Nobody models it, because it is slow, and the model goes to site without it.
- On site, a duct or a drain meets a beam. The fix is a core cut, a drop, or a re-route, decided in a hurry.
- When the MEP revision arrives, the hand model is out of date.
- Quantities (metres of pipe by diameter, duct area by size, tray length) are measured by a QS from the PDF.

**What MEP Tracer does, in five steps.**

1. **Layer mapping.** On load it lists the drawing's layers, guesses the service by name and by what is on them
   ("SWR", "CHW", "supply", "tray") and shows the guess for the user to fix in one table. The mapping is saved per
   consultant and reused next time (same idea as *Layer Whisperer*, for services).
2. **Lines to runs.** Lines and polylines on a service layer are joined into runs. Gaps below a tolerance are
   closed; tees, crosses, elbows and reducers are found where lines meet. Vertical risers are recognised from riser
   symbols or tags ("RISER", a circle with a tag, a block).
3. **Tags to size and level.** Text and attributes near a run give its size ("150 dia", "600x300"), service and
   level ("BOD +2.80", "IL -0.45", "at 2.4 m AFFL"). Each run gets its size and level with the *source text* kept.
   Runs without a tag get the schedule's or the drawing note's default (for example "all drains 100 dia unless noted")
   and are marked **assumed**.
4. **Build and route in 3D.** Segments become IFC MEP segments with fittings, at the stated levels. Where a level
   is given only at one end, slope is applied for gravity lines (drain fall, for example 1:60, set by the user).
   Runs with no level are placed under the slab at a user rule ("duct: 150 mm below beam soffit") and marked **assumed**.
5. **Check and count.**
   - Clash against the structure in the project (beams, slabs, walls, columns): hard clash and clearance
     (insulation + 50 mm). Result: a list with element, level, size, and "needs: sleeve / drop / re-route".
   - Clash between services (duct vs pipe vs tray).
   - A **BOQ** by service, size and level: metres, fittings count, duct area (m²), insulation area, supports
     estimate, each quantity clicking through to the 3D elements and to the source lines.

**What the user sees.** A "Coverage" view of the drawing: lines traced (green), lines skipped with the reason (grey,
click to see it: "no size tag", "unknown layer", "ends do not meet"), and tags unused. The aim is that no line is
silently ignored; an unused tag is as visible as a traced line. A **confidence** colour on each run: traced from
tag (green), assumed default (amber), guessed (red).

**Revision.** A new MEP DXF re-traces and shows *new runs*, *moved runs*, *size changes* and *removed runs* with a
quantity delta; ids of unchanged runs are kept, so BCF issues and notes keep their pins.

### Implementation

Browser only, files stay on the device, no GPL code, no change to existing source outside the new module. This
reads and writes through the one cad2bim API.

1. **Reading the drawing.** Reuse the CAD core's reading (`packages/cad-engine`) for lines, polylines, arcs, text,
   attributes and blocks. A DWG already goes through the Rust CAD core; no new reader.
2. **A new MEP stage in the Rust pipeline (`packages/pipeline`),** beside the structural one, with its own input
   type like CH v2's `v2::Input`; never change v1's `Entity` or `analyze`. Pure functions, unit tests on small
   synthetic drawings with known answers:
   - `join_runs` (tolerance, snapping, topology of tees and bends),
   - `read_tags` (nearest tag by distance and direction, size grammar: `150 dia`, `DN150`, `600x300`, `Ø100`),
   - `levels` (BOD, TOD, IL, AFFL, with slope for gravity lines),
   - `fittings` (elbow, tee, reducer from topology).
3. **Output.** IFC writer: `IfcPipeSegment`, `IfcDuctSegment`, `IfcCableCarrierSegment` and the fitting classes,
   with a Pset carrying service, size, level source (tag or assumed) and the source layer and line ids. Stable ids
   from the source entity so a revision keeps them (same approach as the existing Revision of the cad2bim tab).
4. **Clash.** Reuse the engine's QA/clash code and the model's spatial index; add a clearance parameter per service.
   Results go to the existing issues list and can export as BCF.
5. **BOQ.** Reuse the BOQ module; add rows by service and size. Duct area and insulation area by formula, shown on
   every row.
6. **UI.** One new group in the cad2bim tab: *Services*: Layer map, Trace, Coverage, Clash, BOQ. Design system
   components only; colours and spacing from tokens. Defaults documented in a design note, one decisions row.
7. **Agent tool.** Add `trace_mep`, `mep_coverage` and `mep_clashes` to the MCP catalogue
   (`packages/mcp/src/tools.ts`), so an agent can answer "which drains hit a beam on level 3?".
8. **Testing.** Golden-file tests on synthetic drawings; two or three real consultant drawings in a scratch folder,
   never in git.
9. **Delivery order (small steps), estimates not measured.**
   - Step 1: layer map + joined runs + coverage view (still 2D). About a week.
   - Step 2: tags to size and level + 3D pipes and ducts. About two weeks.
   - Step 3: fittings and risers. About a week.
   - Step 4: structure clash + clearance list + BCF. About a week.
   - Step 5: BOQ and revision deltas. About a week.

**Risks and honest limits.**
- 2D drawings are ambiguous. Anything the tags do not state is *assumed*, shown amber, and listed; the tool must never
  look more certain than the drawing. The result is a coordination model, not a fabrication model.
- Consultants draw differently. Mapping is per consultant and learned; the first drawing from a new consultant
  takes longer. Say so in the Coverage view rather than hiding lines.
- Not an MEP design check. It does not size pipes or ducts, and does not replace the consultant's responsibility.
- Complex equipment (AHUs, pumps, chillers) comes in as simple boxes from block names, not full families.

### Future update

- **Equipment from blocks:** map block names to IFC equipment types with simple 3D boxes and connection points.
- **Auto re-route:** for each clash, propose a drop, a shift or a sleeve that clears the structure, show the
  cost and headroom effect, and let the user accept (links to *Sleeve Punch*, *Flush Fit*).
- **Hangers and supports:** feed traced runs to *Hanger Hero* for support layout and the support BOQ.
- **Fire and drainage checks:** hydraulic and fall checks on traced runs (links to *Fire Line*, *Drain Line*).
- **4D:** services get tasks (first fix, second fix, testing), so a delayed slab shows what MEP is blocked
  (links to *Ready Board*).
- **PDF-only MEP:** accept a PDF through the *Plan Tracer* path when no DXF exists, with lower confidence.
- **Back to Revit:** send the traced services to the Revit add-in as native MEP elements once Sync with Revit ships.
- **Team memory:** keep each consultant's mapping, tag habits and default levels so the second project is one click.
- **Agent flow:** "trace the services on level 2 and list every beam they hit", answered with a 3D proof.
