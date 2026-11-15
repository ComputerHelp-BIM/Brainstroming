# 20261115 Idea: Stage Shield — the floor you are about to pour is loaded onto the floors below it. Stage Shield reads the 4D plan, the slab ages and the prop layout, and says, floor by floor and day by day, "safe to pour above", "add back-props here", or "wait 3 days" — with the weak slab and the weak column lit up in 3D

## Entry 3

```
date-time:   2026-11-15 (scheduled run, UTC)
idea name:   Stage Shield (construction-stage load check for slab-on-slab building: props, back-propping, early-age
             strength, punching at columns, and a "safe to pour above" gate tied to the 4D plan)
potential:   High for any team that builds or models a multi-storey cast-in-situ RC building, and the idea is
             fully inside this toolkit's lane (IFC + 4D + rebar + QA). Most striking and propping decisions on site
             are made from habit ("strike at 7 days, keep two floors propped"). Yet the heaviest load a slab ever
             carries early in life is often NOT the finished building. It is the wet floor above, passed down
             through props, onto a slab that is only a few days old. When that goes wrong it is a cracked slab, a
             punching failure at a column, or a collapse. When it goes right but too carefully, a floor cycle is
             lost: 2 or 3 extra days per floor on a 30-floor tower is a month.
             Time and money saved: NOT measured here. The honest way to find out: take one finished tower, run the
             tool on its real pour dates, and compare its prop plan with what the site actually did. If the site
             kept props longer than the check needed, that is the saving. Do not quote days until a team has done it.
             Market check (2026-11-15, one standard search, not exhaustive; verify before marketing):
               - The method is old and published (Grundy-Kabaila 1963; Gross & Lew, NBS; McGurl & Johnston, spreadsheet
                 method). Papers and Excel sheets exist.
               - Software that does it is mostly desktop FEM with a "construction stages" module (for example
                 StruSoft FEM-Design, with a trial) and a few small commercial reshoring tools (Apatos Reshoring,
                 a hackathon-stage page). Some formwork suppliers do the check in-house as part of selling props.
               - I did NOT find a free browser tool that takes the IFC and the 4D schedule and answers the
                 daily site question, and I did NOT find one that draws the answer on the 3D model. That is the gap.
                 This is one search; treat "not found" as "not found", not as "does not exist".
             Extends tools already in this repo (it does not repeat them):
               - Cube Clock: "can we strike this slab today?" from cubes. Cube Clock's own future-update list names
                 "compare the required striking strength with the construction-stage load from the slab above" as
                 the next step. Stage Shield IS that step, built as its own tool: it computes the load that
                 Cube Clock's strength is compared against.
               - Shutter Plan: props and plywood layout for one pour. Stage Shield checks the whole stack of floors
                 over time and where props must stay.
               - Pour Planner (4D) and Raft Run: when and how to pour. Stage Shield says whether the floor below
                 is ready to take that pour.
               - Quake Preview / Load Path X-Ray: completed-building behaviour. Stage Shield is the building while
                 it is still being built, which those do not look at.
             Trend: LinkedIn and press this month keep returning to site safety, to AI that "explains its sums",
             and to digital twins that help on site and not only at handover. Stage Shield is rules and arithmetic
             (no AI guess), shows every number, and sits on site safety. I could not measure LinkedIn or Instagram
             counts directly, so this is an impression from press and vendor pages, not a count.
             The wow: press play on the 4D model. As each floor is poured above, the slabs below flicker
             green / amber / red by how much of their early-age capacity is used. The red one is pinned with a
             sentence: "Level 4 slab, 6 days old, carries 1.3x its allowed early load when Level 7 is poured.
             Keep Level 5 and Level 6 props, or delay the Level 7 pour to day 9." A bar at the top reads
             "28 stages checked, 2 need action". Drag the pour date and the colours move at once.
```

### Description

**The pain.**
- In slab-on-slab construction, the new floor's weight does not go to the ground by itself. It goes down through
  props into the floors below, which share it by stiffness, and those floors are young and weak.
- The decisions that govern it are made by rule of thumb: how many floors of props stay, when to strike, whether
  to add back-props under a transfer area or a heavy slab. The rule often came from a different project.
- The engineer of record checks the finished building. The construction-stage check is often left to the
  contractor, the formwork supplier or nobody, and the result is a PDF or a hand sum that does not follow the
  schedule when the schedule slips.
- When the schedule changes (a monsoon week, a late steel delivery), the prop plan is not checked again. The very
  moment the pour order changes is the moment the risk changes.

**The idea in one line.** Put the pour dates, the slab ages and the prop layout in one place, work out the load
each slab sees at each stage with a published method, compare it with what that slab can carry at that age, and
draw the result on the 3D model.

### How it works (user view)

1. **Open the model and the 4D plan.** The IFC (slabs, beams, columns with levels) and the pour schedule that cad2bim's
   4D already has. No schedule yet? Type the floor cycle once (for example "one floor every 7 days").
2. **Set the site practice in four fields.** Number of floors propped under the pour, when props are struck, the
   props' stiffness class (a table the user can edit, with a "typical" start), and the site construction
   load (workers, pump and pipe, stacked material) as a number per m². Each field has a plain-language hint.
3. **Press Check.** Stage Shield makes one "stage" for each pour. For each stage it works out, for each floor
   under the pour, the share of the wet load that floor takes, and the age of that floor's concrete on that day.
4. **Read the board.** One row per stage, one coloured cell per floor below:
   - *Green:* load used is under the early-age capacity of that slab on that day.
   - *Amber:* under the limit, but within a user-set margin.
   - *Red:* over the limit. A sentence says which check failed (flexure, shear/punching at a named column) and by
     how much, and offers three fixes to try: **Add back-props** at a named floor, **Delay the pour above** by N days,
     **Keep props one more floor**. Each fix re-runs the check right away.
5. **See it in 3D.** The 4D play bar drives colours on the slabs. Click a red slab: the column that punches is
   outlined, the pour above is shown ghosted, the props in the load path are drawn as thin lines.
6. **Safe-to-pour-above gate.** Each future pour gets a status in the 4D list: *OK*, *OK if props kept*, or *Not yet*,
   with the earliest safe date. The date moves if the cube results change (when linked to Cube Clock).
7. **Hand-off.** A one-page PDF per stage (assumptions, numbers, result, who checked), and an Excel of the board.
   The page says clearly "screening check by simplified method; not a substitute for the engineer's construction-stage
   design". It carries the method name, the version and the inputs, so a checker can repeat it.

### Why a BIM team would use it

- **Modeller / BIM lead:** the 4D model gains a meaning beyond animation: it becomes a safety and cycle-time tool, and
  the same model that makes the BBS also answers "can we pour Level 7 tomorrow?".
- **Site engineer / planning engineer:** one screen to answer the daily question, and a reason to give the
  formwork supplier when props are requested or released.
- **Structural designer:** a quick view of which slabs are loaded hardest during construction, before issuing the
  drawings (a note on the drawing: "keep L5 props until L8 is cast").
- **Project manager:** the cost of prop-hire days against the cost of a delayed pour, shown by one slider.
- **Why it is a wow:** construction-stage risk is invisible in normal BIM. Seeing a slab turn red during the 4D play
  is a different kind of demo from another coloured clash.

### Implementation

Everything below is a design for the next person to pick up. Nothing in cad2bim's source was touched.

**Where it plugs in (no new foundation needed).**
- `packages/engine`: reads the IFC for slabs, beams, columns, levels, spans, thickness, concrete grade, plus the 4D
  tasks already planned in the 4D roadmap item. The stage calculation is a **pure function** (inputs in, numbers
  out) with no UI and no network, so it is testable with a few known worked examples.
- `apps/web`: a new panel in the dock beside 4D (the "Stage board"), colours on the 3D through the same path the
  Warning Doctor and QA pins use, a "safe to pour" column in the 4D list.
- `packages/mcp`: one more read-only tool, so the AI agent can answer "Is Level 7 safe to pour on the 20th?" with
  the board's numbers and cite them, rather than guess.
- Offline and on-device like everything else: nothing is uploaded.

**Method, kept simple on purpose (version 1).**
1. *Rigid-slab / equal-stiffness shore-load method* (the Grundy-Kabaila family of methods), which assumes props are
   stiff compared with the slabs and slabs share load by their own stiffness. It is the standard first-pass method and
   needs few inputs: floors propped, slab self-weights, construction load, the stage order.
2. *Age-dependent capacity.* Early strength from the grade and age by a code relation (IS 456 gives a growth table
   and the cube result from Cube Clock overrides it). Flexure and shear capacity from the strength at that day.
3. *Punching shear at columns* using the early-age strength: the check that most often governs flat slabs; for
   beam-slab floors, flexure and shear at the beam face.
4. *Prop and back-prop forces* (kN per prop or per m²) compared with a user-entered prop capacity.
5. *Staging.* For every pour, the previous pours' ages come from the 4D dates. Strike dates are the user's rule, or the
   output of Cube Clock when linked.

**Build order (smallest useful first).**
1. A typical-floor model: identical floors, flat slab or beam-slab, one pour per floor, equal prop rule; board only
   (no 3D colours). Compare against two published worked examples (the research papers cited below contain tables to
   check against) before anything else is built.
2. Read real slab areas, spans and thickness from the IFC; per-slab (not per-floor) checks; 3D colours on the slabs.
3. Real 4D dates; "safe to pour" column; fix buttons re-running the check.
4. Punching at named columns with the column's loaded area from the model.
5. PDF stage sheets and Excel export; MCP read-only tool.
6. Cube Clock link: replace table strengths with cube-based strengths.

**Tests to write first.** (a) Worked examples from the published method give the same shore-load shares within a
stated tolerance; (b) changing the floor cycle from 7 to 5 days raises the lower-slab load in the direction the
method says; (c) setting the grade high enough turns every cell green; (d) a stage with no props under it fails loud,
not silent; (e) the board with no schedule gives a clear "no dates" message and does not guess dates.

**Cost to build.** Estimate, not measured: the calculation core is small (a few hundred lines) and easy to check
against the published examples; the larger part is the 4D hook, the panel and the PDF sheets. The risk is not the
arithmetic. The risk is that users trust a screening check as if it were a design, so the wording and the
sign-off box on the sheet are part of the build, not an afterthought.

### Future update

- **Finer analysis.** Replace the rigid-slab method by a stiffness model of the slab stack (age-adjusted slab
  stiffness, prop stiffness, creep), checked against the simplified one and flagged where they differ.
- **Cube Clock link.** Live cube strengths move the "earliest safe date"; a low cube turns a green stage amber.
- **Shutter Plan link.** Prop positions from the formwork layout, so the load-per-prop check uses the real grid.
- **Transfer floors, podiums, cantilevers.** Special cases where the usual rule of thumb is least reliable.
- **Sensor option.** If the site has maturity sensors or prop load cells, a CSV import replaces the table with
  measured values (kept optional; no hardware is assumed).
- **Reel.** A 20-second 4D clip with the red slab flashing and the fix applied (see Model Reel).
- **Other codes.** ACI 347 / ASCE 37 / EN 12812 style construction-load rules as selectable profiles.

### Reel idea for LinkedIn / Instagram

15 seconds. A tower rises in 4D. A slab three floors down blinks red with the caption "Level 4: 6 days old,
carrying a pour 3 floors up." A finger drags the pour date one day later; the red goes green; the caption reads
"One day. No new props." Closing frame: "Checked before the pour, not after the crack."

### Honest limits

- This is a **screening** tool by a simplified published method. It does not replace the construction-stage design by
  the engineer of record or the formwork designer. It should make that conversation start earlier, with numbers.
- Results depend on inputs the model does not hold: actual prop stiffness, actual striking dates, actual stacked
  material. Defaults are labelled defaults, and the sheet lists every one.
- The rigid-slab method can be unconservative for some layouts (for example very flexible slabs or soft props). The
  tool must say so when inputs fall outside the method's range, not hide it.
- Time saving and risk reduction are **not measured**. They are hypotheses to test on one finished project.
- Code clauses and formulas named here (IS 456 strength growth, punching rules) must be checked against the current
  codes before build; I did not re-read the clauses in this session.

### Sources (checked 2026-11-15)

- ACI abstracts: [Analysis of Shoring Loads and Slab Capacity for Multistory Concrete Construction](https://www.concrete.org/publications/internationalconcreteabstractsportal/m/details/id/1772)
- ACI abstracts: [Spreadsheet Analysis of Load Distribution in Shoring and Reshoring Systems](https://www.concrete.org/publications/internationalconcreteabstractsportal/m/details/id/51683926)
- ACI abstracts: [Construction Load Analysis of Slabs and Shores Using Microcomputers](https://www.concrete.org/publications/internationalconcreteabstractsportal/m/details/id/3247)
- University of Ottawa thesis on flat-slab shoring and reshoring (Excel-based approach with age-adjusted stiffness and creep): [ruor.uottawa.ca](https://ruor.uottawa.ca/bitstreams/b1372dce-9aa5-4276-80a2-31903bb73269/download)
- Commercial examples found by the search (paid or trial): [StruSoft FEM-Design construction stages](https://strusoft.com/?p=8565), [Apatos Reshoring (Devpost page)](https://devpost.com/software/apatos-reshoring)
- In this repo: Cube Clock (20261101), Shutter Plan (20261017), Pour Planner (20261014), Raft Run (20261112), Model Reel (20261018).
