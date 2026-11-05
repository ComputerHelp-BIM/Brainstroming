# 20261105 Idea: Save-to-See — draw in AutoCAD, press Save, and the 3D model updates in seconds in the browser beside it, with the changed elements lit up, template errors pinned to the drawing line, and a one-click push to Revit

## Entry 1

```
date-time:   2026-11-05 (scheduled run, first entry of the day)
idea name:   Save-to-See (Live DXF-to-3D Preview Loop: Watch the Drawing File, Re-run the Pipeline on Every
             Save, Diff Against the Previous Build, Light Up What Changed, Pin Template Errors to the
             Exact Drawing Entity, Keep Camera and Selection, Push to Revit / Export IFC on Demand)
potential:   High. The CH-template modeller works in a loop: edit the DXF, save, convert, open the result,
             spot a mistake, go back. Today every turn of that loop is manual and the feedback arrives late
             (often a day later, after someone else opens the model). If the loop drops from minutes to
             seconds, mistakes are caught while the line is still under the cursor, when they cost seconds
             instead of an RFI. Time saved (an estimate, not measured): each convert-and-check turn costs
             about 2 to 5 minutes by hand (export or upload, wait, open, find the spot); a drafter does
             30 to 80 turns on a G+7 plan set. That is roughly 1 to 4 hours per project, and more
             important, fewer errors that reach the model reviewer. It is also the best demo the toolkit
             can have: two windows, AutoCAD on the left, 3D on the right, and a column grows when the
             line is moved and saved. Nobody forgets that on a LinkedIn clip.
```

### Description

**The pain (what the drafter and the modeller actually do today).**
- The drafting rule set (the CH template, `docs/design/ch-template-v2.md`) is strict by design: layers,
  labels, plan levels, schedule tables, mark lists. A wrong label, a missing level number or an unclosed
  outline gives a wrong or missing element. The pipeline already reports checks, but only after a full
  open-and-convert, in a panel, away from the drawing.
- So the drafter works blind for an hour, converts, and finds forty problems at once, some caused by the
  first one. Finding the cause means reading a check message, then hunting the entity in a crowded plan.
- The people who check the 3D (a modeller, a structural engineer) get the file by email or chat, open it,
  and send comments back. The drafter fixes, resaves, resends. Each round is hours or a day.
- There is no feedback between "I moved this line" and "this column is now 300 mm off its grid".

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Trace Lens (2026-11-03) goes from a 3D element **back** to its DXF source and
  lists skipped lines; Layer Whisperer (2026-10-30) cleans up a messy file **before** conversion; Rev Merge
  (2026-11-02) merges a **new revision** into a hand-fixed model; Sheet Sync (2026-10-17) mentions a "live
  link mode" for Excel only (and in the desktop bridge). **None of them closes the loop while the drawing is
  being drawn**: no watch, no re-run on save, no diff of what the last save changed, no error pinned back
  to the entity.
- In cad2bim (read from `CLAUDE.md`, `docs/` and the source): the DXF/DWG to 3D pipeline is Rust compiled
  to WebAssembly (`packages/pipeline`, run by the CAD engine's core), so it already runs in the browser
  with no upload. The app already keeps file handles of opened files (`apps/web/src/lib/recent.ts`, uses
  `showOpenFilePicker` in `openFile.ts`), so re-reading a file the person opened once needs no new
  permission model. The pipeline returns checks with the plan, level and entity they refer to
  (`ChResult { plans, levels, elements, grids, checks }`). The pieces exist; the loop does not.
- In the market (not exhaustively checked): hot-reload on save is normal in software development and in a few
  scripting tools (Dynamo and Grasshopper recompute when inputs change), but I did not find a tool that
  watches a **2D DXF** and rebuilds a **structural 3D / IFC model** beside the CAD window, in a browser,
  for free. Rebuilding in Revit from a changed DWG is manual or is a paid import add-in. Treat "nothing
  like it" as unverified.

**The idea.** A **Live** switch on the open drawing in the cad2bim web app. Nothing leaves the device.

1. **Watch.** The person opens the DXF with a "Watch this file" choice. The app keeps the file handle and
   checks the file's modified time (polling every second or two; Chromium browsers allow reading a handle
   again after the first grant). On a change it waits for the writer to finish (stable size and time for
   300 ms, so a half-written save is never read), then reads it.
2. **Rebuild fast, quietly.** The pipeline runs in a worker, so the viewer never freezes. The old model stays
   on screen until the new one is ready, then the swap is instant and the camera, section box, hidden
   items and selection are kept (selection is matched by the element's stable key: the pipeline's
   GlobalId seed is already layer, mark, level and position). If the build fails, the last good model stays
   and a banner says so; the person never loses the view.
3. **Show what the save changed.** Compare the new build with the previous one by stable key:
   added (green glow), removed (red ghost for two seconds), moved or resized (amber with a short slide),
   unchanged (normal). A small counter reads "this save: +2 columns, 1 beam resized, 1 wall removed".
   Plain colour plus a text count, not colour alone.
4. **Pin errors to the drawing, not to a list.** Each pipeline check that has an entity gets a pin at that
   position in the 3D and in a 2D plan view of the drawing, with the sentence from the check and, where the
   pipeline can tell, the one-line fix ("level 3 is claimed by two plans: plan A and plan C"). Fixing the
   line and saving makes the pin disappear on the next build; new problems from this save are marked "new",
   so the drafter sees what they just caused.
5. **Second-screen mode.** A "Pop out" opens the 3D in its own window so it can sit on the second monitor
   beside AutoCAD, with a plan and 3D linked: click a column in 3D and the plan view zooms to its outline
   and prints the layer, label and entity handle. The drafter can then type `HANDLE` in AutoCAD or use a
   small copy button for the handle, to jump straight to the entity. (No AutoCAD plug-in needed in V1.)
6. **Keep it honest.** A header shows the build time, the file's save time and the counts, so nobody
   confuses a 3-minute-old model with the drawing on screen. A stale build is shown with an amber tag.
7. **One-click outputs when it looks right.** "Export IFC", "Send to Revit" (through the existing Revit
   bridge) and "Save a checkpoint" (a named, dated snapshot in the browser's storage) take the current build
   without a second conversion.
8. **Quantities tick over live.** Concrete m³, formwork m², steel kg (once rebar is on) and element counts
   update on each build with their change since the previous one. A drafter sees that moving one wall added
   0.8 m³ of concrete.
9. **Team use without a server.** Two people can use it on one shared drive: the drafter saves, the
   reviewer's browser watches the same file path on the shared drive (or a synced folder) and refreshes. No
   upload to cad2bim; the file only moves where the team already moves it.

**Why it will be popular and shareable.** The two-window clip is the whole pitch: drag a column line in
AutoCAD, hit Ctrl+S, and the 3D column slides on the right with a green glow and the quantity ticker
changes. It shows the toolkit's best property (the pipeline is fast and local) in five seconds, and it solves
a real daily annoyance. Suggested post: "I draw in AutoCAD. The 3D updates when I press Save. No upload,
free." Record the real build time; do not promise a number before measuring.

### Implementation

- **Where it fits.** `apps/web` (Live switch, banner, pins, the diff colours, pop-out window);
  `packages/cad-engine` (the pipeline call stays as it is; a small API returns the result with stable keys
  and the check positions); `packages/engine` (the viewer: swap a model while keeping the camera, selection
  and section box; highlight sets); `packages/pipeline` (no change in V1, only the use of keys and check
  positions it already returns). No new dependency. No upload; everything runs on the device.
- **Watch loop.** Use the file handle from `recent.ts`; call `getFile()` on a timer, compare `lastModified`
  and `size`, debounce 300 ms, read, hash (SHA-256 is already in the toolchain for pinning) and skip if the
  content is the same (AutoCAD sometimes saves with no change). Fall back for browsers without the file
  access API: a "Reload" button and drag-and-drop of the same file, with a note saying why.
- **Worker and cancel.** One build at a time; a newer save cancels or supersedes the running build (only the
  newest result is shown). Keep the last good result in memory, so a failed or cancelled build changes
  nothing.
- **Diff.** Index elements by stable key; compare type, geometry fingerprint (a hash of the footprint,
  levels and size) and properties. Output four lists. Keep the list of the last 20 builds as a "history
  strip" so the person can scrub back to see the model before an unlucky save.
- **Pins.** Reuse the QA finding list (`packages/engine/src/qa/`): a check becomes a finding with a
  position, a message and an optional fix text; the QA panel already selects and zooms. Add the "new in
  this save" flag by comparing finding keys between builds.
- **Handle to the entity.** Carry the DXF entity handle (hex) and layer from the CAD core into the
  element's source info (the same data Trace Lens needs; if that idea is built first, share the code).
- **Pop-out.** `window.open` with a message channel (BroadcastChannel) so the second window only renders
  and the first holds the data; or a single window with a detach option if the browser blocks pop-ups.
- **Tests.** (1) A synthetic DXF modified step by step (move a column, add a beam, delete a wall, break a
  label): the diff lists exactly those changes. (2) A half-written file (truncated) is not built, and the
  last good model stays. (3) Same content saved twice: no rebuild. (4) Fast double save: only the newest
  build is shown. (5) Selection and camera survive a rebuild. (6) A build that throws leaves the view
  unchanged and shows the message. Test with files in a scratch folder, never in git (`CLAUDE.md`: drawings
  are never committed).
- **Measure before claiming.** Record on three real drawings (small, typical G+7, large podium): time from
  file save to new model on screen, on a mid-range laptop; and the memory used. Publish those, not guesses.
- **Risks.** (1) Long builds on large drawings break the "seconds" promise: show a progress bar and, if
  needed, rebuild only the plans whose content hash changed (the pipeline reads plan by plan). (2) Browsers
  without the file access API: a clear fallback and a printed list of supported browsers. (3) Saves from
  cloud-synced folders arrive in two steps (temp file then rename): the stable-for-300 ms rule and the hash
  check handle it, tested on one synced folder. (4) A reviewer mistaking a live preview for the issued
  model: a visible "LIVE PREVIEW, not issued" tag, removed only by an explicit checkpoint or export.
  (5) DWG files: the pipeline reads them through the CAD core's conversion; watching works the same, but the
  build is slower, so say so in the banner.
- **First 2 weeks.** A bare watch loop on one DXF with the swap that keeps the camera; the diff colours and
  the counter; check pins with "new" flags. No pop-out, no shared drive, no history strip in the first
  version. Record: build time on three drawings, how often a half-written file was seen, and how many errors
  a real drafter fixed before ever exporting.

### Future update

- A small **AutoCAD / BricsCAD / DraftSight command** (`C2B_JUMP`) that selects the entity whose handle the
  3D sends, so a click in 3D selects the line in the CAD window. Needs an add-in per CAD product; it is
  separate work and must be written from scratch (no GPL code).
- **Draw-time hints**: as the check list shrinks, show "model health 92%" and a coach line ("3 columns
  have no label; the fastest fix is layer CH-COL-TEXT").
- **Link to Trace Lens**: the pins and the "skipped lines" overlay shown together, so ignored lines are visible
  while drawing.
- **Link to Layer Whisperer**: unrecognised layers get a "map this layer" prompt the moment they appear in a
  save.
- **Link to Rev Merge**: the same diff used to merge a consultant's new revision into the hand-fixed model.
- **Link to Model Capsule and Room Link**: share the live build with a reviewer in the same moment, keeping
  the "no upload" rule (device-to-device only).
- **Excel and BBS live**: the same watch idea for the schedule workbook, rebuilding the rebar cages when the
  sheet is saved (this folds Sheet Sync's "live link" into one shared watch service).
- **Time-lapse export**: turn the build history into a short video of the model growing as the drawing was
  drawn (a 4D of the drafting itself), for a team showcase post.
- **Pre-flight on save for the whole team**: a shared-drive mode where a saved file is checked and a
  one-line verdict ("12 problems, 3 new") is shown to the drafter's lead, without opening the model.
