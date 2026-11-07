# 20261107 Idea: Déjà Model — "we did this before": find the closest past element, core or floor in your own old projects by look, and borrow it, with its rebar and BBS, into the open model

## Entry 1

```
date-time:   2026-11-07 (scheduled run, third entry of the day)
idea name:   Déjà Model (Look-Alike Search across the office's past IFC / .c2b / DXF projects, with a
             "Borrow" button that brings the match, its rebar and its BBS into the open model)
potential:   High, and daily. Every modeller has said "we did a lift core like this on the Pune job, where is
             it?" and then lost 20 to 60 minutes opening old folders, then redrew it anyway (estimate from how
             this usually runs, not measured here). The office's best work sits in finished projects that
             nobody can search. Déjà Model makes the archive searchable by LOOK and by NUMBERS, on the
             modeller's own machine, and lets them pull the part into today's model in one click. The wow: you
             select a stair core in the new job, press one key, and six thumbnails of your own past stair
             cores appear, ranked, each with "used on 3 projects, steel 112 kg/m3, checked and issued". You pick
             one, press Borrow, and it lands in the new model, resized to the new grid, rebar and BBS included.
```

### Description

**The pain.**
- Offices repeat themselves. Transfer beams, lift cores, ramps, water tanks, typical podium slabs, parapet
  and sunk-slab details: the second time should take minutes, but the first one is buried in an old folder.
- Finding it depends on who remembers. When a senior leaves, the memory leaves. Juniors redraw from nothing.
- Copying from an old Revit project is painful: different grids, levels, families and units; the part arrives
  with the old project's warnings and wrong types.
- Folder names and file names are the only index. Nobody tags old projects.
- The part that is worth most, the **checked** reinforcement and its BBS, is exactly what is not reused.

**What exists today (checked 2026-11-07, not exhaustive).**
- In this repo's `idea` branch: Ratio Radar compares **ratios** (kg/m3 and similar) with past projects. Scope
  Scout calibrates **hours** from past projects. Family Forge makes a **new** family from a datasheet. Mark
  Merge consolidates marks **inside** one model. Tower Stacker copies a typical floor **inside** one job. A
  search of the idea files for "similarity", "lookalike" and "find similar" found nothing that searches old
  projects for a **part** and brings it in. Déjà Model fills that gap.
- In cad2bim (`docs/design/`): the app already opens IFC and `.c2b` on the device, keeps model identity by
  GlobalId, has the Revit bridge, BOQ, BBS, QA, "Select Similar", saved views, and (planned) several projects
  open at once (`multi-project.md`) and the reopen cache (`reopen-cache.md`). Those are the pieces; the **index
  of the archive and the Borrow step** are new.
- In the market (found by web search today; vendor claims are marketing and unverified): **ARKI** (listed in the
  Autodesk University 2026 exhibitor catalogue) builds a searchable library of standards and details from past
  Revit projects and drops reusable details into an open Revit project. **PiAxis AI** learns from a firm's
  past Revit projects to find and reuse details. **Nomic** does semantic search over Revit **family** libraries.
  Academic work (a 2025 *Buildings* paper on BIM search by deep embedding of geometry, meaning and topology;
  a "BIM library transplant" Revit add-on) shows whole-element similarity is a live research area. So the idea
  **exists, but as paid, cloud, Revit-only products** (pricing not published on the pages I saw). I did not
  find a free tool that works from IFC, `.c2b` and DXF on the user's own device, includes the rebar and BBS, and
  never uploads the archive. Treat that as unverified until a proper search.
- Trend: "agentic BIM", Revit with MCP and "ask your model" are everywhere in 2026 posts. One source says
  Autodesk released an open-source Revit MCP server on 18 June 2026 (a single AI-written news summary, so
  verify). Individual LinkedIn posts pair Revit with Claude through MCP. I could **not** measure what is
  trending on LinkedIn or Instagram, and nothing here is trend data. The honest hook is: everyone is posting
  "AI for my model"; a modeller's real question is "show me what **we** already did".

**The idea.** A **Déjà** panel (and a command "Find like this"). It has three parts: the index, the search,
and Borrow. Everything runs in the browser; the archive never leaves the device.

1. **Index the archive once.** Point cad2bim at a folder of old models (IFC, `.c2b`, DXF/DWG). The File System
   Access API gives read access to that folder; nothing is uploaded. For each file the app reads it with the
   existing readers and stores a small **card** per part in the browser's own database: category, type and
   size, level, bounding box, the count and spacing of its bars if it has rebar, quantities, a few numbers that
   describe its shape, and a tiny preview picture. The cards are small (kilobytes); the models are not copied.
   It runs in the background and can be paused. New files are picked up on the next visit.
2. **Search by selecting, by words, or by a sketch.**
   - *Select and press "Find like this":* pick a core, a stair, a beam run, a slab bay, or a whole floor.
   - *Words:* "ramp 1:10 with retaining wall", "transfer beam over 1200 deep", "UGT 50 kL" using the English
     grammar the app already has for commands (no AI needed for the basics).
   - *Numbers:* "span 8 m, depth 600 to 750, M30" as filters.
3. **Ranked, explained matches.** Each result shows the preview, the project it came from, the date, and **why
   it matched** ("same footprint within 5%, same level height, same number of lifts"). It also shows what the
   office learned there: the later **revision count**, QA findings that were open when it was issued, the
   steel ratio, and whether the project was marked "built" or "issued" in the card. A part with open errors is
   shown but dimmed, so a bad example is not copied by accident.
4. **Compare before you borrow.** Put two matches next to each other, or one next to the part in the open
   model, with the differences listed: depth 750 against 600, 12 bars against 16, wall 230 against 200.
5. **Borrow.** One click brings the match into the open model: geometry, type, rebar and BBS rows. It is
   **re-fitted**: moved to the picked grid intersection, rotated to the new grid, resized to the new bay with a
   preview of what will stretch and what will not (a stirrup spacing yes, a column size no). Every changed
   number is listed; nothing is silent. The result is one undo step, marked with its source ("from Pune
   Phase 2, C14") so anyone can trace it. It then goes through the usual QA and cover checks of the open model.
6. **For Revit users.** The Revit bridge sends the borrowed part as native elements (types matched by name or
   created from a template), so the modeller can stay in Revit and use cad2bim as the finder.
7. **Office memory view.** A plain list of what the office has built most: "lift cores: 41, stair cores: 63,
   transfer beams: 9", so a manager can see the strengths and the gaps and decide which parts to turn into
   standard families. (No names of people, only projects.)

**Time saved (estimate, to be measured).** Finding a past example: from 20 to 60 minutes to under a minute.
Re-making a lift core with rebar: from half a day to about 15 to 30 minutes with checks. The saving that is not
an estimate: reuse of a **checked** part carries its checks with it, so fewer new mistakes are made.

**Why it is a "wow".**
- The 20-second clip: a modeller selects a plain concrete box in a new job and presses one key; six lookalikes
  from the office's old projects fan out with their steel ratios; one click on Borrow and the core with all its
  bars appears on the new grid, with the "changed 7 numbers" list beside it. "Your office's memory, in one key."
- It makes every past project **pay again**, and it gets better with each project finished, with no extra work
  from the team.

### Implementation

- **Where it lives.** The reader and card builder go in `packages/engine` (it already reads IFC and the BOQ/BBS
  data); the index store and ranking are pure functions with unit tests; the panel and the Borrow flow are in
  `apps/web`, using design system tokens. DXF cards come from the CAD core's reading. No new package at first.
- **Cards, not copies.** A card holds: part id (file + GlobalId), category/type, dimensions, level height,
  footprint outline simplified to a few points, bar pattern summary, quantities, QA state at the time, a
  thumbnail rendered once by the existing viewer. Stored in IndexedDB; rebuilt if the file's modified time
  changes. Files stay where they are; if a file is moved, its cards are marked missing, not deleted.
- **Ranking, step one (no AI).** A weighted distance over the card's numbers (category, footprint outline
  compared after normalising scale and rotation, depth, bar pattern, quantities), with the weights shown to
  the user and adjustable. Explanations come straight from which terms were close. This is predictable and
  testable.
- **Ranking, step two (optional).** A small on-device model (WebAssembly, no server) for a shape embedding, so
  looks-alike works across different modelling styles. Used only to re-order the step-one top 50, so a wrong
  guess cannot hide a good rule-based match. Licences checked first (`CLAUDE.md`: no GPL, Apache/MIT/BSD ok).
- **Borrow.** Reuse the native editing layer (`docs/design/native-editing.md`): a Borrow is a group of creates
  with their rebar sets in one transaction. Re-fitting uses the datums and constraints design
  (`datums-and-constraints.md`): the part is anchored to a grid intersection and a level; stretch rules are
  stored per category and shown before applying. Rebar bars are regenerated from the pattern (count, spacing,
  cover, hooks) rather than copied bar by bar, then BBS rows come from the live model as usual.
- **Safety.** Nothing is uploaded; reading the archive needs one folder permission, which the browser asks for
  and which the user can revoke. Borrowed parts carry a source tag. The part's old QA findings are shown so a
  known-bad detail is not copied. Names of clients in old file names are never shown outside the device.
- **Steps (one developer).** (1) folder permission, card builder for IFC columns, beams, slabs, walls and a
  rebar summary, and a flat list with filters (about 1 to 2 weeks); (2) "Find like this" ranking and the
  compare view (about 1 week); (3) Borrow for single elements and runs, with re-fit preview (about 2 weeks);
  (4) cores, stairs and whole floors, and `.c2b` cards (about 2 weeks); (5) the Revit bridge hand-off (about 1
  to 2 weeks); (6) optional embedding step.
- **Testing.** Synthetic archive of 20 small IFC files made by the test generator (no client files in git), with
  hand-known "correct neighbour" answers; checks that the ranking puts the intended match in the top 3; a
  Borrow test that the bar count, spacing and cover after re-fit equal the hand calculation and that one undo
  restores the model; a test that a moved file marks cards missing; and a privacy test that no network request
  is made while indexing.
- **Rules to respect in cad2bim** (`CLAUDE.md`): nothing leaves the device unasked; tokens only for colours; a
  version bump with a changelog entry for each package touched; the decision written to `docs/decisions.md`.

### Future update

- **Ask by photo or sketch:** photograph a hand-drawn section or an old drawing, and the search ranks by that.
- **Team archive without a server:** share the card index (not the models) as a file, so a new joiner can see
  the office's memory on day one and ask a colleague for the model only when needed.
- **Auto-standards:** when the same part has been borrowed three times, offer to turn it into a standard
  family or template (links to Family Forge and the `.c2f` families design).
- **Lessons attached:** let a reviewer pin a note on a card ("do not copy: the stirrup spacing failed at
  site"), shown whenever it is a match. Link to Cover Cop and Joint Judge checks.
- **Cost memory:** show what the matched part cost last time (from the BOQ rates saved with that project) next
  to the match, then feed Rate Rocket.
- **MCP tool:** expose "find like this" and "borrow" through the cad2bim agent helper (`packages/mcp`), so an
  AI assistant can answer "what did we use for a 10 m transfer beam?" from the office's own models and cite the
  project.

### Sources

- [AU 2026 exhibitor catalogue: ARKI](https://conferences.autodesk.com/flow/autodesk/au2026/exhibitorcatalog/page/catalog/exhibitor/1784290702935001BJmO)
- [Nomic: AI for Revit families](https://www.nomic.ai/glossary/ai-for-revit-families)
- [Intelligent BIM Searching via Deep Embedding of Geometric, Semantic, and Topological Features (Buildings, 2025)](https://doaj.org/article/ca4d9557347d46ebbd028f705a98fbf8)
- [BIM Library Transplant, Yonsei](https://yscholarhub.yonsei.ac.kr/item/800fa357-4d43-4c51-9d28-5e4a9d68acdb)
- [BIM in 2026: AI and collaboration, PBC Today](https://www.pbctoday.co.uk/news/digital-construction-news/bim-in-2026-artificial-intelligence-best-in-class-collaboration/162865/)
- [Autodesk open sources Revit MCP (secondary summary, verify)](https://gentic.news/article/autodesk-open-sources-revit-mcp)
