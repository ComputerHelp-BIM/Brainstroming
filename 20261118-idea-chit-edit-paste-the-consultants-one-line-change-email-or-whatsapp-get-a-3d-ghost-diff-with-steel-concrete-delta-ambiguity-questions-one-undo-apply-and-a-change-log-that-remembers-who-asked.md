# 20261118 Idea: Chit Edit — paste the consultant's one-line change ("C5 to 300x750 from L3 up, B12-B15 depth 600") from an email, WhatsApp or call note; get it ghosted in 3D with the steel and concrete delta, asked back where it is unclear, applied as ONE undo step, and logged with who asked and when

## Entry 1

```
date-time:   2026-11-18 (scheduled run, UTC; second idea of the day)
idea name:   Chit Edit (plain-text change instruction to a previewed, priced, undoable model edit, in the browser, nothing uploaded)
potential:   High, and it hits every modeler every day. Most design changes do not arrive as a new drawing
             revision. They arrive as a sentence: an email line, a WhatsApp voice-to-text, a "please make
             B12 to B15 600 deep" after a call. Today the modeler reads it, hunts the elements by hand,
             edits them one by one, forgets one, and nobody records that the edit came from that sentence.
             Weeks later: "who changed the C5 size and why?" has no answer. Time saved: NOT measured.
             First test: collect 30 real one-line change messages from one project (kept out of git, names
             removed). A modeler applies them by hand and with the tool. Compare minutes per change, wrong
             or missed elements, and how many messages the tool could not read. Do not quote hours or
             "errors prevented" until that is done.
             Market check: I did NOT search the web in this run, so this is "I did not check", not "none".
             Inside this repo the closest ideas are listed below; none takes a typed or pasted sentence and
             turns it into a previewed edit. To do before building: search Revit add-in stores and the
             Speckle / BIMcollab / Dalux feature lists for "natural language edit".
             Extends tools already in this repo (does not repeat them):
               - Toolsmith: turns "say it once" into a reusable Revit button. Chit Edit is one-off and
                 specific ("C5, L3 up, 300x750"), with a preview, a price and a log, not a saved recipe.
               - Sheet Sync: edits many elements from an Excel grid. Chit Edit takes the sentence the
                 consultant actually wrote, no spreadsheet to make first.
               - Markup Mirror: reads red-cloud comments on a PDF. Chit Edit reads TEXT (email, chat, notes).
               - Rev Merge and Change Radar: handle a new drawing revision and its diff. Chit Edit handles
                 changes that never become a new revision, and feeds Change Radar's delta view.
               - Meeting Mind: turns spoken decisions into BCF issues. Chit Edit turns a written instruction
                 into the EDIT itself; the two chain (an issue's text can be sent to Chit Edit).
               - Ask & Watch: asks questions, changes nothing. Chit Edit changes things, always after a
                 preview.
               - Guess Gauge and Mark Keeper: Chit Edit writes each applied change into their registers
                 (the size is now "confirmed by email of 18 Nov").
             Trend: AI agents driving BIM tools through MCP is a standing LinkedIn topic. cad2bim already
             has the safe part (an agent tool catalogue with edit tools, an "allow changes" switch, one call
             = one undo step). The missing piece for a daily user is the human-facing front door: a box where
             a normal person pastes the sentence. I could not measure LinkedIn or Instagram counts, so this
             is not a social-media claim.
             The wow: paste "C5 to 300x750 from L3 up. B12 to B15 depth 600, same width. Skip basement."
             The model shows a ghost: 14 columns and 4 beams glow amber (before = grey outline, after =
             solid). A card reads: "+3.1 m3 concrete, +212 kg steel (re-detailed), +Rs 41,000 at your rate
             card. Level 3 to roof." One yellow line: "'B12 to B15' on Level 3 AND Level 4 — which? [both] [L3]".
             Tap "both", press Apply. Done in 20 seconds; Ctrl+Z undoes all of it; the change log now says
             "email from R. Mehta 18-Nov 10:42, 18 elements".
```

### Description

**The pain.**
- Changes come as short sentences with shorthand: marks (C5, B12-B15), "from L3 up", "same width", "all
  edge beams", "skip basement". Humans read these easily; tools do not.
- Applying one by hand means finding the elements, editing each one, then checking the BOQ and BBS. A missed
  element shows up months later as a mismatch between two floors.
- No record links the model change to the message that caused it. That matters when the cost or time
  claim comes: "the client told us on 18 Nov" must be provable.

**What it does.**
1. **Paste or speak.** One box. Paste an email, a chat line, or dictate. English, Hinglish and common
   shorthand are read. Several changes in one message are split into a numbered list.
2. **Understand.** Each change becomes a plain, visible instruction: WHICH elements (by mark, grid
   range, level range, type, or "all"), WHAT (size, grade, level, move, delete, copy, a property), HOW MUCH.
   The page shows its reading in simple words next to the original sentence, so the person can fix a
   wrong reading by editing the words, not code.
3. **Ask, don't guess.** Anything unclear becomes one short question with buttons: "B12-B15 exist on L3 and
   L4. Which?", "'300x750' — width x depth or depth x width?", "C5 is not on L1; did you mean C15?".
   Nothing unclear is ever applied silently. (Guess Gauge can record any value the person still has to assume.)
4. **Preview.** Ghost in 3D (before outline, after solid), a count of elements touched, and the delta:
   concrete, steel (from the app's own rebar generator, not a rule of thumb), formwork area, cost at the
   team's rate card. Clashes the change would cause (a deeper beam against a door head, a wider column
   against a wall line) are listed from the checks the app already has.
5. **Apply.** One click, one undo step, only while "Allow changes to the model" is on. If Revit is
   connected, the same change set is queued for the Revit side; conflicts are decided by a person.
6. **Log.** Each applied change stores: original text, who pasted it, who it came from (a name field),
   time, elements, before/after values, and a hash of the model before. Export as a one-page "change
   sheet" for the variation record, or push the entries into the BCF/RFI list.

**What it is not.** It does not design anything. "Make the slab stronger" is refused with the question
"which size or grade?". It does not check that the new size is structurally adequate (that is the
engineer's call, or Twin Check / Quake Preview later). It shows the money and clash effect, not a
verdict.

### Implementation

- **Where it lives.** A "Chit" box in the cad2bim web app and the same endpoint exposed as one more tool
  in the agent catalogue (`packages/mcp`), so an AI agent can send the sentence too and get the same
  preview. Reading and checking stay in the browser; no model data leaves the device. An AI model is used
  only if the person has switched on "AI chat with your own key" (on the roadmap); the first version
  works with rules and a small word list, with no AI at all.
- **Parsing in two layers.**
  1. *Rules first* (fast, testable): patterns for sizes (`300x750`, `300 x 750`, `230/450`), mark lists and
     ranges (`B12-B15`, `C5, C7`), level phrases (`from L3 up`, `L2 to roof`, `except basement`), verbs
     (`make`, `change`, `shift`, `raise`, `delete`, `copy`, `same as`), units (mm, m, ft-in). Unit and
     language word lists are plain data files so a team can add its own shorthand.
  2. *Optional AI second pass* for sentences the rules cannot read: the model must return the same
     structured instruction (a small JSON schema), never code. The instruction is validated against the
     real model (do those marks exist, are the levels real) before anything is shown.
- **Selecting elements.** Reuse the existing filters (mark, level, type, grid zone) used by the read tools
  and `set_property`; an instruction that matches nothing is refused, as `set_property` already does.
- **Edits.** Compose the existing edit tools (`set_property`, `move`, `delete`, `copy`, `create_*`) into one
  batch so the whole message is a single undo step, all or nothing, with the same time limit the helper
  already enforces.
- **Delta.** Run the BOQ, BBS and QA on the after-state in a throwaway copy; diff against now. Cost uses
  the team's rate card (Rate Rocket, when built; until then a simple table).
- **Log storage.** In the project file on the device, as a list that can be exported to CSV/PDF and later
  to BCF. Names are free text typed by the user; no account lookups.
- **Safety.** Preview is mandatory (no "apply without looking" in v1). Edits only with the allow-changes
  switch. A hard cap (for example 500 elements per message) and a "this changes more than 20% of all
  columns, are you sure?" stop. Everything is undoable.
- **Test first.**
  1. Write 30 real-style change sentences (from the team's own mail, anonymised) with the expected edit.
  2. Measure: share read correctly with no question, share that needed a question, share misread (must
     be near zero; a misread is worse than a question), and minutes per change vs by hand.
  3. Seed 10 ambiguous or impossible sentences; every one must end in a question or a refusal, not an edit.
  Publish the numbers, even if poor.
- **Risks.** Over-trust ("the tool said OK"); shorthand that differs per consultant (mitigate with the
  per-sender word list from the log); a message that mixes a change with a question; dictation errors in
  numbers (always read the numbers back on the preview card, large and plain).

### Future update

- **Inbox mode:** forward the email to a private address or drop the `.eml`/`.msg` file; the attachments go
  to Drop Zone and the body to Chit Edit in one go. (Local file only first; no mail server access.)
- **Per-sender profiles:** learn that "TH" means top, "D" means depth for this structural consultant.
- **Reverse mode ("who said so?"):** click any element and see the sentences that last changed it.
- **Reply draft:** one click writes the confirmation back: "Done: 14 columns on L3-roof now 300x750, +3.1 m3,
  +212 kg. Two beams now touch the door head on L3 (list attached)."
- **Variation record:** group Chit Edit entries by client request and price them as a variation order with
  the rate card; hand to Delay Detective when the change delays a task.
- **Voice on site:** the site engineer dictates a change on the phone; the office sees the preview first.
- **Chain with Twin Check / Quake Preview:** run a quick structural sanity screen on the after-state before apply.
