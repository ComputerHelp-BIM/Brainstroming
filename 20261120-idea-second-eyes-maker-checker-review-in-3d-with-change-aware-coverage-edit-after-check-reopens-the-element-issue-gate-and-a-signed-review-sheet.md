# 20261120 Idea: Second Eyes — The checker paints the 3D model green as they review it. Anything edited after it was checked turns amber again. "Reviewed 94% of 14,200 elements, 212 changed since checked" before every issue.

## Entry 1

```
date-time:   2026-11-20 (scheduled run, UTC)
idea name:   Second Eyes (element-level maker-checker review with change-aware coverage, in the browser,
             nothing uploaded)
potential:   High, and it is used every week. Every BIM team that issues drawings or models has a
             "checker" step. Today it lives in a marked-up PDF, an Excel tick list or memory. Nobody
             can answer two questions: "What did the checker actually look at?" and "What changed after
             they looked?" The second question is where late errors come from. Time saved: NOT measured.
             First test: take 2 real models (kept out of git) with a planted set of 20 errors. One
             checker uses the usual way (PDF + Excel), another uses Second Eyes. Report (a) minutes,
             (b) errors found, (c) elements the checker skipped without knowing. Do not quote hours or
             catch rates until that is done.
             Market check (2026-11-20, two standard searches; not exhaustive, so "I did not find one",
             not "none"):
               - Solibri (rule-based checker; Solibri CheckPoint is a cloud version launched Jan 2025):
                 checks rules, not human review. Paid.
               - Revit Model Review add-in, Navisworks review sets and ACC / BIM Collaborate Issues:
                 they track issues and clashes, not "which elements has a person reviewed, and has it
                 changed since". Paid or inside paid suites.
               - A NOVEDGE guide (Jan 2026) recommends pre-issue checks before milestones and triage as
                 "fix now / assign / accept with rationale". That is a process, not a coverage tool.
               - Solibri's 2025 interview pushes "shared responsibility": each person owns the quality of
                 their own part. Good intent, no way to see it.
               - I did not find a free, browser-based tool that records per-element human review and
                 re-opens it when the element changes. I did not check every add-in store. I could not
                 measure LinkedIn or Instagram counts; "QA/QC before issue" and "AI model checking" are
                 common post themes, but that is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Twin Check, Sheet Sentry, Warning Doctor, Ratio Radar, Export Doctor: machines check
                 rules. Second Eyes records what the human checked and what they have not.
               - Issue Desk: issues sheets. Second Eyes gives it a gate: "review coverage 100%".
               - Change Radar and Rev Merge: show what changed between revisions. Second Eyes uses the
                 same diff to re-open only the changed elements.
               - Meeting Mind, Clash Sorter: raise BCF issues. Second Eyes writes its findings the same way.
               - Dojo Drill and Lane Lead: training and splitting work. Second Eyes uses the same zones
                 to split the review between two checkers.
             The wow: the checker opens the model. All 14,200 elements are grey. They click a column
             line and the whole grid line, level by level, turns green with one "Checked" press; they
             click one wrong beam, type "size 300x450 per schedule, model has 300x400", and it turns red
             with a pin. The bar reads "Reviewed 61%". The maker fixes the beam and the next day
             edits some other walls. The checker opens the model again: 31 green elements are now amber
             "changed since checked", each one with a 3D ghost of what changed. The bar says 94%. They
             press "Sign" and a one-page review sheet is made.
```

### Description

**The pain.**
- The checker has no honest coverage number. "I checked the second floor" means different things to
  different people.
- A reviewed model changes afterwards. The checked part and the edited part are mixed, and the issue
  goes out with "checked" on it anyway.
- Findings live in four places: the PDF, the chat, the Excel and someone's head. The maker cannot tell
  which are done.
- The senior who signs has only the checker's word.

**The idea.** Five moves.
1. **Paint to review.** Select by grid line, level, type, category or a lasso, press Checked (green),
   or Problem (red with a note and a pin). Untouched elements stay grey. A bar shows reviewed % and,
   per level and per category, what is left.
2. **Change-aware.** Every check stores a fingerprint of the element (geometry, type, key properties
   and rebar if present). When a new revision loads, any element whose fingerprint changed goes amber
   "changed since checked", with a ghost of the old shape. Elements that did not change stay green.
   The checker reviews only the delta, not the whole model again.
3. **Checklists per element type.** A short list per type (for a beam: size vs schedule, mark, level,
   cover, bar spacing; for a slab: thickness, openings, edge). Tick lists are saved as templates and
   reused. The checker can skip a list for a type and say why.
4. **Close the loop.** Each red element becomes a BCF-style issue for the maker, with the element
   attached. When the maker fixes it, the element comes back amber for a re-check, not silently green.
5. **Gate and sign.** Issue Desk can require "coverage >= your rule" before it lets a sheet or IFC go
   out. "Sign" makes a one-page review sheet: who, when, revision, coverage by level and category,
   issues opened and closed, what was left unreviewed and the reason. Two-step sign (checker, then
   senior) is optional.

**What a modeler sees.**
- A "Review" panel: coverage bar, levels, categories, open problems, and a colour-by switch.
- Click an element: its checks (who, when, which list), its problems, and "changed since: yes/no".
- On a new revision: "212 changed since checked; 9 new, 3 deleted". One click selects them all.
- A "who checked what" view for the BIM manager, so a hole in coverage is visible before the issue.

### Implementation

Inside the cad2bim IFC viewer and its project file; no server, nothing uploaded.
- **Review layer** (packages/engine, new module): a store of {GlobalId, status, checker, time,
  fingerprint, note, list results}. Pure functions, so they are testable with Vitest on small IFC
  fixtures. It is saved in the project file and exported as JSON/CSV; it does not touch the IFC itself
  unless the user asks (a "reviewed" property set is optional).
- **Fingerprint:** a hash of the element's class, type, placement, geometry bounds and a chosen set of
  properties (the user can pick which; default is size, level, material, mark). Rebar hosted on the
  element is included when present. Matching across revisions by GlobalId; if ids are not stable, fall
  back to the same type + level + position (same idea as Rev Merge). Say so on screen when it falls back.
- **Colour layer:** reuse the viewer's colour overrides and its selection tools (grid line, level,
  lasso). Grey/green/amber/red are tokens from the design system, never raw values.
- **Issues:** reuse the BCF writer already planned for QA autofix and Meeting Mind; each red element
  becomes an issue with a viewpoint.
- **Gate:** a rule in Issue Desk ("coverage by category >= X; no red open"). It is a warning with an
  override and a reason, never a hard lock, so a deadline day is not blocked by a tool.
- **Review sheet:** a one-page PDF/HTML from the existing report code; every number links back to the
  3D selection that shows it.
- **Agent access:** MCP tools in packages/mcp: "list unreviewed", "list changed since checked",
  "mark checked with note". An AI agent can pre-check obvious things and mark them "suggested", which a
  person must confirm; it can never mark an element "Checked" itself. The reason: an agent that signs
  its own work would defeat the whole point.
- **Build order:** (1) paint + coverage bar + fingerprint and amber on a new revision; (2) problems as
  BCF issues; (3) checklists per type; (4) Issue Desk gate and review sheet; (5) two-step sign and
  MCP tools.
- **Risks:** fingerprint too strict (every tiny move re-opens) or too loose (a real change is missed).
  Mitigation: choose properties, set a position tolerance, and always show the diff so the checker
  decides. A green mark can give false comfort, so the sheet states what was not covered. People may
  see it as surveillance of the checker; keep it as the team's record, shown per project, not per
  person league tables.
- **Check before release:** the 20-planted-errors test above; and a revision test where 50 known
  elements are edited, to count how many are re-opened and how many are missed.

### Future update

- **Review scope from the schedule:** "check everything poured next week" using Ready Board's
  lookahead, so review effort follows the site, not the model order.
- **Two checkers, one model:** split by zone (Lane Lead's zones) with live coverage for both.
- **Learned focus:** show which element types and levels had the most red marks in past projects and
  suggest where to look first (kept on the device, from the team's own history).
- **Pairs with Calc Bridge, Twin Check and Sheet Sentry:** a machine finding can be attached to the
  element and shown in the same review, so the human sees the machine's result and the human's own
  marks together.
- **Handover:** the review record joins the model capsule so a client or auditor can see the QA trail
  without asking for it.
