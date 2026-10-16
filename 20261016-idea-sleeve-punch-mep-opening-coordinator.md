# 20261016 Idea: Sleeve Punch — MEP openings and sleeves through beams, slabs and walls, found, checked against structural rules, placed and scheduled in one click

## Entry 1

```
date-time:   2026-10-16 (scheduled run)
idea name:   Sleeve Punch — drop in the MEP drawing (DXF / IFC / PDF-traced) next to the structural model; the tool finds
             every pipe, duct and cable-tray crossing a beam, slab, wall or footing, tells you which crossings are SAFE,
             which need a RE-ROUTE or a TRIMMER/REINFORCEMENT detail, auto-places the sleeves/openings in the IFC,
             and issues the opening schedule, the sleeve-location drawing and an RFI/BCF list for the MEP team
potential:   High. "Missed sleeve" and "core-cut after the pour" are among the most common and most expensive site
             defects: breaking a beam to pass a 150 mm drain costs days, rework, and sometimes structural risk. Today the
             structural modeller opens the MEP set, eyeballs crossings, and types opening sizes into a schedule by hand
             (half a day to several days per floor). Sleeve Punch turns this into minutes per floor and, more important,
             makes the answer *rule-based and auditable* ("this opening is in the middle third of the beam span and
             below the neutral axis, within the allowed zone: OK" or "within 2d of the support: NOT ALLOWED").
             Paid equivalents exist (Hilti PROFIS/BIM add-ins, Autodesk Revit sleeve-placement plug-ins, Navisworks
             clash + manual follow-up, BIM Interoperability Tools, Solibri rules) but all need licences, a Revit/
             Navisworks seat and a native model; none produce a checked structural verdict for small and mid-size
             contractors working with DXF + IFC. A free, browser-local, "files stay on your device" version does not exist.
             Strong LinkedIn/Instagram fit: "AI-free auto openings", "no more core-cutting", "MEP coordination in 5 minutes"
             — a 20-second reel of red crossings turning into green sleeves with a dimensioned drawing is highly shareable.
```

### Description

**The pain (daily life of a BIM modeller / structural coordinator).**
- MEP drawings arrive late and often change. The structural team finds out about a 300 mm duct through a transfer
  beam only when the sleeve is missing on site.
- Crossings are found by eye, in a few clipped views, or via a Navisworks clash test that reports thousands of
  hard clashes without saying *which are legitimate sleeves vs real problems*.
- Opening schedules are typed by hand (level, grid, size, invert, element), copied between Excel, DXF and the model, and
  go out of date at the next MEP revision. Nobody rechecks.
- Structural rules of thumb (no opening in the support zone of a beam, opening depth ≤ 1/4 of the beam depth, opening
  in the mid-depth zone only, minimum spacing, no cut through column or shear-wall boundary elements, slab openings
  larger than a threshold need trimmer bars) live in a senior engineer's head, not in the toolkit.

**The idea.** A **Sleeve Punch** command in cad2bim (browser, files stay on the PC):

1. **Input** — the structural IFC (or DXF → 3D, already in cad2bim) plus any of:
   - an **MEP IFC** (ducts, pipes, cable trays, with sizes),
   - an **MEP DXF** where services are lines/polylines on layers like `PIPE-*`, `DUCT-*`, `TRAY-*` with a size in the
     layer, block attribute or a text tag ("150Ø CI", "600x300 DUCT"), plus an elevation or "invert/level" tag, or
   - a simple **Excel / CSV list** (level, grid, element, size) for early-stage coordination.
   Services without elevation can be assigned a default (e.g. "above false ceiling, 300 mm below slab").
2. **Find** — 3D intersection of each service segment (a swept rectangle or cylinder) with structural elements
   (`IfcBeam`, `IfcSlab`, `IfcWall`, `IfcColumn`, `IfcFooting`), using the engine's spatial index so a 50k-element model
   stays interactive. Duplicates (one pipe, several nearby crossings) are merged into one opening.
3. **Judge** — each crossing gets a **traffic-light verdict** from a transparent, editable rule table, for example:

   | Element | Rule (default, editable, code-referenced in the report) | Verdict |
   |---|---|---|
   | Beam | Opening only in the middle third of span, centred on neutral axis ± d/6, depth ≤ D/4, clear to support ≥ 2D, clear to next opening ≥ 2× larger dimension | OK / RE-ROUTE / DETAIL |
   | Slab | ≤ 150 mm free; 150–300 mm with trimmer bars; > 300 mm needs designer sign-off; not within 2 × slab thickness of a column face | OK / TRIMMER / DESIGN |
   | Shear wall / core | Not through boundary element or coupling beam; small sleeves only in the web with minimum edge distance | OK / NO / DESIGN |
   | Column | Never | NO |
   | Footing / raft | Only below a user-set level, with sleeve + water-stop | OK / DETAIL |

   Each verdict shows *why* in one sentence and the distance/measure that triggered it ("120 mm from beam support,
   needs ≥ 2 × 450 = 900 mm"). The modeller can override with a reason (stored in the report).
4. **Resolve** —
   - **Safe** crossings: auto-place **sleeves/openings** (`IfcOpeningElement` + a sleeve object with a stable
     `GlobalId`) so quantity takeoff and the BBS see the correct concrete deduction and trimmer bars.
   - **Re-route** crossings: show *how far* the service must move to become legal, and export the suggestion as a
     **BCF issue** with a clipped viewpoint and an MEP-friendly sentence ("move 250 mm east, or lower 120 mm").
   - **Detail** crossings: attach the standard **trimmer / additional reinforcement detail** (callout, bar count and
     length) that flows into the BBS (Bar Optimizer) and drawings (Drawing Autopilot).
5. **Output** (one button):
   - **Opening & sleeve schedule** (Excel): ID, level, grid reference, element, size, centre offset, invert level,
     service, verdict, detail ref, MEP revision.
   - **Sleeve-location drawing** (DXF/PDF): per floor, dimensioned from gridlines, with the same sleeve ID tags used on
     site and in the model — ready to hand to the formwork gang *before* the pour.
   - **Updated IFC** with the sleeves, plus a **BCF** pack of RFIs for MEP.
   - **Rev-to-rev diff**: when a new MEP revision arrives, a diff of "new / moved / removed / now violating" crossings
     (same logic as Change Radar).

**Why it is unique / extensible.** It reuses what is already there:
- cad2bim loads IFC in a worker, has the element model, picking and selection, edits properties and saves IFC, and the
  DXF → 3D pipeline — the crossing search and sleeve placement sit on top of the existing engine and geometry.
- cad2bim already exports quantities and BBS to Excel: the opening schedule and trimmer bars slot into the same exports.
- Structura already has member geometry, rebar generation (`rebar.js`) and the BOQ (`boq.js`) so trimmer-bar logic and
  concrete deduction have a home; its `viewer.js` can show the red/amber/green crossings.
- The other ideas in this folder get *better inputs*: IDS Gatekeeper's "clash-lite → sleeve/opening-vs-beam" future item
  becomes real; Pour Planner's pour card gets a verified list of sleeves and MEP boxes; Joint Congestion Doctor avoids
  openings near joints; Site QR Pass puts a QR on the sleeve location sheet; Drawing Autopilot places sleeve tags on the
  plan; Change Radar powers the MEP revision diff.

**What makes it a "wow".** The demo is 30 seconds. Load a floor, drop in the MEP DXF. Red markers pulse where a duct hits
a beam in the support zone; amber where a slab hole needs trimmers; green where a sleeve is simply fine. Click "Punch":
sleeves appear in 3D, the list builds itself, a dimensioned sleeve drawing downloads, and a BCF file with "move this duct
250 mm" for the MEP lead. Replace the MEP file with Rev 2 and only the 4 changed crossings flash.

### Implementation

Stages that can ship independently, each useful on its own:

1. **Service reader (1–2 weeks).** (a) IFC: `IfcDistributionFlowSegment`/`IfcFlowSegment` with swept-solid or profile
   geometry from the existing worker; (b) DXF: lines/polylines/blocks on MEP layers with a **layer-to-system map**
   (editable JSON saved per project; sensible defaults for Indian MEP layer names such as `PLUMB`, `HVAC`, `FIRE`,
   `ELEC-TRAY`); parse size from layer/attribute/text with editable regex templates; (c) CSV/Excel list import.
   Always show a confidence/"unparsed items" table before computing.
2. **Crossing engine (2 weeks).** Convert every service segment into an oriented box/cylinder; build a simple BVH over
   structural elements (the engine already has bounding boxes and a picking structure); run segment–solid
   intersection (box-vs-prism using SAT, with a cheap cylinder approximation for pipes). Merge nearby hits into
   openings; record local coordinates (distance from support, from the element face, from centre of depth) so rules can
   be evaluated without re-doing geometry. Unit-test with the 50k-element fixture and with hand-calculated cases.
3. **Rule engine (1–2 weeks).** Rules are data, not code: a JSON schema `{element, condition, check, verdict, message,
   source}` with defaults for typical RC practice (and the national code references the project chooses). Rules show
   their formula and a "reference" field so the report is defensible, plus a visible banner: **"Screening tool — the
   engineer of record decides; overrides are logged."** Thresholds are editable per project and per element type.
4. **Resolve actions (2 weeks).** (a) Create `IfcOpeningElement` + `IfcVoidingRelation` and a sleeve proxy with
   properties (`SleeveID`, `System`, `Size`, `Invert`, `Verdict`); (b) trimmer-bar standard details feeding the BBS
   (trimmer around a slab opening = 2 + 2 additional bars of length `opening + 2 × Ld`; extra stirrups and diagonal bars
   at beam openings — configurable); (c) "required move" calculator for re-route suggestions (shortest translation
   that satisfies all rules, snapped to a tolerance).
5. **Outputs (1–2 weeks).** Excel via the existing xlsx writer (schedule + rules used + overrides); DXF sleeve plan via
   the Drawing Autopilot writer (grid dimensions, tag layer `SLEEVE`, section marks); BCF 2.1/3.0 writer shared with IDS
   Gatekeeper (one viewpoint per issue, camera on the crossing, the MEP element highlighted); IFC save with the new
   openings.
6. **Revision diff (1 week).** Hash each crossing (service ID + element GUID + rounded position) and diff against the
   previous run, using Change Radar's comparison view; keep a project history so "who changed what and when" is
   answerable.
7. **UI (parallel).** A "Sleeves" bottom-panel tab: table with filters (verdict, level, system), isolate-in-3D,
   section-box on selection, bulk accept / override-with-reason, and a one-click "Punch all green". Uses the
   existing design tokens (Paper/Ink themes), keyboard-first, with the command search entry `Sleeve Punch`.

Risks and mitigations: false confidence (clear "screening only" language, every verdict explains its numbers, overrides
logged, never auto-approves anything amber/red), messy MEP layers (editable map + preview of what was understood),
service elevations missing (explicit default and a bulk-set tool; unelevated items flagged, not guessed), huge MEP sets
(index and stream per level, process per system), data privacy (all processing local, nothing uploaded; the report says so).

### Future update

- **Fire-stopping and acoustic schedule** for each sleeve (rating, collar/sealant type), exported for the fire
  consultant.
- **Core-drill mode** for existing buildings: scan a photo of the wall or slab, mark the opening, and check against the
  as-built model and rebar-scan data before drilling.
- **Dynamic MEP re-route assistant**: propose the nearest legal path for a pipe or duct (A*-style on a coarse voxel of
  the legal zones), output as a BCF with before/after images.
- **Clash-priority learning**: remember per-project overrides so the next MEP revision opens with the team's decisions
  already applied.
- **Site QR Pass link**: scan a sleeve box on site → see the sleeve ID, size, invert, verdict and the pre-pour
  "sleeve fixed and checked" tick; status feeds 4D.
- **Sleeve prefab and BOM**: sleeve type, length and quantity per floor ordered directly from the schedule; cutting list
  for PVC/steel sleeves.
- **Cost impact** (5D): show cost of rework avoided and extra reinforcement added per floor; weekly "coordination health"
  chart for the project manager.
- **Revit/Bridge round-trip**: push sleeves back to the native model through the cad2bim Bridge for Revit, and pull MEP
  revisions automatically.
- **Agent mode** through the MCP helper: "check floor 3 against the latest MEP DXF and list the crossings I must send
  back to the plumber".

### Market check

- Existing tools are mostly paid and platform-bound: Revit/Navisworks coordination (licence + native model), Hilti
  PROFIS / sleeve add-ins, MagiCAD/CADS or Dynamo packages that place sleeves but do **not** judge them against structural
  rules, and cloud coordination suites (Autodesk Construction Cloud, Trimble Connect) with clash lists that need manual
  triage. Open-source IFC clash checkers (IfcOpenShell-based scripts, ifcclash) find intersections but give no structural
  verdict, no sleeve placement, no DXF sleeve plan, no schedule, and are not usable by a site engineer.
- A free, browser-local pipeline of "MEP DXF/IFC + structural IFC → checked crossings → sleeves + schedule + drawing +
  BCF + trimmer bars" with transparent, editable rules: not found. Partial overlaps exist only as paid add-ins or as
  consultant services.
- Trend note: the 2026 BIM conversation on LinkedIn/Instagram is about practical, "boring but expensive" wins:
  auto-openings, clash triage that is not drowned in noise, validation before issue, and paperless coordination. "No
  more core-cutting after the pour" is a message site and design teams repost.
