# 20261103 Idea: Clash Sorter — turn 3,000 raw clashes into 40 root-cause groups, ranked by cost and rework, assigned to the right trade, with one suggested fix per group, BCF issues and a weekly burn-down

## Entry 1

```
date-time:   2026-11-03 06:00 UTC (scheduled run)
idea name:   Clash Sorter (Clash Triage: Raw Clash Report to Root-Cause Groups, Ranked by Cost, Assigned to a Trade, One Fix per Group, BCF Issues, Weekly Burn-Down)
potential:   Very high. Every BIM team that coordinates models gets a clash report with thousands of rows,
             and the real work is reading it: which are noise, which share one cause, whose job is it,
             which can wait. Today that reading is done by hand in Navisworks or Excel before every
             coordination meeting. Estimated saving (an estimate, not measured): 4 to 8 hours per
             coordination cycle for one coordinator, and fewer meetings spent on the same clash twice.
             It makes the clash list something a site engineer can act on, not only a BIM person.
```

### Description

**The pain.**
- "Clash noise" and "clash fatigue" are common topics in BIM and coordination posts on LinkedIn (read this as
  a trend I saw in feeds, not a measured statistic). A test run returns 2,000 to 5,000 clashes; maybe 40 are
  real decisions.
- One badly placed duct run clashes with 60 beams: that is one problem, listed 60 times.
- Hard clashes between a pipe and the slab it already sleeves through, a conduit inside a wall by design,
  or a rebar and a cast-in sleeve are not errors, but they sit in the same list.
- Nobody agrees who owns a clash (MEP, structure, architecture), so it bounces between consultants.
- The same clash returns in the next run with a new ID, so progress cannot be counted.

**What exists today (one thin search plan, to be re-checked before any "nothing like it" claim).**
- Navisworks Manage clash tests, Solibri, Autodesk Model Coordination in ACC, and BIMcollab all find and
  group clashes in some way. They are paid or tied to a subscription, and I did not verify their current
  prices or features here. Their grouping is mostly by element or by a fixed rule (same level, same pair of
  categories), not by cause and cost.
- Free route: export an XML or CSV and filter in Excel. Nothing I know of is free, in the browser, reads
  plain IFC from the device, and proposes a fix per group.
- In this repo: `Clash checks` is on the roadmap as an idea (overlapping members found and listed).
  Sleeve Punch (MEP openings), Joint Congestion Doctor (rebar vs rebar), Flush Fit (architecture vs
  structure) and Meeting Mind (BCF from a meeting) each solve one clash family. None sorts a general clash
  list. Clash Sorter would sit on top of the roadmap's clash checks, and it can also read a list exported
  from any other tool.

**The idea.** A **Clash Sorter** panel in cad2bim's web viewer:
1. **Input.** Two or more IFC models loaded from the device (federated by their shared coordinates), or a
   clash report from another tool (Navisworks XML, BCF, CSV of GUID pairs). Nothing is uploaded.
2. **Pre-filter ("by design" rules).** A short, editable rule list marks known non-issues: element passes
   through a sleeve or opening that exists; clash volume under a tolerance (for example 5 mm for hard
   clashes, set by the team); conduit in a wall or slab chase; fixing, bracket or hanger touching its host;
   duplicates. Each hidden clash records which rule hid it, so nothing disappears silently.
3. **Group by cause.** Clashes are clustered when they share an element, a line (one duct run), a level and
   a zone (grid cell), or a repeated pair of types. One group = one decision. A 3,000-row report might
   become 40 groups.
4. **Rank.** Each group gets a score from its size, severity (hard, soft or clearance), the trade chain
   affected, the level's pour date if a 4D schedule is loaded, and whether it blocks a concrete pour (an
   embedded item that cannot be changed later ranks high). Score weights are visible and editable.
5. **Owner.** A group is assigned to the trade that is easiest and cheapest to move, using a short, editable
   priority list: for example, drainage and ducts move before beams; structure moves last unless the team
   decides otherwise. The coordinator can override.
6. **One suggested fix.** For common patterns: raise or lower a duct by a stated offset to clear a beam;
   add a sleeve at these coordinates (sent to Sleeve Punch); shift a pipe run sideways by a stated amount;
   reduce a section only as an RFI to the structural engineer (never changed silently). Each fix shows the
   moved element, the new position and the clashes it would clear, so the person can accept it.
7. **Issues.** Accepted groups become BCF issues, one per group, pinned to the elements, with the group
   image, the owner and a due date. Rejected groups keep the reason.
8. **Next run.** Re-running the check matches clashes to the last run by element GUIDs, so the report says:
   "212 new, 1,480 closed, 38 still open, 6 reopened". Open groups carry their old issue, owner and age.

**The wow 1: the "3,000 to 40" moment.** The first run shows the raw count falling live as rules and grouping
apply, then a list of the top 10 groups with a picture each. A coordinator can lead a meeting from this
screen.

**The wow 2: "what-if I move it".** Pick a group and drag the suggested offset in the 3D view; the clash
count for that group and for new clashes it would create updates at once, so the fix is tested before it
is raised.

**The wow 3: burn-down and heat map.** A weekly chart of open groups by trade and age, and a 3D building
coloured by clash density per zone. A director sees "Level 4 east is the problem" in one picture; the
image can go into the Model Reel video or a weekly PDF.

### Implementation

Everything below is a plan; nothing was built or measured.

- **Place in the toolkit.** A panel in `apps/web` using the IFC reading and federation in
  `@cad2bim/engine`. The clash finder itself is the roadmap's "Clash checks" item; Clash Sorter is the layer
  above it and works on a list of pairs. Keep the sorter in the engine as a pure function (list of clash
  records in, groups out) so it is testable and can run in a worker.
- **Clash record.** {id, guidA, guidB, type (hard/soft/clearance), distance or volume, point, level,
  grid zone, categories, systems}. Imports: BCF, a Navisworks XML clash report, CSV with GUIDs. Each parser
  is small and tested against sample files that stay out of git (people's drawings never go in git).
- **Geometry.** Bounding-box and mesh intersection from the loaded geometry (web-ifc); broad phase with a
  spatial grid or BVH, narrow phase on triangles, in a worker so the page stays responsive. Million-element
  behaviour is covered by `docs/design/million-elements.md`; check it before promising speed.
- **Grouping.** Start simple and explainable: union-find over clashes that share an element or lie within a
  small distance along the same MEP run (via IfcDistributionSystem or connected ports where present), then
  split by level and zone. No black box: each group states why its clashes belong together. A learned model
  can come later, only if it stays explainable.
- **Rules.** A plain JSON/YAML rule file the team can version. Rules are tested against the IFC (for example
  "IfcOpeningElement contains the penetrating element's bounding box").
- **Fix suggestions.** Rule-based first: vertical or horizontal offset search within allowed ranges for a
  system type, checked against the same clash test. The IFC is never edited silently; an accepted fix is
  written as a BCF issue with a viewpoint, or as a change set for the native edit tools when those ship.
- **Matching runs.** GUID pairs as the key; fall back to nearest centroid when GUIDs change after a Revit
  re-export. Store history in the project file (`docs/design/project-file.md`).
- **Privacy.** Local only. Reports and BCF are files the person saves; sharing is an explicit export.
- **Licences.** Any BVH or geometry library must be MIT, Apache-2.0, BSD or ISC; no GPL (repo rule).
- **Risks.** Wrong grouping hides a real clash: keep every hidden clash one click away, show counts by rule,
  and never delete. Owner priority is a team policy, not a fact; ship it as editable defaults. Models in
  different units or origins give nonsense: pair with a grid alignment check before running.
- **First 2 weeks.** CSV/BCF import + the three simplest rules (tolerance, duplicate, by-design opening) +
  union-find grouping by shared element and level + a ranked list with pictures. Test with a real two-model
  federation kept outside git, and time one coordinator doing the same job by hand. Record measured times
  in the next entry.

### Future update

- Auto-assign by contract: read the team's responsibility matrix from an Excel sheet instead of the default
  priority list.
- Learn from past projects: remember which rule or fix the team accepted for a clash pattern, and offer it
  first next time (kept on the device and exportable).
- Link to the 4D timeline and Pour Planner: rank clashes by days before the pour, with a "must be solved
  by" date on every group.
- Link to Rate Rocket: a rough rework cost per group to show on the ranking (marked as an estimate).
- Coordination meeting mode: Room Link shares the screen; decisions are written back as BCF by voice using
  Meeting Mind.
- Revit side: a bridge command that jumps to the group's elements and applies an accepted offset after a
  person confirms (conflicts are always decided by a person, as in the roadmap).
- Public sample: a small open sample federation so new users can try the three-click flow without their own
  files.
