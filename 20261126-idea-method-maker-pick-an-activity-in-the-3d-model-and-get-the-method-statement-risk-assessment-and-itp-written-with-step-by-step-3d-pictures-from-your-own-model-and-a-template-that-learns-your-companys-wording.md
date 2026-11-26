# 20261126 Idea: Method Maker — pick an activity in the 3D model and get the method statement, risk assessment and ITP written, with step-by-step 3D pictures from your own model and a template that learns your company's wording

## Entry 1

```
date-time:   2026-11-26 (scheduled run, fourth entry of the day)
idea name:   Method Maker — "Every site activity needs a method statement. The model already knows the activity."
             Select an activity in the cad2bim model or the 4D plan ("L5 slab pour", "Column cage fixing, grid
             C-F / 3-7", "Raft excavation and PCC", "Cuplock scaffold, east face"). Method Maker writes the
             submission pack in the company's own format: (1) Method Statement with numbered steps, (2) Risk
             Assessment (hazard, who is exposed, control, residual risk), (3) Inspection and Test Plan with hold
             and witness points, (4) resource list (gang, equipment, material quantities). Every step has a 3D
             picture captured from the real model with the working elements highlighted, and the numbers (m3, kg,
             m2, floor height, span, crane reach) come from the model, not from a copied old document.
             Output: Word and PDF, on the device.
potential:   High. Method statements, RAMS and ITPs are asked for on almost every contract, per activity and per
             revision, and in practice the site engineer or BIM coordinator copies the last project's document
             and edits it. Estimates below are mine, not measured:
             - Today: 2-4 hours per method statement to copy, re-word, re-number, re-measure and screenshot;
               the document often keeps the old project's quantities or a wrong floor height, and the
               consultant returns it with comments, which costs another round.
             - With Method Maker: a first complete draft in about 2 minutes, then 15-20 minutes of review.
               A pack of 10 activities drops from about 3 days to about half a day.
             Why it wins: it uses data nobody else uses for this. The model knows the quantity, the height, the
             neighbours, the hazards (edge, opening, crane, excavation depth); the 4D plan knows the sequence
             and the dates; the BBS knows the steel. A generic template cannot, and a text-only AI cannot see
             the model. Rewording to the client's format is a one-time job.
             Market: to my knowledge (not verified for this run), method statements and risk assessments are
             written in Word or bought as paid safety-management or RAMS-generator subscriptions that work from
             text forms, not from a BIM model. Revit and the common 4D tools make no method statement. A free,
             on-device generator that reads the IFC and pictures the steps is not one I know of.
             Trending: AI-written safety documents and "RAMS in minutes" are common talking points among
             construction-tech posts on LinkedIn, and a short reel of a method statement building itself
             step by step over the 3D model suits Instagram. Treat this as a signal of interest, not a
             measured trend.
```

### Description

**The pain (daily life of a BIM modeller, planning engineer and site engineer).**
- The client says "submit the method statement for the L5 slab pour by Thursday." Someone opens the previous
  project's file, changes the name and date, fixes the quantities by hand, and screenshots the model for the
  pictures. The pictures are wrong because the model has moved on.
- The risk assessment says "work at height" and "excavation" in the same words on every document, because
  nobody has time to find the real hazards on this floor. The real ones (an open slab edge at the lift core,
  a 6 m deep raft pit next to a boundary wall, a crane swing over the neighbour) are in the model.
- The ITP lists the same hold points every time, with no link to what is being built.
- When the schedule or the design changes, the pack is stale and re-writing it is what everyone avoids.

**What Method Maker does.**

1. **Pick the activity.** Select elements in the model, a task in the 4D plan, or a saved selection. Method Maker
   names it ("RCC slab, level 5, zone B, 42 m3, 186 m2, thickness 150 mm") from the elements.
2. **Choose the activity type.** A small library of starter recipes (editable): excavation, PCC, footing and raft,
   column and wall, slab and beam, scaffold and formwork, rebar fixing, concrete pour, curing, de-shuttering,
   masonry, plaster, waterproofing, MEP sleeve and hanger, lift installation, facade fixing. The recipe is a
   plain-text list of steps with tags; the user and the team can edit it.
3. **Fill from the model.** Each tag is filled by a rule: quantities from the BOQ, bar sizes and weights from the
   BBS, prop spacing and plywood area from the formwork data, pour size from the pour planner, crane and reach from
   the site layout, floor height and opening list from the model. Anything it cannot find is left as a visible
   blank ("[needs: pump make]"), never guessed.
4. **Find the hazards.** Method Maker looks at the geometry: slab edges and shaft openings (fall), pit depth
   next to a boundary (collapse, neighbour), working near a crane path or an overhead line, hot work near a
   shuttering, a confined space (tank, lift pit), heavy lifts over a public road. Each becomes a row in the risk
   table with a standard control from the user's list and a pin in 3D.
5. **Draw the steps.** For each step it captures a clean picture from the model: the working elements in colour, the rest
   ghosted, a callout and a grid reference. A 6-picture sequence for a pour ("1. Pump position, 2. Pour start
   corner, 3. Pour direction, 4. Construction joint, 5. Finishing, 6. Curing") is made in seconds.
6. **Build the ITP.** Hold, witness and review points come from the recipe, with the element list and the check
   values from the model ("cover 25 mm, from the cover table; bars as BBS marks B12-B48").
7. **Export.** The user's own Word template (header, logo, document number, revision table) is filled. A PDF is
   made alongside. A revision number and a "what changed since Rev 0" list are added when the pack is re-made.

**The company's wording.** The first time, the user drops three past method statements the company is proud of. Method Maker reads their
headings, tone, step wording and standard controls into the recipe library on the device ("Rev table at top, 'Scope /
Resources / Sequence / Safety / Quality' headings, 'shall' not 'will'"). Next packs sound like the company. No
document leaves the device.

**Safety and honesty rules.**
- It writes a **draft for a qualified person to review and sign**. Every page says so. It does not replace
  the site safety officer, the structural engineer or the temporary-works designer; for scaffolding, props and
  deep excavation it names the design that must come from them.
- Hazards found by geometry are marked "found from the model"; hazards from the library are marked "standard".
  A hazard the model cannot show (weather, public, night work) is left for a person to add.
- Starter controls are labelled *example only*; the company's list replaces them.

### Implementation

1. **Recipe format (1 week).** A recipe is a small JSON or Markdown file: steps, tags, hazard rules, ITP points.
   Ship 6 recipes first: column, slab pour, rebar fixing, excavation, scaffold and formwork, masonry.
2. **Model facts (1 week).** A function that takes a selection and returns the facts: quantities by class
   (existing BOQ code), heights, openings, edge lengths, neighbours within a distance, depth below ground, levels.
   Reuse the engine's QA and BOQ readers; no new IFC reading.
3. **Hazard rules (1 week).** Plain geometry tests: exposed slab edge with no wall or parapet; opening larger than
   a set size; excavation deeper than a set depth; distance to boundary or neighbour; crane radius overlap
   from the site-layout file. Each rule has an id, the text of the hazard and a pin.
4. **Step pictures (1 week).** Use the viewer's section box, highlight and ghost modes with a scripted camera
   (the same camera path code planned for Model Reel). Render to PNG at the page size, callouts drawn as SVG.
5. **Documents (1 week).** DOCX through a small client-side writer, PDF from the same layout. A user template
   is a DOCX with `{{tags}}`; Method Maker fills it and leaves the styles alone.
6. **Wording learner (1 week).** Read the user's sample DOCX files on the device, extract headings and common
   phrases, and offer them as the template and the phrase list. Rules, not a hosted model; an optional
   "tidy the wording" step can use the person's own key, as the AI chat does (nothing sent unasked).
7. **Checks and release (1 week).** A sample project in `docs/examples/`, a test that the facts and hazards
   for it are stable, a design note `docs/design/method-maker.md` in the app repo, and an entry in the roadmap.

Total: about 6 weeks for one developer. Everything runs in the browser; no file leaves the device.

### Future update

- **Live pack:** when a revision changes the elements, re-make the pack, show a redline of what changed and flag
  the pages that need re-approval (pairs with Change Radar and Issue Desk).
- **Toolbox talk and induction cards:** one page per activity with the three hazards found from the model, in the
  crew's language (pairs with Bhasha Sheet and Edge Guard).
- **Permit to work:** hot work, confined space, work at height and excavation permits pre-filled from the same
  facts, with a QR link on the 3D pour card (pairs with Site QR Pass).
- **Photo proof:** the ITP point opens a phone form; photos land on the element (pairs with Photo Placer and
  Second Eyes).
- **Approval tracker:** status lights per pack in 3D, "work blocked until method statement approved" (pairs with
  Submittal Scout and Ready Board).
- **Video:** the picture sequence plays as a 30-second 4D clip for the toolbox talk (pairs with Model Reel).
- **Lessons loop:** incident and near-miss notes from site feed the recipes so the next pack carries them
  (pairs with Repeat Offender).

### Related ideas in this repo

Complements Edge Guard (fall hazards and scaffold), Cage Manual (rebar fixing steps and ITP for one cage),
Pour Planner and Raft Run (pour sequence), Ready Board (is the work ready) and Submittal Scout (approvals). It differs from them
because it writes the document the client actually asks for, for any activity, from one selection, in the
company's own format. None of them produces a method statement and risk assessment pack.
