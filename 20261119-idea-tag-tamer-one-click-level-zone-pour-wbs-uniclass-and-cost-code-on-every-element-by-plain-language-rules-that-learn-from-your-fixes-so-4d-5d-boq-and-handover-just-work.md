# 20261119 Idea: Tag Tamer — Every element gets its Level, Zone, Pour, WBS, Uniclass and Cost code in one click, from rules written in plain words that learn from your fixes. 4D, 5D, BOQ and handover then link themselves.

## Entry 1

```
date-time:   2026-11-19 (scheduled run, UTC)
idea name:   Tag Tamer (rule-based, learning auto-tagging of IFC elements, in the browser, nothing uploaded)
potential:   High for daily use, and it is the missing base under 4D and 5D. Every 4D/5D, BOQ-by-zone,
             pour card or COBie sheet needs the same thing first: each element carries the right
             Level, Zone, Pour, WBS/activity code, classification and cost code. Today a modeler fills
             these by hand, by Excel paste, or by a one-off Dynamo script. Time saved: NOT measured.
             First test: take 3 real models (kept out of git), tag by hand with a stopwatch, then with
             Tag Tamer. Report (a) minutes, (b) elements needing a manual fix, (c) rules written.
             Do not quote hours or accuracy until that is done.
             Market check (2026-11-19, two searches; not exhaustive, so "I did not find one", not "none"):
               - Navisworks TimeLiner has rule-based auto-attach, but it matches a property value to a
                 task field exactly, so you must already have the codes in the model. It does not make
                 the codes. Paid.
               - BEXEL Manager (4D/5D, from about $537/year per a review site), Synchro Pro, Vico Office:
                 paid, and they also assume the elements are already coded.
               - ArchiCAD, BricsCAD and Vectorworks map classification systems to IFC types inside
                 their own authoring tools. Not a browser tool, and not for Zone, Pour or WBS.
               - Research (2025, Buildings): Random-Forest auto-classification of BIM objects to
                 Uniclass from IFC data. A paper and a prototype, not a daily tool. Its accuracy
                 figures come from one dataset; I have not reproduced them.
               - Community Claude "skills" suggest UniFormat/OmniClass/Uniclass codes from properties.
                 They suggest a code per element; they do not do zones, pours, WBS, learning or write-back.
               - I did not find a free, browser-based tool that tags Level + Zone + Pour + WBS + a
                 classification + a cost code together from plain-language rules and then learns from
                 corrections. I did not check every Revit add-in store. I could not measure LinkedIn or
                 Instagram counts; "data-ready BIM" and "AI agents in AEC" are common post themes, but
                 that is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Sheet Sync: edit values in Excel. Tag Tamer makes the values for you, then Sheet Sync
                 can still be used to review them.
               - Rate Rocket / 4D / 5D roadmap items: they consume codes; Tag Tamer produces them.
               - Pour Planner, Ready Board, Order Clock, Crew Curve, Bill Painter: all need Zone/Pour/
                 WBS on elements. One tagging pass feeds all of them.
               - Export Doctor and BEP Bolt: they check that Psets and classification exist; Tag Tamer
                 fills them.
               - Mark Keeper / Type Diet: marks and types. Tag Tamer is about grouping and coding.
             The wow: open a model. A bar shows "Tagged 3% of 14,200 elements". The modeler types one
             line: "Zone = the grid bay each element's centre falls in; Pour = slab and its columns
             below; WBS = Level + Zone + Category". The 3D repaints by zone in about a second and the bar
             climbs to 96%. The 4% left glow amber with the reason ("sits across two bays"). Fix one,
             and the rule offers: "Apply to the 11 others like it?" Then "Play 4D": the building rises
             zone by zone with no manual linking.
```

### Description

**The pain.**
- 4D and 5D demos look easy; the real work is linking. Tasks and elements only match when a code
  already exists on both sides.
- Codes are typed by hand, pasted from Excel, or made by a script that one person owns and nobody
  else can read. When the model changes, the codes go stale and nobody knows.
- Each tool wants its own field: the planner wants activity, the QS wants cost code, the client wants
  Uniclass, the site wants Pour no. Same job done four times.
- Untagged elements are silent. They simply miss the schedule, the BOQ or the handover sheet.

**The idea.** Four moves.
1. **Say the rule.** Plain language (English or Hinglish) or a small form: "Zone = grid bay", "Pour =
   slab plus the columns under it, per level", "Cost code = category + concrete grade + size band",
   "Uniclass = from IFC class and predefined type". The rule is shown as readable text and as a
   preview on the 3D, never as hidden code.
2. **See coverage.** A tagged-% bar and a 3D that colours by any tag. Untagged and doubtful elements
   glow, each with its reason. This is the check nobody has today.
3. **Teach once.** Fix one element by hand; Tag Tamer proposes the same fix for lookalikes (same type,
   same level, same neighbours). One click accepts, one undo reverts. Rules are saved per client or
   per template and reused on the next project.
4. **Write back everywhere.** Values go into IFC properties and classification references, and into
   an Excel/CSV mapping, so the other tools (4D timeline, 5D cost, BOQ, pour cards, COBie) read them.
   Every change is in the revision timeline with who, when and which rule.

**What a modeler sees.**
- A "Tags" panel: a tagged-% bar, the list of rules, and a colour-by switch.
- Click an element: each tag with the rule that made it, or "manual" if a person set it.
- After a new revision: "212 elements changed; 9 lost their zone." One click re-tags and lists the
  9.

### Implementation

Inside the cad2bim IFC viewer and its project file; no server, nothing uploaded.
- **Rule engine** (packages/engine, new module): a small, typed rule language over IFC data
  (class, type, level, grid-bay from centre point, material, quantities, neighbours). Rules are data,
  versioned in the .c2b file. Pure functions, so they are testable with Vitest on small IFC fixtures.
- **Plain-language entry:** the existing AI chat (user's own key) turns a sentence into a rule and
  shows the rule text for approval. The model never edits elements itself; only the rule engine does.
  Without a key, the form builder still works.
- **Learning:** nearest-neighbour matching on type, level and position (no training, no model file),
  so it is explainable: "matched because same type, same level, 0.4 m from C12".
- **Coverage and colour-by:** reuse the viewer's colour overrides and QA panel; the coverage bar is a
  QA check like the existing ones.
- **Write-back:** IfcPropertySet and IfcClassificationReference through the existing IFC writer; CSV
  export for Excel; BCF-style issue for each element left untagged, if wanted.
- **Classification tables:** user imports their licensed Uniclass/OmniClass table (check the licence
  per table before shipping any in the app); cad2bim ships none by default.
- **Agent access:** expose "list untagged", "apply rule", "explain tag" as MCP tools in packages/mcp,
  so an AI agent can tag a model with the same rules a person can read.
- **Build order:** (1) rules for Level/Zone/Pour by grid bay, coverage bar; (2) cost code and WBS
  rules, CSV write-back; (3) learning from fixes; (4) classification import; (5) plain-language and MCP.
- **Risks:** rule mistakes spread fast, so every apply is previewed, counted and undoable; grid bays
  need a clean grid (fall back to levels only and say so); licensed classification tables.
- **Check before release:** 3 real models, stopwatch test above; compare tagged-% and wrong-tag rate by
  hand on a sample of 200 elements.

### Future update

- **Template rule packs** for common clients and codes (CPWD/DSR, NRM, a contractor's own WBS).
- **Schedule import:** read a Primavera/MS Project/Excel schedule and propose the matching rules
  from task names ("Raft L-1 Zone B") instead of asking for them.
- **Live watch:** tags re-check on every new revision and flag what lost its code (pairs with Change
  Radar and Ask & Watch).
- **Reports:** quantity and cost per zone, pour or WBS node in one click, and a "ready to link"
  check before a 4D/5D run.
- **Team rule library** shared in a project, with an owner and a change log.
