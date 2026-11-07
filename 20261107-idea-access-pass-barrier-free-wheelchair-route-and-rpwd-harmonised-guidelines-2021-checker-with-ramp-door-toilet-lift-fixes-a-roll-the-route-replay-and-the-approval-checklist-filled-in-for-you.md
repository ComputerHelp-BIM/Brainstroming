# 20261107 Idea: Access Pass — barrier-free check for the whole building: a wheelchair route from the gate to every room, ramps, doors, toilets and lifts checked against India's Harmonised Guidelines 2021, red spots with a fix, a roll-the-route replay and the approval checklist filled in for you

## Entry 1

```
date-time:   2026-11-07 15:30 UTC (scheduled run, fourth entry of the day)
idea name:   Access Pass (Barrier-Free / Universal Accessibility Checker and Fixer: route finder from site gate
             to every room, ramp slope + landing + handrail, door clear width + threshold, accessible toilet
             turning space, lift cabin size, corridor width, tactile path, parking bay; RPwD / Harmonised
             Guidelines 2021 checklist auto-filled; "Roll the route" replay)
potential:   High and mandatory. In India, building permission and the completion certificate can be refused
             for a new building that does not meet the Central Government's accessibility standard (RPwD Act
             2016, sections 40 and 44; the standard is now the Harmonised Guidelines 2021). Today the check is
             done late, by hand, on a printed checklist, by whoever has time, and it comes back as an
             objection from the authority or as a court notice after the building is built. Fixing a ramp at
             the drawing stage costs minutes; fixing it after the slab is cast costs a break-out and a
             re-pour (cost not measured here). The wow: press one button and a blue wheelchair rolls from the
             gate to every flat, shop and toilet in the 3D model; where it gets stuck, the spot flashes red
             with a plain reason ("door 750 mm, needs 900") and one fix button.
```

### Description

**The pain.**
- Accessibility is checked **last**, on paper, from 2D drawings. A 1:12 ramp that is really 1:9 once the level
  difference is read from the section, a 800 mm toilet door, a lift cabin 100 mm short, a 1,100 mm corridor
  behind a column: all are found at scrutiny or at handover.
- The authority's checklist is long. Filling it by hand means reading 40 to 80 dimensions across plans,
  sections and schedules (count estimated, not measured). Nobody enjoys it, so it is done once, badly.
- Architects own most of these numbers, but the **structure** decides some of them: column and beam positions
  narrow corridors and doors, a sunk slab in a toilet creates a step, a ramp needs a slab level and a landing.
  A BIM modeller on a structural-plus-architecture team gets blamed for clashes they could have caught.
- Clients, hospitals, schools, offices and government jobs now ask for an "access audit report" as a deliverable.
  It is billed as a separate service by access consultants.

**What exists today (checked 2026-11-07, not exhaustive).**
- In this repo's `idea` branch: Exit Map (20261020) builds a walk graph for **fire escape** and mentions
  "accessibility pack" as a one-line future update. Stair Smith designs stairs. Park Pilot checks ramp swept
  path for cars. Lift Lord sizes lifts and the shaft. Wall Wizard, Window Wright and Flush Fit work on walls,
  doors and fit. A search of all 98 idea files for "wheelchair", "barrier-free", "RPwD" and "accessib" found
  only that one line in Exit Map. Nothing makes accessibility the main subject, uses the Harmonised
  Guidelines, or fills the approval checklist. Access Pass is that.
- In cad2bim (`docs/design/`, read today): IFC reading, saved views, QA with findings (`actionable-qa.md`),
  levels (`levels.md`), walking in the 3D view, BCF and IDS-style rules (planned), the Python console and the
  one API. These are the pieces; the **route graph, the rules and the checklist** are new.
- In the market: general BIM rule checkers (for example Solibri) can hold accessibility rule sets, and some
  national rule packs exist for other countries; they are paid desktop products and I did **not** find an
  India-specific Harmonised Guidelines 2021 rule set in them. A web search today (standard mode) found the
  Guidelines, their legal status and policy papers recommending that approval portals use the checklist
  appended to the 2021 Guidelines, but **no BIM or IFC tool** that checks it. That is a search result, not
  proof; treat "no free tool exists" as unverified until a deeper search.
- Trend: the 2026 BIM conversation on LinkedIn is mostly "AI + Revit + MCP" and "agentic BIM". I could not
  measure what trends on LinkedIn or Instagram and have no trend data. The honest hook: "Accessibility is law
  and a court-notice risk, and nobody has made it a one-click check." Short clips of a wheelchair rolling
  through a building and turning red where it fails are the kind of thing that travels.

**The idea.** An **Access** panel in the viewer that runs on any IFC or `.c2b` (and later the Revit
bridge model). It works in four steps.

1. **Read the building as a walk graph.** The app finds floors, doors, ramps, stairs, lifts, rooms, toilets and
   parking from IFC classes and from the names it already understands. Each door joins two rooms with its
   **clear width** (frame and leaf, not the nominal size) and **threshold height**. Each ramp has a run and a
   rise. A lift has a cabin size and door width. If something is only a proxy or has no size, the panel lists
   it first as "needs a value" instead of guessing.
2. **Run the rules** from a rule table the user can read and edit (default: Harmonised Guidelines 2021 values,
   each rule with its clause number so it can be checked against the document). Examples of what is checked,
   with the numbers to be **verified against the current official PDF before shipping** (I am not quoting
   figures from memory as final):
   - route from the plot entrance / accessible parking to the main entrance and to each floor, stepless;
   - ramp slope, rise per run, landing size, handrail height on both sides, kerb edge, width;
   - door clear width, threshold height, space beside the handle side;
   - corridor width and passing places;
   - accessible toilet: door, turning circle, grab bar zone, basin and WC clear space, one per floor / per
     number of users, with the right sign;
   - lift: cabin depth and width, door width, button height, one lift reaching every floor;
   - accessible parking bays: count by total, width, nearest to the entrance;
   - tactile path and signage points on the route.
3. **Show the result on the model.** Green route where it works. At each failure a red pin on the exact element
   with the reason and the gap ("ramp 1:9, limit 1:12; needs 3.6 m more run or a 150 mm lower landing"),
   the clause, and a **fix**: for example "widen door D14 to 900" (architect), "move column C7 by 120 mm"
   (structural), "lower toilet slab step" (structural), "add a lift stop at basement". Each pin can become a
   BCF issue to the right person.
4. **Roll the route.** A wheelchair figure follows the route from the gate to a picked room at wheelchair eye
   height and speed. It stops and flashes at each failure. A "reach map" colours every room: reachable
   (green), reachable with help (amber), unreachable (red), with a score for the floor and the building.

**Approval checklist and report.** One click writes a PDF (and an Excel) that mirrors the authority's
checklist: each row filled with the model's value, the limit, pass / fail / not found, the element id, and a
screenshot of the failing spot. It is a **pre-check to save time**, not a certificate, and says so on the page;
a registered access auditor still signs. The report carries the rule table version so a reader knows which
clauses were used.

**Time saved (estimate, to be measured).** The checklist pass: from about half a day of reading drawings to
about 10 to 20 minutes including fixes. The saving that matters more: errors found when the model is still
cheap to change, and one fewer round of authority objections.

**Why it is a "wow".**
- The clip: a gate, a wheelchair, a model. The chair rolls, hits a 750 mm door, flashes red; one click shows
  the fix, the chair rolls on. At the end the whole building is green and a checklist PDF drops out.
- It turns a legal chore into a visible, shareable result, and it is useful in every residential, school,
  hospital and commercial job.

### Implementation

- **Where it lives.** Rules and the walk graph are pure functions in `packages/engine` (next to QA), with unit
  tests; the panel, the pins and the replay are in `apps/web` using design-system tokens; the report uses the
  existing PDF / Excel export paths. No new package at first. It should reuse the graph the Exit Map idea
  needs, so build the graph once.
- **Graph build.** From IFC: `IfcDoor` (clear width from the type's quantity or from the opening and frame
  parameters), `IfcRamp` / `IfcRampFlight` and stairs, `IfcTransportElement` (lifts), `IfcSpace` and names for
  toilets, `IfcCovering`/slab steps for thresholds. Where a value is missing the item is "needs value", with
  a field to type it in; typed values are saved with the project.
- **Rule table.** A JSON file: id, clause, what is measured, limit, unit, severity, and a short fix template.
  The default set is entered from the official Guidelines by a person and reviewed by a registered access
  auditor before release; every number carries a "verified on" date. Offices can add state by-law variations.
- **Routing.** Standard shortest path on the graph, with step-free as a hard condition, then a second pass for
  clear-width limits. Ramps are checked from geometry (rise and run from the bounding shape or the Z of the
  ends), not from the name.
- **Fixes.** A fix is a suggestion with the changed number and who owns it; it is applied through the cad2bim
  API only when the user presses it (one undo step), or sent as a BCF issue. Nothing changes silently.
- **Privacy and rules of the repo** (`CLAUDE.md`): runs on the device, nothing uploaded; colours from tokens;
  a version bump with a changelog entry per package touched; the decision recorded in `docs/decisions.md`.
- **Steps (one developer).** (1) door, ramp and corridor rules on a plan with red pins (about 1 to 2 weeks);
  (2) toilets, lifts, parking and the route graph with the reach map (about 2 weeks); (3) checklist PDF / Excel
  with screenshots (about 1 week); (4) the replay and BCF hand-off (about 1 week); (5) auditor review of the
  rule table and a pilot on three real buildings (about 2 weeks).
- **Testing.** A small synthetic IFC set with known answers (a 1:12 and a 1:9 ramp, 800 and 900 mm doors, a
  toilet of 1,400 and 1,500 mm); tests that the right pins appear and that the PDF rows match a hand-filled
  checklist; a test that an unknown value is reported, not passed; a test that nothing is sent over the
  network.

### Future update

- **Fix-in-place suggestions that move structure:** show which column or beam shift clears a corridor, and
  check the frame again with Column Compass and Ratio Radar.
- **Other countries' packs:** UK Part M, ADA, ISO 21542, as extra rule tables others can add.
- **Wayfinding and signage:** place tactile paving, signs and contrast strips from the route, with a BOQ.
- **Evacuation for everyone:** share the graph with Exit Map so refuge areas and assisted escape for
  wheelchair users are checked in the same run.
- **Phone audit of the built building:** measure real door and ramp values on site and write them back to the
  model for the completion check (links to Audit Eye and Snag Sweep).
- **MCP tool:** let an assistant answer "can a wheelchair reach every flat on level 5?" and cite the pins.

### Sources

- [Harmonised Guidelines and Standards for Universal Accessibility in India 2021 (DEPwD)](https://divyangjan.depwd.gov.in/upload/uploadfiles/2262.pdf)
- [Achieving Universal Accessibility in Indian Cities (Vidhi)](https://vidhilegalpolicy.in/blog/achieving-universal-accessibility-in-indian-cities/)
- [Implementation Roadmap (Vidhi, 2025)](https://vidhilegalpolicy.in/wp-content/uploads/2025/12/Implementation_Roadmap.pdf)
- [Understanding recent amendments to RPwD Rules, 2017 (LiveLaw)](https://livelaw.in/articles/understanding-recent-amendments-to-rpwd-rules-2017-231269)
- Related ideas in this branch: `20261020-idea-exit-map-...`, `20261022-idea-park-pilot-...`, `20261028-idea-lift-lord-...`
