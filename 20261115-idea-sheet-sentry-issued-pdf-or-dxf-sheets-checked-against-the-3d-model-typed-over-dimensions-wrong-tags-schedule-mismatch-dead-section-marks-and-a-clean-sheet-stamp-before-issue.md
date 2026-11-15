# 20261115 Idea: Sheet Sentry — the drawing sheet you are about to issue (PDF or DXF) is read and checked against the 3D model: typed-over dimensions, tags that disagree with the element, schedules that do not match the model, section marks that point nowhere, old notes — each finding pinned on the sheet and in 3D, with a "clean sheet" stamp before issue

## Entry 1

```
date-time:   2026-11-15 (scheduled run, UTC)
idea name:   Sheet Sentry (pre-issue check of the DRAWING against the MODEL, in the browser, nothing uploaded)
potential:   High for every modeller and BIM lead who issues sheets. The last 20 minutes before every issue are
             spent flipping through sheets to find a dimension someone typed over, a beam tag that says B12 where
             the model says B14, a schedule that was not refreshed, a section mark that points to a sheet that
             no longer exists. These errors reach site and become RFIs, rework and arguments.
             Time saved: from how this work usually runs, NOT measured here. A first test: take one issued set,
             have the tool and a person each find the errors, and compare the lists. Do not quote hours until a
             team has timed it.
             Market check (2026-11-15, two searches, not exhaustive). It is NOT empty, so I do not claim unique:
               - In Revit, paid add-ins: Ideate BIMLink / Ideate Explorer find overridden dimensions and run
                 SpellCheck on sheets; pyRevit scripts can audit tag completeness. They live inside Revit and
                 check the Revit file.
               - AI drawing review for PDF sets (Nomic, InspectMind, Buildcheck and others, mostly paid, vendor
                 pages): they read 2D PDFs without a model and look for missing information and conflicts
                 between sheets.
               - AEC Magazine describes a QA/QC platform that links BIM models and drawing sets (paid).
               What I did not find in the search: a free tool in the browser that takes the ISSUED sheet
               (PDF/DXF, the thing the contractor really gets) and the model on the same device and says
               "this number on the sheet is not what the model says". That is the gap. Only a claim from one
               search; verify before marketing.
             Extends tools already in this repo (it does not repeat them):
               - Trace Lens: model element -> its DXF source. Sheet Sentry is the other way: sheet text ->
                 model element.
               - RFI Forge: audits consultants' drawings BEFORE modelling (plan vs schedule vs section).
                 Sheet Sentry audits OUR output AFTER modelling, against our model.
               - Issue Desk: revision clouds, register, transmittal. Sheet Sentry is the check that runs just
                 before it and gives it a "clean" stamp.
               - Twin Check / Warning Doctor: model-vs-design and model health. Neither reads the sheet.
             Trend: LinkedIn / press this month is about AI agents inside Revit and about governance (preview,
             confirm, audit trail before anything changes) and about AI drawing review as a funded category.
             Sheet Sentry is read-only, deterministic (rules, not guesses), and shows a preview. I could not
             measure LinkedIn or Instagram counts directly; the sources are press and vendor pages.
             The wow: drop the sheet PDF and the IFC. The sheet comes up with small red and amber flags on it.
             Click a flag on "3000" and the 3D flies to the beam and shows 3150; click a tag "C7" and the
             column lights up as C9. A bar at the top reads "41 checks, 3 errors, 5 warnings". Fix them, drop the
             new PDF, it reads "clean", and a stamp with the date and the model revision is offered.
```

### Description

**The pain.**
- Drawings are made from the model, but they carry typed text: a dimension with its value overridden, a note
  copied from the last project, a tag typed by hand, a schedule pasted as a picture or exported weeks ago.
- Every model change after the sheet was made leaves the sheet quietly wrong. Nobody sees it until site does.
- The check is a person with a highlighter and a PDF, late on issue day. It is skipped when the deadline is near.

**The idea in one line.** Read the sheet like a person would, read the model like a computer does, and list every
place where they disagree.

### How it works (user view)

1. **Drop the sheet and the model.** A PDF with real text (what Revit and AutoCAD print), or a DXF/DWG sheet, and the
   IFC or the open cad2bim project. A scanned PDF with no text is told so plainly ("no text layer, cannot check").
2. **It finds the sheet's objects.** Dimensions (a value with a measured line), tags (a mark near an element),
   schedule tables, section/elevation marks (a circle with a sheet number), notes, scale text, revision.
3. **It matches them to the model.**
   - *Dimensions:* the line is measured on the sheet at the sheet's scale and compared with the written number and with the
     model distance between the same two elements. "Written 3000, drawn 3150, model 3150: typed over."
   - *Tags:* the mark under a leader is matched to the element the leader touches (by position in the view, using
     the view's known level and grid). "Tag B12, model B14."
   - *Schedules:* each row is matched to the model's elements by mark; sizes, counts and totals are compared.
     "Schedule says 24 C1; model has 26."
   - *Section/elevation marks:* "points to A-301; no sheet A-301 in the set" or "points to a view that does not cut
     the element it names".
   - *Notes and text:* a note naming another project or a grade that the model's materials do not use
     ("M25" on a sheet where the model says M30) is flagged; plain spell check on the rest.
   - *Sheet hygiene:* scale text vs the measured scale, north arrow present, revision matches the register,
     title-block fields filled.
4. **Every finding has a reason and a place.** A flag on the sheet and a pin in 3D; a plain sentence ("sheet says,
   model says, difference"); three buttons: *Real error*, *Ignore (with reason)*, *Rule is wrong*.
5. **Re-check.** Drop the fixed sheet; the list shrinks. When there are no errors left, offer a "checked against
   model rev X on date Y" stamp for the title block or the transmittal. It states what was checked and what was not.
6. **A checks file.** The list exports to Excel and BCF, so a reviewer or the drafter gets the same list.

### Why a BIM team would use it

- The last-hour hunt becomes a button, so it actually happens on every issue.
- It catches the errors people really make: typed numbers and stale schedules.
- The BIM lead gets one repeatable check instead of "I looked through it".
- It stays on the device: the issued set never goes to a server, which matters for client drawings.

### Implementation

Runs in the browser. Reuses what cad2bim already has: the IFC reader, the viewer's pins, the PDF reader of the
document viewers, and the DXF/DWG reader of the CAD engine.

1. **Read the sheet.**
   - PDF: extract text with positions and the vector lines (the document viewers already read PDFs on the device).
     Group text into dimension, tag, table and note candidates by simple geometry (a number near a line with end
     ticks is a dimension; a table is aligned text on a ruled grid).
   - DXF/DWG: the CAD engine already gives DIMENSION entities with the measured value and the text override, plus
     text, blocks and layouts. This is the easy and exact case; build it first.
2. **Calibrate.** Sheet scale from the scale text, or from a known pair (two grid lines whose model distance is
   known). Show the calibration and let the user correct it. No check runs on an uncalibrated view.
3. **Link sheet to model.** Per view, a transform from sheet to model plan coordinates, found by matching grid
   bubbles and column symbols to the model's grids and columns. If it cannot be solved, the view is skipped and
   says so, never guessed.
4. **Rules.** A list of small, separate rules, each with an id, a plain sentence and a tolerance (e.g. 1 mm on
   paper). Rules are data, so a company adds its own. All deterministic; no AI is needed for the checks. An
   optional AI step (the user's own key, like the existing AI chat) can only *explain* a finding or draft a note,
   never decide one.
5. **Findings model.** One record each: rule, sheet position, model element guid, sheet value, model value,
   status. Saved in the project file so a re-check keeps the "ignored with reason" decisions.
6. **Output.** Flags drawn over the sheet, pins in 3D, a list, Excel, BCF, and the stamp text.
7. **Test plan.** (a) Build a test sheet with known planted errors (10 typed-over dimensions, 5 wrong tags, 3 stale
   schedule rows, 2 dead section marks) and report how many are found and how many false alarms. (b) Run it on a
   real issued set with a senior person and compare with their list. Publish found / missed / false alarms; do not
   ship a rule whose false-alarm rate is high.

### Future update

- **Fix suggestions that write back.** For a typed-over dimension, offer the model value; for a stale schedule,
  regenerate it; with the Revit bridge, push "reset dimension override" back to Revit as a preview-then-confirm
  action (never silent).
- **Sheet diff between issues.** Compare issue 1 and issue 2 of the same sheet and list what changed on paper, against
  the model change list from Change Radar: "model changed, sheet did not" and "sheet changed, model did not".
- **Company rule packs.** Rules for IS 456 / IS 13920 detailing notes (cover, lap and anchorage notes consistent
  with the model's actual values), a BEP-based naming and title-block pack (links to BEP Bolt).
- **Bluebeam-style comments out.** Findings written as PDF comments on the sheet so the drafter sees them in any reader.
- **Whole-set mode.** A drawing register built from the folder (Drop Zone) and one score for the set.
- **Overlay mode.** The model's linework drawn over the PDF in a colour, a quick visual proof that the sheet
  matches the model shape.

### Reel idea for LinkedIn / Instagram

12 seconds: a normal PDF sheet, then red dots appear on it one by one with a counter. Zoom on "3000" -> 3D flies to
the beam -> "model: 3150". End card: "41 checks. 3 errors. Found before site did." Caption: "Your last 20 minutes
before issue, as a button." (Use a real or clearly planted example, say which, and do not post invented numbers.)

### Honest limits

- Scanned PDFs have no text; they cannot be checked (OCR is a later, weaker option).
- Matching a sheet view to the model is the hard part. Where it cannot be solved with confidence the view is skipped
  and listed, not guessed.
- It checks what can be compared (numbers, marks, references). It does not judge design or detailing quality.
- Rules have tolerances; false alarms will happen. The *Ignore (with reason)* and *Rule is wrong* buttons exist for
  that, and the test plan must report the false-alarm rate.
- The hours-saved claim is an estimate to test. The market check is two searches; paid tools I did not try may do
  parts of this.

### Sources (checked 2026-11-15)

- Revit overridden-dimension and sheet checks, paid in-Revit tools: [Ideate: find overridden dimensions](https://ideatesoftware.com/how-can-i-easily-find-overridden-dimensions-in-my-revit-model),
  [Ideate BIMLink health check](https://support.ideatesoftware.com/support/help/ideate-bimlink/how-to/health-check),
  [ArchiLabs: Revit annotation QA after model change](https://archilabs.ai/posts/revit-annotation-qa-after-model-change).
- Model-and-drawing QA/QC platform (paid): [AEC Magazine](https://aecmag.com/bim/qa-qc-platform-links-bim-models-and-drawing-sets/).
- AI drawing review on PDF sets (vendor pages, claims unverified): [Nomic: automated drawing review](https://www.nomic.ai/glossary/automated-drawing-reviews),
  [InspectMind: AI plan checking](https://www.inspectmind.ai/guides/ai-plan-checking),
  [Buildcheck Series A](https://yespress.io/buildcheck).
- These are press and vendor pages. I could not measure LinkedIn or Instagram trends directly.
