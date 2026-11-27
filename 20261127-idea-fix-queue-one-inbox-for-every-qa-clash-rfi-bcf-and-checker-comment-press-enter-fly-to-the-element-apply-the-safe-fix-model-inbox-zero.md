# 20261127 Idea: Fix Queue — one inbox for every QA finding, clash, RFI, BCF issue and checker comment; press Enter, fly to the element, apply the safe fix; "Model Inbox Zero"

## Entry 1

```
date-time:   2026-11-27 (scheduled run)
idea name:   Fix Queue — "Your model's to-do list, sorted. Press Enter, fix, next."
             Everything that needs a modeller's attention lands in one list: QA findings, IDS failures, clashes,
             BCF issues, RFIs, the checker's comments, drawing-revision changes and the site's snag pins. The list
             is ranked (what blocks a pour or a drawing issue first), de-duplicated (ten findings with one root
             cause become one card) and worked card by card. Each card flies the camera to the element, says what
             is wrong in one sentence with the measured number, offers the safe fix as a button, and has three
             other keys: Skip, Send to engineer, Not an error (with a reason that is remembered). Finish the list
             and you get "Inbox zero" with a one-page summary to paste into the daily stand-up.
potential:   High, and it multiplies the value of the checks the toolkit already has or has planned. Checks that
             nobody works through save nothing. Today a modeller's day is split across five places: the QA panel,
             a BCF or Navisworks clash list, WhatsApp, an Excel RFI log and the checker's mark-ups. Switching
             between them, finding the element again and re-explaining the problem is the hidden cost.
             My estimate (not measured, to be tested with two real projects): 30 to 60 minutes a day per modeller
             goes to finding and reaching the next thing to fix, not to fixing it. For a team of eight that is
             about one person a day. A team lead also stops asking "what is left?", because the queue says so.
             Market: to my knowledge (not verified for this run), the pieces exist as separate paid products:
             issue trackers (BCF-based, per-seat cost), clash tools and model checkers each with their own list.
             A free, in-browser queue that merges all of them, runs on files that stay on the device, fixes the
             safe ones with one key and explains the root cause is not one I know of.
             Trending: "inbox zero", "AI agent clears your backlog" and "one-key triage" are common formats on
             LinkedIn productivity posts. The visual hook is simple to film: a 30-second screen recording of the
             camera flying from problem to problem while a counter falls from 142 to 0. Treat this as a signal of
             interest, not a measured trend.
```

### Description

**The pain (a BIM modeller's day).**
- Problems arrive from many senders and in many shapes: a red row in the QA panel, a clash group from the
  coordination meeting, a BCF from the architect, a WhatsApp photo from site, a line in the checker's Excel, a
  new DXF revision that moved three columns.
- Each has to be found in the model first. "Column C12, level 3" takes a minute to locate; at 100 items that is
  an afternoon.
- Duplicates are common: one wrong column size shows up as a QA finding, a clash, a BOQ mismatch and an RFI.
- Nobody can answer "how many open items are left on level 3, and which stop the pour?" without building a
  spreadsheet.
- Fixing in order of arrival, not of consequence, means the item that blocks the drawing issue is found on the
  day of the issue.

**What Fix Queue does.**

1. **Collect.** Items come from sources, each a small adapter that produces the same card:
   - the QA findings the app already produces (`Finding` in `packages/engine/src/qa/`: id, severity, title,
     detail, measured, limits, elements);
   - IDS and requirement checks, clash results, BCF files (in and out), and the RFI list from *RFI Forge*;
   - the checker's comments from *Second Eyes*, snags from *Snag Sweep*, drawing-revision changes from
     *Change Radar* and *Rev Merge*;
   - a pasted list (names, marks, a WhatsApp message), read like *Chit Edit* reads one.
   Each card keeps its source, so closing it can write back (close the BCF, mark the RFI, clear the snag).
2. **Merge and rank.** Cards are grouped by root cause (same elements, same underlying fact). The group's
   rank is shown with its reason: "blocks pour of L3 slab (4D)", "affects 26 BOQ lines", "due in 2 days (issue
   date)", "client-visible". The rank is a plain score from stated inputs, not a hidden model, and the person
   can reorder.
3. **Work it.** One card on screen. The camera flies to the elements, the rest of the model fades, the card says
   *what is wrong and the number* and shows the proposed fix as a ghost (before and after in the same view).
   Keys: `Enter` apply the fix, `S` skip, `E` send to engineer (with the snapshot), `N` not an error (asks why, saves
   the reason as a rule for next time), `Z` undo (one undo per card, using the app's History).
4. **Safe fixes only, shown first.** A fix is "safe" when it is reversible, local and has one obvious answer:
   a missing level link, a mark clash, an element off its grid by less than a tolerance, a wrong grade tag, a
   duplicate. Anything with several answers or any structural effect (a size change) is a *proposal* that needs
   a person, never applied alone.
5. **Close the loop.** Re-run the sources after each batch. Cards that disappear are closed with evidence
   ("fixed: now continuous"); new ones appear marked *new since you started*. At zero, a summary:
   items closed by fix, by person, by "not an error", and what is still waiting on others.

**Why it is a wow.**
- It feels like a game and works like a work-planner: flying through problems, one key each, with a falling
  counter. People will film it.
- The "not an error" key teaches the queue. After a month the false alarms of the team's own checks stop
  coming back (this links to *Repeat Offender*, which mines the history for checks to add).
- Team view: the lead sees per-level and per-person counts, and what is stuck waiting for the architect.

**Day in the life.**
1. 9:00. Open the project. The queue says 87 items: 12 block this week's pour on level 3, 31 are safe fixes,
   24 wait on the architect, 20 are low rank.
2. Press `Enter` through the 31 safe fixes (each shown as a before and after): 4 minutes.
3. The 12 blocking items are worked one by one; three become "send to engineer" with a snapshot attached.
4. 9:45. The queue shows 24 waiting items with owner and age, and 20 low-rank items for the afternoon.
5. A new architect DXF arrives at noon. 6 cards appear marked *new*, ranked by what they affect.
6. 5:30. Press `Summary`: a paragraph and a table for the stand-up. Nothing was lost between tools.

### Implementation

**Reuse what exists.** (The repository's design notes were read for this idea; the engine's code was not run.)
- `Finding` in the QA engine already has a stable id (check + hash of the elements' GlobalIds), severity, text,
  measured and limit values and element indices. That is nearly a card. The id being stable across runs is what
  lets the queue know a finding is the same one tomorrow.
- The QA panel already runs `runChecks`; Fix Queue is a second view over the same results, plus other sources.
- The history/undo system gives one undo per card. The viewer gives the fly-to, fade, isolate and ghost.
- The agent helper (MCP, `packages/mcp`) already exposes `qa`, `select`, `isolate` and edit tools, so an agent
  can read and work the queue the same way a person does ("work through the safe fixes and stop at anything
  that changes a size").

**New pieces.**
1. **Card model** (`packages/engine`): `{ id, source, sourceRef, severity, title, detail, measured, limits,
   elements, fix?: { kind: 'safe' | 'proposal', preview, apply }, rankInputs, state }`. `state` is one of open,
   skipped, sent, dismissed (with reason), closed (with evidence).
2. **Adapters**, one per source, each small and independent: QA (first, nearly free), BCF read and write,
   pasted list, IDS, clash, RFI. A new source is a new adapter, not a change to the queue.
3. **Merge by root cause:** group cards by shared elements and by a "cause key" the adapter can supply
   (for example the same wrong type). A simple rule first: same element set or one a subset of another.
4. **Rank:** a plain function of stated inputs: severity, whether a 4D task or a drawing issue date depends on
   the element, number of BOQ lines affected, age, client-visible tag. Every input is shown on the card.
5. **Work screen** (`apps/web`): one card, keys as above, ghost preview using the existing overlay and colour
   overrides, the *Skip / Send / Not an error* actions. The design rule applies: no helper text; the card is
   the sentence, the number and the button.
6. **Dismiss memory:** "not an error" saves a rule (check id + element pattern + reason) in the project file,
   so it is shared by the team through the project, never sent anywhere.
7. **Summary:** a table (closed, by whom, waiting on whom) as text and Excel, using the existing Excel writer.

**Privacy.** Everything runs in the browser on the device. BCF and Excel are files the person chooses to open
or save; nothing is uploaded to read or convert it.

**Build order (my estimate, not measured).**
   - Step 1: the card model, the QA adapter and the work screen with fly-to, skip and dismiss. About a week.
   - Step 2: safe-fix previews and apply for the checks that already have an obvious fix (the QA panel's "fix"
     buttons from the actionable-QA design). About a week.
   - Step 3: merge by root cause and the rank, with the reasons shown. A few days.
   - Step 4: BCF in and out, the pasted-list adapter, write-back on close. About a week.
   - Step 5: the team view and the stand-up summary. A few days.

**Risks and honest limits.**
- A wrong "safe" fix is worse than no fix. Keep the safe list short, preview every change as before and after,
  one undo per card, and log every applied fix. Anything touching a structural size stays a proposal.
- Ranking can annoy if it is opaque. Show the reason on every card and allow reorder; do not claim it predicts
  risk.
- Dismiss rules can hide real errors. Rules expire after a set time or on a new revision, and the summary lists
  what was dismissed and why.
- Value depends on the sources. With QA alone it is a nicer QA panel; the saving grows with each adapter. The
  first test should be on one real project with BCF and the checker's Excel.
- Team sharing through the project file needs the multi-user story in the roadmap (versions, co-editing). Until
  then the queue is per person, with the summary and BCF as the way to share.

### Future update

- **Agent mode:** "clear the safe fixes and bring me the rest", with a report of what it did and every undo kept.
- **Learning:** the dismissed reasons and fix choices feed a small per-team model of what is usually right
  (links to *Repeat Offender* and *Deja Model*).
- **Phone view:** the site engineer works a short queue of snags and photo requests on a tablet or phone
  (links to *Snag Sweep* and *Site QR Pass*).
- **Time to clear:** the queue's history gives a plain measure per project: items per day, median age, what
  stays open longest. Useful for the BIM manager and for the BEP's KPIs (links to *BEP Bolt*).
- **Nightly:** *Night Shift* runs the sources overnight so the queue is ready and ranked at 8 a.m.
- **Meeting mode:** in the coordination meeting, work the clash cards live and assign owners by name, with the
  list exported as BCF when the meeting ends (links to *Meeting Mind* and *Clash Sorter*).
- **Revit:** the Revit bridge shows the same queue as a panel in Revit and applies the safe fixes natively.
- **Public inbox-zero card:** an optional image ("142 → 0, level 3 ready for the pour") for the team's own
  posts, with no model data on it unless the person adds it.
