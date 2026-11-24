# 20261124 Idea: Grade Gate — find every column-slab junction where the column grade is much higher than the slab grade, draw the hold-back line, split each pour into two grades with m3 by grade, and slide the grade steps up the tower to see the saving

## Entry 1

```
date-time:   2026-11-24 14:00 UTC (scheduled run)
idea name:   Grade Gate (Mixed-Grade Junction Finder + Hold-Back Line + Two-Grade Pour Split + Grade-Step Saving Slider)
potential:   High. Every tower above about 15 floors has M50-M70 columns and M30-M40 slabs. The grade
             change sits at the slab joint, and it is the part nobody models. Today it is a note on the
             drawing ("column concrete to extend 600 mm into slab") and a site argument on pour day.
             Grade Gate makes it a fact in the model: where the junctions are, what the hold-back line
             is, how many m3 of each grade each pour needs, and what it costs. It also answers a
             money question the structural team gets every project: "where can we drop the column
             grade, and what does it save?". Estimated saving (an estimate, not measured): half a day
             of quantity work per floor type, fewer wrong RMC orders, and one grade-step study that is
             normally skipped because it takes two days in Excel.
```

### Description

**The pain (daily for the modeller, the quantity engineer and the site team on any high-rise).**
- Columns are designed in a higher grade than slabs to keep sizes small. The drawing says "column
  grade extends X mm into the slab" and "slab concrete held back at the column". The 3D model has
  one concrete type per element and no junction, so none of this is visible.
- Quantities: the BOQ says "M60: 840 m3". It does not say how much of that M60 is the 600 mm collar
  that sits inside the slab thickness, so the RMC order is wrong by a few m3 per floor. Across 40
  floors that is a real cost, and a real shortage on pour day.
- Site: the carpenter must build a hold-back (a mesh or ply collar, a "puddle" ring) at each column,
  place M60 in the column and the collar first, then M35 in the slab inside the plastic life of the
  first. A forgotten hold-back means one grade for both, and a structural query three months later.
- Design: the team asks "can we drop the column grade from M60 to M50 above floor 30?". Nobody can
  say what it saves without re-running the BOQ by hand.

**What exists today (a short check; to be re-verified before any "not available" claim).**
- The rule itself is known. ACI 318 has historically asked for the stronger column concrete to
  extend into the floor (600 mm) when the column concrete is more than about 1.4 times the slab's,
  and later guidance allows higher ratios if the slab confinement is checked. Sources disagree on
  clause numbers and limits, so the engineer must set the rule per project
  ([Structure magazine](https://www.structuremag.org/article/high-strength-concrete-comes-of-age/),
  [eng-tips thread](https://www.eng-tips.com/threads/using-different-concrete-strengths-for-columns-and-slabs.356639)).
  I did not find a rule of this kind in IS 456; the project specification or the engineer decides.
  Forum posters also say placing two grades in one pour is hard, especially pumped.
- Revit, Tekla and Solibri (paid) model concrete as one material per element. My search did not find
  a feature that splits a slab by grade at the column, schedules m3 by grade per pour, or checks the
  grade ratio at a junction. One web search is not proof: look again before posting any claim.
- This is practical, not a trend. I could not read LinkedIn or Instagram from here, so I cannot cite
  trend data. The hook is plain: "the 600 mm of M60 hiding inside your slab, counted".
- In this repo: Pour Planner and Raft Run plan pours and truck rosters; Joint Judge plans construction
  joints; Tag Tamer tags pours; Rate Rocket prices a BOQ; Floor Face-Off compares slab systems;
  Column Compass sizes columns; Mark Merge reduces marks. None splits concrete by grade at a junction
  or compares grade steps up a tower.

**The idea.** A **Grade** panel in the cad2bim web viewer (and the same operations through the agent
helper and Python API), working on the IFC the person already opened. Nothing is uploaded.

1. **Find the junctions.** The engine reads each column's and slab's grade (from the IFC material or
   a property, with a fix-up table for messy names such as "C60/75", "M 60", "Conc-60"). For every
   column that meets a slab or beam, it gets the ratio. Result: "312 junctions. 164 ratio over 1.4.
   Floors 4 to 31."
2. **Rule table the engineer owns.** Ratio limit, collar extent past the column face (600 mm default
   as a starting value), collar depth (full slab, or slab plus drop), perimeter and corner columns
   treated stricter. All editable, each with a "from specification" note. Defaults are labelled
   "starting values, set by your engineer".
3. **Hold-back line in 3D.** A collar outline is drawn around each flagged column, in the slab, in a
   second colour. Click one: "C14, level 12. M60 collar 600 mm. Slab M35 held back. Ratio 1.71."
4. **Two-grade pour split.** Each pour is split into volumes by grade: "Pour L12: M35 412 m3, M60
   (columns plus collars) 38 m3". A volume is the real intersection, not an estimate by area.
5. **Order sheet.** Excel and PDF per pour: grade, m3, order sequence (columns and collars first,
   then slab), plastic-life reminder as a number the engineer sets, and a hold-back count. The RMC
   supplier gets one clear sheet.
6. **BOQ by grade.** The quantity table adds a grade column and a "collar" line, so "M60: 840 m3"
   becomes "M60 columns 802 m3 + M60 collars 38 m3". The cost uses the person's rates.
7. **Grade-step slider (the money view).** Drag a column grade step up the tower ("M60 up to floor 25,
   M50 to floor 40, M40 above") and see the cost, collar volume, and a "needs engineer check" flag
   where the column design would fail. The slider does not design the column; it asks the engineer's
   own capacity ratio (an Excel or ETABS utilisation they paste in) and shows green or red per
   column. Saving appears as a number in rupees.
8. **Write-back.** The split goes to the model as separate `IfcBuildingElementPart` volumes with a
   grade `Pset`, so other tools read it. A diff shows what changed.

**The wow 1: the hidden concrete.** Switch on "grade colours": every slab shows a small M60 square
at each column. "38 m3 of M60 you were not counting."
**The wow 2: the slider.** Drag the grade step and watch the saving change, with red columns where it
fails. That is a design meeting in one screen.
**The wow 3: the order sheet.** One click gives a supplier-ready order per pour. Easy to screenshot
and post: "the order sheet BIM forgot".

### Implementation

How this fits the repo as it is today (read from `CLAUDE.md` and `docs/`, not from code, so check the
names before starting):

- **Where it lives.** A grade module in `@cad2bim/engine` (`packages/engine/src/`) reading the
  material and geometry web-ifc already gives it. The panel is in `apps/web` on the design system.
  Dependencies point down only; each package gets its own version bump with changelog, README and
  `guide.ts`. A first version needs no new package.
- **Grade reading.** Material name and property parser with a person-editable table; unknown grades are
  listed for the person to map once, then remembered in the project file.
- **Junction finding.** Use the same element-contact search the BBS uses to find the host of a bar.
  A column-to-slab contact gives the junction; the footprint comes from the column outline plus the
  collar extent, clipped to the slab. Do this in the existing worker with a spatial grid.
- **Volumes.** Boolean intersection of the collar prism with the slab solid. Use a tolerance setting
  and report the mismatch with the model's own quantity; the sum of the parts must equal the whole.
- **Pour split.** Attach each slab to its pour (from Pour Planner or the person's tag) and sum by grade.
- **Excel / PDF out.** Use the exporters the BOQ already has. No outside service.
- **Slider.** A pure function of grade per column and the cost table. The capacity ratio is an
  imported column of numbers (matched by mark); the tool shows it and never invents one.
- **First version (smallest useful).** Grade reading, ratio flag, collar in 3D, m3 by grade per pour,
  Excel order sheet. Slider and write-back second.
- **Test.** A small synthetic tower in the repo tests; one real tower IFC kept out of git (people's
  drawings never go in git). Measure: junctions found against a hand count of 10 floors, volume sum
  against the model total, order sheet checked by a site engineer, and time against the by-hand
  method. Report only what was measured.
- **Rules to respect.** Runs on the device; nothing is uploaded. No GPL code. No AI and no key needed.
  Short labels in the app, no helper text (the design rule). The structural engineer owns the rule
  table and every grade decision; the tool counts and shows.

### Future update

- **Grade-aware Pour Planner.** Add the extra time and the two-mix truck order to the pour cycle.
- **Hold-back drawing.** A detail sheet per column type through Drawing Autopilot, with the collar.
- **Cube Clock link.** Track cubes per grade per pour, so the M60 collar has its own result line.
- **Cost into Rate Rocket.** Carry the grade premium into the tender BOQ from the first day.
- **Carbon.** Show the cement saving of a grade step in Carbon Lens (lower grade, less cement).
- **Learn from the site.** Import the real delivered m3 by grade (Pour Proof) and compare with the plan.
