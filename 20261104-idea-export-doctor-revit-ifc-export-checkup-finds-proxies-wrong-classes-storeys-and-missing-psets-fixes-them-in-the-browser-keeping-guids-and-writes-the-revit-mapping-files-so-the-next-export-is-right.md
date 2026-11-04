# 20261104 Idea: Export Doctor — drop in the IFC your Revit just exported, see what the export broke, fix it in the browser with GUIDs kept, and get the Revit mapping files that make the next export right at source

## Entry 1

```
date-time:   2026-11-04 (scheduled run)
idea name:   Export Doctor (Revit IFC Export Checkup + Browser Fix-and-Write-Back + Auto-Written Revit
             Class-Mapping and Pset Files)
potential:   High. Every BIM team sends IFC to a consultant, a client, a checker or a contractor every
             week, and the first export is almost never right. The loop today is: export (5 to 20 min),
             open in a viewer, find a problem, go back to Revit, change a setting or a parameter, export
             again. Three or four rounds per issue is normal. Export Doctor cuts the loop to one round:
             it finds the problems in seconds, fixes most of them in the exported file, and writes the
             settings that stop them coming back. Estimated saving (an estimate, not measured): 1 to 3
             hours per IFC issue round for a modeller; more for the BIM manager who answers "why is my
             beam a proxy?" every week. It also protects the team's name: a clean IFC is what the other
             side sees first.
```

### Description

**The pain (what a modeller actually sees).**
- Elements arrive as `IfcBuildingElementProxy` because the Revit category or family has no IFC class
  mapped, or the family was made from a generic template. They then cannot be filtered, costed or checked.
- The right class but the wrong one: a beam exported as `IfcMember`, a slab as `IfcRoof`, a column
  as `IfcBeam` (family hosted or rotated). Structural checks and quantity take-offs go quietly wrong.
- Elements on the wrong storey, or on no storey, because of level offsets, or elements that span storeys.
- Property sets missing or half-filled: shared parameters not mapped, `Pset_*Common` without
  `LoadBearing` / `IsExternal` / `FireRating`, the client's required properties absent.
- New GUIDs after a re-export, so the other side's issues and links break. Duplicate GUIDs after a copy.
- Materials and colours lost, 10 000 tiny parts for one wall, units or north angle wrong, a 600 MB file for
  a small building.
- The fix is always "go back to Revit". A BIM manager keeps the knowledge in their head: which setting,
  which mapping file, which family parameter.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: IDS QA Autofix checks a model against requirements and suggests fixes;
  Warning Doctor scores the Revit model's health inside Revit; Layer Whisperer cleans DXF layers; BEP Bolt
  writes the rules. None of them looks at what the *IFC export* broke, fixes the exported file, and
  feeds the correction back into the export settings.
- In the market (not exhaustively checked): free viewers show the problem but do not fix it. Rule-checkers
  (Solibri and similar) report it, mostly paid, and fix nothing in the file. Scripting with IfcOpenShell can
  do every fix, but needs a developer and its licence is LGPL, so it cannot go into this product as it is.
  Revit's own IFC exporter has a class-mapping file and a user-defined Pset file, but few teams know the
  format, and the file is hand-written. I could not confirm that a one-step "check, fix, and write the
  Revit mapping" tool exists; treat that as unverified.
- In cad2bim (from `CLAUDE.md` and `docs/`): the IFC reader (web-ifc), the IFC writer that keeps ids and
  writes parameters (`docs/design/ifc-writer-parameters.md`), QA findings with select / isolate / zoom
  (`docs/design/actionable-qa.md`), the Revit add-in (bridge) and the Excel writer. The pieces for
  fixing and writing are already there.

**The idea.** An **Export Doctor** tab in the cad2bim web viewer. The person drops the IFC; nothing
leaves the device. In a few seconds they get a **checkup card** and a list of findings, each one clickable
(select, isolate, zoom) as in the existing QA panel.

1. **Checkup card.** One screen: how many elements, how many proxies, how many with no storey, how many
   with missing required properties, duplicate GUIDs, file size and what is in it (geometry, properties,
   spatial tree). A score out of 100 the BIM manager can quote in a meeting. Not a pass or fail on its own;
   every point loss says why.
2. **Findings with a proposed fix.** Each finding carries its evidence and one suggested action:
   - "212 proxies. 140 have the name `Beam-...`, a beam profile and a horizontal axis: propose
     `IfcBeam`." The proposal comes from the element's name, Revit category and type (kept in the IFC
     as properties), and geometry (axis direction, section, thickness). The person sees the reason and
     the confidence (high / medium / low) and can accept all high-confidence ones at once.
   - "38 elements on no storey; their base level is L2 within 20 mm: assign to L2."
   - "Pset_BeamCommon has no LoadBearing on 96 beams; the BEP says it is required: set from the type."
   - "14 GUIDs repeat: give the later ones new GUIDs, record them in a changes list."
3. **Fix in the browser.** Accepted fixes are written into a new IFC with the existing writer. Every GUID
   that was not asked to change stays the same, so the other side's issues still point to the right object.
   The original is never overwritten. A changes list (CSV and the model's own log) says what changed on
   which element.
4. **Write the fix back to the source (the wow).** The same findings are turned into the two files Revit's
   IFC exporter reads: an updated **class-mapping file** (Revit category / type to IFC class) and a
   **user-defined property-set file** (which Revit parameters become which Pset properties). The
   person puts them in the exporter's folder (or the add-in does it), exports again, and the same
   problems do not come back. A second drop of the new IFC shows the score going up and the changes list
   shrinking to nothing. This closes the loop that today lives in one BIM manager's head.
5. **Team memory.** The accepted rules are stored as a small, readable "export profile" for the project or
   the company ("on this job, `Beam-*` types are `IfcBeam`; footing families are `IfcFooting`"). The next
   project starts from it. It is a plain text file the team can review in Git.

**The wow 1: the proxy count falling.** On screen, 212 grey proxy elements turn into coloured beams,
columns and slabs in front of the room as one click accepts the high-confidence fixes. The counter goes
212, 40, 12. On a call with the client, the team's IFC goes from "needs work" to "clean" in the time it
takes to say it.

**The wow 2: it teaches Revit.** "Download your Revit mapping file" is the line people will repeat. The tool
does not only repair the symptom; it hands the BIM manager the exact file that stops the cause.

**LinkedIn / Instagram angle.** A 30-second before and after: a Revit IFC full of grey proxies, then the
same model coloured by class after one click, with the score 41 to 94. The hook, "your IFC export is
lying to you", suits openBIM posts (buildingSMART, IFC4.3 and IDS talk is common on LinkedIn). I cannot
measure what is trending today from here; check the angle against what the team sees in its feed before
posting.

**Who benefits, day to day.**
- The modeller: stops the "re-export and hope" loop; sends a clean IFC the first time.
- The BIM manager: one place to see the state of every export, and the settings written for them.
- The structural engineer / checker: gets beams as beams and columns as columns.
- The QS: quantities by class are now trustworthy (Measure Right, Rate Rocket, Ratio Radar all depend
  on the right class).

### Implementation

- **Reading.** web-ifc through `@cad2bim/engine`: spatial tree, class, name, type, properties,
  quantities, relationships. Revit's category and family type are normally kept as properties in the IFC;
  the first work is a map of what is actually there in exports from Revit 2023 to 2025, using the sample
  models the team can keep outside git.
- **Class proposals.** A small rule table first (category, name patterns, section shape, axis direction,
  thickness-to-span ratio), each rule with a reason string, then a confidence. No machine learning in the
  first version: rules are explainable, testable and fast. Rules ship as data, not code.
- **Fixes.** Re-class an element (`IfcBuildingElementProxy` to `IfcBeam` and so on, keeping geometry,
  placement, GUID and relationships), set storey containment, add or fill property sets and quantity sets,
  reassign duplicate GUIDs. Done in the existing IFC writer; an IFC schema check (the writer's own
  validity checks plus a round trip through web-ifc) is run on the output before it is offered.
  Re-classing between entity types has schema rules (predefined types, which relationships are allowed):
  the first version supports a short, tested list of transitions and declines the rest with a reason.
- **Revit files.** Generate the class-mapping and user-defined Pset files in Revit's published formats.
  The format must be read from the current Revit documentation and tested against a real export before it
  is shipped; I have not verified the exact syntax or the folder rules for each Revit version, so the
  first task is a spike: write the files by hand, export, and confirm.
- **Revit add-in (`bridge/revit`).** A "Install export profile" command copies the files to the right folder
  and sets the exporter's options, with an undo. Optional: an "Export and check" button that runs the
  export and opens the result in cad2bim. Add-in work is C# on .NET 8 as today.
- **Checkup score.** A transparent formula (weights shown on screen), versioned. Not a standard; the card
  says so.
- **Privacy and licences.** Everything runs on the device; no file is uploaded (repo rule). No GPL or LGPL
  code in the app, so IfcOpenShell is not used; only the repo's own reader and writer. Any new library must
  be MIT, Apache-2.0, BSD or ISC.
- **Tests.** Each rule gets a fixture IFC (kept small and synthetic, not a client's drawing) and an
  expected result. A round-trip test: fix, write, read, check that GUIDs and geometry are unchanged except
  where intended. Measure and record: how many proxies the rules classify correctly on two real exports
  (kept outside git), how many wrongly, how many left alone.
- **Risks.** (1) A wrong re-class makes a quiet error: default to showing the proposal, never apply
  low-confidence ones without the person's click, and keep the changes list. (2) Some proxies are
  correct (temporary works, special items): the rules must leave unknown things alone. (3) Re-saving an IFC
  can lose data the writer does not know about: warn when the source has entities the writer cannot
  carry, and offer "report only" mode. (4) Revit mapping file format differs by version: ship per-version
  templates and say which versions were tested.
- **First 2 weeks.** Checkup card (counts, proxies, no-storey, missing Pset, duplicate GUIDs, size) on one
  real Revit export; six class rules for structural elements (beam, column, slab, wall, footing,
  stair); the fix-and-write path for re-class and storey assignment; the changes CSV; and the Revit
  class-mapping file as a hand-verified spike for one Revit version. Record how many findings a BIM manager
  agreed with in the next entry.

### Future update

- Pset fill from the BEP / IDS file (BEP Bolt, IDS QA Autofix): "the client needs these 14 properties" turns
  into the user-defined Pset file.
- Compare two exports (Change Radar): what the last export changed, element by element.
- Batch mode and an agent-helper command (`packages/mcp`): "check these five IFCs and tell me the worst
  one", for a Monday-morning model review.
- Link to Trace Lens: for a model built by the DXF pipeline, show which element came from which drawing line
  next to the IFC finding.
- Federated check: drop the structure, architecture and MEP IFCs together and find class and storey
  names that do not agree across disciplines (pairs well with Grid Lock).
- Per-consultant profiles: remember what each outside party's IFC usually gets wrong and check for that
  first.
- A "send to the client" gate: a one-page export certificate (score, rules, date) that is attached to a
  transmittal (Issue Desk).
