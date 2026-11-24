# 20261124 Idea: Same-Same — find every "typical" flat, floor and bay in the model, flag the odd one out with a 3D diff, and edit once and apply to all, with a per-unit BOQ

## Entry 1

```
date-time:   2026-11-24 00:30 UTC (scheduled run)
idea name:   Same-Same (Typical-Unit Finder + Odd-One-Out Detector + Edit-Once-Apply-to-All)
potential:   High. Towers, villa clusters, hostels, hospitals and hotels are mostly repeats. A modeller
             "copies the typical flat" 30 times and then fixes things by hand. Two things go wrong: (1) a
             fix made in flat 1 is not made in the other 29, (2) a slip in one copy (a beam 50 mm off, a
             missing sleeve, a bar dia changed by accident) is never seen until site. Same-Same finds the
             repeats on its own, shows which copy differs and how, and pushes one edit to all copies with
             one undo. Estimated saving (an estimate, not measured): 0.5 to 2 days per tower per major
             revision, and fewer "typical but not typical" RFIs from site. It is also a very shareable
             demo: "found the one flat out of 40 that was different, in 4 seconds".
```

### Description

**The pain (daily for a modeller on any repeat-heavy building).**
- A revision arrives: "change the toilet door to 750 and drop the kitchen slab by 50". The modeller
  makes it in one flat, then repeats it in every flat on every floor by hand, or re-copies and loses
  the other local fixes.
- "Typical" is a promise in the drawing, not a fact in the model. Copies drift: someone nudges a
  column, resizes a beam to clear a clash in one flat only, or forgets to copy a sleeve. Nobody
  compares flat 7 to flat 31 element by element.
- Consultants and checkers review one typical floor and assume the rest match. Errors found at site
  (a beam that is 450 deep on floor 9 only) cost far more than in the model.
- Quantities: the BOQ is one big number; the team cannot say "one Type-A flat is 18.4 m3 of concrete
  and 1,310 kg of steel, and we have 36 of them".

**What exists today (a short check; to be re-verified before any "nothing like it" claim).**
- Revit has Groups, Assemblies and linked models for repeats. These make copying easier but do not
  find repeats that already exist in an imported IFC, and an edited copy inside a group is not
  reported as a difference. I did not find a built-in "odd one out" report.
- Model checkers: Solibri (paid) has consistency rules such as wall heights per type per floor, and
  Autodesk Model Checker (the Revit-side API was reported as no longer supported from the 2026 tools
  release) checks against requirement sets. These check against rules, not against the building's
  own other copies. My web search found no tool that discovers typical units and reports the
  outlier ([Solibri community thread](https://society.solibri.com/post/1726), [Autodesk Model Checker](https://www.autodesk.com/learn/ondemand/module/check-standards-compliance-on-building-projects-using-revit)). A
  search is not proof; check again before saying "not available".
- Model compare tools (Navisworks, Solibri) compare two revisions of a model, not flat 7 against
  flat 31 inside one model.
- Trade press in late 2026 is full of AI agents inside BIM tools and AI QA/QC (Snaptrude, Qonic,
  Pirros; see [AEC Magazine on agentic BIM](https://aecmag.com/bim/the-agentic-future-of-bim/)). That is the
  theme in the feeds; "AI that finds the one wrong flat" is the same story told with a plain,
  checkable algorithm and no key needed. (This is trade press, not LinkedIn data; I could not read
  LinkedIn from here.)
- In this repo: Mark Keeper finds the same mark on different elements; Tower Stacker builds a tower
  from one typical floor; Mark Merge consolidates types; Twin Check compares design against BIM;
  Rev Merge three-way merges a new architect revision. None finds repeats inside an existing model,
  compares each copy to the others, or propagates one edit to all copies.

**The idea.** A **Same-Same** panel in the cad2bim web viewer, and the same operations through the
agent helper and the Python API.

1. **Pick one, find the rest.** Select one flat (or draw a box, or pick an IfcSpace, or pick a
   floor). Press *Find same*. The engine finds every place the same arrangement occurs, including
   rotated by 90 degrees and mirrored (hand-of-flat), and lists them: "Found 38 more. 36 match,
   2 differ." Auto mode scans a level or the whole model and proposes typical groups without a seed
   ("Type A x 36, Type B x 12 mirrored").
2. **Odd one out.** Each copy gets a match score and a list of differences: missing element, extra
   element, moved by N mm, resized, different type or grade, different bar diameter or spacing,
   different property value. In 3D the differences are ghosted on the master (red = missing in the
   copy, green = extra in the copy, amber = changed). Click a difference to fly to it. Tolerance is a
   setting (default a few mm, shown, never hidden).
3. **Three choices per difference.** *Make this copy match the master*, *Make all copies match this
   one*, or *Keep: it is intentional* (remembered with a name and a note, so it is not reported again
   and it shows in the project's change log).
4. **Edit once, apply to all.** With typical groups known, an edit on the master (resize a beam,
   move a column, change a door) can be applied to every matching copy in the group. A preview
   shows the count and any copy where the edit does not fit (that copy was an intentional variant,
   or something else is in the way). One undo step reverses the whole propagation.
5. **Per-unit quantities.** The BOQ and BBS get a *By typical* view: one Type-A flat = X m3, Y kg,
   Z m2 formwork, times 36, plus the odd ones listed separately. Cost per flat for the sales team,
   and a quick check that every Type-A flat measures the same.
6. **Typical-floor mode.** For a tower: compare every floor with the typical floor and list the
   floors that differ and where (expected: ground, podium, refuge, terrace; unexpected: floor 9).
   This is the pre-issue check a design lead wants before a drawing set goes out.

**The wow 1: the odd flat.** "40 flats, 39 identical, flat 27 has a beam 450 deep instead of 300."
A red ghost on the 3D view, found without reading a single drawing.

**The wow 2: one edit, 36 flats.** Change the master flat's toilet door; preview "36 will change, 2
kept as intentional"; apply; undo restores all of it in one step.

**The wow 3: the shareable clip.** A 10-second screen recording of the scan lighting up repeats in
colour groups, then the one red outlier. A natural post for LinkedIn or Instagram.

### Implementation

How this fits the repo as it is today (read from `CLAUDE.md` and `docs/`, not from code, so check
the names before starting):

- **Where it lives.** The matching engine is a new module in `@cad2bim/engine` (IFC side, reading
  elements already in the one model that BOQ, BBS and QA follow). The panel is in `apps/web` using
  the design system. No new package. Dependencies point down only. Each package gets its own
  version bump with changelog, README and `guide.ts` entry (see `docs/workflow.md#versions`).
- **Element signature.** For each element: IFC class, type, material or grade, a size signature from
  its geometry (extent along its own axes), and, for rebar hosts, a hash of the rebar set (diameter,
  count, spacing, shape code) reusing what the BBS already reads. Position is stored relative to a
  unit anchor, not absolute.
- **Matching.** Seed unit S has elements e1..en with signatures and local positions. For each element
  in the model with the same signature as e1, propose a transform (translate, rotate 0/90/180/270,
  optional mirror = 8 options). Vote on transforms (a simple RANSAC or Hough vote on translation
  and rotation). For each transform with enough votes, check how many other elements of S have a
  partner within tolerance. The score is matched / total with penalties for extras. A spatial grid
  index keeps this fast on a large model; run it in the existing worker so the view stays smooth.
- **Auto mode.** No seed: cluster by IfcSpace or by the grid bays between columns, hash each cluster
  by relative layout, and group clusters with equal hashes. Slower and less certain than seeded
  mode, so it only proposes; a person confirms a group before anything is edited.
- **Differences.** After matching, unmatched seed elements are "missing in copy"; unmatched copy
  elements inside the unit's footprint are "extra"; matched pairs with different size, type, grade
  or properties are "changed". Report each with the element ids so the viewer can ghost them.
- **Edit propagation.** Express the master edit as the cad2bim API's operations (the same
  operations Python, commands and the agent use, so it is one undo step). Transform the edit into
  each copy's frame (rotate or mirror the move vector) and run it as a batch. Skip a copy and say
  why if its partner element is missing or marked intentional.
- **Memory.** Groups, intentional variants and their notes are saved in the project file (`.c2b`)
  and survive revisions. After a new revision (the pipeline keeps each element's IFC id), re-run the
  match by id first so groups are stable and only changed copies are re-scored.
- **Agent helper.** Expose `same_same_find(selection)`, `same_same_diff(group)` and
  `same_same_apply(group, edit)` through the MCP helper. Follows `docs/agent/protocol.md`.
- **First version (smallest useful).** Seeded find on one level, rotation and mirror, differences
  for missing, extra, resized and retyped elements, the 3D ghost, the three choices, and the BOQ
  *By typical* view. Rebar-level diffs, auto mode, typical-floor mode and edit propagation come
  second.
- **Test.** One real tower IFC kept out of git (people's drawings never go in git), plus a small
  synthetic model in the repo tests: 5 identical flats, one with a moved column, one mirrored, one
  missing a sleeve. Measure: precision and recall of differences against a hand-made answer, time on
  a 14,000-element model, and the minutes a modeller needs to check the same by hand. Report only
  what was measured.
- **Rules to respect.** Runs on the device; nothing is uploaded. No GPL code. No AI, no key needed
  (plain geometry and hashing). Short wording in the app; no helper text (the design rule).

### Future update

- **Township mode.** Place one villa or tower type on a master plan by plot, with per-plot
  variants (foundation depth, orientation), instanced so 200 units stay light, and a per-plot BOQ
  and total (links to Yard Smith and Plot Pilot).
- **Library of typicals.** Save a typical flat with its rebar as a reusable unit for the next project
  (links to Deja Model).
- **Drawing side.** The same finder on a DXF: find repeated flats in a plan and warn when one is
  drawn differently before the model is built (links to Trace Lens and Rev Merge).
- **Revision notes.** When a revision touches one copy only, say so in the change note
  ("flat 27 changed; 38 others did not") (links to Change Radar and Chit Edit).
- **Formwork reuse.** Group counts feed formwork and kit planning: how many sets of Type-A shutters
  for the cycle (links to Shutter Plan and Kit Map).
- **Plain explanations.** An optional "why does flat 27 differ?" using the person's own key, only
  when asked, reading the change log; off by default.
- **Honest scoreboard.** After 5 projects, publish measured hit rate and time saved, not estimates.
