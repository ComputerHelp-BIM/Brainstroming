# 20261120 Idea: Repeat Offender — Your team's own closed BCF issues, RFIs and snags are mined for the mistakes you make again and again, turned into one-click checks, and run on every new model before it is issued, with a "same mistake, 4th project" board.

## Entry 4

```
date-time:   2026-11-20 (scheduled run, UTC)
idea name:   Repeat Offender (closed BCF / RFI / snag history to a team-own check library,
             run before every issue; on the device, nothing uploaded)
potential:   High for BIM leads and QA; medium-high for every modeller (fewer rework loops).
             Every team has the same 10-20 mistakes coming back: wrong beam depth at a lift pit,
             sleeve missing at a shear wall, level off by 25 mm, slab step missing at toilets,
             door swing against a column. Each one was found, fixed and closed in a BCF or an RFI
             log, and then forgotten. The next project starts with an empty checklist.
             Time saved: NOT measured.
             First test: take 3 closed projects' issue exports (kept out of git). Mine them with
             Repeat Offender. Then take the NEXT project's issues as the answer key. Count (a) how
             many of the next project's issues one of the learned checks would have caught,
             (b) how many learned checks fire on a clean model (false positives). Do not quote hours
             or hit rate until this is done.
             Market check (2026-11-20). One standard web search only. Found: Speckle and others run
             IDS / COBie rules on every version and export failures as BCF; Solibri is described as the
             reference for rule-based checking; BIMcollab pairs rule checks with BCF tracking; a TUM
             paper proposes case-based reasoning that stores BCF issues as reusable cases and finds
             recurring patterns. These are vendor pages and one academic paper; I did not verify
             their price, how far they go, or whether a product ships the TUM idea. What I did NOT
             find: a free, in-browser tool that reads a team's own closed issue history, proposes
             checks from it, and lets a person approve each one. "I did not find one", not "none
             exists". Practitioner commentary in the same search was sceptical of black-box AI
             checking, so every learned rule here is shown in plain words and needs a person's OK.
             Trend: AI model checking and "learn from past projects" are live topics in the sources
             above. I could not measure LinkedIn or Instagram counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - IDS QA + autofix + BCF (Oct 14): runs rules you already have. Repeat Offender
                 writes the rules, from your own history.
               - Spec Sage: rules come from the spec. Here they come from what actually went wrong.
               - Clash Sorter / RFI Forge / Snag Sweep: each groups one project's issues. This
                 groups across projects and turns the groups into checks.
               - Deja Model and Dojo Drill: reuse and training. This is memory of mistakes.
               - Second Eyes: gate before issue. Repeat Offender feeds that gate its checks.
             Library: Warning Doctor scores model health on generic warnings; this is your own team.
             The wow: drop the last 3 projects' BCF files. A "Top offenders" board appears: "Sleeve
             missing at wall-slab junction: 4 projects, 31 issues, avg 2.1 days to close." Press
             "Run on this model": 6 spots light up in 3D, each with the old issue it matches ("same
             as Tower B, issue #212, fixed by moving the sleeve 150 mm"). One click makes the BCF.
```

### Description

**The pain.**
- Lessons live in closed issue lists nobody reopens; a new modeller cannot learn them.
- Checks (Solibri / IDS) catch what someone thought of in advance, not what *this team* gets wrong.
- The BIM lead's own memory is the real checklist, and it leaves with them.
- The same RFI is raised on project after project because the drawing habit never changed.

**The idea.** Four moves.
1. **Read the history.** Drop BCF files (and, optionally, an Excel RFI or snag log). Each issue has a
   title, comment text, status, element GUIDs and viewpoint. Closed issues with a fix comment are the
   gold.
2. **Group them into "offenders".** Same wording, same IFC class / property / location pattern, same
   fix. A group is shown as a plain sentence with its count, projects, days-to-close and examples.
   Nothing is a rule until a person picks it.
3. **Turn an offender into a check.** For each approved offender the tool drafts a rule from a small
   fixed set of check types: *missing relation* (wall-slab has no opening), *value out of range* (beam
   depth under span/depth), *level mismatch* (top of slab differs from the level by more than 5 mm),
   *missing property*, *geometry near geometry* (door swing hits column). The person edits the numbers
   and names it. Rules are saved in a team library as plain data (and exportable as IDS where IDS can
   express them).
4. **Run before issue.** "Run on this model" checks the open model, lights failures in 3D with the
   matching past issue beside each, and writes BCF for the ones the person keeps. The offenders board
   shows which rules fire most and which never fire (candidates to retire).

**What a user sees.**
- An "Offenders" panel: ranked list, each row a sentence, counts and a "make check" button.
- A "Run" result: pass/fail per rule, 3D highlights, each failure with "seen before: project, issue
  id, how it was fixed".
- A rule history: when a rule was made, by whom, how many true hits and how many dismissed ones
  (so false positives are visible, not hidden).

### Implementation

Inside the cad2bim app; the library lives in the project file or a team file the person saves.
Nothing leaves the device unasked; no model or issue text goes to an outside service.
- **Reading issues:** a BCF reader (zip of XML, versions 2.1 and 3.0) in packages/engine; GUIDs link
  to elements when the same IFC is open. RFI/snag logs by column mapping, reusing the Sheet Sync reader.
- **Grouping:** deterministic first: normalised title text, IFC class of the linked elements, and
  location (level, zone). Plain text-similarity on the device (no network). The optional AI chat
  (user's own key, already on the roadmap) may *suggest* names for groups; it never creates a rule
  and never sees an issue unless the person sends it.
- **Rule types:** a closed set of five checker types (above), each a small pure function over the IFC
  data model, tested with Vitest on hand-made cases. A rule is data (type, parameters, text), not code.
- **IDS export:** where a rule fits IDS (missing property, value range) write the .ids file; where it
  does not (geometry near geometry) keep it in the library and say so.
- **3D and BCF out:** failure highlighting via the existing viewer colour code; BCF writing reuses the
  existing IDS QA BCF writer.
- **Agent access:** MCP tools in packages/mcp: "top offenders", "run library on this model",
  "why does this rule exist" (returns the source issues). An agent gets the same evidence a person does.
- **Build order:** (1) BCF reader and offender board (read only, shows value on day 1); (2) the three
  simplest rule types and "Run"; (3) 3D highlight and BCF out; (4) IDS export; (5) geometry rule
  types; (6) MCP.
- **Risks:** thin or badly worded issue history gives weak groups (the board says how many issues it
  could not group); a learned rule can be too broad and cry wolf (so every rule shows hits vs
  dismissed and can be retired in one click); issue text can hold client names, so the library keeps
  only the sentence the person approved, not the raw comments, unless asked.
- **Check before release:** the 3-project key test above, plus a BIM lead reading the first 20
  proposed offenders and marking each "real / not real".

### Future update

- **New-joiner mode:** a modeller opens "Offenders" in week one and gets the team's ten most costly
  mistakes with 3D examples (feeds Dojo Drill).
- **Live while you model:** run the cheap rules on save (see Save to See) and show a small badge
  instead of waiting for the issue gate.
- **Cost of a mistake:** link each offender to days-to-close and rework hours from Model Pulse, so the
  board ranks by cost, not count.
- **Share a rule pack:** export a vetted pack ("RC frame, India, lift and stair") for another team to
  import, with the source counts but no issue text.
- **Drawing-habit fixes:** when the cause is in the incoming DXF (Layer Whisperer, Trace Lens), offer to
  add the pre-flight warning there so the mistake is stopped at the start of the pipeline.
- **Retire and prove:** after six months show which rules stopped firing, as a simple "mistakes
  that went away" chart for the team and the client.

Sources used for the market check (mostly vendor material, treat as claims):
[TUM, case-based reasoning on BCF issues](https://mediatum.ub.tum.de/doc/1856017/1856017.pdf),
[Speckle, BIM model checker](https://speckle.systems/bim-model-checker/),
[Nomic, AI for BIM model checking](https://www.nomic.ai/compare/best-ai-for-bim-model-checking),
[AEC Magazine, rethinking the BIM platform](https://aecmag.com/ai/rethinking-the-bim-platform/).
