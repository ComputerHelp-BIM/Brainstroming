# 20261029 Idea: Mark Merge (Type Diet): ETABS or the model gives 240 "different" beams; Mark Merge finds the 40 marks the site will actually build. It groups near-identical beams, columns and slabs, picks the strongest member of each group as the group's detail (so it only ever upsizes, never weakens), shows a live slider of "marks left vs extra steel in kg and rupees", colours the 3D model by group, and rewrites marks, schedules, BBS and shuttering counts in one click

## Entry 1

```
date-time:   2026-10-29 (scheduled run, UTC)
idea name:   Mark Merge (Type Diet). "Every beam is unique in the analysis model. No site
             wants 240 unique beams."
             Today: the analysis (ETABS) or the first rebar run gives each beam its own
             reinforcement. The detailer then spends two to four days with Excel and a
             print-out: 'B14 and B27 are nearly the same, give both the bigger one', 'B3
             and B9 differ by one bar, merge', repeat for 240 beams and 90 columns. The
             result decides how many beam marks go on the drawings, how many different
             cages the bar-bending yard makes, how many shuttering sizes the carpenters
             set, and how many mistakes happen on site (a wrong cage in the wrong beam).
             Nobody measures what the merge cost in steel, because the merge is done by
             eye. Here: the modeller or detailer opens the model in cad2bim and presses
             Mark Merge. In a second the panel says: '238 beams -> 41 marks at 0 % extra
             steel (exact twins only); slide right for fewer.' The slider is the wow:
             drag it to '24 marks' and the panel says '+1.9 % beam steel = +1,420 kg =
             about 1.1 lakh'. Drag to '12 marks': '+6.4 %'. The 3D model recolours live by
             group, so the engineer sees the 'B-12 family' lit across five floors. They
             pick the mark count that is worth it. Press Apply and the marks, the
             schedules, the rebar in 3D, the BBS and the shuttering quantity all update,
             with an undo. A report says for every group which member 'governs' (the one
             whose detail everyone gets), why, and the extra steel each member received."
             One engine, five front ends:
               (1) cad2bim web app: a "Marks" dock panel with the slider, a group table
                   and a 3D colour-by-group, built on rebar (0.66-0.70), BBS (0.73) and
                   the one live model (0.80.0),
               (2) Revit (bridge add-in + pyRevit AnonGee button): writes the new Mark,
                   Group and Governing-member parameters onto Revit elements and updates
                   the rebar sets so the Revit schedules match,
               (3) ETABS / Structura / Detail Drop: reads each member's required steel
                   as the starting data, so the merge can be done before any 3D rebar,
               (4) agent helper (MCP): 'merge the typical-floor beams, allow up to 3 %
                   more steel, keep transfer beams unique',
               (5) Sheet Sync / Excel: the group table round-trips through Excel for
                   the engineer who wants to edit by hand.
             Outputs: merge report (Excel + PDF), new mark list, group map (3D colour and
             a plan per floor), steel penalty table (kg and cost per group and in total),
             updated beam / column / slab schedules, updated BBS, shuttering sizes list
             (how many different forms), and a change log for the drawing revision.
potential:   Very high for the structural and detailing team; used on every RCC project,
             once at the start and again at every redesign.
               - It replaces 2 to 4 days of manual comparison on a mid-size building (this
                 is my estimate from how detailing offices describe the job; it was not
                 measured. The first test should time a real project, both ways).
               - Fewer marks means fewer drawing sheets and fewer cages to fabricate,
                 and fewer site mistakes. Repeating forms also lower shuttering cost,
                 because the same panels are reused floor after floor.
               - It turns a decision made by feel into a priced choice. 'Is 1.9 % more
                 steel worth 24 fewer marks?' is a real conversation between the
                 structural consultant, the contractor and the client, and today there is
                 no number to put in it.
               - It is safe by design: a member only receives a detail that is equal to
                 or stronger than its own required steel, never weaker (see the rules
                 below). The engineer signs the result; the tool shows every change.
               - It makes the other ideas better: Detail Drop (ETABS to detailed rebar)
                 gets fewer details to draw, Bar Optimizer gets longer repeats to cut,
                 Shutter Plan gets repeated forms, Rate Rocket prices fewer unique items,
                 and Drawing Autopilot draws fewer sheets.
             Market: from a quick web search (not a full survey; the sources are vendor
             pages and trade press and I did not verify claims or prices, so check before
             relying on any of this):
               - The 2026 AI-in-Revit tools I found (BIMlogiq Argus, Pele AI, ArchiLabs)
                 are about tagging, dimensioning, views and sheets, i.e. the drawing
                 production side. I did not find one that groups reinforced concrete
                 members by required steel, picks a governing detail and prices the extra
                 steel (AEC Magazine, 'BIMlogiq boosts Argus AI platform for Revit').
               - Rebar detailing suites (for example Autodesk Advance Steel / Reinforcement
                 Detailing, Tekla Structures with its own numbering, SOFiSTiK, and Indian
                 packages such as the beam-grouping features inside design tools) have
                 numbering or grouping functions. They are paid, tied to one desktop
                 product, and I could not confirm any shows a live trade-off slider with
                 the steel cost. Tekla's numbering groups identical parts; that is the
                 'exact twins' step here, not the tolerance step.
               - Revit has no mark-merge; its Mark parameter is typed by hand or by an
                 add-in.
             So the 'exact twins' part is available in paid tools; the tolerance merge
             with an IS-safe governing member, a visible cost, and a browser tool that
             keeps the file on the device is the gap I could not find. If a tool does it,
             it is likely paid and tied to one modeller.
             Why it makes people say wow: one slider turns 238 beams into 24 marks and
             tells you what it costs in kilograms and rupees; the 3D model lights up in
             group colours; and 'two days of Excel, now a slider' is a sentence every
             detailer understands. Good reel for LinkedIn or Instagram: 'G+14 tower,
             238 beams. Slide: 41 marks, 24 marks, 12 marks. Extra steel: 0 %, 1.9 %,
             6.4 %. Pick one. Done in a minute.'
description: Mark Merge reads the members and the steel each one needs, groups them under
             rules the engineer sets, and writes the result back. Grouping is plain
             arithmetic on data cad2bim already holds, so it runs fast on the device
             and nothing is uploaded (the cad2bim rule).
             1. Feature vector. For each member (beam, column, slab panel, footing) build
                a vector: section (b x D), span or height class, concrete grade, cover,
                and the required steel at the standard stations (beam: support left,
                midspan, support right, top and bottom; shear spacing by zone; column:
                longitudinal total and tie spacing; slab panel: bottom and top bars each
                way). Required steel comes from ETABS results via Detail Drop, from the
                rebar already in the model (0.66), or from a CSV.
             2. Hard rules (never crossed). Members group only if they share: type,
                section (or an allowed section family), grade, floor-type class (typical
                versus transfer / podium / terrace), and cover. Never merged, by default:
                cantilever beams, transfer beams, beams carrying a wall load tagged as
                heavy, edge beams with a different torsion class, columns of different
                seismic detailing zone (IS 13920 confining zones). The engineer can lock
                any member as 'unique'. These rules are a list in one editable file, so
                the office sets its own.
             3. Governing member. In each group, the detail given to everyone is the
                envelope: at every station the largest required steel in the group,
                rounded up to a real bar combination (for example 3-16 + 2-12) that meets
                IS 456 spacing, minimum and maximum steel, and the IS 13920 ductile rules
                where they apply. Shear: the smallest spacing in the group. Because it
                is an envelope, no member is below its own requirement. The tool checks
                this after the merge for every member and reports any failure (there
                should be none; the check is there as a proof, not a hope).
             4. Cost of the merge. For each member, extra steel = envelope bars minus its
                own, times length, times unit weight. Total and per group; rupees from
                the office's steel rate (a single number). Concrete and shuttering do not
                change, because merging works on reinforcement within a fixed section.
                Optionally allow 'section families' (230 x 450 and 230 x 500 merge to
                the bigger) and then concrete and shuttering extra is counted as well.
             5. The slider. Groups are built by agglomerative clustering on the steel
                vector, with 'extra steel %' as the distance. The slider walks the merge
                tree from 'exact twins' to 'as few as the rules allow' and shows the
                mark count and the penalty at each step. Standard choices are saved as
                presets (Exact only; Economy 2 %; Site-friendly 5 %).
             6. Apply. Marks are renamed by a pattern (B-1, B-2... by floor type, or
                B-T1-01), the group and governing member are stored as properties
                (a Pset 'C2B_MarkMerge': Group, Governing, ExtraSteelKg), the rebar in
                3D is regenerated for each member from its group's detail, and the
                BBS, schedules and shuttering list read the live model so they update
                by themselves (0.80.0). One undo step restores the earlier marks.
             7. Explain. Click a beam: 'B-12 (Level 5, grid C-D/3) is in group B-12 with
                4 other beams. It needs 3-16 top at the support; B-12-governing (Level 3)
                needs 3-20. All five get 3-20. Your beam gets +0.9 kg per metre.' A
                reason a person can read is what lets an engineer trust the result.
             Honest limits: this is a detailing-economy tool, not a design check. It does
             not re-run the analysis, and it does not decide that a member's required
             steel is right; it only gives members at least what the design said. If the
             design changes, run Mark Merge again. A merge that makes a beam heavier can
             matter for the building's seismic mass at the margin (it is very small, but
             it is reported as a total so the engineer can see it). The tool says
             'for engineer review' on every report.
implementation:
             Phase 0 (3 days): grouping of exact twins only, from the rebar already in the
               model. No cost, no slider. Proves that marks, schedules and BBS update from
               one change (they should, since they read one live model). The marks it
               finds are usable on day one. Test: one real project, count marks before
               and after, time both ways.
             Phase 1 (1 week): the feature vector, the hard rules file, the envelope and
               its post-merge check, the extra-steel table, and the report in Excel.
             Phase 2 (1 week): the clustering and the slider, with the 3D colour-by-group
               (the colour-by-parameter feature of 0.38.0 is reused, so a group is just a
               parameter value) and the presets.
             Phase 3 (1 week): Revit write-back through the bridge and a pyRevit button;
               agent commands through MCP; the explain card.
             Phase 4 (later): section families (merging sizes), columns by storey group
               (a 'column schedule by zone'), slab panels, footings.
             Where it sits in the repo: a new module beside the BBS code in
             packages/engine (grouping and envelope are pure functions of arrays, easy to
             test), a dock panel in apps/web, and commands in the agent helper. No new
             dependency; the clustering is under 200 lines (single linkage on a few
             hundred points needs no library). A test set that matters: members that
             must NOT merge (cantilever with a simply supported beam of the same
             section; confined column zone versus a normal zone), and a proof test that
             every member's merged steel is at least its own at every station.
             Checks before delivery: for three real projects, run it and verify (a) no
             member ends with less steel at any station, (b) the BBS total after equals
             the BBS total before plus the extra steel in the report, to the kilogram,
             (c) undo restores the exact original marks. Report the mark count and
             penalty measured, not claimed.
future update:
- **Section families:** merge 230 x 450 and 230 x 500 into one size when the extra
  concrete is cheaper than a second form set, with the concrete and shuttering cost shown.
- **Column zones:** group columns by storey bands (for example 'floors 1-4, 5-9, 10-14') so
  the column schedule becomes one table per zone, the way drawings are read.
- **Slab panels and footings:** the same slider for slab bar layouts (Slab Sketch) and
  isolated footings (a family of F1-F6 instead of 40 sizes).
- **Form-reuse score:** count how many times each form is reused across floors and show
  the shuttering cost per use; ties to Shutter Plan.
- **Cage library:** each group's cage becomes a saved cage (3D and bar list) for the
  Bar Optimizer and for the yard's cutting list, with QR tags on the cages.
- **Redesign diff:** when the analysis changes, show which groups changed and which
  members moved between groups (Change Radar's diff), instead of starting again.
- **Agent:** 'find the cheapest merge that keeps the beams under 30 marks and leaves the
  transfer level alone' runs the options and returns the ranked list with the cost.
- **Other codes:** the rules file is data; ACI 318, Eurocode 2 and BS 8110 rule sets.

### Improvement suggestions for existing tools in the repos

- **cad2bim (rebar / BBS):** store a per-member 'required steel' next to the 'provided
  steel' on the rebar record, so a merge (and any QA check) can compare them; today
  the model holds what is provided.
- **cad2bim (properties):** a reserved Pset namespace (`C2B_*`) for tool-written
  properties, with a documented way to read and clear them, so Mark Merge, Bill Painter
  and Rate Rocket do not collide.
- **cad2bim (colour by parameter):** allow a legend that lists the group names and the
  member count of each, clicking a name to select the group; this is the same control
  that Mark Merge, Pour Planner and Bill Painter would all use.
- **cad2bim (QA):** a check 'two members with the same mark but different steel', which
  is a common mistake in hand-merged drawings and costs nothing to add.
- **Sheet Sync:** its 'Smart Mark generator' (auto-numbering identical size, grade and
  reinforcement) is the exact-twins step of this idea; build them as one tool, so the
  mark rules are written once.
- **Detail Drop:** output the required-steel vector per station in its JSON schema (it
  already has a design-run schema) so Mark Merge can read it without a conversion.
- **pyRevit-addons (AnonGee):** a button that writes Mark, Group and Governing-member
  parameters from an exported merge report, for offices that merge in cad2bim and detail
  in Revit.
```
