# 20261103 Idea: Joint Judge — draw where the concrete pour stops and get the construction joints placed by rule, checked against the rebar and the structure, with stop-end, waterstop and starter-bar quantities and a 3D joint map for the pour card

## Entry 1

```
date-time:   2026-11-03 18:00 UTC (scheduled run)
idea name:   Joint Judge (Construction Joint / Pour Break Planner: Rule-Placed Joints, Rebar-Aware Check,
             Stop-End + Waterstop + Starter-Bar Add-Ons to BBS and BOQ, 3D Joint Map for the Pour Card)
potential:   High. Every large pour has a point where the concrete must stop for the day. Where that point
             falls is decided on site, by the foreman, often at the last hour, and it is one of the most
             common causes of cracks, leaks and "why is there a cold line here" disputes. The modeller never
             draws it, so it is in no drawing, no BBS and no BOQ. Joint Judge puts it in the model, before
             the pour. Estimated saving (an estimate, not measured): 2 to 4 hours of planning per slab level
             and a lot fewer site arguments; the real value is fewer defects and a cleaner basement and
             podium waterproofing story. It is the missing step between the Pour Planner (when) and the
             Bar Optimizer / BBS (what steel).
```

### Description

**The pain.**
- A raft, podium slab or typical floor is rarely poured in one go. The pour is split by batching-plant
  output, pump count, the working day and the weather. Where it is split is a structural decision (joints go
  where shear and moment are low), a waterproofing decision (basement joints need waterstops) and a
  formwork decision (a stop-end must be fixed through the rebar). Today it is one conversation on the slab
  the evening before.
- Drawings usually carry a generic note ("construction joints as per IS 456 cl. 13.4, location to be
  approved by the structural engineer") and nothing else. So the approval is a phone call, the joint is
  made by feel, and the structural engineer meets it for the first time as a photo on WhatsApp.
- The things a joint needs are never counted: stop-end shuttering (expanded metal, mesh or timber), extra
  bars through it, starter bars, PVC or steel waterstop, surface roughening, bonding agent. They appear as
  site purchases and wastage.
- A joint cut across a beam's support zone, or through a column-beam junction, or beside a lift pit, is
  how a "temporary" decision becomes a permanent defect.

(Read the rule details below as the rules as I know them. The exact clauses and limits must be checked
against the current published edition of IS 456 and the project specification before anything is built,
shown to a user or sold. The tool must carry the rule's source and who verified it.)

**What exists today (one thin check, to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Pour Planner (4D RMC / formwork cycle) names construction joints in passing
  and plans pours by volume and time; Gap Guard handles expansion and seismic gaps; Cube Clock and Ready
  Board work on the pours that are already planned. None of them decides where a pour stops, checks that
  place against the structure and the steel, or counts the joint hardware.
- In the market: general 4D planners (Synchro, Navisworks Timeliner, Bentley, Vico) can sequence pour zones
  you draw, but they do not know the structural rules for joints. Concrete-supplier and formwork-vendor
  tools plan pump and form layouts, not rule-placed joints. Revit has no construction-joint object, so
  teams draw lines in a view or sketch on a PDF. I did not find a free tool that places joints by rule,
  checks the rebar and prices the hardware. I could not open every vendor's current product to confirm
  this; treat it as unverified.
- In cad2bim (from `CLAUDE.md` and `docs/`): the IFC viewer, QA findings with select/isolate/zoom, the BBS
  and rebar modelling, 4D (roadmap, "Building now / Next"), sheets and PDF output (`@cad2bim/documents`),
  the Revit add-in and the agent helper. The pieces are there; the joint logic is not.

**The idea.** A **Joint Judge** panel in the cad2bim web viewer, with a matching Revit add-in command. The
person picks a floor or raft and a pour size limit ("max 120 m3 per pour" or "2 pumps, 10 m3/h each, 8 h").
Joint Judge then does four things.

1. **Proposes the pour breaks.** It cuts the slab, beams and walls into pour zones that fit the size limit
   and puts every cut where the rule allows it. The rule set (a draft file, reviewed by an engineer) says
   for example: in a beam or slab, the joint goes near the middle third of the span, where shear is low and
   away from a secondary-beam junction; in a column, at the underside of the beam or the top of the slab,
   not partway up; in a wall, vertical joints at the midspan between openings or at a stated spacing and
   horizontal joints at a set height above the slab; in a raft, strips ("pour strips" or "post-pour bands")
   at the places the engineer names. The person can drag any line; the tool recolours it green, amber or
   red at once.
2. **Checks each joint against the model.** For every cut it reads the real model: the member, its span, the
   rebar passing through, the beam above or below, openings, sleeves, lift pits and the level of the
   neighbouring pour. Findings are cards in the existing QA panel (select, isolate, zoom, as in the
   Actionable QA design): "Joint J-07 is 480 mm from the support, outside the middle third of beam B14";
   "Joint J-12 cuts 3 sleeves of the MEP set"; "Joint J-03 runs through the lap zone of the bottom bars";
   "Pour 4 is 148 m3, over the 120 m3 limit". Each card says what was checked and what the check does not
   prove (the same pattern as `docs/design/actionable-qa.md`).
3. **Details each joint.** From a small library it picks the type: stop-end of expanded metal or mesh with
   the bars passing through, a keyed or roughened face, or a joint with a PVC or steel waterstop for a
   basement wall or raft. It writes the add-ons: extra starter bars and their bend shapes into the BBS
   (using the BBS Weaver / cad2bim BBS shapes), stop-end area, waterstop metres, roughening area, and
   bonding agent where the specification asks for it. The BOQ gets a "construction joints" heading with
   units the site actually buys.
4. **Draws the joint map and writes the pour card.** A plan with joint lines, pour numbers and volumes, as
   a sheet in the existing DXF / PDF writer; and a 3D view with each pour zone in its own colour. The same
   data feeds the Pour Planner's 4D sequence (each zone becomes one task with its volume and its
   neighbour-cure dependency) and the Site QR Pass pour card (scan the slab, see "Pour 3, 96 m3, joints J-04
   and J-05, stop-end detail D-2, pour after pour 2 + 3 days").

**The wow 1: the red line that moves.** In a review meeting, the engineer drags a joint line along a beam
in 3D. It turns red (outside the middle third), then green as it enters the allowed zone, while the volume of
the two pours on either side changes live and the pump-hours update. Everyone sees the compromise between
"engineering says here" and "the plant can only pour 100 m3" in one gesture, instead of an hour of talk.

**The wow 2: "you are about to cut a lap."** The tool puts a joint on top of the bottom-bar lap zone, shows the
bars in 3D with the lap highlighted, and offers two fixes: move the joint by 600 mm, or stagger the laps. A
mistake that is normally found with the form already fixed is found at the desk.

**The wow 3: the joint hardware list nobody had.** One click gives: 412 m of stop-end, 86 m of waterstop, 640
starter bars (with the bar bending shapes), 38 m2 roughening. The purchase manager has it three weeks before
the pour, not on the morning.

**Why this fits LinkedIn and Instagram right now (read as a trend from feeds, not a measured one).** Site
quality and "defects caught before they happen" posts do well, and short clips of a 3D slab changing colour
as a line is dragged are easy to understand with no sound. A before/after of a basement joint leak and the
joint map that would have prevented it is a story every site engineer recognises. A short reel: draw the
pour limit, the tool cuts the slab, a red joint turns green, the hardware list pops up.

### Implementation

Everything below is a plan; nothing was built or measured.

- **Place in the toolkit.** A pure planner in `@cad2bim/engine` (`model/`, beside the BOQ and QA code): input
  is the elements of one level (members, spans, openings, rebar if present) plus a rule set and a limit;
  output is pour zones, joints, findings and add-on lines. It runs without a browser and in a worker. The
  panel and the 3D overlay are in `apps/web`. The Revit add-in and the agent helper (MCP) call the same
  function, so "plan the level 5 pours" works from chat as well.
- **Zone cutting.** Start with slabs and beams on a regular frame: use the member graph (columns, beams,
  slab panels, grid lines) and cut along grid lines or beam midpoints that satisfy the rule, choosing the
  cuts that give pours closest to the limit. A greedy plus local-search pass is enough for version one; a
  proper optimiser is later. Walls and rafts come after the slab version is tested.
- **Rule files.** JSON or YAML, schema-checked: member type, allowed zone as a fraction of span, forbidden
  zones, minimum distance to openings and junctions, joint type by member and by environment (basement,
  terrace, normal), `source` (standard, edition, clause) and `verified_by`. Do not ship a rule file as "IS
  456" unless its author has read the current edition; mark unverified rules as drafts in the UI. The
  standard is copyrighted: store the rule logic and clause number, not the text.
- **Rebar-aware checks.** If rebar is modelled (cad2bim rebar, BBS Weaver, Slab Sketch, Detail Drop), find
  laps and the bars crossing the joint; otherwise state "no rebar in the model, rebar checks skipped" and do
  not guess. Use the same IFC rebar reading as the BBS tools.
- **Add-ons.** Starter-bar, stop-end and waterstop items come from small templates keyed to the joint type,
  written as BBS lines and BOQ items through the existing writers (Excel writer, BBS shapes). Quantities are
  geometry from the model; any assumed lap, embedment or spacing is a named parameter the person can see.
- **4D and pour card.** Export zones as tasks with volumes and dependencies to the 4D model once it exists
  (roadmap), and to Pour Planner's data until then. The Site QR Pass card shows zone number, volume, neighbours
  and detail.
- **Sheet.** A "joint plan" sheet through `@cad2bim/documents` (PDF, DXF/DWG), with legend and the detail
  drawn for each joint type. Standard details stay in an editable library per company.
- **Privacy.** Local only. Files stay on the device, as in the repo rules; a shared joint map is an explicit
  export.
- **Licences.** Rule files and detail library are our own data. Any geometry or optimisation library must be
  MIT, Apache-2.0, BSD or ISC; no GPL (repo rule).
- **Risks.** (1) A wrong rule gives a wrong joint: show the rule and clause on every finding, ship rule sets
  as drafts until a structural engineer signs them off, and say plainly that the tool proposes, and the
  structural engineer approves, as the specification requires. (2) Real structures are not regular frames
  (transfer beams, post-tensioned slabs, flat slabs): detect them and decline to propose, rather than guess.
  PT slabs have their own rules and must be out of scope until an engineer writes them. (3) Site reality
  (rain, breakdown, pump failure) forces unplanned joints: provide a "pour stopped here" quick tool that places
  an emergency joint, checks it and prints the detail, so the plan is useful on the worst day too.
  (4) Waterproofing detail varies by product and specialist: the library is a starting point and the vendor's
  detail wins.
- **First 2 weeks.** Zone cutting for one regular slab level (beams plus slab panels), a draft rule set with
  five rules (beam middle third, slab middle third, column joint at beam underside, distance to openings,
  pour size limit), QA cards for those five, the 3D recolour on drag, and the joint hardware list as an Excel
  sheet. Test on one real project level kept outside git against the joint positions the site actually used,
  and record how many the tool agreed with, how many it flagged and why, in the next entry. Ask one
  structural engineer to review the rules before any wider use.

### Future update

- Walls, rafts and pour strips (post-pour bands) with their curing-time dependency in 4D.
- Link to Cube Clock: joints and the neighbour's strike-ready date decide the earliest next pour.
- Link to Steel Ledger and Order Clock: the stop-end, waterstop and starter bars enter the material order
  calendar with lead times.
- Site feedback: the foreman photographs the real joint; the photo is pinned to the joint in the model and
  compared with the plan ("planned 7.2 m from grid B, built 8.4 m"), feeding a record for the as-built file.
- Learn from past projects: the tool remembers where engineers moved its proposals and starts to propose the
  same way for that engineer or firm.
- A leak-risk score for basements: count joints at wet levels, their detail and workmanship photos, and
  list the ones to inspect first after the water test.
