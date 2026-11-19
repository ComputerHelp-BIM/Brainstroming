# 20261119 Idea: Bend Bridge — Press one button and the 3D rebar model becomes the file the bar-bending machine reads: every bar bent, tagged and bundled in pour order. A free viewer in the browser shows each bend before the steel is cut, and a weight proof ties the machine file back to the model.

## Entry 1

```
date-time:   2026-11-19 (scheduled run, UTC; second entry of the day, after Tag Tamer)
idea name:   Bend Bridge (3D rebar model / BBS -> BVBS machine file -> tagged, pour-wise bundles)
potential:   High for any yard or contractor with a cut-and-bend machine or a service-bureau yard, and
             medium-high for modelers who today stop at a BBS PDF. The chain "3D rebar -> BBS -> yard"
             usually breaks at the last metre: someone types the schedule again into the machine. That
             is slow, and every retype is a chance for a wrong length or a wrong count.
             Time saved: NOT measured. First test (needs a real yard and its machine): take one real
             slab or one floor of columns and beams; time (a) the usual way: BBS PDF -> operator keys
             it in; (b) Bend Bridge file -> machine. Count (c) wrong or missing bars found after
             bending, (d) minutes per floor. Do not quote hours or accuracy before that is done.
             Market check (2026-11-19, two searches; not exhaustive, so "I did not find one", not
             "none"):
               - BVBS is a public machine-readable ASCII format for cut-and-bent rebar, files end in
                 .abs. Shape groups include BF2D (flat bars), BF3D (3D bars), BFWE (spirals), BFMA
                 (mesh), BFGT (lattice girders). Tekla's docs say it exports BVBS 2.0 (2000), and
                 versions from 2021 support BVBS 3.0 (2018, adds coupler and thread data).
               - Tekla Structures exports BVBS and has an extension that embeds BVBS into IFC. Tekla
                 is paid, and a modeler in a Revit shop does not use it.
               - SOFiSTiK has "Export For Production (BVBS)". CYPE sells an "export reinforcement
                 detailing IFC and BVBS" module. Both paid, both tied to their own design software.
               - BuildingPoint Scandinavia (Construsoft) gives away a free BVBS Viewer. It reads a
                 file and shows a table and one bar in 3D. It does not make the file, does not link
                 to a model, and shows no pour, no tag and no weight check.
               - I did not find a free Revit add-in that writes BVBS directly, and I did not find a
                 free, browser-based tool that goes from an IFC rebar model to BVBS and then to
                 labelled pour-wise bundles. I did not check every Revit add-in store, and I did not
                 check Indian machine makers' own software. I could not measure LinkedIn or Instagram
                 counts; "construction automation", "BIM to site" and "robots on rebar" are common
                 post themes, but that is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - BBS Weaver / Bar Optimizer: they make the schedule and the cutting plan and mention
                 "BVBS-style files" as one line each. Bend Bridge is the finished, tested path from
                 those to the machine, with machine profiles, preview and proof.
               - Cage Manual: the fixers' step-by-step order. Bend Bridge feeds it a tag per bar so
                 the bar in the hand matches the bar on the screen.
               - Steel Ledger: reconciles steel bought vs steel built. Bend Bridge adds the middle
                 number: steel cut and bent, per pour.
               - Pour Planner / Order Clock / Ready Board: they say when a pour happens. Bend Bridge
                 says the steel for it will be bent and bundled in time.
               - Gang Pay: pay per kg. Bend Bridge's bent-kg log is a clean source for it.
               - Mark Keeper / Bar Swap: marks and diameter substitution. Bend Bridge uses their
                 output and re-issues the machine file when a diameter changes.
             The wow: open a model with 3D rebar. Click "Send to bender". A panel shows "Level 3
             slab and columns: 1,842 bars, 11 bend shapes, 6 bundles, 9.4 t" and plays the first
             bundle: each bar flies out of the 3D cage, unfolds flat to its cut length, and folds
             back through its bends in the order the machine will do them. Press "Write file": a
             .abs and a tag sheet. On the yard PC the operator opens the file, or scans the QR on
             the sheet. A "Proof" strip shows: model 9,412 kg, machine file 9,409 kg, the 3 kg
             difference explained (rounding of lengths, listed per bar).
```

### Description

**The pain.**
- A BBS is made, checked and printed, then the yard keys the same numbers into the machine, or a
  helper reads them out. Every retype is a chance for a wrong length, count or shape.
- Bars come off the machine in shape order, not in pour order. The fixers then hunt for "the bar
  for B14 on Level 3". Time is lost on site, not at the desk.
- When the engineer changes a diameter or the architect moves a wall, the machine file and the
  printed tags go stale. Nobody knows which bundles are already bent.
- Free tools can read a BVBS file; none (that I found) makes one from a model and ties it back.

**The idea.** Four moves.
1. **Pick the scope and the machine.** Choose a level, a pour or a set of elements. Choose a machine
   profile (BVBS 2.0 or 3.0 to start; maximum bar length, diameters it can bend, minimum bend
   radius, stirrup-only or all shapes). A bar the machine cannot make is flagged with the reason, and
   routed to a "hand bend" list instead of being silently dropped.
2. **See every bar before it is bent.** A 3D cage view with a play button; click a bar to see its
   shape, cut length, bends and the code. Shapes that look wrong glow: a hook shorter than the IS
   rule, a bend radius under the minimum, a length over the stock bar.
3. **Write what the yard needs.** The machine file (BVBS `.abs`), a tag sheet with a QR per
   bundle, and a bundle list in pour order: Level, Pour, Element, Mark, Dia, Shape, Count, Length,
   Weight. Same data as the BBS the site already trusts, so nothing new to learn.
4. **Prove it and keep it fresh.** A weight proof (model kg vs file kg vs BBS kg, with per-bar
   differences listed). After a change, the file re-issues only the changed bars, with a "replaces
   tag 118" note so the old tag is pulled from the yard.

**What a modeler sees.**
- A "Send to bender" button next to Export BBS, and a panel with the three numbers above.
- A bar table you can sort: shape, dia, bends, and a "cannot bend on this machine" filter.
- After a revision: "37 bars changed, 5 already bent." One click writes a change file.

**What the yard sees.** A tag sheet and a file. The operator does not need cad2bim at all.

### Implementation

Inside the cad2bim IFC viewer; the file is made on the device and nothing is uploaded. A machine's
folder or USB stick receives it by the normal browser download.
- **Input.** The 3D rebar from IFC (`IfcReinforcingBar` with swept-solid or sweep geometry,
  reading the bar's centreline and diameter) and the existing BBS data model in `packages/engine`.
  Where a model has no bars, use the BBS rows with their shape codes (BBS Weaver output).
- **Bar to shape.** A pure function from a 3D centreline to a list of straight segments and bend
  angles in a plane (BF2D) or out of plane (BF3D). It rounds to the machine's step (for example,
  whole millimetres), checks the minimum mandrel radius, and reports the length change from bend
  deductions. Pure function, so it is tested with Vitest on a small set of known shapes (straight,
  L, U, stirrup with 135-degree hooks, crank bar, spiral).
- **BVBS writer.** BVBS is public. Build the writer from the published specification (BVBS e.V.),
  not from any GPL tool. Licence check before shipping: read the spec's terms and write down the
  result in `docs/decisions.md`. I have not read the full spec; the shape groups and versions
  above come from vendor docs, so the first task is to read the spec itself.
- **Machine profiles.** A small JSON per machine (limits, version, header fields, units). Start
  with the generic BVBS profile; add a second profile only after a real yard tests the first with
  its own machine. A profile that has not been run on a real machine is labelled "untested".
- **Preview.** Reuse the viewer's overrides and the cage view from Cage Manual; the unfold
  animation is a simple interpolation of the polyline, drawn with three.js.
- **Tags and QR.** PDF with a QR per bundle (existing PDF path plus a small QR library, checked
  for licence). The QR opens the bundle page (offline in the Model Capsule if used).
- **Proof.** Weight per bar from diameter and cut length (the same formula the BBS uses);
  compare three sums and list every bar whose lengths differ by more than the machine's rounding.
- **Change files.** Diff two revisions by bar identity (the GUID plus mark); write only the changed
  bars with a "supersedes" tag.
- **Agent access.** MCP tools in `packages/mcp`: `list_bars_for_pour`, `check_bendability`,
  `write_bender_file`, so an AI agent can prepare the file and a person approves it.
- **Build order:** (1) bar to shape for the five common shapes plus a weight proof, CSV out;
  (2) BVBS writer for BF2D and the free BVBS Viewer as a test oracle; (3) preview and tags with QR;
  (4) pour-wise bundles and change files; (5) BF3D and the first real-yard machine profile.
- **Test plan:** write a file, open it in the free BVBS Viewer, and check every bar's length and
  shape against the model; then a real machine at a partner yard, one floor, with an engineer
  present. Keep real files out of git.
- **Risks:** a wrong bend deduction makes bars short, so the first release writes the file only
  after a person confirms the proof; machine makers extend the format in their own ways, so only
  profiles tested on a real machine are marked "tested"; scrap and safety stay the yard's
  responsibility, and the tool says so on the export screen.

### Future update

- **Optimise before bending:** hand the bars to Bar Optimizer first, so the file carries the cut
  plan that wastes the least steel, with offcuts reused across pours.
- **Mesh and cages:** write BFMA (mesh) and pre-welded column cages for a yard that makes them in a
  jig, and a jig layout view.
- **Bent-as-you-go log:** the operator scans a bundle QR; the model colours that bar "bent", and the
  Steel Ledger and Gang Pay get a real bent-kg number per day.
- **Robot tie and AR hand-off:** the same tags feed Cage Ghost (phone AR check) and, later, any
  tying robot that reads bar positions.
- **Machine marketplace:** a shared, community-tested library of machine profiles, each with the
  yard that tested it and the date.
- **Reverse path:** read a consultant's or a supplier's BVBS file and place the bars in the model
  to check them against the 3D cage (pairs with Bar Reader and Trace Lens).
