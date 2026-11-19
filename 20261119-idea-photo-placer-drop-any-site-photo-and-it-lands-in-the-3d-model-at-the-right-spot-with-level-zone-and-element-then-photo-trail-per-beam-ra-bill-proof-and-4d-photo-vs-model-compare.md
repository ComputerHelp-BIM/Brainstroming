# 20261119 Idea: Photo Placer — Drop 200 site photos from any phone. Each one lands in the 3D model at the spot it was taken, tagged with Level, Zone and the elements in view. Every beam, column and slab gets a dated photo trail; the RA bill, the pour card and the dispute file fill themselves.

## Entry 1

```
date-time:   2026-11-19 (scheduled run, UTC)
idea name:   Photo Placer (site photos placed in the 3D model, in the browser, nothing uploaded)
potential:   High for daily use on any site-facing BIM team. Site photos are the evidence for the RA bill,
             the pour check, the snag list, the delay claim and the handover file. Today they sit in
             WhatsApp and phone galleries, named IMG_4821.jpg, and someone spends hours finding "the
             photo of the B21 reinforcement before pour". Time saved: NOT measured.
             First test: take one real site's 150 photos (kept out of git). Time a person finding the photo
             evidence for 10 named elements by hand, then with Photo Placer. Report (a) minutes,
             (b) how many photos landed on the right level, the right zone and the right element,
             checked by a person. Do not quote hours or accuracy before that.
             Market check (2026-11-19, two searches; not exhaustive, so "I did not find", not "none"):
               - OpenSpace (which took over Disperse in 2025) places 360 photos in BIM models and floor plans
                 with its own AI engine ("BIM Sync"). It needs a 360 camera capture routine and a paid
                 subscription; the price is not public (quote only). Source: openspace.ai product pages.
               - Research, 2025-2026: phone-camera localisation against BIM (PolyU, RWTH Aachen, TUM, an
                 ISPRS 2026 paper). I only read search snippets. One reports a mean 3D error of 0.92 m for
                 the smartphone step. RWTH stores camera positions back in the IFC as documentation points,
                 in a web tool. These are research prototypes, not daily tools.
               - I did not find a free, browser-based tool that takes ordinary phone photos (not 360), places
                 them in an IFC, links them to elements by level/zone, and writes the trail to the
                 BOQ, RA bill and pour card. I did not check every Revit add-in store.
               - I could not measure LinkedIn or Instagram counts. Site-capture, "reality capture" and
                 "AI progress tracking" are common AEC post themes; that is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Snag Sweep / Audit Eye / Chat-to-Model: photos pinned by hand or by a voice note. Photo
                 Placer places them on its own, in bulk, from the gallery.
               - Scan Check: point cloud vs IFC (needs a scan). Photo Placer works with plain photos.
               - Cage Ghost: AR over live steel, one photo at a time, on site. Photo Placer is the office
                 side: all photos, any day, searchable.
               - Bill Painter and Delay Detective: both need proof. Photo Placer is the proof store.
               - Site QR Pass / Pour Card: a QR scan opens the 3D. Photo Placer adds "what did it look
                 like on 14 Oct".
             The wow: drag a folder of 200 photos onto the model. Little camera cones appear in the 3D
             one by one, each with its photo. Click column C12: a strip of 9 dated photos, oldest to
             newest, "binding", "shuttering", "poured", "deshuttered". Slide the 4D date: the photo
             nearest that day opens beside the model, next to what the schedule says should be there.
```

### Description

**The pain.**
- Photos are taken all day and mean nothing a week later. The file name has no place, no element.
- The RA bill needs "photo proof of work done". The QS asks for it, the engineer scrolls WhatsApp.
- In a dispute, the question is "what was the state of this beam on that date". The answer is a
  guess, or a search through 1,000 photos.
- Existing tools that place photos need a 360 camera and a paid plan. Most Indian and small-firm
  sites use phones.

**The idea.** Four moves.
1. **Drop.** Drag photos (or a WhatsApp export zip) onto the model in the browser. Nothing is
   uploaded. Time and, if present, GPS come from the photo's own EXIF data.
2. **Place.** Three levels of help, from certain to clever, and the tool always says which it used:
   - *Level and zone from clues:* time order of the walk, the person says "this is 3rd floor", or a
     visible floor or room sign. Cheap and mostly right.
   - *Two-click anchor:* the modeler clicks 4 spots in the photo and the same 4 in the 3D; a standard
     pose solve (PnP) gives the exact camera. Photos shot just after it (by time) are placed relative to
     it and the modeler only confirms.
   - *Auto match (later):* match photo edges and shapes to the model's rendered views, as the
     research papers do, and keep it only above a confidence level.
3. **Link.** Each photo gets the elements in view (from the camera cone, hidden ones removed) and a
   confidence. "C12, C13, B21 (partly)". A person can fix it by one click.
4. **Use.** Photo trail per element; a photo strip along the 4D timeline; one-click evidence packs
   (PDF/zip) for an RA bill line, a pour card, an RFI or a delay claim.

**What a modeler sees.**
- A "Photos" panel: count placed / doubtful / unplaced; the doubtful ones glow amber with the reason.
- Camera cones in 3D, filter by date, by level, by who took it.
- Click any element: photo trail. Click any photo: the model view from the same camera, side by
  side, with a slider to blend them.

### Implementation

Inside the cad2bim IFC viewer and project file; no server.
- **Photo store:** photos are kept as references in the .c2b project (and in an optional local folder),
  never uploaded. EXIF read in the browser.
- **Pose solve:** PnP from user-picked point pairs, in TypeScript (a small, tested module in
  packages/engine). The viewer's camera is set to the solved pose, so the model is drawn behind the
  photo for a check.
- **Propagation:** photos in time order get a starting pose from the previous one; the user confirms
  or corrects. Visual odometry is a later step, not in the MVP.
- **Element linking:** ray or frustum test against the engine's spatial index, with occlusion, to list
  the elements in view. Store as IFC documentation links (IfcDocumentReference) plus a per-photo
  JSON sidecar with the pose and confidence.
- **Evidence pack:** reuse the BOQ / BBS PDF writers; a pack = elements, dates, photos, pose
  confidence and who confirmed.
- **Agent access:** MCP tools in packages/mcp: "photos of element", "photos between dates",
  "unplaced photos". An AI agent can then answer "show me proof for the level 4 slab pour".
- **Privacy:** faces and number plates blur is optional (on device); strip GPS on export by default.

**MVP (3 weeks).**
1. Drop photos, read time, show on a timeline.
2. Two-click anchor with PnP, camera cone in 3D, blend slider.
3. Element list per photo from the camera cone; photo trail on element click.
4. Evidence pack PDF for one element or one RA bill line. Test on one real site's photos and report
   the placement hit rate honestly, including failures.

### Future update

- **Auto-place:** edge matching to model renders; accept only above a confidence level; show the rest as amber.
- **360 and video:** a phone walk video split into frames, placed along a path (links to Scan to Frame).
- **Progress hint:** compare photo with the model's expected state for that date (4D) and suggest "looks
  behind"; always a suggestion, a person decides (links to Bill Painter and Delay Detective).
- **Photo diff:** the same view on two dates, blended, to see what changed.
- **Quality coach:** warn at upload: blurred, too dark, no reference visible, "take one wider shot".
- **WhatsApp bot:** send a photo with a caption "L3 C12 binding"; it arrives placed (links to Chat-to-Model).
- **Handover:** the photo trail becomes part of the as-built file and Handover Hub's asset pages.

### Improvement suggestions for existing tools in the repos

- **cad2bim:** keep an element-id to document-reference list in the project file, so photos, BCF
  issues, pour cards and QR passes share one "attachments" store instead of four.
- **cad2bim viewer:** a "set camera from pose" API and a photo-over-model blend, also useful for
  Cage Ghost, drawing overlays and markup.
- **Snag Sweep / Audit Eye:** accept the same placed photos, so a snag pinned once is also in the trail.
- **Bill Painter:** add a "proof" column per bill line that points to the photo pack.
- **C2B:** let the Excel review workbook list photo counts per element, so a client DXF review can also
  show "no photo yet" gaps.
