# 20261104 Idea: Cover Cop — find which face of every beam, column, slab and wall touches soil, weather, water or fire, set the right clear cover and minimum grade (IS 456), repaint the 3D cover heatmap, update every bar length in the BBS in one click, count cover blocks, and check site covermeter readings against the model

## Entry 2

```
date-time:   2026-11-04 (scheduled run, second entry of the day)
idea name:   Cover Cop (Exposure-Aware Clear Cover + Minimum Grade Assigner, Cover Heatmap,
             BBS Length Re-Cut, Cover-Block BOQ, Covermeter Site Check)
potential:   High, and it fits the toolkit's rebar line. Wrong or uniform cover is one of the most common
             site defects: the drawing note says "cover 25 mm unless noted", and the toilet slab, the
             external beam face, the basement wall earth face and the footing bottom all get the same
             number, or the number is typed by hand per element. Modeller time saved (an estimate, not
             measured): 2 to 4 hours per typical G+10 to set cover element by element in Revit, and
             another 1 to 2 hours to re-cut and re-issue the BBS after the checker changes it. The bigger
             value is rework avoided: low cover means rust, spalling and a repair cost many times the
             cost of a cover block. Cover on the BBS also decides bar length, so it moves steel quantity
             and money (a 5 mm change on every stirrup of a frame is a real number).
```

### Description

**The pain (what a BIM modeller and the checker actually do today).**
- Cover is a parameter on each rebar set. Revit lets you set it per host or per face, but the person has to
  know which face is which: external, internal, in contact with soil, in a wet area, in a basement, near
  the sea. Most models carry one default, so the BBS and the 3D bars are wrong in places and nobody sees it.
- The rule itself is simple and written down: IS 456 Table 16 (nominal cover by exposure: mild 20, moderate
  30, severe 45, very severe 50, extreme 75 mm), cl. 26.4.2 (footings 50 mm, against earth), Table 16A
  (cover for fire resistance by hours) and Table 5 (minimum grade and maximum water-cement ratio by
  exposure). Applying it is a lookup that depends on *where each face is* — which the 3D model knows and
  the 2D drawing note does not.
- When the structural engineer changes the exposure ("basement is now severe", "this toilet is wet") the
  cover for dozens of elements changes. The modeller edits them by hand, then the BBS has to be
  re-issued, and drawings and schedules disagree for a week.
- On site the question is the other way round: the bar fixer or the QA engineer measures cover with a
  covermeter before the pour and writes numbers on paper. Nobody compares them with the model.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Slab Sketch, Tank Smith, Basement Box and Pile Pilot mention cover inside
  their own design of one element type. Detail Drop writes ductile detailing. Ratio Radar checks steel and
  concrete ratios. None of them reads the *whole model*, works out the exposure of each face, assigns cover
  and grade as one rule set, and re-cuts the bars. None takes covermeter readings from site.
- In cad2bim (from `CLAUDE.md` and `docs/`): "cover rules" are listed as part of Rebar in
  `docs/design/revit-gap.md` (row 22) and the rebar layout with cover is planned in `docs/design/etabs.md`
  (E4). The BBS engine (`packages/engine/src/bbs/`: `bar.ts`, `shape.ts`, `schedule.ts`, `standards.ts`)
  already turns bar shapes (IS 2502 codes) and member geometry into cut lengths, and the QA module
  (`packages/engine/src/qa/`) already lists findings that can be selected, isolated and zoomed. So the
  receiver for a cover value exists; the *decider* does not.
- In the market (not exhaustively checked): Revit sets cover per host by hand; rebar add-ins mostly take
  one cover per run. Design software applies cover to its own design model, not to the Revit or IFC
  model. Covermeters are sold with their own app that shows readings, not the model. I could not find a
  free browser tool that reads a model, derives exposure per face and re-cuts the BBS; treat this as
  unverified.

**The idea.** A **Cover** panel in the cad2bim web viewer (and the same through the MCP agent and Python
console, so the BIM manager can script it). Nothing leaves the device.

1. **Exposure per face, derived, not typed.** From the 3D model cad2bim finds, for every concrete element,
   which faces are: against soil (below the ground or inside a basement wall's earth side), external (the
   outer face of the envelope, from a ray cast against the storey outline and openings), inside a wet area
   (rooms named toilet, kitchen, utility, terrace, tank — from room or space names, and the person can
   correct the keyword list), inside a plant room, or plainly internal. The result is a face label per
   element, shown in 3D as a **cover heatmap** (blue = earth, orange = external, teal = wet, grey =
   internal) with the usual select / isolate / zoom.
2. **A rule table the team owns.** A small, editable table that starts as IS 456 Table 16 and 16A and
   cl. 26.4.2, with the project's own override per project or per client: exposure class by label, cover
   by element type (footing, beam, column, slab, wall, stair), fire-rating hours by building use, and the
   minimum grade and maximum water-cement ratio from Table 5. The person states the **governing
   environment** once ("coastal", "moderate city") and the table fills. Every number shows the clause it
   came from; the tool never invents a value that is not in the table.
3. **One click to apply, with a preview.** "Show me what changes": a before / after list per element
   (cover 25 to 40 mm on the external beam faces), the count of elements, and the effect on steel — new
   total kg and the change against the current BBS. If the person accepts, cad2bim writes the cover back
   into the model's rebar parameters (the IFC writer already keeps ids and writes parameters), re-cuts every
   affected bar in the BBS (stirrups and ties change length with cover; main bars change by the end covers),
   and updates the BOQ and the bar schedule. Undo is one step.
4. **Minimum grade check.** Next to cover, a second finding type: elements whose assigned grade is below
   the Table 5 minimum for their exposure (for example M20 in a severe zone). The finding says what the
   rule asks for and offers to set the grade, which feeds the concrete BOQ.
5. **Cover-block BOQ.** The number and size of cover blocks (or spacers) per element type, from cover and
   bar layout — the item that usually gets forgotten in the tender and then bought on site at a higher
   rate. Output as a CSV for the purchase team.
6. **Covermeter site check.** The person imports a CSV (element mark or QR tag, face, measured cover — the
   format most covermeter apps can export; or types them on the phone through the existing site pages).
   Cad2bim compares each reading with the required cover plus tolerance (IS 456 allows -0 / +10 mm for
   nominal cover) and shows the pass / fail on the 3D model: a red dot on the face, the number, and a
   "re-fix before pour" list that goes into the pour card (Site QR Pass, Pour Planner) and into BCF issues
   (IDS QA Autofix route).

**Why it is practical.** It replaces a hand lookup repeated per element with a rule that the whole team
shares, and it turns a drawing note into a number each bar actually uses. A checker can read one page
instead of opening 300 beams.

**Wow factor (what the person sees and can post).**
- A 3D model that turns into a four-colour exposure map in one click, with the *outside skin glowing
  orange* — the first time most teams see where the weather really touches their frame.
- The slider "Coastal ↔ Normal" that changes the whole building's cover and shows the extra steel in kg
  and rupees live.
- A 20-second screen recording: "same G+10, cover set properly in 12 seconds, BBS re-issued, 480 kg
  difference" — a clear before / after for a LinkedIn or Instagram reel.
- Site side: scan the member QR, type the covermeter number, and the dot turns green or red in the model.

**Trend link (a judgement, to be checked against live feeds before posting).** Structural-failure, spalling
and monsoon-damage clips and "why does my building have rust stains" posts repeat every year on short-video
feeds; durability and embodied-carbon discussions are active in AEC. The post writes itself: "cover is the
cheapest 25 mm in the building".

### Implementation

Keep it inside cad2bim's own code (no GPL; see `CLAUDE.md`). Phases, smallest useful first:

1. **Rule table + manual labels (1 to 2 weeks).** A JSON rule table in `packages/engine` (IS 456 Table 16,
   16A, cl. 26.4.2, Table 5), with clause references. A Cover panel in `apps/web` where the person assigns
   an exposure label to a selection (by storey, by element type, by selection) and sees the heatmap.
   Reuses colour-by (already reads the live model).
2. **Re-cut the BBS.** Extend the cut-length functions in `packages/engine/src/bbs/` to take a per-face
   cover instead of one value; test against hand-calculated bars (stirrup, main bar with bends, per IS 2502
   shapes). Keep the old single-cover path as default so nothing changes for people who do not use it.
3. **Derived exposure.** Face classification from the geometry the engine already holds: below-ground test
   against the storey and site level, ray cast outward from each face for "external", room-name keywords
   for "wet". Always show the evidence (why a face was called external) and let the person override; wrong
   guesses are corrected once and remembered in the project.
4. **Findings and write-back.** New QA findings (`qa/checks.ts`): "cover below rule", "grade below
   minimum", "cover changes BBS by X kg" with select / isolate / zoom; accept writes parameters through the
   IFC writer, one undo step.
5. **Cover-block BOQ and CSV.** Per element type from cover and bar layout.
6. **Covermeter import.** CSV mapping dialog (element mark, face, reading), comparison, 3D dots, pour-card
   line. Phone entry reuses the site pages, nothing else.
7. **Agent and console.** Expose `cover.apply`, `cover.check`, `cover.report` through the one cad2bim API
   (0.84.0) so an AI agent or a Python script can run the same rules.

Checks and honesty: the tool applies tabulated values; it is **not a design check** and the structural
engineer signs the rule table once per project. State in the UI what was measured (reading from site) and
what was derived (exposure from geometry and names). Test on two public sample IFC models first; people's
drawings never go in git.

Risks: room names are inconsistent (mitigate: keyword list plus manual override); curved or complex
envelopes confuse the ray cast (mitigate: fall back to "ask"); fire-rating needs a use-class input the
model rarely holds (mitigate: ask once per project). Effort estimate (an estimate): about 5 to 7 weeks for
steps 1 to 5 with one developer.

### Future update

- **Cover-aware congestion:** feed the new cover into Joint Congestion Doctor so that bar spacing at
  beam-column joints uses the real cover, not a default.
- **Revit write-back** through the bridge so the same exposure map and cover land in the Revit rebar sets.
- **Durability report:** a one-page PDF for the client (exposure map, cover table with clauses, grade table)
  as part of the design basis and handover (Handover Hub).
- **Other codes:** BS EN 1992 nominal cover (c_nom = c_min + Δc_dev), ACI 318 cover and the Gulf-region
  exposure classes, as extra tables in the same panel.
- **Repair-aware mode:** for Audit Eye findings (spalling, rust), read the existing cover from the
  covermeter data and propose a repair class and quantity.
- **Cover from photos:** a phone photo of the cover block spacing before the pour, checked by vision for
  missing blocks, pinned to the member.
- **Learning from the team:** remember which exposure labels the checker overrides most often and suggest
  them next time.
