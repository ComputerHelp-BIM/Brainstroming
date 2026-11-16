# 20261116 Idea: Detail Deck — the "General Notes + Typical Details" sheet written from the model: only the details this building needs, filled with this project's own grades, covers and lap lengths, and a check that no note contradicts the model

## Entry 1

```
date-time:   2026-11-16 (scheduled run, UTC)
idea name:   Detail Deck (model-driven general notes, lap/Ld tables and typical-detail sheet, in the browser,
             nothing uploaded)
potential:   High, and it happens on every structural set. Every project ends with the same chore: copy the
             last job's "General Notes" and "Typical Details" sheet, then fix it by hand. Someone deletes the
             details the job does not have (no PT slab, no sunk slab), forgets one it does have (a cantilever,
             a lift pit), and leaves old numbers in the notes ("M25 for columns" when the model has M30). The
             wrong note is read by the site as an order. It is the least glamorous sheet and one of the most
             quoted in disputes.
             Time saved: NOT measured here. First test: take the last finished set, time a person making the
             notes sheet from the old one, time Detail Deck on the same model, and count the contradictions
             each version still has. Do not quote hours until a team has done this.
             Market check (2026-11-16, two searches; not exhaustive, so "I did not find one", not "none"):
               - Revit add-ons I found automate views, dimensions, tags, rebar images and schedules (AGACAD
                 Smart Documentation, Naviate Structure). I found none that writes general notes or picks
                 typical details from what is in the model.
               - Notes sheets are described in tutorials as a master sheet placed by hand (Noble Desktop,
                 Autodesk University "intelligent details"). That is a library, not model-driven.
               - Auto-drawing tools (ARES drawings automation, AEC Magazine "auto drawings") go for plans,
                 sections and dimensions; the notes sheet is not what they sell.
             Not verified inside any paid product.
             Extends tools already in this repo (does not repeat them):
               - Drawing Autopilot: makes plans/sections/elevations from 3D. Detail Deck makes the sheet it
                 leaves out: notes, tables and typical details. They share one sheet set.
               - Spec Sage: reads a tender spec into model rules. Detail Deck goes the other way and writes
                 the notes from the model, then lets Spec Sage rules check them.
               - Bar Reader: reads someone else's notes (cover, grades, lap rule) into the model. Detail Deck
                 writes yours, and the two can be compared on a consultant's set.
               - Cover Cop, Bar Optimizer, Mark Merge: they hold the cover, lap and Ld tables and the marks.
                 Detail Deck prints the same tables, so the sheet cannot disagree with the model.
               - Sheet Sentry: checks issued sheets against the model. Detail Deck makes the notes sheet
                 correct before it is issued; Sheet Sentry checks it after.
             Trend: AEC trade press this year (AEC Magazine, October 2026) is full of "auto-drawings" and
             AI QA/QC launches (Pirros, Snaptrude agent). Drawings are called the low-hanging fruit, and almost
             all of it is plans and dimensions. The notes sheet is the quiet gap. I could not measure LinkedIn
             or Instagram counts; the sources are press pages.
             The wow: press one button. In about the time of a coffee a full A1 sheet appears: grade table
             (M30 columns L1-L8, M25 slabs), cover table by exposure, lap and development length table for
             Fe500D at each concrete grade, and ONLY the typical details this model needs, each with a small
             3D thumbnail of the real spot it applies to ("sunk slab, toilets T1-T14, tap to see them"). A
             side panel says: "2 contradictions: note 6 says M25 for beams, model has M30 on L5-L8. Detail 9
             needs a trimmer bar on 6 openings the model does not have." Fix, refresh, done.
```

### Description

**The pain.**
- The notes sheet is copied from the last job and edited by memory. It is never fully right.
- Typical details are chosen by a person scanning the drawings for conditions. Missing one means the site
  improvises; keeping an unused one clutters the sheet and confuses.
- Numbers in notes (grades, cover, lap and development length) are typed twice, once in the model and once in
  the notes. When the design changes, only one of them is updated.
- Revisions: nobody re-reads the notes when a level changes from M25 to M30.

**The idea in one line.** The model knows its grades, covers, bar sizes and conditions. Let it write the notes
and choose the details, show where each one applies, and warn when a typed note says something else.

### How it works (user view)

1. **Open the project** (a .c2b or an IFC with rebar). Press "Detail Deck" in the drawings group.
2. **Reading the model.** The tool lists what it found, with counts, and you tick or untick:
   - Concrete grades by member type and level range; steel grade and ductility; exposure class if set.
   - Cover by member (from the bar data or the cover rules in the project).
   - Conditions that need a detail: sunk slabs, cantilevers and chajjas, slab openings by size band, beam
     over a wall opening, lift pit, stair landing support, expansion joints, construction joints, column
     offset, beam-column joint, transfer beam, shear wall boundary element, tank walls, parapet, and so on.
3. **The sheet is built.**
   - **General notes**: a numbered list written from the model. Each note has a small "source" tick that
     jumps to the elements it came from. Notes that need an engineer's choice (loads, design code edition,
     special tests) are left as marked blanks, never invented.
   - **Tables**: concrete grade table, cover table, lap length and development length table for each grade
     combination in the model, bar bend and hook table. Same numbers the BBS and Bar Optimizer use.
   - **Typical details**: only those whose condition exists. Each is a parametric detail (the standard
     arrangement of bars, with this project's diameters, spacing, cover and lap), drawn as 2D linework and
     paired with a 3D thumbnail of one real instance. A "where used" list shows the marks or levels.
   - A **detail index** with callout tags that can be dropped on the plans (via Drawing Autopilot).
4. **Contradiction check.** If notes already exist (typed on the old sheet, or read in from a DXF/PDF), the tool
   compares each stated value with the model and lists: wrong grade, cover not as modelled, a detail on the
   sheet with no matching condition, a condition with no detail. Each line has "fix the note" or "fix the
   model" (the second is only a suggestion; it never edits the model silently).
5. **Edit and lock.** Notes are normal text boxes; edits are kept per note ("manual"). On the next refresh,
   manual notes are not overwritten, but a manual note that now disagrees with the model is flagged.
6. **Out.** DXF/DWG sheet, PDF, and a Revit sheet with detail items through the Revit bridge (preview, then
   confirm). A company border and text style can be set once.

### Why a BIM team would use it

- The one sheet nobody wants to make takes minutes and is traceable to the model.
- Juniors learn which condition needs which detail, because every detail says why it is there.
- Revisions: on a grade or cover change, "refresh" shows exactly which notes changed, as a red-line list.
- The checker reads one list, "model says X, note says Y", not a whole sheet.
- It makes a firm's own standard visible: the team's detail library becomes data, not a pile of old DWGs.

### Implementation

- **Facts from the model.** Reads grades, cover, bar data and element types through the same property readers
  the BOQ and BBS use; conditions are found by small deterministic rules (a slab with a lower top than its
  neighbour is "sunk"; a slab edge with no support beyond is "cantilever"; an opening above a size threshold in
  a slab is "opening, trimmer needed"). Each rule is a function with unit tests and a "why" string.
- **Detail library as data.** A detail is a JSON recipe: a condition, parameters (diameters, spacing, cover, lap
  from the shared tables) and a vector drawing function. First pack: the common IS 456 / SP 34 style details,
  drawn and checked by a structural engineer of the company, not by the tool. Firms add their own.
- **Shared tables.** One module for cover, lap, Ld, hooks, used by Cover Cop, Bar Optimizer and Detail Deck so
  they cannot disagree. Values come from the company's rule pack, with the code edition named on the sheet.
- **Notes as templates.** Each general note is a template with slots filled from model facts and a rule for
  when it is included. Missing facts leave a visible blank.
- **Contradiction check.** Text from an existing DXF (and later a PDF text layer) is parsed by pattern for
  "M<number>", "Fe<number>", "cover <number> mm" and similar; ambiguous matches are shown, not guessed.
- **Output.** Reuses the Drawing Autopilot sheet writer and DXF export; PDF in the browser; thumbnails are the
  viewer's own render. Nothing leaves the device.
- **Tests.** (1) Rule tests with hand-checked cases per condition. (2) A set of old real notes sheets (kept out
  of git) run through the check; report found vs missed contradictions counted by an engineer. (3) Round trip:
  refresh with no model change gives no diff. (4) A timed trial before any time claim.

### Future update

- **Firm standards mode**: import the company's existing typical-detail DWGs, tag each with its condition once,
  and Detail Deck places them instead of the built-in ones.
- **Consultant pack**: other codes (ACI, Eurocode, BS) as rule packs, chosen per project.
- **Revision memory**: each refresh stores a small diff, so the revision cloud on the sheet is automatic
  (feeds Issue Desk).
- **Live callouts**: tap a callout tag on a plan or in 3D and the typical detail opens beside it; on site via
  the QR pass.
- **Read the other side**: run the contradiction check on the consultant's notes against your model during RFI
  (Bar Reader and RFI Forge).
- **Spec link**: notes from the tender spec are matched to Spec Sage rules, so a spec change flags notes.
- **Steel and PT**: notes and details for steel connections and PT slabs once those tools exist.

### Reel idea for LinkedIn / Instagram

10 seconds: a stack of old notes sheets with red pen marks. Text: "Last job's notes. Wrong in 6 places." Cut to
a 3D building; tap "Detail Deck". The sheet draws itself, details pop in with a small 3D thumbnail of the real
spot. A red banner: "Note 6: M25. Model: M30." Tap fix, green tick. End card: "The notes sheet, written by the
model." Use a real sheet or mark it as an example; do not post invented numbers.

### Honest limits

- It drafts. The engineer of record owns the notes, loads and design choices; blanks stay blank.
- Detail quality depends on the library; the first pack must be reviewed against the current code editions by
  an engineer before use on a live job.
- Condition detection can miss odd cases; the "found / not found" list is shown so a person can add one by hand.
- Reading notes from old DXF/PDF is pattern matching and will miss free-form wording; it says what it could
  not read.
- Time saved and error counts are estimates to be tested. The market check is two searches; paid tools I did
  not try may do parts of this.

### Sources (checked 2026-11-16)

- Revit documentation add-ons (views, dimensions, tags, rebar images; no notes generation found):
  [AGACAD Smart Documentation](https://agacad.com/?p=20704),
  [Naviate Structure](https://www.autodesk.com/integrations/partner/naviate-structure).
- Notes sheets placed by hand from a master sheet:
  [Noble Desktop, Revit Structure notes and details](https://www.nobledesktop.com/learn/revit/revit-structure-recap-and-cleanup---placing-notes-plans-details-and-schedules),
  [Autodesk University, intelligent details](https://www.autodesk.com/autodesk-university/article/Creating-Intelligent-Details-Revit).
- Auto-drawings from BIM (plans and dimensions): [ARES drawings automation](https://onlinehelp.graebert.com/EN/ARESCMD/26/bim/chapter_bim_automation.htm),
  [AEC Magazine, the productivity promise of auto drawings](https://aecmag.com/features/the-productivity-promise-of-auto-drawings/).
- AI in AEC news, October 2026: [AEC Magazine AI section](https://www.aecmag.com/ai).
- These are web and press pages. I could not measure LinkedIn or Instagram trends directly.
