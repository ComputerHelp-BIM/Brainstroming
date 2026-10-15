# 20261015 Idea: Joint Congestion Doctor — will the concrete actually go in? Rebar clash and clear-spacing check at beam–column joints

## Entry 1

```
date-time:   2026-10-15 (scheduled run)
idea name:   Joint Congestion Doctor — rebar clash, clear-spacing and "can the vibrator / tremie go in?" check at
             beam–column joints, with a 3D congestion heatmap and ranked fix suggestions
potential:   High. Beam–column joints, wall boundary zones and footing–pedestal junctions are where drawings fail
             on site: 4 beam bars + 4 beam bars + column cage + ductile stirrups (IS 13920 / ACI 318 ch. 18 joint
             confinement) simply do not fit. Today it is found by the site foreman when the cage is half tied, then
             a phone call, a sketch on a cigarette pack, and a delay. Checking it needs Tekla / Revit rebar
             modelling and a licensed clash tool (paid, slow, and nobody models rebar for this). A free, in-browser
             check that works from the bar data the team already produces (BBS / IFC rebar) is not available.
             LinkedIn/Instagram angle: "rebar congestion", "constructability", "clash detection for rebar",
             "BIM that saves rework" — reels of a red-hot 3D joint that turns green after one click are
             exactly the format that travels.
```

### Description

**The pain (daily life of a structural BIM modeller / detailer).**
- The detail drawing says 8-25 mm top bars in the beam, 12-25 mm column bars, 10 mm stirrups at 100 c/c in the
  joint. On paper each is fine. In 3D at the joint, bars from two beams cross at the same level, column bars sit
  in the way, and the clear gap between bars is 18 mm for 20 mm aggregate: concrete cannot flow, honeycombing
  follows.
- Reviewers ask "is it buildable?" The honest answer today is "we'll see on site".
- Nobody has time to model every bar in Revit/Tekla just to find the 5% of joints that are a problem.

**The idea.** A **Joint Congestion** panel that:
1. Takes the rebar already available in the toolkit — cad2bim's IFC rebar (`packages/engine/src/ifc/rebar.ts`),
   the BBS bars, PlanWin AI Pro design output (Ast, bar dia, spacing per member), or just the member
   schedule (size + bars + stirrups) when no 3D rebar exists — and **builds the bar cage at every joint
   on the fly** (only the joint zone, a few hundred bars at most; the rest of the model is not touched).
2. Checks, per joint, the rules a site engineer actually applies:
   - **Clear spacing between parallel bars** ≥ max(bar dia, aggregate size + 5 mm, 25 mm) (IS 456 cl. 26.3.2 /
     ACI 318 25.2), layer by layer, and between bar layers (vertical clear gap).
   - **Bar-vs-bar crossing at the same level** (beam A bar meets beam B bar inside the column core), including
     hooks and 90° bends from IS 13920 / SP 34 anchorage into the joint.
   - **Vibrator / tremie access**: a 40-60 mm needle must find a vertical free gap through the joint; reports the
     largest free opening and its position.
   - **Confinement stirrup fit** (IS 13920 cl. 7.4 / ACI 18.8) — hoops at the required spacing must not collide
     with beam bars passing through.
   - **Cover and lap overlap**: laps landing inside the joint or at the same section (staggering check).
   - **Reinforcement ratio** of the joint cross-section (e.g. > 4% flagged, > 6% hard stop).
3. Shows a **3D congestion heatmap**: each joint coloured green / amber / red by the worst rule, with the
   offending bars outlined; click a joint to see a "gap map" of the section (a slice through the joint with the
   free gaps drawn to scale, like a CAD section) and a rule table.
4. Offers **ranked auto-fix suggestions**, each with quantified impact on steel weight and cost (reusing the
   toolkit's BBS):
   - switch 2 × 16 mm to 1 × 25 mm (fewer, bigger bars, mind the Ast),
   - bundle bars (IS 456 cl. 26.1.1: max 4 in a bundle),
   - change layer arrangement or stagger bar ends,
   - use a mechanical coupler or headed bars instead of 90° hooks,
   - widen the column or beam by 25–50 mm (shows the architectural impact),
   - reduce aggregate to 12.5 mm (flag as concrete-spec change).
   A fix is a proposal; the modeller approves one-by-one (same safe pattern as other AnonGee/cad2bim tools).
5. Outputs: a **joint report** (PDF/Excel, one page per red joint with the gap map, ready to send with an RFI),
   a **BCF issue** per red joint pinned to the 3D view, and a **tagged list** for the detailer ("J-C12-L3, red,
   top layer clear gap 14 mm, suggest 1×25").

**Why it is unique / extends the repo.** cad2bim already reads IFC rebar and produces BBS; Structura already
shows ETABS member sizes and a rebar module; PlanWin AI Pro already has Ast and IS 456/IS 13920 knowledge.
Joint Congestion Doctor is the first feature that *uses* that rebar data to answer "can it be built?" rather than
"how much steel is it?". Commercial tools that do this (Tekla Structures rebar collision check, Revit +
Navisworks, Autodesk Advance Steel/Reinforcement) need full 3D rebar models and a paid seat; none runs
in the browser with files kept on the PC.

### Implementation

- **Data in.** Three tiers, so it works on day one:
  1. *Schedule-only (Tier 0):* member sizes + longitudinal bars + stirrup spec from Structura/PlanWin/Excel →
     the engine lays out bars by standard rules (corner bars, spacing, hook geometry from IS 2502 / SP 34).
  2. *IFC rebar (Tier 1):* `IfcReinforcingBar` geometry from `ifc/rebar.ts`, used as-is.
  3. *Revit/Bridge (Tier 2):* bars sent from the cad2bim Bridge for Revit.
- **Geometry core.** Bars as polylines with radius → capsule/cylinder segments. Spatial hash (uniform grid
  of ~50 mm cells) over the joint bounding box; segment–segment minimum distance for clear spacing
  (`d_centre − r1 − r2`), run in a Web Worker, results streamed to the viewer. Typical joint = 150–400 bars →
  well under 50 ms. Whole 500-joint building < 30 s.
- **Vibrator access.** Rasterise the joint plan section into a 5 mm grid, mark bar discs as blocked, flood-fill /
  distance transform to find the largest inscribed free circle; pass if ≥ 60 mm (configurable).
- **Rule engine.** A small JSON rule pack (`rules/is456.json`, `is13920.json`, `aci318.json`, `eurocode2.json`)
  with parameters (aggregate size, needle dia, min clear spacing), so the checker is code-agnostic and users can
  add a company standard. Same pattern as the IDS Gatekeeper rule packs in this folder.
- **Heatmap + gap map.** `Viewer.ts` already has subset colouring and section support
  (`render/viewer/section.ts`, `subset.ts`); colour the column/beam subset by joint score, and a React
  panel draws the SVG gap map from the same grid.
- **Fix engine.** Enumerates candidate re-arrangements that keep Ast_provided ≥ Ast_required (reads required from
  PlanWin/ETABS via the Twin Check idea), re-runs the check on each, ranks by (steel kg delta, congestion score,
  number of members touched). Never writes to the model without approval; writes an updated IFC rebar set or
  a revised BBS Excel.
- **Phases.**
  1. Tier-0 beam–column joint check, clear-spacing + stirrup-fit rules, red/amber/green table (1 week).
  2. 3D heatmap + gap-map SVG + PDF/Excel report (1 week).
  3. IFC rebar (Tier 1) + BCF export (1 week).
  4. Fix suggestions with steel-weight delta (2 weeks).
- **Tests.** Fixture joints with known answers (hand-checked IS 456 examples); a property test that a generated
  cage is always at least "bar-dia clear" to itself; golden SVG snapshots for the gap map.
- **Safety note.** A design aid. The tool reports geometry-based constructability; final detailing is signed by
  the structural engineer.

### Future update

- **Walls, footings, pile caps:** boundary elements of shear walls, pile-cap bar mats, raft–column pedestals.
- **Precast / PT:** duct and anchorage congestion, prestress anchor zones.
- **Live in Revit:** the Bridge highlights red joints directly in the Revit view and posts a Revit issue.
- **Photo check:** a site photo of the tied cage is compared with the model's cage (spacing detector) to give an
  "as-built vs as-designed" tick — a very shareable "AI on site" moment.
- **Learning loop:** the team's own solved joints become a library ("how we fixed C12 at L3") that suggestions
  are ranked against.
- **Combine with Bar Optimizer:** re-optimise cutting plan after the congestion fixes, with the new weight and
  CO2e shown immediately.
- **Combine with Pour Planner:** flag pours where a congested joint needs a self-compacting mix (SCC) or a
  smaller aggregate, and add it to the RMC order sheet.
