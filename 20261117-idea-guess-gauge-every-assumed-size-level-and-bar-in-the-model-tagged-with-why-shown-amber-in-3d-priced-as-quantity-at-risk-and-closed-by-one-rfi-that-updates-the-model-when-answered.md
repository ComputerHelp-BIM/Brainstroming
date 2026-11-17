# 20261117 Idea: Guess Gauge — Every size, level or bar the modeler had to assume (the drawing was silent or unclear) is tagged in the model with a reason, shown amber in 3D, counted as "quantity at risk" in the BOQ and BBS, and closed by one RFI. When the consultant answers, the model, quantities and sheets update and the amber turns green.

## Entry 1

```
date-time:   2026-11-17 (scheduled run, UTC; fourth idea of the day)
idea name:   Guess Gauge (assumption register for the model, in the browser, nothing uploaded)
potential:   High, and it fixes a quiet daily pain. Real drawing sets are never complete. The modeler
             guesses: "beam size not shown, put 230x450", "level not given, copy the floor below",
             "bar dia unclear, take 12 mm". The guess lives in the modeler's head. A week later the
             BOQ, the BBS and the 3D hold it as if it were fact, and it is quoted to the client or
             sent to site. When the modeler is on leave or leaves the company, nobody knows which
             parts are real. Time saved: NOT measured. First test: on one real drawing set (kept out
             of git), a modeler logs every guess for two weeks with the tool, and we count (a) how
             many guesses, (b) what share of concrete and steel they touch, (c) how many were later
             wrong. Do not quote hours, rupees or "errors prevented" until that is done.
             Market check (2026-11-17, two searches; not exhaustive, so "I did not find one", not "none"):
               - The TU Munich "EarlyBIM" work (Abualdenien, Zahedi, Borrmann) built a Revit plug-in
                 prototype with a multi-LOD meta-model that shows missing values and uncertainty and
                 logs changes. Academic, early design stage, not a product.
               - Edinburgh research (Scan-to-BIM confidence index) scores how reliable each modelled
                 element is. Research, for point-cloud models.
               - BIMForum LOD and ISO 19650 LOI describe how developed a model is, but have no
                 per-element "guessed or confirmed" flag. A 2026 ITcon paper (Level of History) says
                 the same gap exists for heritage models.
               - I did not find a commercial or free tool that tags each guessed element with a reason,
                 prices the guess as "quantity at risk" in the BOQ/BBS, and closes it through an RFI
                 that updates the model. I did not check Revit add-in stores one by one.
             Extends tools already in this repo (does not repeat them):
               - RFI Forge: finds conflicts in the drawings BEFORE modelling and makes RFIs. Guess Gauge
                 covers what the modeler had to decide DURING modelling, including things no conflict
                 shows (a silent drawing). It reuses RFI Forge's RFI list and numbering.
               - Trace Lens: explains lines the pipeline skipped. Guess Gauge records the values the
                 pipeline or the modeler filled in by default (a type per size, a default height).
               - Twin Check, Sheet Sentry: check consistency. Guess Gauge checks certainty, a different
                 question.
               - Rev Merge, Change Radar, Mark Keeper: when a new revision arrives, a guess the new
                 drawing now states is auto-closed, and a guess it contradicts is flagged.
               - Ask & Watch, Measure Right, Ratio Radar: a quantity can show "of which assumed: 9%".
               - Scope Scout, Basis Book: the assumptions list goes straight into a design basis or a
                 quote's "assumptions and exclusions" page.
               - Model Pulse: shows who built what; Guess Gauge shows how sure each part is.
             Trend: AI agents and MCP are the standing BIM talk on LinkedIn; an agent that builds a
             model fast makes unmarked guesses MORE likely, so an honest "how sure" layer matters more.
             I could not measure LinkedIn or Instagram counts; this is not a social-media claim.
             The wow: press "Gauge". The model recolours: green = read from a drawing, amber = assumed,
             red = assumed AND contradicted. A dial at the top says "Concrete 612 m3: 91% confirmed,
             9% assumed (55 m3). Steel: 74% confirmed." Click amber beam B14: "Size assumed 230x450 —
             schedule row missing — RFI-07 open — 3 sibling beams share this guess." Type the
             consultant's reply "B14 = 230x600" into RFI-07: 4 beams grow, the BOQ and BBS change, a
             diff card shows "+1.8 m3, +96 kg", and the dial moves to 93%.
```

### Description

**The pain.**
- Every drawing set has holes. Modelers fill them with sensible guesses, and that is right: the model must
  go on. The harm is that the guess leaves no trace.
- The BOQ and BBS then look exact to the last kilogram while part of them is a guess. Managers and clients
  cannot tell which part.
- RFIs get written from memory, late, or never. When the answer comes by email or WhatsApp, someone has to
  remember which elements to change.
- A new modeler inherits a model with no memory of what was assumed.

**The idea.** Make the guess a first-class thing, with four moves.
1. **Capture, almost free.** When the modeler (or the DXF → 3D pipeline, or an AI agent over MCP) creates
   or edits an element with a default or a copied value, a one-tap chip appears: *Why?* with choices
   "not on drawing", "unclear", "copied from level below", "typical", "client verbal", "other". The tool can
   also guess the reason itself (the pipeline knows when it used a default) and asks only to confirm.
2. **See.** A Gauge mode colours the model green / amber / red and shows a dial per quantity. Filters:
   by reason, by level, by who guessed, by age.
3. **Price the doubt.** Each BOQ and BBS line gets an "assumed share" column and a range, e.g.
   "Beam concrete 48.0 m3 (of which 6.2 m3 assumed; if the beam is 600 deep: +1.8 m3)". The range uses a
   simple what-if the modeler enters ("could be 450 to 600"), never a made-up probability.
4. **Close.** Group guesses that share a cause (all beams with the missing schedule row) into one RFI
   (via RFI Forge), numbered, with 3D snapshots. The reply is typed or pasted once; the tool applies it to
   every element in the group, shows the diff, and turns them green with the RFI number and date.

**Smart parts.**
- **Guess groups.** Fifty beams guessed for one reason become one question, not fifty.
- **Stale guesses.** An amber element older than N days that is already in a sheet issued for construction
  is red. This is the real danger: a guess that went to site.
- **Revision-aware.** A new DXF revision (Rev Merge) that now states the size closes the guess with a
  match, or raises a conflict when the drawing disagrees with the assumed value.
- **Issue gate.** Issue Desk can warn: "This sheet shows 14 assumed elements. Issue anyway?" The sheet's
  title block can print "Assumptions open: 14 (see list)". The modeler decides; the tool only reminds.
- **Assumptions page.** One click writes a clean "Assumptions and open points" list (PDF / Excel) for the
  client, with RFI numbers. It is also the handover note for the next modeler.
- **Agent-honest.** Elements created through MCP tools carry `assumed: true` by default unless the agent
  cites a source (drawing, schedule row, message). The agent is told this in the tool description.

### Implementation

- **Data:** an IFC property set per element, `Pset_C2B_Certainty` with `State` (Confirmed / Assumed /
  Contradicted), `Reason`, `Source` (sheet, schedule row or note), `RFI`, `By`, `Date`, `RangeLow`,
  `RangeHigh`. Stored in the project file and written to IFC, so it survives export and opens in any viewer
  that shows property sets. No new file format.
- **Capture hooks:** (a) the editing API's create / edit calls (the one cad2bim API) accept an optional
  `certainty`; (b) the Rust pipeline emits a list of "defaulted" fields per element (it already knows when
  it falls back to a default; this needs a small output addition, not a change to v1 `Entity`); (c) MCP
  tools set `Assumed` unless a `source` is given.
- **Views:** a colour-by in the existing Colour By system (State), a dial in the status bar, a docked
  "Assumptions" panel (a table grouped by reason, with Select and Isolate).
- **Quantities:** BOQ and BBS read the live model already ("one model, every view"), so the assumed-share
  column is a sum over a property. The what-if range re-runs the quantity with the low and high values.
- **RFI link:** reuse the RFI Forge list; each RFI stores its element ids; applying a reply is the
  existing "edit many elements" call with one undo step.
- **Export:** assumptions page as PDF / XLSX through the document tools already in the app; BCF topic per
  RFI so a coordination tool can carry it.
- **Privacy:** nothing leaves the device; the register is part of the project file.
- **Cost:** small. About 3 weeks for capture, colour and dial; a further 3 for RFI grouping and apply-reply.

### Risks

- If capture is annoying, people skip it. So: one tap, optional, and auto-filled whenever the tool knows
  (pipeline defaults, agent creation). Measure the skip rate in the first test.
- A register that nobody closes becomes noise. The stale-guess rule and the issue gate are what make it
  useful; if they are too strict they will be switched off. Make them settings.
- Confirmed does not mean correct: a drawing can be wrong. Twin Check and Sheet Sentry still apply.
- The "what if" range is only the modeler's own range. It must never be shown as a probability or a
  guarantee.

### Future update

- **Phase 1 (about 3 weeks):** the property set, one-tap Why chip, Gauge colour mode, dial, Assumptions
  panel, pipeline defaults tagged.
- **Phase 2:** assumed-share and range columns in BOQ and BBS; the assumptions page (PDF / Excel).
- **Phase 3:** guess groups, RFI Forge link, apply-reply with a diff card and one undo step.
- **Phase 4:** revision-aware closing and conflict flags (Rev Merge), the Issue Desk gate and title-block
  line, stale-guess alert.
- **Phase 5:** MCP: agents must cite a source or the element is Assumed; a "list open assumptions" tool so
  an agent can draft the RFI text.
- **Phase 6:** learn from past jobs: "on this architect's sets, missing beam sizes were 230x600 in 7 of 9
  cases" as a suggestion with the count shown, never as a silent default (needs Layer Whisperer's
  per-architect profiles). Also a Revit add-in column so the Revit side shows the same colour.

### Sources (market check)

- EarlyBIM: Revit plug-in prototype for uncertainty and missing information — https://mediatum.ub.tum.de/node?id=1518646
- Multi-LOD meta-model and uncertainty in design models (ITcon 2020) — https://itcon.org/paper/2020/13
- Consistent management and evaluation of building models in early design — https://structurae.net/de/fachliteratur/fachartikel/consistent-management-and-evaluation-of-building-models-in-the-early-design-stages
- Level of History parameter for heritage BIM (ITcon 2026) — https://itcon.org/papers/2026_34-ITcon-Kwast-Kotlarek.pdf
- Confidence in Scan-to-BIM models (Edinburgh) — https://www.research.ed.ac.uk/en/publications/quantifying-the-confidence-in-models-outputted-by-scan-to-bim-pro/
