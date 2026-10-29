# 20261029 Idea: RFI Forge — audit the consultant's DXF/DWG set before you model it, and get the RFI list in 5 minutes

## Entry 1

```
date-time:   2026-10-29 (scheduled run, UTC)
idea name:   RFI Forge (Pre-Model Drawing Audit). "Find the drawing's mistakes on day 1, not in week 3."
potential:   High. Every modeller knows the loop: you start modelling from the consultant's DWG set, and
             somewhere in week 2 you find that column C7 is 300x600 on the plan and 300x750 in the column
             schedule, that grid C is 4,200 from B on one sheet and 4,500 on another, that a beam mark
             exists on the plan but not in the beam schedule, or that the typical-floor note says "as per
             L3" and L3 is not in the set. Each one stops the work, becomes an email, then an RFI, then a
             wait. Typically 20-60 such conflicts per mid-rise set; each costs 1-3 days of waiting or a
             silent guess that surfaces on site as rework. RFI Forge finds them all before modelling starts,
             writes each as a ready-to-send RFI pinned on the drawing, and then models everything that WAS
             consistent, leaving the conflicting items flagged, not guessed. No free tool does this. Paid AI
             drawing-review products exist, mostly cloud-upload, mostly priced per sheet or per seat; none
             that I know of ends in a 3D start model with the open questions pinned in it.
```

### Description

**The pain (daily life of a BIM modeller / structural detailer).**
Garbage in, garbage out. The 2D set from the structural consultant is the "source of truth", and it is wrong
in small ways that only show when two sheets are compared. Today the check is done by eye while modelling, so
errors surface one at a time, at the worst moment, and the RFI is written from memory. The modeller either
waits (schedule slips) or guesses (rework, disputes, "who authorised this size?").

**Why now (trending).** LinkedIn and Instagram AEC feeds in 2025-26 are full of three things this idea sits on:
(1) *AI drawing review / "AI reads your drawings"*, (2) *RFI reduction and "rework is the biggest hidden cost"*,
(3) *openBIM and data-ready models*. Most of those demos are cloud, upload your drawings, and stop at a
report. cad2bim already reads DXF/DWG on the device and builds the 3D, so it can do the check and
the build in one run, and the drawings never leave the PC.

**The idea.** Open the incoming set in the CAD workspace and press **RFI Forge**. It runs a set of
cross-checks on the drawings, not on a model, and returns an **RFI list** where every row is pinned on the
drawing (click the row, the viewer zooms to the spot and highlights both conflicting places).

1. **Plan vs schedule.** Every mark on a plan (C1..Cn, B1..Bn, S1..Sn, footing marks) is looked up in its
   schedule. Reports: mark on plan, not in schedule; mark in schedule, never used on a plan; size on plan
   text differs from schedule size; same mark, two different sizes on two floors with no "typical" note.
2. **Grid and dimension sums.** Grid-to-grid dimensions on each sheet are compared with the grid geometry and
   with other sheets; a dimension string must add up to its overall dimension (4,200 + 4,500 + 3,600 =
   12,300, not 12,600). Grid labels missing, duplicated or out of order.
3. **Levels and heights.** Level names/elevations across sheets (FFL, SFL, slab top, plinth) compared; floor
   heights that do not add up to the section's overall height; a level used in a note but absent in the set.
4. **Geometry sanity on the 2D itself.** Column outside its grid intersection by more than a tolerance (default
   75 mm), beam that starts or ends in air (no column, wall or beam under it), slab bay with no boundary beam
   or wall, open polylines where a closed outline is expected, overlapping columns, beams on two layers for one
   line. (Most of these are what blocks a clean 2D to 3D run today.)
5. **References that go nowhere.** "Refer detail D5", "as per Sheet S-12", "typical floor L3-L9", "see note 7":
   each reference must resolve to a detail, sheet or note in the set; a revision cloud with no revision-table
   row; sheet title-block revision different from the index.
6. **Standard-code smell tests (IS 456 / IS 13920 style), warnings only.** Beam width below the minimum,
   a column minimum dimension or bar count below code for its storey, stirrup spacing in the confinement zone
   larger than the ductile detailing limit, cover missing, grade missing. Not a design check: a
   "ask the engineer" list.

**What you get.**
- **Traffic-light summary:** "Set is 86 % consistent: 31 conflicts (9 blocking, 14 should-fix, 8 note)".
  *Blocking* means a 3D element cannot be built without a decision.
- **RFI cards, ready to send:** "RFI-014. Sheet S-03 plan: C7 = 300x600. Sheet S-11 column schedule: C7 =
  300x750. Which governs? Impact if 750: +0.27 m3 concrete x 9 floors = 2.4 m3, +186 kg steel." Each card has
  the two crops of the drawing as images, the proposed assumption, who is asked (picked from the title block's
  consultant) and a due date.
- **One-click outputs:** an Excel RFI register (log number, sheet, item, status, due date); a PDF with the
  crops (to attach to an email); **BCF** topics so the same RFIs reach Revit through the Bridge or any BCF
  tool; and a short WhatsApp-style summary text for the project manager.
- **Clean start model, not a blocked one.** The existing pipeline models everything that is consistent;
  each conflicting element is built with the *most likely* value (the one the majority of references agree
  on, or the schedule's), tagged `RFI-014 (assumed)`, and drawn in an amber "assumption" colour in 3D. When
  the consultant answers, press **Resolve** and the element, its schedule rows and its quantities update.
  The model is never silently wrong: an assumption can always be listed (`Assumed elements: 23`).
- **Assumption ledger:** a running list of what the modeller assumed, who answered, when. This is the audit
  trail that ends the "who authorised this size?" argument.

**The wow moment (one screen, 30 seconds, perfect for a LinkedIn/Instagram reel).** Drop the consultant's
DWG on cad2bim. The drawing fills the screen. A scanning line sweeps the sheets and pins pop up like a
radar: red pins on conflicts, with a counter ticking up: 31 conflicts, 9 blocking. Click a pin, the plan and
the schedule slide side by side with "300x600 vs 300x750" circled. Press *Forge RFIs*: 31 cards appear and
the 3D model builds beside them, with 23 amber "assumed" columns glowing. Total time on screen: under a
minute for a job that is two to three days by hand.

**Why this is unique / extends what is already here.**
- cad2bim already has **Audit and Overkill** (file health: missing layers, blocks, styles). RFI Forge is the
  *content* audit: whether the drawings agree with each other. These two stack: file health first, then
  content.
- Other ideas in this repo check *model against design* (Twin Check), *model against IDS* (IDS Gatekeeper),
  or *revision against revision* (Change Radar). None checks the *source drawings against themselves before
  any model exists*, the one point where an error is cheapest to fix.
- It reuses the readers that already exist (CAD core, CH v1/v2 template reader in Rust) and the hosted
  pipeline, so most of the "understand this drawing" work is done; the new part is the cross-check rules
  and the RFI output.

**Day-to-day use cases.**
- Start of every project: audit the tender/IFC (issued for construction) set.
- Every revision: run it on the new set; only *new* conflicts are shown (compare with the previous
  ledger), so Rev D does not repeat Rev C's open RFIs.
- Contractor's design-coordination team: send the register to the consultant on day 2.
- Consultant's own QA: run before issuing, so the set goes out clean (the same tool, other side of the table).
- Freelance modellers: attach the RFI register to the quote ("31 issues found in your set before I started"),
  a strong commercial argument.

### Implementation

Fits the architecture: the browser, a worker, no server, files stay on the device. The cad2bim repo was read
for reference only; nothing was changed there.

1. **Facts layer (reuse):** take the drawing's entities from the CAD core (`packages/cad-engine`, Rust
   to WebAssembly) and the CH reader in `packages/pipeline/core` (v1; v2 for the CH template). These already
   find columns, beams, slab bays, marks, schedules (tables are drawn from `*T` blocks), grids and levels.
   Add a *facts* output: a flat list of `{kind, mark, size, level, sheet, position, source entity ids}`.
2. **Rule engine (new, Rust, `packages/pipeline/core/src/audit/`):** each rule is a small function over the
   facts; rules are data-driven (a JSON pack: id, severity, tolerance, message template) so a team can add its
   own ("our company never uses 230 wide beams"). Same shape as the IDS Gatekeeper rule runner in this
   repo, so the two can share one results panel.
   - Rule groups: `marks`, `dims` (sum-of-string, grid vs sheet), `levels`, `geometry`, `refs`, `code-smell`.
   - Each result: `{ruleId, severity, message, evidence: [{sheet, bbox, entityIds}], proposedValue, confidence}`.
3. **Mark and text matching:** normalise marks (`C-7`, `C7`, `C07` equal); read dimensions with units and
   the drawing's unit setting; tolerate the usual typing noise. Matching is deterministic first; an
   *optional* AI step (user's own key, as already decided for cad2bim) only rewrites messages and reads
   free-text notes ("typical floors 3 to 9"), never decides a pass or fail alone.
4. **RFI composer:** the evidence boxes become cropped images (render the viewer region to canvas),
   the impact numbers come from the existing BOQ/BBS engine (`packages/engine`), computed for both
   versions of the conflict. Output with the existing Excel/PDF writers; BCF through the BCF writer planned
   for IDS Gatekeeper.
5. **Assumption mode:** extend the pipeline's input with `overrides` (mark to size/level) and a per-element
   `assumption_id`; the IFC writer carries it as a property (`Cad2Bim_Assumption = RFI-014`), so it survives
   export to Revit. **Resolve** edits the override table and uses the existing *Revision* rebuild (rebuilds
   only what changed, keeps ids, as in 0.101.0).
6. **UI (React, the design system):** a CAD-workspace dock panel "RFI Forge" with three tabs (Summary, RFIs,
   Ledger), pins drawn as an overlay layer in the 2D viewer, a split view for "A vs B", amber colour token for
   assumptions (a token, not a raw value). A Python/MCP tool `audit_drawings()` so an AI agent can run it too.
7. **Tests:** a folder of synthetic DXF fixtures, each with one planted error (never real client drawings,
   which stay out of git); golden RFI output; precision/recall counted on a planted-error set. Report what
   was measured and what was estimated (cad2bim's own documentation rule).

**Phases.**
- *Phase 1 (about 2-3 weeks):* marks, dims, levels, refs on the CH template, RFI register to Excel, pins in the
  viewer. This alone is the wow demo.
- *Phase 2:* assumption mode + Resolve + ledger; impact numbers (concrete/steel delta) in the cards.
- *Phase 3:* BCF/PDF output, custom rule packs, geometry-sanity rules for non-template DWGs, code-smell pack.

**Honest risks.** Drawings from other consultants do not follow the CH template, so the first release works
best on a known layer/mark convention and says so. A false alarm costs trust; therefore every rule has a
severity, a tolerance and a "dismiss with reason" (remembered per project). Code-smell rules must be worded as
questions for the engineer, never as design approval.

### Future update

- **Cross-discipline mode:** architect's plan vs structure plan (column positions, slab steps, openings), the
  same engine with more fact sources; feeds Flush Fit and Sleeve Punch.
- **Scanned PDF input** through Plan Tracer, so even a PDF-only set can be audited.
- **Learning from the ledger:** after 20 projects the common error types per consultant are known; show
  "this consultant's sets usually conflict on schedule vs plan sizes" and tune the tolerances.
- **Consultant-side "issue gate":** a locked-down mode that blocks the Export-to-issue button until blocking
  conflicts are zero (pairs with IDS Gatekeeper's readiness certificate).
- **Live RFI status board** (open, answered, overdue) shared as a link, so the PM sees the cost of waiting.
- **RFI turnaround analytics:** days open per consultant, cost of delay per RFI, the data a contractor uses
  in an extension-of-time claim.
- **A reel-ready export:** the "radar sweep" scene as a short video (like the Model Reel idea), so the
  tool markets itself.
