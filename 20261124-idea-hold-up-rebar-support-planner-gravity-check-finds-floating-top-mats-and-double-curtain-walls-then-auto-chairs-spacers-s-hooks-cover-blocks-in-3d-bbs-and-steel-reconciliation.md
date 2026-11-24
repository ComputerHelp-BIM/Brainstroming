# 20261124 Idea: Hold-Up — a rebar support planner: a "gravity check" finds top mats and double-curtain walls that nothing holds up, then adds chairs, spacers, S-hooks and cover blocks in 3D, in the BBS and in the steel reconciliation

## Entry 1

```
date-time:   2026-11-24 06:00 UTC (scheduled run)
idea name:   Hold-Up (Rebar Support Planner + Gravity Check + Accessory BBS)
potential:   High. Every raft, pile cap, transfer slab, basement wall and shear wall on site needs
             steel that holds the real steel up (chairs, spacer bars, S-hooks or cross-ties, cover
             blocks, tie wire). It is carried in the contractor's head or a rule of thumb, almost
             never in the model, rarely in the BBS. So "steel on site" never matches "steel in the
             BBS", and the gap is argued at the bill. Hold-Up puts it in the model: it checks that
             every bar layer is physically supported, adds the supports, schedules them, and shows
             the tonnes. Estimated saving (an estimate, not measured): one steel-quantity dispute
             avoided per raft, and no missing line in the tender BBS. It also has a plain, visual
             demo: the top mat "falls" in 3D when the chairs are removed.
```

### Description

**The pain (weekly for a rebar modeller / BBS engineer).**
- A 2 m raft has a bottom mat and a top mat. Between them, something must hold the top mat up while
  the concrete is poured: chair bars (often T20 to T25), usually one per square metre or as the
  engineer says. Revit draws the two mats and nothing between them, so the 3D model has a top mat
  floating in air and a BBS that is short by the chairs.
- Walls with two curtains need S-hooks, cross-ties or spacer bars per m2; deep beams need separator
  bars between layers; columns and walls need cover blocks per m2 of face; every joint takes tie
  wire. None of it is in the model, so the tender BBS is light and the site steel is "over".
- The contractor says "you gave me 142 t". The BBS engineer says "the drawing says 142 t". The truth
  is 142 t of drawn bars plus roughly 6 to 9 t of supports that nobody scheduled (my figure for a
  raft-heavy tower; an estimate to measure on a real project, not a published number).
- Checkers look at bar sizes and spacing. Nobody asks "what holds this mat up?" until a labourer
  walks on it.

**What exists today (a short check; re-verify before any "not available" claim).**
- Tekla Structures has a Chair Bars component: pick a slab, draw a polygon, chairs are arranged in
  it ([Tekla Chair Bars](https://teklastructures.support.tekla.com/2018/en/ext_chair_bars)). It is
  paid, desktop, and for slabs; it is not a check that finds what is unsupported.
- ideCAD has chair rebar features inside its paid design package
  ([ideCAD](https://help.idecad.com/ideCAD/chair-rebar-features)).
- SOFiSTiK has a spacer command for Revit, with a stated limit: only for floors and foundations
  without slope ([SOFiSTiK](https://docs.sofistik.com/2023/en/reinforcement/cmdref/create/spacer.html)).
  Paid.
- Revit rebar add-ins (Graitec PowerPack, BARBOX, pad and pile-cap tools) cover cages and meshes;
  my search did not find chairs or spacers in their listings. One web search is not proof: look again.
- Chair quantity is hand-calculated in practice: a worked example gives about one T25 chair per
  1 m x 1 m of a 2 m raft from the mat's load over the chair's capacity (a forum example, not a code
  rule). Specifications set the real rule per project.
- The trade-press theme right now is AI QA/QC and agents in BIM. I could not read LinkedIn or
  Instagram from here, so this is not trend data. The angle "the check nobody does, shown as a
  falling mat" is a plain geometry check, no AI and no key.
- In this repo: Slab Sketch places chairs for slabs it designs; Steel Ledger reconciles site steel
  against the model; Bar Optimizer cuts lengths; Joint Congestion Doctor finds bar clashes; Cage
  Manual and Cage Ghost show cages. None checks whether any rebar from an existing Revit model is
  held up, or adds and schedules supports for rafts, walls and beams.

**The idea.** A **Hold-Up** panel in the cad2bim web viewer (and the same operations through the
agent helper and Python API), working on rebar already read from the person's Revit IFC.

1. **Gravity check.** For each bar layer, the engine finds what is below it: the form face, the
   layer below, a chair, a spacer. A layer with no support within a span limit is flagged
   "floating" and shown red in 3D. Result: "Raft R1: top mat unsupported over 412 m2. Wall W7:
   two curtains with no ties."
2. **Auto supports by element type**, from a rule table the person owns (defaults for IS 456 and
   typical Indian practice; BS 8666 and others as choices, all editable):
   - raft, pile cap, thick slab: chair bars or chair frames under the top mat, size from depth, grid
     from a rule (e.g. 1 per m2, staggered), edge offsets;
   - double-curtain walls: S-hooks or cross-ties per m2, with 135 / 90 degree hooks;
   - deep beams, transfer beams: separator bars between layers;
   - columns, walls, beams, slabs: cover blocks per m2 of face, at the model's cover;
   - tie wire as kg per tonne of bar (a setting, not a fact).
3. **A chair is a real bar.** Chair bars and S-hooks become real bars in 3D with shape codes, so the
   BBS gets new marks (as a bar bender would write them), weights and a "support" tag. Cover blocks
   and tie wire are counted as items, not drawn one by one.
4. **The falling mat.** A toggle removes the supports in the view: unsupported layers drop to the
   form face and flash red. Switch them on and the mat rises. That is the demo.
5. **Steel reconciliation line.** The BBS summary shows *Drawn bars* and *Supports and ties*
   separately, per element and per pour, so "142 t drawn + 7.4 t supports" is one clear page.
6. **Pour-day walk.** Click a pour and see its support list: "412 chairs T25 at 1 m, 36 m cover
   blocks 50 mm per face", exported to Excel as a store-issue list for the site.

**The wow 1: the falling mat.** "This top mat is floating over 412 m2" with a red drop, then fixed
in one click.
**The wow 2: the missing tonnes.** The BBS total moves up by a clear, itemised 7 t, with the reason.
**The wow 3: the clip.** A 10-second screen recording of the mat dropping and rising, ready to post.

### Implementation

How this fits the repo as it is today (read from `CLAUDE.md` and `docs/`, not from code, so check
the names before starting):

- **Where it lives.** A support module in `@cad2bim/engine` next to the BBS (`packages/engine/src/bbs/`)
  reading the same rebar the BBS reads (one `IfcSweptDiskSolid` per bar, read as centrelines). The
  panel is in `apps/web` on the design system. No new package; dependencies point down only. Each
  package gets its own version bump with changelog, README and `guide.ts`.
- **Layer finding.** Group bars into layers by direction and height inside their host (the BBS already
  finds the member by geometry: the concrete element a bar lies in). Sort layers by height.
- **Support test.** For each layer, sample points on its bars; for each, look for a support within a
  span limit (default from the rule table) or a face within cover. Use a spatial grid so a
  million-element model stays fast; run in the existing worker.
- **Placement.** Raft and slab chairs: lay a staggered grid over the layer's footprint minus openings;
  chair height = gap between layers minus bar diameters; bar size by depth from a table. Walls: a grid
  on the wall face between curtains. All go through the cad2bim API's operations so one undo step
  removes them, and the same calls work from Python, commands and the agent.
- **BBS.** New bar marks use the existing shape-code engine (BS 8666 and IS 2502 lengths), with a
  `support` flag so the schedule can show or hide them and the totals can show both.
- **IFC out.** Supports written as `IfcReinforcingBar` with a `Pset` marking them as accessories, so
  other tools can read them.
- **Rule table.** Plain JSON in the project file (`.c2b`), editable in a small table, with the code
  clause or specification line next to each value. Defaults are labelled "starting values, set by
  your engineer".
- **First version (smallest useful).** Gravity check on rafts and slabs, staggered chair grid,
  3D chairs, BBS marks and the Drawn / Supports split. Walls, beams, cover blocks and tie wire second.
- **Test.** One real raft IFC and one wall IFC kept out of git (people's drawings never go in git),
  plus a small synthetic raft in the repo tests. Measure: unsupported area found against a hand
  check, chair count against the engineer's rule, time on a large model, and minutes saved against
  the by-hand count. Report only what was measured.
- **Rules to respect.** Runs on the device; nothing is uploaded. No GPL code. No AI, no key. Short
  labels in the app, no helper text (the design rule). The engine suggests; the structural engineer
  owns the support rule and sizes.

### Future update

- **Load-based chair sizing.** Size chairs from top-mat weight, concrete placing load and chair
  capacity, shown as a one-page calculation for the engineer to sign.
- **Pre-pour walk.** Hand the list to Cage Ghost so the AR check on site also confirms the chairs.
- **Bar-bender files.** Send chair bars to Bend Bridge machine files with the rest of the pour.
- **Cost line.** Price supports in Rate Rocket so the tender BOQ carries them from day one.
- **Learn from the site.** Compare the support list with Steel Ledger's real issue and tune the
  rule table to the team's own practice.
