# 20261031 Idea: Demo Map — paint the existing building Retain / Remove / Modify / New; the tool flags members left hanging, sequences the strip-out top-down, counts debris in tonnes and truck trips, credits the scrap, lists what can be reused, and draws the Existing / Demolition / Proposed sheets

## Entry 1

```
date-time:   2026-10-31 (scheduled run, UTC)
idea name:   Demo Map (Brownfield Renovation, Demolition and Retrofit Planner)
potential:   High. A large share of Indian work is now not a new plot but an old building: redevelopment,
             floor additions, change of use, structural-audit repairs, shop and office fit-outs, hospital
             and factory expansions. In every one of these the first question is "what stays, what goes,
             what do we cut, and what holds up what while we do it?". Today the answer lives in a red-pen
             markup on a PDF, a separate Excel for demolition quantity, and the site engineer's memory.
             The numbers are all derivable from a model: volume to tonnes, tonnes to truck trips, steel
             to scrap credit, retained-versus-new quantity for the BOQ. Revit has Phases (Created /
             Demolished) but gives no demolition quantity in tonnes, no debris trips, no "this retained
             beam just lost its column" warning and no sequence. Paid options are 4D suites (per seat per
             year) or material-passport platforms aimed at European buildings. I did not find a free,
             browser-based tool that joins retain/remove marking, a support check, a sequence, debris and
             the three phase drawings; this is a search result, not proof, so check again before using
             "nothing like it exists" in marketing.
             Estimated saving (an estimate, not measured): 1-2 days per renovation job on phase drawings
             and the demolition BOQ, and fewer "we cut the wrong beam" moments on site.
```

### Description

**The pain (daily life of a modeller / site engineer).**
An old building comes with a scan of drawings, a few site photos, and the owner's list: "remove these partitions,
open this slab for a new staircase, add a floor, strengthen those columns". The modeller then
(1) rebuilds the existing structure, (2) colours things red and green in a PDF for the demolition drawing,
(3) counts brickwork and concrete to be removed by hand, (4) asks the structural consultant "can I take this
wall out?", (5) guesses how many tipper trips the debris needs, and (6) prepares three plan sets: Existing,
Demolition, Proposed. Each step is manual and none of them talk to each other. When the owner changes his mind
about one wall, all six redo.

**Why now (trending).** Renovation, adaptive reuse and "the greenest building is the one already standing"
dominate AEC posts, and the reuse of materials is moving into rules and research. A 2026 Chalmers study of a
Swedish renovation found BIM supported reuse from early design to disassembly and reinstallation, with large
embodied-carbon reductions and some reuse cases cost-competitive with new material
([Chalmers](https://research.chalmers.se/en/publication/554278)). Material passports for in-place materials,
read by phone from a QR code and fed back into BIM, are being proposed
([Building Design](https://bdonline.co.uk/news/orms-proposes-live-material-passports-to-reduce-construction-waste/5111828.article)),
and Madaster and Fimble link BIM models to passports for use through renovation and demolition
([Madaster](https://madaster.com/inspiration/fimble-new-partner-of-madaster/)). One Italian report says rules
effective February 2026 require a deconstruction plan with 70% of weight recoverable on public projects
([LavoriPubblici](https://www.lavoripubblici.it/news/material-passport-materiali-edifici-infrastrutture-38816));
I did not check that against the decree, so treat it as unverified. These sources are mostly European and
research-led: they show direction, not Indian market adoption. The Indian pull is practical: redevelopment,
structural-audit repair orders and fit-outs, where nobody has time for a research-grade tool.

**The idea.** Open the existing building in cad2bim (from an IFC, from old drawings through the DXF-to-3D
pipeline, or traced from a scanned PDF). Press **Demo Map**. Click or box-select elements and **paint** them:

| Paint | Meaning | Colour (existing diff tokens) |
|---|---|---|
| Retain | stays untouched | unchanged (grey) |
| Remove | demolished | removed (red, ghosted in the later phase) |
| Modify | cut, core-drilled, jacketed, opened | changed (amber) |
| New | added in this job | added (green) |
| Temporary | props, shoring, screens | its own outline colour |

Painting is the only input. Everything below is computed from it, live, and updates when you re-paint.

**What it computes.**

1. **Orphan check (the wow).** Remove a column and every beam it carried glows red: "B14 lost its support at
   the right end". Remove a wall and the slab edge, the parapet or the retained wall above it is flagged. A
   retained element with no remaining load path to a footing is called an **orphan**. The check follows the
   same support logic as the Load Path X-Ray idea (what rests on what), animated: the load lines that used to
   run through the removed member break and stop in mid-air. For each orphan it suggests the fix as a *prompt
   for the engineer*, not a design: "needs temporary props" or "needs a new transfer beam" or "re-paint as
   Remove". Props painted as Temporary clear the flag during the works and are listed with their duration.
2. **Sequence.** Default order is top-down and outside-in by floor and zone, with propping before cutting and
   non-structural stripping (partitions, finishes, services) before structural cutting. The user drags the
   steps; the orphan check re-runs for every step, so "slab cut before the props go in" shows up as a warning on
   that step. The 4D timeline (roadmap item) plays it as a replay, floor by floor.
3. **Quantity and debris.** Per paint class and per floor: volume of concrete, brickwork, plaster, tiles, steel
   sections and so on from the model; volume to tonnes from an editable density table (RCC, brick, block, stone,
   steel, timber); bulking factor to loose volume; tipper capacity to **truck trips and days**. Concrete and
   brick to a crusher or filling yard, steel to scrap, hazardous items (asbestos sheets, old tanks, lead paint)
   to a separate list with a "specialist required" mark. Densities, bulking and tipper size are *assumptions you
   edit*, shown next to every number.
4. **Scrap credit.** Steel in removed RCC, from the rebar quantities when the model has them, otherwise from an
   editable kg/m3 default by member type. Result: tonnes of rebar recovered, today's scrap rate you type in, rupees
   back. The same sheet nets it against the demolition cost.
5. **Reuse list.** Doors, windows, sanitary ware, steel sections, flooring, bricks that can go back in or be sold.
   Each reusable item gets a row with count, condition (pick from a list) and where it goes. One click makes
   QR tags for them (pairs with Handover Hub), so the "material passport" idea works at the level a small
   contractor actually uses: a list and a sticker.
6. **Openings and cuts.** Draw a new stair opening or lift shaft through a retained slab. The tool lists the
   retained beams and columns it touches, the rebar that will be cut (if the model has bars), and the strip
   to be propped. It marks "scan before cutting" on every core and cut, a reminder for a rebar scanner, not a
   replacement for one.
7. **BOQ split.** Demolition (remove), alteration (modify) and new work in one BOQ, priced from the rate table
   (Rate Rocket idea), with the **retained** value shown separately so the owner sees what the job really adds.
8. **Three drawing sets.** Existing, Demolition (red dashed, hatched) and Proposed plans per floor, with the
   same title block and a legend, from the paint alone (pairs with Drawing Autopilot). One re-paint redraws all
   three. A single **demolition permit pack** (plans, quantities, sequence, debris plan) as one PDF.

**Unique angle.** It joins four things that live in four tools today: the phase marking (Revit), the
load-path logic (a structural program), the 4D replay (a planning suite) and the debris/passport sheet (a
spreadsheet or a European platform). The fast, visible part is the orphan glow while painting; that is the
screenshot that gets shared.

### Implementation

Built from what cad2bim has or has planned. Nothing here needs code changed in cad2bim's source today; this note
only describes the idea.

| Piece | Uses | New work |
|---|---|---|
| Existing building | IFC reading in the engine; DXF-to-3D pipeline; Plan Tracer idea for scanned PDFs; Scan Check idea for point clouds | none for IFC; "existing" model flag |
| Paint state | reserved `diff-added`, `diff-changed`, `diff-removed`, `diff-unchanged` tokens; per-element colour overrides in the viewer (also used by the revision timeline design) | a `phase` property per element, saved in the project file |
| IFC mapping | IFC `Status` in the `Pset_*Common` sets has the values NEW, EXISTING, DEMOLISH, TEMPORARY; write them on export so other tools read the same marks | mapping table, plus an export option |
| Revit | the add-in's bridge and the planned *Sync with Revit*: map Retain/Remove/New to Revit "Phase Created / Phase Demolished" | add-in mapping, a test on a sample model |
| Support logic | the "what rests on what" graph that Load Path X-Ray needs (slab to beam to column to footing) | build once, reuse for both ideas |
| Sequence | the planned 4D timeline | steps with a dependency rule: props before cut, strip before structure |
| Quantities | BOQ and BBS in the engine; Schedules (roadmap) | density, bulking, tipper tables as editable rows with an owner and a date |
| Drawings | the planned Sheets and PDF; Drawing Autopilot idea | phase-aware line styles and legend |
| Replay and reel | planned *Camera moves and video*; Model Reel idea | a "wrecking" scene: removed elements fade red, dust-free, 20 seconds |

**Phases.**
- *Phase 1 (1-2 weeks of work, my estimate):* paint tool, five colours, saved with the project, live quantity per
  class in tonnes with editable densities, and the three-phase colour view in 3D. Already a useful demo.
- *Phase 2:* the orphan check on columns, beams, walls and slabs; Temporary props clearing flags; the debris and
  truck-trip sheet; scrap credit.
- *Phase 3:* the sequence editor with the 4D replay; the opening-through-slab tool; Existing / Demolition /
  Proposed drawing sets; the demolition permit PDF; the reuse list with QR tags; Revit and IFC phase write-back.

**Honest risks.**
- The orphan check is a **geometry and support screening aid**. It does not know about hidden beams, slab
  continuity, masonry infill acting as bracing, or the real load. A structural engineer decides; the tool
  says so on every screen and in every export, and never writes "safe".
- Old buildings are badly drawn and the existing model may be wrong. Show a **confidence** per element (from
  IFC, from old DWG, from a scan, from a guess) and let the user mark "to verify on site".
- Densities, bulking, tipper size, scrap rates and dump distances vary by city and site: all editable, none
  hard-coded as truth.
- Demolition has safety law: a permit pack is a draft for the contractor and the authority; the tool does not
  replace the method statement or a licensed demolition engineer.
- Quantities from an IFC of an old building depend on how it was modelled; a missing slab is a missing tonne.
  Run the Ratio Radar check first.

### Future update

- **Hindrance and delay log:** when the owner holds back a wall, mark it "Held" with a date and reason; the
  sequence re-plans around it and the log becomes the evidence for an extension-of-time request.
- **Asbestos and hazard layer:** mark hazard zones from a survey sheet; they show as a separate colour and
  force a "specialist required" step ahead of any cutting there.
- **Condition survey import:** combine with Audit Eye (crack and spall finder) so a repair finding becomes a
  Modify element with its repair BOQ line in the same project.
- **Neighbour and noise zones:** a radius around the site with working-hour limits per activity, so noisy
  breaking is scheduled away from the clinic next door.
- **Embodied carbon of reuse:** show carbon saved by retaining and reusing against a demolish-and-rebuild
  option (pairs with Carbon Lens), a chart that sells itself in a client meeting.
- **Option compare:** "retain the old block" versus "demolish all" side by side, with cost, time, debris and
  carbon, from the planned versions-and-history feature.
- **Phased occupancy:** "floors 1-3 stay occupied while 4-6 are stripped": mark occupied zones; the tool
  flags any step that cuts their escape route or services (pairs with Exit Map).
- **Reel export:** the strip-out replay as a short vertical video for LinkedIn and Instagram, "this 1980s block
  became this", in one click.
