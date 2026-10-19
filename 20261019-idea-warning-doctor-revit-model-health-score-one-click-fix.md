# 20261019 Idea: Warning Doctor — Revit model health score, warning triage and one-click safe auto-fix

## Entry 1

```
date-time:   2026-10-19 10:35 UTC (scheduled run, third entry of the day)
idea name:   Warning Doctor — "Your model has 1,284 warnings. Here are the 1,190 we fixed safely, and the 94 that need you."
             A new AnonGee pyRevit tool (Essential panel, next to Bulk Tool) that reads every Revit warning in the
             model, groups it by cause, gives the model a 0-100 Health Score, colours the offending elements in a
             3D "warning heatmap" view, and fixes the boring 80-90 % in one click with a preview: duplicate
             instances, slightly-off-axis lines, joined-but-not-intersecting, duplicate Marks, overlapping floors
             and walls, unenclosed rooms, beams not attached to columns, orphaned rebar. Each fix is a rule with a
             preview row, one undo step per rule, and an Excel / HTML report that the BIM lead can drop into the
             weekly coordination meeting.
potential:   Very high, used every day. Every Revit model a BIM team receives, links or generates carries warnings;
             structural models made by automation (BIM Generation, FramewinToBIM, cad2bim in this very toolkit)
             produce them in bulk: one bad join rule or duplicate Mark makes hundreds. A large warning count slows
             opening, syncing and regeneration, breaks schedules (duplicate Marks), breaks quantities (overlaps
             counted twice) and fails client BIM-execution-plan audits that set a "max warnings" KPI.
             Today a modeller clears warnings one by one in Manage > Review Warnings, a modal list with no
             filter, no grouping, no zoom-to-many and no fix button: realistic cost is 2-6 hours per model per
             week, and the work is redone after every regeneration.
             Time saved: on a 1,000-warning structural model, roughly 90 % fall in fewer than 10 rule families
             that a script can fix deterministically, so a 4-hour job becomes ~10 minutes of reviewing previews.
             Trending angle: "Model health" and "0 warnings" before/after screenshots are a recurring high-engagement
             post type in the Revit/BIM LinkedIn and Instagram community (warning counts dropping 1,284 -> 94 on a
             gauge, heatmap going from red to grey). A shareable Health Card PNG is built in, so every run can be
             a post.
             Market: free options only report (Autodesk "BIM Interoperability Tools - Model Checker", pyRevit's
             basic warnings listing); fixing in bulk is behind paid per-seat products (Ideate Review / Ideate
             Explorer, some paid Dynamo-package and plug-in bundles). To my knowledge none ship structural-specific
             fix rules (beam-column attachment, duplicate Marks after a generator run, rebar outside host) or close
             the loop back to the generator that caused them.
```

### Description

**The pain (daily life of a BIM modeller and BIM lead).**
- Revit's own warning dialog is a flat, modal list: no grouping by type, no count per category, no "select all of
  this kind", no fix, no export. Most people ignore it until a client audit or a crash forces a clean-up.
- Generated models are the worst offenders. This toolkit already knows it: `revit/transactions` carries a
  `SuppressWarningsPreprocessor`, and the backlog records runs where "seven warnings, one per level" or a
  duplicate-Mark warning went unseen. Suppressing hides warnings during a run; nothing reports or removes them after.
- Duplicate Marks break the bar-bending, beam and column schedules that `Export Schedule` sends to Excel.
- Overlapping floors / walls / beams double quantities; "joined but do not intersect" joins and stale joins slow
  regeneration and produce ugly sections.
- Lines 0.01 degrees off axis (from imported DXF in cad2bim, or a rotated survey) make dimensions read 4999.99 and
  break alignment constraints.
- Warning count is a BEP / ISO 19650 audit KPI. Leads need a number and a trend, not a 40-page dialog dump.

**The idea.** One modeless window with four tabs; Revit stays usable while it is open (same pattern as Auto
Level Manager).

1. **Scan.** `doc.GetWarnings()` returns every `FailureMessage`. For each one keep the failure-definition id
   (`GetFailureDefinitionId()`, matched against `BuiltInFailures.*`), severity, description text, failing and
   additional element ids, their category, level, workset and creator (worksharing tooltip). Scanning a
   5,000-warning model takes a few seconds and changes nothing.
2. **Triage.** Warnings are grouped by **cause, not text**: one row per rule family with a count, the categories
   involved and a severity badge (Data-breaking / Quantity-breaking / Performance / Cosmetic). Click a row to
   select all its elements; "Isolate in 3D" opens a dedicated `Warning Doctor` 3D view with graphic overrides:
   red = data-breaking, orange = quantity-breaking, yellow = cosmetic, everything else halftone. That is the
   heatmap.
3. **Fix (preview first, always).** Each rule family has a fixer. Pressing it shows a table of exactly what will
   change (element, before, after); ticked rows are applied in one transaction per rule, so each rule is one
   Ctrl+Z. Pinned, grouped, linked and other users' borrowed elements are skipped and listed, the same rule
   `Round Element Dist` already follows.
4. **Report.** Health Score, per-rule counts before / after, list of what remains for a human, and history of
   previous scans (stored in Extensible Storage in the model, so the trend survives across users). Export to
   Excel (reusing the `Export Schedule` styling) and a one-page HTML / PNG **Health Card**.

**First rule pack (deterministic, safe, the high-volume ones).**

| Warning family (BuiltInFailures) | Safe fix | Reuses |
| --- | --- | --- |
| Identical instances in the same place (`OverlapFailures.DuplicateInstances`) | Keep the oldest id (or the one with more hosted elements / tags), delete the rest | `Bulk Delete` UI pattern |
| Joined elements do not intersect (`JoinElementsFailures.JoiningDisjoint`) | `JoinGeometryUtils.UnjoinGeometry` on that pair only | `Unjoin by Category` |
| Wrong join order / element cut by the wrong host | Re-apply the category priority | `Join Priority` |
| Line / wall / beam / grid slightly off axis | Snap the `LocationCurve` direction to the nearest 0 / 45 / 90 deg (or to the grid) within a tolerance you set | `Round Element Dist` |
| Elements have duplicate Mark / Number values | Renumber the later duplicates by level + grid ("B-3F-C/4-01") or by rule; preview shows old -> new | `Multi Filter Parameter` |
| Overlapping floors / walls / framing | Report the overlap volume; for exact duplicates delete, for partial overlaps open the pair for review (never guess) | `Draw Floor` inner-face logic |
| Beam end not attached to column / slightly off column | Extend or trim to the column face when the gap is below tolerance | cad2bim geometry utilities |
| Room / area not enclosed or in the same region | Zoom-list only (needs design intent) | — |
| Rebar outside its host / host deleted | Select and isolate; optional delete of orphans after confirm | `Obscured Rebar` view handling |

Rules that need human judgement are never auto-fixed: they get a "Review queue" with zoom-to, a note field and an
owner, so the meeting assigns them instead of discussing a dialog screenshot.

**Health Score (simple, explainable).** Start at 100; subtract weighted counts per 1,000 elements
(data-breaking x5, quantity-breaking x3, performance x1, cosmetic x0.2), plus small penalties for unused families,
in-place families, unplaced views and unused line/fill patterns (already handled by `Bulk Delete`). The weights live
in a JSON file the BIM lead can change; the card always prints the formula so the number is defensible in an
audit.

### Implementation

**Where it lives (pyRevit-addons).**
- New button `Essential.panel/Bulk Tool.stack/Warning Doctor.pushbutton` (or a new `QA.panel`), script thin as
  the house style requires; logic in `lib/py3/anongee_toolkit/operations/warning_doctor/`:
  - `scan.py` — `FailureMessage` -> plain dataclass `WarningRecord` (no Revit objects held, so the window can stay
    modeless and the logic is unit-testable off Revit like the existing `tests/`).
  - `rules/` — one module per family: `match(record) -> bool`, `plan(doc, records) -> [FixRow]`,
    `apply(doc, rows)`. Adding a rule = adding a file; rules can be shipped as a pack.
  - `score.py` — weights JSON + history read/write (Extensible Storage schema on `ProjectInfo`).
  - `report.py` — Excel via openpyxl (already bundled), HTML card, PNG via the view-export API.
- Transactions through the existing `revit/transactions.RevitTransaction` and a `TransactionGroup` per run, so the
  whole session can be undone too. Each rule re-reads warnings after it applies and reports "fixed / remained /
  new warnings created" — a fix that creates new warnings is rolled back automatically.
- UI from the existing `UI_Templates` (modeless WPF with an `ExternalEvent` for writes, as in Auto Level Manager).

**Close the loop with the generators (the unique part).**
- After `BIM Generation`, `FramewinToBIM` or `cad2bim` finishes, offer "Run Warning Doctor" and write the result
  into that run's `FINDINGS` sheet. Each rule family maps back to a generator setting ("duplicate Marks came from
  the C1/C1 text on layer S-COLS", "off-axis came from DXF rotation 0.003 deg") so the next run is clean at the
  source rather than cleaned after.
- In **cad2bim (web)** and **Structura**, the same rule ids run on IFC (duplicate GlobalIds / Tags, overlapping
  solids, off-axis members) so the score is comparable between the Revit model and the web/IFC model.

**Phases.**
1. Week 1: Scan + Triage + heatmap view + Excel report (read-only, zero risk, immediately useful).
2. Week 2: first five fixers (duplicate instances, disjoint joins, off-axis, duplicate Marks, join order) with
   preview and undo; tests on recorded `WarningRecord` fixtures.
3. Week 3: Health Score + history + Health Card PNG; hook into the three generators' `FINDINGS` output.
4. Week 4: beam-to-column attachment fixer, rebar orphan handling, review queue with owners.

**Test plan.** A seeded test model with known counts per family (e.g. 50 duplicates, 30 disjoint joins, 20 off-axis
beams, 40 duplicate Marks); assert counts after each rule, no new warnings, element count change equals the planned
deletes, and that pinned / grouped / linked elements were untouched.

### Future update

- **Scheduled health watch:** run the scan on open / sync (read-only) and post the score to a team channel or the
  BIM lead's dashboard; alert when a sync adds more than N warnings, naming the user and rule.
- **Model-wide performance checks beyond warnings:** heavy in-place families, imported CAD in views, unused
  view templates and views not on sheets, oversized groups, design options left on — each as a rule in the same pack.
- **Client / BEP profiles:** load a client's KPI (max warnings, naming rules, required parameters) and print a pass /
  fail card for submission; pairs with the IDS QA idea (20261014) for parameter checks.
- **Learning rules:** record which review-queue items people resolve and how, and propose a new deterministic rule
  when the same manual fix is done 20+ times.
- **Multi-model league table:** scan all project models on a server folder overnight, rank by score and trend —
  the "wow" slide for management and a natural LinkedIn post ("our office average went from 61 to 92 in a month").
- **Natural-language fix requests:** "remove all duplicate beams on level 3 except the ones with rebar" translated
  into a rule plan, always shown as a preview before apply.
