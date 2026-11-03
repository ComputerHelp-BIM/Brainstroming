# 20261103 Idea: Trace Lens — click any 3D element to see the DXF lines it came from, and see every drawing line the pipeline skipped, with the reason, a one-click fix and a coverage score

## Entry 1

```
date-time:   2026-11-03 20:00 UTC (scheduled run)
idea name:   Trace Lens (2D <-> 3D Traceability for the DXF/DWG -> 3D Pipeline: Linked Split View,
             "Dark Lines" Overlay of Everything the Pipeline Did Not Use, Reason per Skipped Entity,
             One-Click Fix, Drawing Coverage Score)
potential:   High. cad2bim's best-known promise is "drop a drawing, get a 3D model". The moment that
             promise breaks is when the model is wrong or something is missing and the modeller cannot
             tell WHY. Today they re-read the DXF by eye, line by line, to find the one beam with a layer
             typo. Trace Lens turns that hunt (often 1 to 3 hours per floor on a messy drawing, my
             estimate, not measured) into a few clicks, and it is the proof a team lead needs to trust
             a generated model. It also makes the 2D -> 3D feature the first one in its class that can
             be audited. It is the natural next step after Layer Whisperer (fix the drawing BEFORE the
             run) and RFI Forge (find the drawing's own conflicts): Trace Lens explains the run AFTER it.
```

### Description

**The pain.**
- A generated model has 212 columns; the structural drawing has 218. Which six are missing? Nothing says.
  The modeller overlays the DXF and the IFC by hand, counts, and guesses.
- A beam is 3 m long in 3D and 3.6 m on the drawing. Was it a bad snap, a trimmed line, or a different
  source line? The pipeline knows. The person does not.
- A column appears at the wrong level, the wrong size or the wrong mark. The cause is one of a small number
  of things: the label sat outside the search distance, two labels competed, a layer was named oddly, a
  block had a nested scale, a polyline was open by 0.2 mm. These are all findable by a machine.
- The whole class of errors is the "silent skip": the pipeline ignored an entity and said nothing. Silent
  skips are the ones that reach site.
- Trust: a senior engineer will not accept a machine-made model without a way to check it. "It looks right"
  is not a check.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Layer Whisperer cleans layers and gives a pre-flight card before the run;
  Plan Tracer turns a PDF or scan into a DXF; RFI Forge audits the drawing set for internal conflicts; Twin
  Check compares a design with the model; Rev Merge handles a new drawing revision. None of them explains
  the pipeline's own decisions, lists what it skipped, or links a 3D element back to its source lines.
- In the market: Revit's "Link CAD" and Dynamo scripts that read DXF give a result and no trace; the usual
  answer to "why is this missing?" is to open both files and compare by eye. General CAD-to-BIM converters
  and AI plan readers (several appear in LinkedIn posts and vendor demos about "AI 2D to 3D") show a model,
  and in the demos I know of, not a per-element reason. I could not open every product to confirm this;
  treat it as unverified. Free tools: none I know of that do it for a structural DXF.
- In cad2bim (from `CLAUDE.md` and `docs/design/cad2bim-pipeline.md`): the CH v2 reader already keeps, per
  element, a `source` (`handle`, `plan` or `rule`: which step of the lookup gave each value). The Rust
  pipeline (`packages/pipeline`) and the CAD engine (`packages/cad-engine`) already read the DXF entities
  with their handles and layers, and the app already shows the DXF (2D) and the IFC (3D) as separate views.
  The raw material for a trace is there; the link and the screen are not.

**The idea.** A **Trace Lens** mode in the web app, on any model that came from a drawing. It opens a split
view: the DXF on the left, the 3D model on the right, and the two are one object.

1. **Click either side, see both.** Click a column in 3D: the DXF lines, label, block and layer it came
   from light up on the left, and a small card shows the chain: "Column C14 <- polyline handle 2B7F on layer
   STR-COL, label 'C14 300x450' found 180 mm away (rule), level from the sheet title (plan)". Click a line
   on the left: the 3D element it became lights up, or the card says "no element".
2. **Dark Lines.** One toggle dims everything the pipeline used and leaves the unused entities glowing red.
   A drawing with a few hundred leftovers usually shows its own problems at a glance: a whole grid of
   columns on a layer nobody mapped, a title block, a revision cloud, text outside any bay.
3. **A reason for every skip.** Each unused entity gets one of a short, fixed list of reasons, so the list
   can be sorted and counted: "layer not mapped", "polyline not closed (gap 0.4 mm)", "label has no shape
   within 600 mm", "two labels compete for one shape", "block nested at scale 0.001", "below minimum size",
   "inside a legend or title block", "duplicate of C14". The reason is the pipeline's real decision,
   recorded as it made it, not a guess made afterwards.
4. **One-click fix, written back as a rule, not a hand edit.** Each reason carries the smallest fix:
   "close the gap and re-run this entity", "map layer A-COL-OLD to column", "attach this label to this
   shape", "ignore this block everywhere". The fix is saved as a project rule (a small, readable file, as
   with Layer Whisperer's per-architect profile), so the next revision of the same drawing runs clean. The
   drawing file itself is never changed.
5. **Coverage score.** One number and one table at the top: "94 % of structural entities used, 41 skipped
   (29 explained by a rule, 12 need a person), 6 elements built from a guess". Elements built from a lookup
   fallback or a rule default (the `rule` source) are marked "guessed" and can be filtered, so the reviewer
   checks the 6, not the 212.
6. **A shareable proof.** "Export trace" writes a one-page PDF and an Excel sheet: every element, its
   source lines, the rule used, and every skip with its reason and its resolution. A team lead can sign it,
   and a client's engineer can see exactly how the model was made.

**Why it makes the team faster.** Checking a generated model is the slow, boring half of automation, and it
decides whether the automation gets used. If a model can be checked in minutes, people trust it, run it on
every revision and stop re-drawing by hand. This is an estimate until measured: it should cut the "check the
model against the drawing" step from hours to under ten minutes per floor on a messy drawing, and remove
most silent skips.

**Why the wow.** Click a column in 3D, the matching lines in the drawing pulse. Press Dark Lines and the
unused parts of the drawing glow red like a heat-map of what was ignored. Fix three red items with one click
each, press Re-run, and watch the missing columns pop into the model. It is a 20-second clip that explains
the whole product, good for a LinkedIn or Instagram reel: "I asked my model why it skipped 6 columns, and it
told me". Short, honest, visual. (I did not check what is trending on LinkedIn or Instagram this week; the
current theme I can see across AEC posts is "AI that you can trust and check", and this answers it.)

**Why it is unique here, and cheap to try.** The data already exists inside the pipeline as decisions it
makes and throws away. The work is to keep them (a trace table), draw the link, and rank the reasons. It
runs on the person's device, the file never leaves it, and no new licence is needed.

### Implementation

- **Trace table in the pipeline.** In `packages/pipeline/core`, record per entity a trace row:
  `{ entity_handle, layer, kind, used: bool, element_id?, reason_code?, source: handle|plan|rule|guess,
  distance?, rule_id? }`. Produced alongside the existing output, behind an option, so the legacy output
  (`hosting` off, byte for byte the Python pipeline's) does not change. The v2 reader already has `source`
  per value; extend it to carry the entity handle and the reason code. Do not change v1's `Entity` or
  `analyze` for v2 (as `CLAUDE.md` says); add the trace as a new output of the v2 path first, and v1 later
  if wanted.
- **Reason codes as a closed list.** A small enum (about 12 to 15) kept in one file with a plain sentence
  each, so the UI, the PDF and the docs read from one source. Every `continue` or early return in the
  analysis that drops an entity must set a code; a test counts that no drop path is silent.
- **ID link.** The pipeline already uses a stable key (layer, mark, level, position, GlobalId seed
  `shanku:`). Store the element id with the entity handle in the trace, and write the handle into the IFC as
  a property (for example `Cad2Bim.SourceHandle`) so the link survives saving and reloading. Check this does
  not alter the GlobalIds of models already sent to Revit.
- **Web app.** A new panel in the CAD workspace and the IFC workspace: the split view reusing the CAD
  engine's 2D canvas (three.js) and the IFC viewer, a shared selection store (one source of truth for
  "selected entity / selected element"), a Dark Lines layer (a material override on used entities), the
  skip list as cards in the existing QA panel (select, isolate, zoom, as in the Actionable QA design), and a
  coverage header. Colours and spacing from tokens only; `npm run lint:css` must pass.
- **Fix actions.** Each reason code maps to a small action that writes to a project rules file (layer map,
  ignore block, label attach, close gap tolerance). After a fix the app re-runs only the affected entities
  where possible, else the whole file; show the diff of elements added or changed before accepting.
- **Export.** PDF through `@cad2bim/documents` and Excel through the existing exchange path. Include the
  pipeline and rules versions in the file so a trace is reproducible.
- **Agent helper.** Expose `trace(element_id)` and `skips()` through the MCP helper (`packages/mcp`) so an
  AI agent can answer "why is column C17 missing" from the real trace, with the entity handle as its
  citation, instead of guessing.
- **Risks and how to handle them.** (1) Trace rows for a big drawing are large: keep them as typed arrays
  and show the first 500 by severity, with filters; measure memory on a very large file and say what was
  measured. (2) A "reason" that is wrong is worse than none: reasons come from the code path that made the
  decision, never from a second analysis, and each has a unit test with a fixture. (3) One-click fixes can
  hide a real drawing error: every fix is listed in the exported proof, and fixes that change geometry (not
  just mapping) need a person's click and a note. (4) DWG goes through the CAD core first; the handle comes
  from the DXF it becomes (`cad-extract --dxf`), so check the handles are stable across two conversions.
  (5) Do not use any drawing of a client in git; test fixtures are made-up or cleaned.
- **First 2 weeks.** Pick the Rust v2 path, add the trace table and eight reason codes (unmapped layer,
  open polyline, orphan label, competing labels, below minimum size, nested block scale, duplicate,
  title block), a test that no drop path is silent, and a read-only panel: click an element, show its
  source handle and rule; list the skips with reasons. No fixes yet, no PDF. Test on three real drawings kept
  outside git, hand-count the true skips on one floor, and record in the next entry how many the trace
  found, how many it missed and how many reasons were wrong.

### Future update

- **Compare two runs.** Trace diff between two drawing revisions: "12 elements changed source, 3 newly
  skipped", feeding Rev Merge and the what-changed note.
- **Learn per architect.** Remember the fixes you accepted per office or per template (Layer Whisperer's
  profiles) so the second drawing from the same architect arrives with fewer red lines.
- **Live while you draw.** In the CAD workspace, show the trace as the drawing is edited, so a gap that
  would be skipped is flagged the moment it is drawn, before any run.
- **Cover other readers.** Extend the same trace to PDF and scan tracing (Plan Tracer) so every vector or
  OCR guess says which pixels or strokes it came from and how sure it was.
- **Trust badge.** A model carries its coverage score and trace hash in its metadata, so the receiving
  consultant sees "built from drawing X rev C, 97 % coverage, 2 guessed elements" before opening it.
- **Team view.** A dashboard of the team's skip reasons across projects, to show which three mistakes in
  the drawings cause most of the lost time, and to turn them into drafting standards for the architects.
