# 20261020 Idea: Toolsmith — say it once in plain English, preview it in Revit, save it as a team button forever

## Entry 1

```
date-time:   2026-10-20 (scheduled run, second entry of the day)
idea name:   Toolsmith — "Type the chore. See the coloured preview. Press Run. Press 'Save as button'. Done for the whole team."
             A chat panel inside the AnonGee pyRevit tab, plus a local Revit MCP server, that turns a modeller's
             plain-English chore ("renumber all L3 beams left-to-right as B301.., then tag them", "set Comments =
             'Pour-2' for every slab and column inside grid A-D / 1-4 on L5", "copy column sizes from this Excel
             into Revit and report the mismatches") into a RECIPE: a short, typed, step-by-step plan built only
             from the toolkit's own safe primitives (select, filter, sort-by-grid, set parameter, rename, tag,
             join, export schedule, bridge commands such as levels.sync / grids.sync). The recipe is dry-run first:
             every element it would touch is coloured in a temporary 3D view with a before/after table. Run is one
             undo step. "Save as button" writes it to the team library as a real pyRevit pushbutton with its own
             icon, input prompts and version number, so the next modeller gets the tool without the chat.
potential:   Very high, every day, every modeller. Most of a BIM modeller's lost hours are not hard modelling; they
             are small repetitive chores that are slightly different on every project, too small to ask a developer
             for and too fiddly to do by hand: renumbering, re-marking, bulk parameter fills, tagging a set,
             isolating "everything on grid C between L2 and L6", pushing an Excel column into parameters. Today
             these are done click-by-click (30 min to 3 h each, several times a week), or with a Dynamo graph that
             only one person on the team can read, or not at all.
             Time saved: a typical chore (200-element renumber + tag) drops from ~60 min to ~3 min the first time
             and ~20 s every time after, because it is now a button. Across a 6-person team, 3-5 such chores per
             person per week ≈ 15-25 hours/week saved; the team tool library grows by itself from real work.
             Trending angle: "AI agent drives Revit" / "Revit MCP" demos are one of the most-shared AEC-AI post
             types on LinkedIn and Instagram in 2025-26 (prompt typed on the left, Revit elements changing colour
             and renumbering on the right). Toolsmith adds the part those demos skip: a safe preview, an undo, and
             a button the whole team keeps. The 20-second screen recording "typed it -> previewed -> saved as
             button -> colleague clicks it" is itself the post.
             Market: Autodesk Assistant (preview, Autodesk Construction Cloud-tied, limited actions); ArchiLabs and
             similar AI Revit copilots are paid per seat and cloud-hosted; open-source "Revit MCP" servers exist but
             let the model run arbitrary C#/Python inside the model with no preview, no undo grouping and no
             reusable output. Dynamo/pyRevit are free but need a scripter. To my knowledge nothing free turns a
             chat request into a reviewed, versioned, shareable pyRevit button built on a whitelisted vocabulary.
```

### Description

**The pain (daily life of a BIM modeller and BIM lead).**
- Chores repeat with small variations: "Mark = B + level + sequence", but this project wants left-to-right per
  grid strip, the next one top-to-bottom. Existing buttons (Bulk Rename, FilterParameter, Parameter Combine,
  Export Schedule) each do one step; chaining them takes many clicks and the chain is lost when the window closes.
- Dynamo graphs are write-once, read-never: the one person who built it leaves, the graph breaks on the next Revit
  version, and nobody else dares to open it.
- Asking a generic AI to "write me a Revit script" gives code that may delete things, runs without a preview, and
  is thrown away after use. A BIM lead cannot let juniors paste unknown Python into a live central model.
- The toolkit already has the building blocks to fix this: an in-Revit HTTP route server and command bus
  (`AnonGee.extension/startup.py`, the Excel `bridge`), typed commands (`levels.sync`, `grids.sync`, declared
  `columns/beams/slabs.sync`), an MCP server pattern in `etabs-mcp/apps/mcp-server`, and the `@cad2bim/mcp` helper.
  What is missing is a vocabulary that sits on top of them and a way to keep what the AI made.

**The idea, in five parts.**

1. **Recipe language (the safety rail).** A recipe is JSON, not code: an ordered list of steps, each a call to a
   whitelisted primitive with typed arguments, e.g.
   `select(category=StructuralFraming, level="L3")` -> `sort(by="grid", axis="X", tolerance_mm=50)` ->
   `set_param(name="Mark", pattern="B{level_no}{seq:02}")` -> `tag(family="Beam Tag", offset_mm=300)`.
   Primitives are grouped by risk: *read* (select, filter, sort, measure, report), *write* (set_param, rename,
   tag, join, move-by-snap), *destructive* (delete, unjoin, change type). Destructive steps need the BIM lead's
   role flag and always show a red preview. No step can run free code, so a recipe can never do more than its
   primitives allow.
2. **Chat to recipe.** The modeller types the chore in the Toolsmith panel. The AI (through the local MCP server)
   sees only: the primitive catalog, the current view/selection summary, category and parameter names, level and
   grid names. Never the whole model. It answers with a recipe plus one-line explanations per step, and asks a
   question when something is ambiguous ("Sequence per level or continuous across levels?").
3. **Dry run and preview.** Every recipe runs first in "plan" mode inside a rolled-back transaction: a temporary
   3D view colours touched elements (green = changed, amber = skipped with reason, red = would be deleted), and a
   table shows old value -> new value per element. Counts are shown big ("212 beams, 212 Marks, 212 tags, 0
   deletes"). Nothing is kept until the modeller presses Run.
4. **Run as one undo.** Run executes the same plan in one `TransactionGroup` named after the recipe, so Ctrl+Z
   undoes it as a whole. A run log (who, when, model, counts, recipe version) is written next to the model.
5. **Save as button (the part that compounds).** One click writes `Toolsmith.panel/<Name>.pushbutton/` into the
   team's shared extension folder: `recipe.json`, a generic `script.py` runner, an icon (picked from the brand
   set), a tooltip from the chat summary, and input prompts for the parts that change per project (level, prefix,
   tag family). Buttons carry a version and an owner; editing one opens the chat with the recipe loaded
   ("also skip cantilever beams"). The BIM lead can promote a button from "personal" to "team" after review.

**Extra modes that reuse the same engine.**
- **Watch me.** Press record, do the chore by hand on 3-5 elements, press stop. Revit's `DocumentChanged` events
  are turned into a draft recipe ("you set Mark on 4 beams in X order with this pattern; apply to the other 208?").
  This is the no-typing path for modellers who think in clicks.
- **From Excel.** Recipes can start with `read_excel(range)` through the existing bridge, so "push this sheet into
  these columns" becomes a saved button that BIM coordinators run every week.
- **Explain this button.** Any saved recipe can be read back in plain English, so a new team member understands
  what a tool does before clicking it (the opposite of an unreadable Dynamo graph).

**Why it creates the wow effect.** The modeller types one sentence, the model lights up green in a few seconds,
Run takes one click, and the result becomes a permanent ribbon button with the team's logo. Every week the
AnonGee tab visibly grows with tools nobody had to program.

### Implementation

**Where it lives.** Mostly in `pyRevit-addons` (new `Toolsmith.panel`), using the existing bridge transport;
a small MCP server package modelled on `etabs-mcp/apps/mcp-server`. No change to other products needed for v1.

1. **Primitive catalog (`lib/py3/toolsmith/primitives/`).** One module per primitive with a JSON-schema of its
   arguments, a `plan(doc, ctx, args) -> PlanResult` (pure, no writes) and an `apply(doc, plan)` (writes). Start with
   ~20 primitives wrapped around what the toolkit already does: category/level/workset/parameter filters
   (FilterParameter), rename patterns (Bulk Rename), Parameter Combine, sort-by-grid (new, uses grid intersections),
   tag-by-category, join priority, round-element-distance, export schedule, `bridge.levels.sync` / `grids.sync`.
2. **Recipe runner (`toolsmith/runner.py`).** Validates a recipe against the catalog schemas, resolves inputs,
   executes `plan` for every step (selection flows from step to step), then `apply` inside one `TransactionGroup`
   with `SuppressWarningsPreprocessor` off so real warnings surface. Hard caps: max elements per step, no step may
   touch linked models, destructive steps require a role flag stored in the team config.
3. **Preview view.** Reuse one temporary 3D view `Toolsmith Preview` with `OverrideGraphicSettings` (green /
   amber / red) and section box fitted to touched elements; results table in the modeless WPF window (same pattern as
   Auto Level Manager), with click-to-zoom.
4. **Local MCP server (`toolsmith-mcp`, Node/TypeScript, like `etabs-mcp`).** Tools exposed to the AI client:
   `list_primitives`, `describe_context` (view, selection summary, level/grid/category/parameter names),
   `plan_recipe` (returns preview counts + sample rows), `explain_recipe`. Deliberately **no** `run_recipe` tool:
   only a human presses Run in Revit. Talks to Revit through the existing `localhost:48884/anongee/*` routes with a
   per-session token. Works with Claude Desktop / any MCP client, and from the built-in chat panel using the
   team's API key.
5. **Button writer.** Generates the pushbutton folder from a template: `bundle.yaml` (title, tooltip, author,
   version), `recipe.json`, `script.py` (5 lines: load recipe, ask inputs with pyRevit forms, call runner), icon
   from `UI_Templates`. Team library path configurable (network drive / synced folder / git repo); `git` mode commits
   each new or changed button so recipes get history and review.
6. **Watch-me recorder.** Subscribe to `Application.DocumentChanged` while recording; map changed element ids +
   parameter deltas to the nearest primitive pattern (same param changed on N same-category elements -> `set_param`
   with inferred pattern via the Bulk Rename pattern solver); show the draft recipe for confirmation.
7. **Tests.** Primitive `plan()` functions are pure and unit-testable with the repo's existing fake-document test
   helpers; a golden-recipe suite (10 real chores from the team) runs on a sample model in CI-like manual checks.

**Rough effort.** v1 (catalog of ~20 primitives, runner, preview, chat via MCP, save-as-button): 4-6 weeks for one
developer. Watch-me recorder: +2 weeks. Excel step and explain mode: +1 week.

### Future update

- **Same recipe, other apps.** Map the vocabulary to `cad2bim` (IFC in the browser) and `etabs-mcp` (ETABS), so
  "rename all L3 beams B301.." is one recipe that runs in Revit, the web IFC editor or ETABS, keeping Marks in step
  across the three and feeding Twin Check / Change Radar.
- **Team marketplace page.** A small web page listing the team's buttons with run counts and "hours saved" (runs x
  measured manual time), a weekly LinkedIn-ready "Toolsmith saved us 41 hours this week" card for the BIM lead.
- **Recipe suggestions.** After Warning Doctor or IDS QA runs, Toolsmith proposes a ready recipe for the top finding
  ("212 duplicate Marks: run 'Renumber by grid'?").
- **Voice input** on site laptops/tablets, and a WhatsApp entry via the Chat-to-Model idea for read-only recipes
  ("how many L5 columns still have no pour tag?").
- **Primitive SDK.** Developers add a primitive once (schema + plan + apply + test) and every past and future chat
  can use it; the catalog becomes the toolkit's public automation API.
- **Signed recipes** for enterprise clients: team-approved buttons carry a checksum so a modified recipe cannot run
  on a central model without re-approval.
