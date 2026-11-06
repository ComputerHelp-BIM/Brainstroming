# 20261106 Idea: Ask & Watch — type or say a question about the model in English or Hinglish, get a number with 3D proof, then pin it as a live "watch" that re-answers itself on every new revision and tells you what changed

## Entry 1

```
date-time:   2026-11-06 (scheduled run, third entry of the day)
idea name:   Ask & Watch (In-Browser Question Box for the IFC Model: Plain English / Hinglish / Voice Questions,
             Answers Computed by the Engine (Never Guessed), Click-to-3D Proof, Pin Any Answer as a Live Watch,
             Re-Run on Every New Revision with a Delta, a Watch Board for the Whole Team, Phone-Friendly Answer Cards)
potential:   Very high for day-to-day use. Every BIM team gets the same 30 questions all day: "how many M30 columns on
             level 5?", "which beams are deeper than 600?", "total shuttering on the podium?", "which footings have
             no rebar yet?", "is there any slab thinner than 150?". Today each one costs a modeller 5 to 20 minutes
             (filter, schedule, export, count) or an interruption of someone who knows the model. Ask & Watch
             makes the answer a 10-second thing that a project manager, QS or site engineer can do alone, with proof.
             The part that is new is the "Watch": the question stays alive. On the next revision it answers again
             and says "was 112, now 118, six new columns, click to see them". This is the part that makes people
             show it on a screen in a meeting. Time saved is an estimate from how this work usually runs; it has
             not been measured here.
```

### Description

**The pain.**
- The model holds the answers, but only a modeller can get them out. Everyone else asks the modeller, who stops
  modelling to filter, schedule and export. Five such interruptions a day is an hour or more lost per modeller
  (estimate).
- Chat with an AI about the model is the loudest trend on LinkedIn right now, and most demos have the same flaw:
  the AI says a number and nobody can tell if it is right. In BIM a wrong count is a wrong order, so "sounds
  right" is not good enough.
- Questions are asked once and forgotten. Next week the revision changes, the number is stale, nobody re-asks,
  and the BOQ in the tender drifts away from the model.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In cad2bim: the AI agent helper (`packages/mcp`, `docs/agent/protocol.md`) already exposes 41 read and edit
  operations (`elements`, `boq`, `bbs`, `qa`, `levels`, `select`, `colour_by`, `command` and others, with a
  common filter of ids, marks, category, level, grade, name and type, `docs/api.md`). So an outside agent such as
  Claude Desktop can already answer these questions, but it needs a separate app, a local helper and pairing, and
  its answers are chat text, not something a project can keep. The viewer also has a `command` operation for
  plain-English commands. **There is no built-in question box, no saved questions, and no re-asking on a new
  revision.**
- In this idea folder: Toolsmith (2026-10-20) turns a repeated recipe into a Revit button, Spec Sage (2026-11-01)
  gives cited answers about **specifications**, Measure Right (2026-11-03) gives click-to-3D proof for
  **quantities in a measurement book**, Ratio Radar (2026-10-29) is a fixed set of sanity ratios, Change Radar
  (2026-10-14) diffs two revisions. None of them lets a person ask an open question of the model, and none keeps
  the question alive across revisions. A search for "ask the model" and "natural language" in the folder found
  no idea about this.
- In the market (not exhaustively checked): vendor assistants and a few paid add-ins and startups offer
  "chat with your model" or natural-language model search, usually as a subscription and usually with the model
  data sent to a cloud service. A free, in-browser version where the model stays on the device, every number is
  computed by the engine, and the question becomes a watch is not something I found. Treat this as unverified
  until a proper search is done.

**The idea.** A **Question box** at the top of the viewer (one keystroke to open, a microphone button beside it),
and a **Watch board** beside the BOQ tab.

1. **Ask.** Type or say a question in English, Hindi or Hinglish ("level 5 pe kitne M30 column hain?",
   "beams deeper than 600 mm on podium", "footing jinme rebar nahi hai").
2. **Plan, then compute (the rule that keeps it honest).** The language model never answers from memory and never
   sees the model. It only turns the question into a **query plan**: a small JSON in the same filter language the
   API already has (category, level, grade, a property test, a group-by, a measure such as count, length, area,
   volume, steel kg). The engine runs the plan on the model in the tab and returns the numbers.
   - The plan is shown in plain words beside the answer: "Columns, level 5, grade M30. Count." The user can
     tap any chip to change it (level, grade, category), and the answer updates at once. This is also how a
     misunderstood question gets fixed in one tap instead of a re-prompt.
   - A question the plan language cannot express is said to be not understood, with the nearest thing it can do.
     It never invents a number.
3. **Proof.** Every answer has three things: the number, a table of the elements behind it, and the same
   elements lit in 3D (the viewer's select and colour-by machinery). Click a row, fly to the element. A "0
   results" answer also shows what the question was matched against (how many columns exist, on which levels),
   so a typo in a level name is visible.
4. **Pin as a Watch.** One click saves the plan (not the answer) with a name, an optional unit and an optional
   rule: "alert me if more than 120", "alert me if it changes by more than 5%", "alert me if it is not zero".
   Watches are saved with the project's model notes on the device, and can be exported as a small JSON file so
   the team shares one set.
5. **Re-answer on every new revision.** Open a new IFC (or a new pipeline output from a new DXF revision) and the
   watch board re-runs all watches and shows **was, now, change, and the elements that moved in or out**, with
   one click to light them. Example row: "Columns M30 L5: 112 -> 118, +6 (C41 to C46), click to see".
   A watch with a broken plan (the level was renamed) says so instead of showing zero.
6. **Answer cards for people with no model open.** Any answer or the whole board exports as a **phone-friendly
   card**: a PNG or a one-page PDF with the number, the question in words, the model name and revision, a small
   3D picture with the elements lit, and a link-free QR-free footer ("computed in cad2bim from file X, revision
   B, on 2026-11-06"). It is made for WhatsApp, not for a portal. Nothing is uploaded; the user shares the file.
7. **Starter pack of watches.** A first-run list the user can switch on in one click, each one a plain plan:
   elements with no level, no mark, no grade, no cover set; duplicates at the same location; beams shorter than
   300 mm; slabs thinner than a set value; columns with no footing below; any category with zero elements on a
   level where its neighbours have some. These are the "quiet bugs" people find late.
8. **Why a project manager would use it.** The board is a model **status screen with no software skill needed**:
   a QS watches steel kg and concrete m3 per level, a site engineer watches "columns modelled vs columns in the
   drawing list" (from a pasted Excel list of marks), a BIM lead watches the starter-pack watches.

### Implementation

1. **Query plan (new, small, pure TypeScript in `packages/engine`).** A typed `QueryPlan`: `{ filter, where[],
   groupBy[], measure, sort, limit }`. It reuses the API's filter and the engine's existing element, BOQ and QA
   readers; the new part is `where` (a property test: equals, range, exists, empty, regex-free "contains") and
   the measures. A run returns `{ rows, total, elementIds }`. Because it is pure, the same code serves the
   Python console and the agent's tools (`ask.plan`, `ask.run`), in line with "one model, every view".
2. **Language step (new, thin).** Question text -> plan, as a function with two implementations behind one
   interface:
   - **Rules first (offline, no AI):** a small grammar for the 40 most common question shapes in English and
     Hinglish (count, total, list, which, how many, more than, less than, between, without, with no, per level).
     This alone covers most day-to-day questions and works with no network.
   - **AI second (optional, opt-in):** when the rules do not match, send **only the question text and the schema**
     (category names, level names, property names, grades; never element data, never geometry) to a model the
     user has chosen, with their own key, and parse the answer into a `QueryPlan` that the engine validates
     against the schema before running. This keeps "nothing leaves the device unasked" true: what is sent is
     shown first, once, and the setting is off by default.
   - The plan is validated, never executed as code. An invalid plan is rejected, not repaired silently.
3. **Voice (reuse the browser).** The Web Speech API where available (Hindi and English); the transcript goes to
   the same box and is shown for correction before it runs. No audio is stored by cad2bim. Where the browser
   sends audio to its own speech service, the settings text says so.
4. **Proof in 3D (reuse).** `select`, `isolate`, `colour_by` and the table component the BOQ tab already has.
   "Moved in / moved out" is a set difference of element ids by GlobalId (and by mark as a fallback, the same
   matching Change Radar and Rev Merge use).
5. **Watch store (new, small).** Plans, names, rules and the last answer, kept per model in IndexedDB; export and
   import as JSON. A watch is data, so it is reviewable and diffable in git by a team that wants that.
6. **Watch board (new UI).** A table of watches with was, now, change and a status colour from the design tokens;
   a sparkline of the last N revisions the device has seen. A "run all" button, and a run on every model load.
7. **Answer card (reuse + small).** Render the table and a viewport snapshot to a canvas, add the footer, export
   PNG; the one-page PDF reuses the print path of the BOQ export.
8. **Tests.** (a) A grammar test table: 120 questions in English and Hinglish with their expected plans (written
   by the team, not generated). (b) A run test on the synthetic model the engine tests already use: each plan
   returns the known count. (c) An adversarial test: questions that must be refused (an unknown level, a
   property that does not exist, an instruction to delete or edit; **the question box is read-only, it never
   calls an edit operation**). (d) A watch test across two revisions of a synthetic model with planted changes.
   (e) For the AI step, a recorded-response test so CI needs no network and no key.
9. **Where it lives.** A box in the viewer chrome and a Watch tab beside BOQ and QA in `apps/web`; the engine side
   in `packages/engine`; the tool names added to `packages/mcp/src/tools.ts` so an outside agent can add and
   read watches too. Versions and changelog per `CLAUDE.md`; a design note in `docs/design/` and a row in
   `docs/decisions.md` before any code.
10. **Phasing.**
    - Phase 1: the query plan, the rules grammar (English), the answer with table and 3D proof, the starter
      watches. (1 to 2 weeks, estimate.)
    - Phase 2: Hinglish, plan chips, pin as a watch, the watch board and re-run on a new revision with the
      moved-in / moved-out list.
    - Phase 3: answer cards, JSON share of watches, the opt-in AI step, voice.
    - Phase 4: watches fed by an Excel list of marks, alert rules, a sparkline across revisions.

**Risks.** A wrong plan gives a confident wrong number -> the plan is always shown in words with editable chips,
and the proof table is one click away; people ask for things the data does not hold (cost, schedule) -> say what
is missing and which module would hold it; the AI step might leak model facts -> only the question and schema
names are sent, shown before sending, off by default; Hinglish spelling varies -> the grammar test table grows
from real questions the team types, kept on the device; watches that break when names change -> a broken watch
says why, with a one-click rebind to a similar level or category name.

### Future update

- **Watches on other modules:** QA findings, BBS totals, the BOQ delta between revisions, so Bid Mirror,
  Ratio Radar and Order Clock can each publish their numbers as watches.
- **Team board:** share the board in a Room Link session (2026-11-02) so everyone sees the same numbers as the
  model is walked through, and a comment on a watch lands as a BCF issue pinned to the elements behind it.
- **Model Capsule (2026-11-04):** embed the board and the answer cards in the offline review file the client opens.
- **Change Radar and Issue Desk:** a watch that crosses its rule can raise an item in the next revision note.
- **Ask about drawings:** the same box over the DXF layers and the pipeline's skipped lines (Trace Lens), so
  "how many lines were skipped on layer S-COL?" has the same proof.
- **Ask across projects:** on the device, compare a question's answer over the firm's past models ("steel kg per
  m3 of slab on our last 10 podiums"), the link to Ratio Radar's norms.
- **Learned phrasing:** per-team synonyms ("shuttering" = formwork, "ghar" = unit) added by the user and kept
  on the device.
