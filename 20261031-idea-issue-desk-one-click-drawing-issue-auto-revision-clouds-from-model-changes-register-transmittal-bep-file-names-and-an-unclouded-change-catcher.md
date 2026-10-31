# 20261031 Idea: Issue Desk — press "Issue" and the tool finds which sheets really changed, clouds the changes, bumps the revision, exports PDF + DWG with BEP names, fills the drawing register and writes the transmittal; and it catches the change you forgot to cloud

## Entry 2

```
date-time:   2026-10-31 (scheduled run, UTC)
idea name:   Issue Desk (One-Click Drawing Issue: Auto Revision Clouds, Register and Transmittal)
potential:   High. Every BIM team issues drawings every week, and the last mile is still manual: pick the
             sheets, guess which ones changed, draw revision clouds by hand, type the revision letter on
             each title block, update the drawing register in Excel, export PDF and DWG with the right
             file names, write the transmittal, mail it. On a 60-sheet set this is 2-4 hours per issue
             (my estimate, not measured) and it is where "wrong revision went to site" mistakes begin.
             Nothing in the earlier ideas here does this step: Change Radar finds what changed in the
             model and the BOQ, Drawing Autopilot makes the sheets, BEP Bolt sets the naming rules, but
             nobody turns those into the actual issue. This idea is that missing last mile.
             Estimated saving (an estimate): 2-3 hours per issue, and far fewer wrong-revision incidents.
```

### Description

**The pain (daily life of a modeller).**
Friday, 5 pm: the consultant wants "IFC set, rev C". The modeller opens 60 sheets one by one and tries to
remember which ones were touched since rev B. Clouds are drawn by hand, so some are missed and some are left
over from the last issue. The revision table on each title block is typed by hand. The drawing register is a
separate Excel that is always one issue behind. PDFs and DWGs are exported with names typed by hand, so
`S-101_GroundFloorBeams_final2.pdf` goes out. The transmittal is a Word file. Any step done wrong sends the
wrong drawing to site, and the contractor builds from it.

**Why now (trending).** AI plan comparison and "revision tracking" tools are all over LinkedIn in 2026, and
vendors say they compare two PDF sets and generate revision clouds
([Nomic](https://www.nomic.ai/glossary/ai-plan-comparison),
[Compare Versions](https://www.aecplustech.com/tools/compare-versions)). Those are vendor pages, so read them
as positioning, not as proof of accuracy. They compare *pictures of drawings after the fact*. They do not know
the model, so they cannot say *why* a line moved or which element it was. Issue Desk starts one step earlier,
from the model diff.

**What exists today (checked, thinly, with two searches).**
- Batch export is solved by several Revit add-ins, for example ProSheets (a free plus paid plan, per its
  vendor; I could not confirm which features are paid)
  ([DiRoots](https://diroots.com/plugins/prosheets-revit-addin/)).
- Transmittal and register: an open-source Revit add-in, *Transmittal* by russgreen, sets revisions and status
  and keeps a transmittal history in SQLite
  ([GitHub](https://github.com/russgreen/Transmittal)).
- After-the-fact PDF comparison with AI clouds: paid web tools (see above).
- I did **not** find a tool that starts from the model diff, decides which sheets are affected, draws the
  clouds from it, and also does naming, register and transmittal in one flow. This is a search result, not
  proof; check again before putting "nothing like it exists" in marketing.

**The idea.** In cad2bim, press **Issue**. The wizard asks four things: *issue purpose* (For Information /
For Coordination / For Approval / For Construction), *new revision* (suggested: next letter), *to whom* (a
list kept per project), *formats* (PDF, DWG, IFC, BCF). Then it:

1. **Finds the changed elements** by comparing the model now with the model at the last issue (the planned
   versions-and-history feature, or the revision-diff engine from Change Radar). Added, moved, resized,
   re-typed, deleted.
2. **Finds the affected sheets**: a sheet is affected if a changed element is visible in any view on it
   (plan, section, elevation, schedule row). Unaffected sheets are listed as "unchanged, carry forward at
   the same revision", which is what the contractor wants to see.
3. **Draws revision clouds** around the changed elements on each affected view, tagged with the revision
   letter. Old clouds from the previous issue are retired, not left behind.
4. **Updates every title block**: revision table, date, drawn / checked / approved initials from the project
   roster, suitability code.
5. **Runs the pre-flight** (below), and stops on red items.
6. **Exports** PDF and DWG with names from the BEP rules (project-originator-zone-level-type-role-number,
   revision, status), into a folder per issue.
7. **Writes the drawing register** (Excel and a sheet in the model): sheet number, title, scale, revision,
   date, status, "changed in this issue: yes / no / why".
8. **Writes the transmittal**: a one-page PDF with the list, the purpose, the recipients and the
   "changes in this issue" summary, plus a ZIP, plus an email draft the person opens and sends. The tool
   never sends anything itself.

**The wow: the Unclouded-Change Catcher.** Before the issue, Issue Desk compares what *you* clouded by hand
(or last time) with what *actually* changed in the model. It shows a short list:
- "Beam B14 on Level 3 grew from 230x450 to 230x600 and appears on S-103 and S-212: **not clouded**."
- "A cloud on S-105 around grid C-D: **nothing changed there** (left over from rev B)."
The first kind is the one that costs money on site. A 60-sheet set is checked in seconds, with every change
tied to an element, and a click jumps to it in 3D.

**The second wow: "Issue Reel".** One click plays the issue as a 20-second before/after of the changed
elements only (red out, green in), the same engine as Model Reel. The project manager sees "what changed in
rev C" without opening a single sheet.

**Pre-flight checks (red / amber / green).**
- Sheet shows a view whose model is newer than the sheet (stale view).
- Title block field empty or in the wrong format (date, revision, drawn-by).
- Sheet number or file name breaks the BEP rule (uses BEP Bolt's rules, if present).
- Revision letter skipped or reused.
- Unplaced view, empty viewport, a schedule with `<<varies>>` or empty cells.
- Warnings count on the model rose since the last issue (uses Warning Doctor's score, if present).
- Changed element has no sheet showing it at all ("changed but not drawn"; amber).

**Unique angle.** It joins things that live in different tools today: the model diff, the sheets, the
revision clouds, the BEP file naming, the register and the transmittal. Competing add-ins do the export and the
transmittal; competing AI tools do the PDF comparison; none that I found *starts from the model element* and
so can explain every cloud ("because B14 changed section"). It runs in the browser, on the person's device.
Files stay on their disk, as in cad2bim.

### Implementation

Built from what cad2bim has or has planned. This note only describes the idea; nothing in cad2bim's source
is changed by it.

| Piece | Uses | New work |
|---|---|---|
| Model diff between two states | planned *Versions and history*; the revision-diff engine in the Change Radar idea | map "changed element" to "visible in which views" |
| Sheets and views | planned *Sheets and PDF*; Drawing Autopilot idea; Revit sheets via the add-in bridge (`docs/bridge/protocol.md`) | a view-to-element visibility index (bounding box against view crop and cut plane) |
| Revision clouds | line work in the DXF writer; Revit `RevisionCloud` through the bridge | cloud outline from element boxes, merged when close, kept off the title block |
| Title block fields | Sheets and PDF design; Revit sheet parameters | field mapping per project, kept in the project file |
| Naming | BEP Bolt idea (naming rules) | parser and generator for the rule string; a manual rule box if BEP Bolt is not there |
| PDF / DWG export | Sheets and PDF (planned); the DXF writer; Revit export through the bridge | batch queue with a progress bar, per-issue folder |
| Register and transmittal | Schedules (roadmap); Excel writing already used elsewhere in the idea notes | register template, transmittal PDF template, ZIP in the browser |
| Pre-flight | Warning Doctor and IDS QA ideas | a checklist runner that reads their results when present |
| Issue Reel | Model Reel idea; planned *Camera moves and video* | filter the reel to changed elements only |
| Project memory | project file (`.c2b`) | an `issues[]` list: id, date, purpose, revision, recipients, hash of each sheet |

**Phases.**
- *Phase 1 (about 1-2 weeks of work, my estimate):* the issue wizard, revision letters, the register as Excel,
  BEP-style file names, and a transmittal PDF, on sheets from cad2bim's own Sheets feature. No clouds yet.
  Already removes the typing.
- *Phase 2:* model diff to affected sheets; auto clouds; the Unclouded-Change Catcher; pre-flight list.
- *Phase 3:* Revit round trip (write clouds and revisions into the Revit sheets through the add-in); DWG per
  sheet; BCF with the issue; Issue Reel; per-recipient filtered sets.

**Honest risks.**
- A cloud is only as good as the *visibility index*. An element hidden by a view filter or a worksharing
  state may be reported as "visible". Show the reason for every cloud and let the person untick it. The tool
  suggests; the person issues.
- Changes that are not geometry (a text note, a dimension, a schedule value) need their own detectors. Phase 2
  covers geometry and parameters; annotation changes can fall back to an optional picture compare of the two
  PDFs, clearly labelled as the weaker method.
- The tool never replaces the checker's sign-off. "Checked by" comes from a person pressing a button, never
  from the tool.
- Issuing is a contractual act. The tool keeps an immutable record (date, hash of each file) but is not a
  legal document-control system; say so in the transmittal footer.
- File naming standards differ per client; the rule must be editable, not hard-coded.

### Future update

- **Recipient views:** the structural consultant gets only S- sheets, the MEP team gets the background
  DWG with the slab edges, the client gets a PDF with a change summary.
- **Acknowledgement tracker:** each recipient has "received / reviewed / commented"; a late reply is shown
  on the register. A link to Markup Mirror closes the loop when comments come back.
- **Superseded stamp:** when a new issue goes out, old PDFs in the shared folder are renamed or watermarked
  "SUPERSEDED rev B" so site never builds from them.
- **QR on the title block:** scanning shows the latest revision of that sheet (pairs with Site QR Pass),
  so the site engineer sees at once if the printed drawing is stale.
- **Change cost note:** each issue lists the BOQ delta of what changed (pairs with Change Radar and Rate
  Rocket), so "rev C added 1.8 lakh of steel" travels with the drawings.
- **Issue digest for WhatsApp / LinkedIn:** a one-image summary of the issue (sheets changed, top five
  changes) and the Issue Reel as a vertical video, with no model geometry leaving the machine unless asked.
- **Learning the firm's habits:** remembers the cloud style, the revision text wording and the usual
  recipient lists per client, so the second issue takes two clicks.

Sources: [DiRoots ProSheets](https://diroots.com/plugins/prosheets-revit-addin/),
[russgreen/Transmittal](https://github.com/russgreen/Transmittal),
[Nomic: AI plan comparison](https://www.nomic.ai/glossary/ai-plan-comparison),
[Compare Versions](https://www.aecplustech.com/tools/compare-versions).
