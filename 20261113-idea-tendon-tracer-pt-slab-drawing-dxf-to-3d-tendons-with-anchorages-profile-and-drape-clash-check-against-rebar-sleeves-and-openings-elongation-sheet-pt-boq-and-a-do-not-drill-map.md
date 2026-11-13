# 20261113 Idea: Tendon Tracer — the post-tension slab drawing (DXF/PDF) turned into 3D tendons with anchorages and a real profile, checked against rebar, sleeves and openings, with an elongation sheet, PT quantities and a "do not drill" map

## Entry 1

```
date-time:   2026-11-13 (scheduled run)
idea name:   Tendon Tracer (PT slab and transfer-plate modelling: tendon layout lines and callouts
             from the specialist's DXF read into 3D tendons with ducts, live and dead-end anchorages,
             a high/low-point profile along each run, a clash check against rebar, sleeves, openings
             and embeds, an elongation and jack-force sheet, a PT bill of quantities, and a
             "do not drill" band map that other tools can read)
potential:   High for structural modellers and PT contractors on podium, parking and flat-slab jobs.
             Modelled in no BIM tool the team has today (checked the 103 ideas in this repo: PT is
             named only as something to avoid, in Hanger Hero; nothing models it). It is a gap, and
             the one clash that hurts most: a drill or core cut through a stressed tendon is a
             structural incident.
             Be honest about the "news" part: a quick search (2026-11-13) found no 2026 source
             showing a trend of BIM tendon modelling in India. Market summaries say PT demand in
             India is growing, led by transport infrastructure (market-research summary, paid report,
             weak source), and a 2016 trade journal says PT detailing in BIM is "very limited".
             So the pitch is the gap and the pain, not a viral topic. Check LinkedIn directly before
             posting anything about it.
             The wow: drop the PT contractor's tendon layout DXF on the 3D slab. Coloured tendons
             appear in the slab, each lifted and dipped along its drape, the anchorages pop on the
             edge. Click "Check": red where a tendon hits a sleeve, a drop panel edge, an opening or
             a rebar mat bar bundle. Then "Do not drill": a band map goes onto the slab and Hanger
             Hero, Sleeve Punch and the site QR card all respect it.
```

### Description

**The pain.**
- The PT specialist (the stressing contractor) sends a 2D layout: tendon lines, spacing, a callout
  such as "5T13 @ 1200" or "banded", live and dead ends, and a table with profile heights. The modeller
  gets a drawing, not a model. Today it is either skipped (the BIM model shows a slab with no tendons)
  or drawn by hand as lines in Revit, one by one. (From the team's experience; not measured here.)
- The clashes are found late: the slab rebar bottom mat, the shear-stud or drop-head rebar, the
  MEP sleeves and the cast-in inserts all sit in the same 200 to 300 mm. A tendon that cannot move
  cannot be simply shifted on site.
- After the pour, nobody has a map of where the tendons are. Core drilling, hangers, anchors and
  fit-out use a GPR scan or a paper copy of the layout.
- Quantities (strand mass, ducts, anchorages, stressing count) are counted from the layout by hand and
  reconciled after.

**What exists today (checked 2026-11-13; not exhaustive).**
- Analysis and detailing packages (ADAPT-Builder, RAM Concept, SOFiSTiK, Tendon-detail add-ons for
  Revit) are paid, built for the PT designer, and use their own models. SOFiSTiK's 2026 documentation
  shows PT slabs modelled in Revit with tendon projections on an analytical panel, then analysed in its
  own desktop tool (vendor tutorial). Revit itself has no PT tendon element: people use rebar,
  lines or generic families.
- Hanger and drill-safety tools use GPR scans after the fact. None read the layout before the pour.
- In this repo: Slab Sketch makes slab rebar. Hanger Hero and Sleeve Punch avoid PT bands by a region you
  draw. Layer Whisperer maps DXF layers. Plan Tracer reads scanned plans. **None reads a PT layout or
  builds tendons.** Tendon Tracer is the missing source of the "PT band" those tools ask for.
- Do not claim it is unique to the market: PT detailing software exists. What is not (as far as I know)
  is a free, browser-only step from the PT contractor's DXF to checked 3D tendons plus a drill map
  that stays on the device. Confirm before marketing.

**The idea in one line.** Drop the PT drawing on the slab; get tendons with a real profile, clash
results, an elongation sheet, PT quantities and a map of where not to drill.

### How it works (user view)

1. **Drop** the PT contractor's DXF (or DWG through the CAD engine) onto the slab or floor level.
   The Layer Whisperer step maps layers (tendon lines, anchorages, notes, profile table). A PDF goes
   through Plan Tracer first.
2. **Read** the layout. Tendon lines (polylines or lines) become runs; the callouts ("4T13 banded",
   "@ 1500") become strand count, duct type and spacing; end symbols become live or dead anchorages.
   A table of profile heights (high at supports, low at midspan, drape per span) is read from text or
   asked for in a small form when missing. What is not found is listed, not guessed.
3. **Build** the 3D. Each tendon gets a parabolic or reverse-parabolic profile between its high and
   low points, and a duct and strand as IFC (IfcTendon, IfcTendonAnchor, IfcTendonConduit in IFC4x3)
   so Revit and others open it. Anchorages sit in the edge with their blister or pocket.
4. **Check.** The same engine as Joint Congestion Doctor tests tendon against: slab bottom and top
   bars, sleeves and embeds, openings and edges (minimum edge distance and spacing the user sets
   from the project's PT specification), shear heads, and the cover at each point. Each finding is
   pinned and goes to BCF.
5. **Sheets and numbers.**
   - **Elongation sheet:** for each tendon, length, friction and wobble losses and the expected
     elongation at the jacking force, with the project's own constants (the tool does not invent
     them; it reads them from the specification form). A ±7% acceptance band is shown as a
     placeholder; the user's code (IS 1343 or IS 18001 style, or the specialist's own) decides.
     This is a check sheet for the stressing crew, not a design of the PT.
   - **PT BOQ:** strand mass, duct length, anchorage count by type, stressing operations.
   - **Do-not-drill map:** a plan with bands and a clear-zone layer. Exports as a DXF layer for the
     site and as the PT band for Hanger Hero, Sleeve Punch and Site QR Pass.
6. **Compare on revision.** A new PT layout from the specialist goes through Change Radar: tendons added,
   moved, or changed in count, and the BOQ and clash delta.

### Why it fits cad2bim

- The DXF reading and layer mapping already exist (CAD engine, Layer Whisperer). Tendon polylines with
  heights are plain geometry.
- IFC writing exists (`packages/engine`, IFC writer parameters). The rebar clash engine and the
  slab rebar from Slab Sketch give the objects to check against.
- Nothing leaves the device: PT layouts are the specialist's commercial drawings. A browser tool that
  never uploads them is easy to accept.
- Small enough to start: one slab, parabolic profiles, one code form.

### Implementation

**Phase 1 (about 2 weeks, estimate).** Read tendon lines and anchorage blocks from a DXF on one level,
callouts to strand count and spacing, flat profile at a user height. Draw in 3D. Anchor count and strand
BOQ. No clash check yet.
**Phase 2 (about 2 weeks).** Profile from a high/low table, parabolic interpolation, IFC export.
Clash check against slab rebar, sleeves, openings and edges. BCF.
**Phase 3 (about 2 weeks).** Elongation sheet from a specification form. Do-not-drill map and its DXF
layer. Hand-off to Hanger Hero, Sleeve Punch and Site QR Pass.
**Phase 4.** Revision compare through Change Radar. Banded and distributed tendon rules, transfer
girders and bonded beams.

Where it goes: the CAD core reads the layout (`packages/cad-engine`); the geometry and clash code
sits in the pipeline or engine package beside the rebar clash code; the app adds a "PT" panel in the
Structure ribbon. This note only proposes; nothing in the source was changed.

**Risks.**
- PT layouts differ by contractor. Start with two or three real layouts from the team, kept out of git.
- It must not be read as PT design. The tool models, checks clashes and lists numbers from the
  specialist's design, and says so on every sheet. The PT designer signs the layout.
- Losses and elongation formulas depend on the code and the system (bonded or unbonded). Take the
  constants from the user, never default them silently.

### Future update

- Read the PT designer's analysis export (ADAPT, RAM Concept CSV) for exact profiles and forces.
- Tendon "as-stressed" record: the site enters measured elongation; out-of-band tendons go red and into
  the handover pack (Handover Hub).
- GPR or phone-scan overlay: compare the scanned post-pour layout with the model before a core cut
  (Scan Check).
- 4D: stressing as a task with a strike-ready rule after the cube test (Cube Clock).
- Bridge girder and cable-stayed tendon support for infrastructure jobs.

### Sources

- [SOFiSTiK 2026 PT slab tutorial (vendor)](https://docs.sofistik.com/2026/en/tutorials/building/pt-slab/pt-slab.html)
- [India post-tensioning systems market (paid report summary, weak source)](https://www.nexdigm.com/market-research/report-store/india-post-tensioning-systems-market/)
- [Baxi, PTI reprint (2016): BIM and PT](https://www.post-tensioning.org/Portals/13/Files/PDFs/Publications/Reprints/2016%20December/Baxi_reprint.pdf)
- [IS 8543:2019 summary (third party; check the standard)](https://infralens.in/code/IS-8543-2019)
