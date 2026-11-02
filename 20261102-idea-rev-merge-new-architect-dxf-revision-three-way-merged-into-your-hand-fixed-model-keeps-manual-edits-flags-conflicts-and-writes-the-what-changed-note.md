# 20261102 Idea: Rev Merge — the architect sends "Rev C" and your hand-fixed model is merged, not rebuilt: manual edits are kept, real conflicts are shown one by one, and the "what changed" note for the architect and the team writes itself

## Entry 1

```
date-time:   2026-11-02 (scheduled run)
idea name:   Rev Merge — "The architect mails Tower-B_RevC.dxf with no change list. The modeller drops it on
             the project that already holds Rev B plus two days of hand fixes. In about ten seconds a
             three-column screen appears: 41 items unchanged, 17 changed in the drawing only (will be
             applied), 6 edited by you only (will be kept), and 3 CONFLICTS — 'you moved column C14 by
             50 mm on site advice; the architect also moved it, by 150 mm'. Each conflict has three
             buttons: keep mine, take theirs, or a number box. Press Merge. The model updates only where
             the drawing changed, keeps every hand fix, keeps all GlobalIds, and draws the Rev B to Rev C
             overlay (red = old, green = new) with a numbered change list. One more click makes the
             'Rev C change note' for the architect (PDF + the overlay DXF with clouds) and a short
             'impact line' for the team: '+9 elements, -2, 6 moved, about +3.2 m3 concrete, +410 kg steel'."
             Runs first in the cad2bim web app (the pipeline's Revise, extended), then as a Revit
             add-in button (merge into the live Revit model through the bridge).
potential:   Very high for every team that models from architect DXF / DWG (which is most small and mid
             structural and BIM offices in India). Revisions are the daily pain: a typical project gets
             10-30 drawing revisions, the architect rarely lists what moved, and each one costs the
             modeller either (a) hours of eyeballing two sets of drawings, or (b) a full rebuild that
             throws away the manual fixes made since the last one. People keep "my edits" in their heads
             and lose them. This is also a hot LinkedIn topic (revision tracking, "single source of
             truth", AI plan comparison, version control for BIM), so the before / after overlay and the
             'three conflicts, one click each' screen make a good short video.
             Time today (estimated from how offices work, not timed): finding what changed between two
             DXFs by eye or by Bluebeam overlay, then fixing the model by hand: 2-6 hours per revision
             for a G+10 tower; re-checking that old hand fixes survived: another 1-2 hours; writing the
             reply to the architect: 30-60 minutes. With Rev Merge: about 10-20 minutes including
             decisions on conflicts. Estimate: 3-8 hours saved per revision, 30-150 hours over a
             project with 15-20 revisions.
             Market: Bluebeam Smart Overlay, AutoCAD 2026's markup and revision-cloud assist, Autodesk
             Automated Change Analysis (BIM 360 / ACC), Navisworks Compare and Solibri model comparison
             all compare two finished sets and show differences; they are paid desktop or cloud tools.
             None that we found does a three-way merge into a model that has been edited since (base
             drawing, edited model, new drawing), none keeps the person's manual fixes by rule, and
             none writes the reply note and the quantity impact from the same step. Cad2bim already has
             the hard part for DXF: stable IDs and a revision plan (see Implementation); the new work is
             the manual-edit memory, the conflict screen and the outputs.
```

### Description

**The pain.** The cad2bim pipeline turns a DXF into a 3D model. Then the modeller works on it: moves a column
the engineer asked for, fixes a beam size, adds a lintel the drawing did not show. Two days later the architect
mails a new revision. Today the choices are bad. Rebuild from the new DXF and lose the fixes, or edit by hand
and hope nothing was missed. Neither tells the architect, the engineer or the QS what really changed.

**The idea.** Treat the three things as a git merge:

1. **Base** — the drawing Rev B as it was read when the model was built (cad2bim already keeps each element
   with the reading it came from).
2. **Mine** — the model now, with every edit made since (editor moves, resizes, deletes, additions, Revit
   edits that came back through the bridge).
3. **Theirs** — the new drawing Rev C, read the same way.

For every element (matched by handle, then by key: layer, mark, level, position) the tool decides:

| Drawing changed? | Model edited by hand? | Result |
|---|---|---|
| no | no | leave it |
| yes | no | apply the new drawing (keep the GlobalId) |
| no | yes | keep the hand edit (shown as "kept") |
| yes | yes, same change | nothing to do |
| yes | yes, different change | **conflict**: person decides |
| gone in drawing | edited by hand | conflict ("architect removed it, you edited it") |
| new in drawing | — | add it |

**The screen (the wow).** A three-column review with counts, the overlay in 3D and in plan, and the conflicts
on top. Hover a conflict and both versions show as ghosts (yours blue, theirs green) at the same place. One
click each. "Apply all non-conflicting" is a single button; nothing is written until the person presses Merge,
and Undo restores the model as it was.

**Outputs from the same step.**
- **Change list** (numbered): "C14 moved 150 mm east", "B22 section 230x450 to 230x600", "Door D7 deleted".
  Excel + on screen, each row jumps to the element.
- **Overlay DXF / PDF** of Rev B (red) against Rev C (green) with revision clouds around each change and the
  same numbers as the list, for the architect: "please confirm these 26 items".
- **Impact line**: elements added / removed / moved, and the concrete m3 and steel kg difference, using the BOQ
  and BBS the engine already computes. For a PM this is the first thing asked.
- **Team note**: a short text for WhatsApp or email, made from the list. Nothing is sent by the tool.
- **Merge record** in the project file: which revision was merged, when, what was kept, what was decided on each
  conflict. Answers "why is this column here?" months later.

**Unique angle.** Change tools show two sets and stop. Rebuild tools forget the person's work. Rev Merge
remembers the person's work and merges. It also explains each change from the drawing's own element, which is
what makes the architect note useful. It runs in the browser; the drawings stay on the person's device.

### Implementation

Built from what cad2bim has. This note only describes the idea; nothing in cad2bim's source is changed by it.

| Piece | Uses | New work |
|---|---|---|
| Drawing reading and matching | pipeline reading; "Revisions" in `docs/design/pipeline-rust.md` (match by handle, else key; keep GlobalIds via `Element::id_key`) | keep the base reading with every element so "drawing changed" is a true Rev B vs Rev C test, not a guess from the model |
| Revision plan | `pipelineRevision` (the plan before Revise: changed, new, gone, same counts) | split "changed" into "changed in drawing only" and "changed in drawing and by hand" |
| Manual-edit memory | editor transaction history; `docs/design/project-versioning.md` | a small per-element "edited by hand" flag with the field changed (position, section, level, deleted, added), written into the project file |
| Conflict rules | none | a field-level rule table (position, section, level, existence); a hand edit to one field does not block drawing changes to another field |
| Review screen | viewer overrides for diff colours (`diff-added`, `diff-changed`, `diff-removed`) from the revision timeline design | three-column list, ghost pair for a conflict, one-click choices, undo |
| Outputs | DXF writer; BOQ / BBS engine; Excel writing used elsewhere | overlay DXF with clouds, change list, impact line, merge record |
| Revit | bridge protocol (`docs/bridge/protocol.md`) | Phase 3: the same merge into the live Revit model, edits made in Revit read back as "mine" |

**Phases.**
- *Phase 1 (about 1-2 weeks of work, my estimate):* edited-by-hand flags, the three-way plan, the review screen
  with conflicts, Merge and Undo, on DXF-built models.
- *Phase 2:* overlay DXF / PDF with clouds, the numbered change list, the impact line, the team note and the
  merge record.
- *Phase 3:* DWG input (already read by the CAD engine), Revit write-back, rule presets per office.

**Licences and rules.** Fits cad2bim's rules: nothing uploaded, no GPL code, colours from tokens, the DXF and
DWG reading stays with the CAD core (CADCraft crates, acadrust).

**Honest risks.**
- Matching a changed element to its old self can fail (a column moved a long way looks like one removed and one
  added). Show "possible move" pairs with a distance limit the person can change, and never merge a guess
  silently.
- A drawing change that is not geometry (a note, a dimension, a level name) is not seen by the element merge.
  Show text changes as a list from the drawing's text, marked as the weaker method.
- Hand edits made outside cad2bim (in Revit, before the bridge was used) are not known. The tool says so, and
  offers a one-time "compare my model with Rev B" to find them.
- The tool suggests; the person decides every conflict and signs the merge. The impact line is indicative
  until the model is checked.

### Future update

- **Architect profile:** learn each architect's habits (always shifts walls 25 mm, renames layers) and warn
  "this looks like a layer rename, not a new wall".
- **Auto-reply with questions:** list unclear changes ("two columns merged into one wall: intended?") as RFIs,
  linked to RFI Forge and Issue Desk.
- **Cascade check:** after a merge, run the flush-fit, clash and column-stacking checks only on what changed
  and say what the change broke downstream.
- **Rebar follow-up:** when a column moves or grows, re-run the cage and BBS for that element only and show the
  steel delta (links to BBS Weaver and Change Radar).
- **Time travel:** step back and forth through Rev A, B, C, D in 3D with the merge record alongside
  (links to the revision timeline).
- **Share the change card:** a one-image summary (overlay plus the top five changes) for WhatsApp or LinkedIn,
  with no drawing data leaving the machine unless asked.

Sources: [Bluebeam Revu Smart Overlay](https://www.bluebeam.com/solutions/revu),
[Nomic: AI plan comparison](https://www.nomic.ai/glossary/ai-plan-comparison),
[Nomic: AI to track drawing revisions](https://nomic.ai/glossary/ai-to-track-drawing-revisions),
[Autodesk: Automated Change Analysis](https://www.engineering.com/?p=43370),
[AutoCAD 2026: what's new](https://blog.cadalyst.com/cadmanagement/design-elevated-whats-new-in-autocad-2026);
in cad2bim: `docs/design/pipeline-rust.md` (Revisions), `docs/design/revision-timeline.md`.
