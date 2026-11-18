# 20261118 Idea: Red Pen — The structural engineer marks columns, beams and sizes by hand on the architect's plan (a phone photo or a scan). Red Pen reads the marks, builds the 3D RC frame on the architect's grid, and turns every unreadable smudge into a question instead of a silent guess.

## Entry 1

```
date-time:   2026-11-18 (scheduled run, UTC)
idea name:   Red Pen (hand markup on an architect plan to a 3D RC frame, in the browser, nothing uploaded)
potential:   High for daily use, and a strong demo. In real Indian projects the first structural
             "drawing" is often a red-pen markup: the engineer circles column positions on the
             architect's plan, writes "C1 300x600", "B 230x450", "S 150" in the margin, and sends a
             photo on WhatsApp. The modeler then re-types all of it into Revit by eye. Time saved:
             NOT measured. First test: take 10 real markups (kept out of git) and measure (a) the
             share of columns and beams read correctly with no edit, (b) the minutes from photo to a
             first 3D model against the modeler's usual hours. Do not quote hours or accuracy until
             that is done.
             Market check (2026-11-18, two searches; not exhaustive, so "I did not find one", not "none"):
               - Snaptrude "Sketch to BIM" and Higharc turn architectural sketches or layouts into
                 BIM. Their focus is rooms, walls and house design, not the structural engineer's
                 marks, sizes and notes. Vendor claims, not tested by me.
               - WiseBIM / Plans2BIM / usBIM.planAI (ACCA) turn 2D plans into BIM. They read the
                 architectural plan, not handwritten structural markup.
               - Academic: Sketch2BIM (hand-drawn floor plans to BIM with a human in the loop) and a
                 2026 paper on sketch plans to IFC with suggested column layouts. Prototypes.
               - Fiverr sellers do it by hand ("hand sketch to Revit model"), which shows the demand.
               - I did not find a tool that reads a structural engineer's handwritten marks (mark,
                 size, steel note) on top of an architect plan and returns a frame with a question
                 list. I did not check Revit add-in stores one by one. I could not measure LinkedIn
                 or Instagram counts; "AI turns a sketch into a model" is a common post style, but
                 that is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Plan Tracer: scanned PDF of CAD-quality drawings to DXF. Red Pen reads handwriting
                 and ink marks on top of a plan, which Plan Tracer does not.
               - Column Compass: the tool proposes three layouts. Red Pen is for when the engineer has
                 already decided and drawn it.
               - Bar Reader: reads printed schedules. Red Pen reads handwritten ones.
               - Guess Gauge and RFI Forge: every unclear mark becomes an amber item and one question.
               - Layer Whisperer / Drop Zone: Red Pen uses the same clean-start path afterwards.
               - Scan to Frame: phone LiDAR of a built structure. Red Pen is for a design not yet built.
             The wow: take a photo of the marked plan on the desk. In under a minute the plan appears
             in 3D in the browser: 38 columns, 61 beams, the slab edge, all on the architect's grid.
             Each element shows a small thumbnail of the ink it was read from. Tap a column: "C7 300x600,
             read from your marks, confidence 94%". Three marks glow amber: "Is this a 9 or a 4?" The
             modeler taps the answer, and the model updates. One button sends the answers back to the
             engineer as a single WhatsApp-ready image with numbered circles.
```

### Description

**The pain.**
- The first structural information often arrives as a photo of a red-pen markup. It is quick for the
  engineer and slow for the modeler.
- The modeler re-types every mark. Handwriting is read wrongly (4 vs 9, 230 vs 280), and nobody notices
  until a quantity or a clash looks odd.
- Markups change by phone call: "move C5 half a metre left." Nothing records it.
- Photos are skewed, shadowed and cut off. Manual alignment to the architect's grid costs time.

**The idea.** Four moves.
1. **Align.** Detect the sheet and the architect's grid bubbles in the photo, straighten the image, and
   fit it to the grid. If the architect's DXF or PDF is available, match to its grid so positions are
   exact. If it is not, the engineer enters one known distance.
2. **Read the marks.** Find the ink that differs from the printed plan (colour and stroke differences).
   Classify it: column (filled or circled square), beam (a line between columns), wall, size note,
   mark label (C1, B12), slab note, arrow ("shift this way"). Read the handwriting with a local text
   model, and use the grid to check it ("C7 sits at B/3 — 3 sibling marks say 300x600").
3. **Build.** Create columns, beams and the slab edge on the architect's grid with the sizes read.
   Unmarked sizes are not guessed silently: they are tagged "assumed" (Guess Gauge). Typical rules from
   the engineer's notes ("all beams 230x450 unless marked") apply to every beam.
4. **Ask.** Each unreadable or conflicting mark becomes a numbered question on an overlay image: "Q3:
   C12 size reads 300x6?0." The engineer taps or types the answer once. The model and the list update.

**Smart parts.**
- **Ink layer.** The tool separates the engineer's ink from the printed plan, so the modeler can toggle
  the original, the ink only, or the 3D model on top. Every element links to its ink crop.
- **Grid snapping.** Hand-placed columns are snapped to grid intersections or to a stated offset
  ("0.5 m from grid line B"). A column that is off by a hand-width is snapped and flagged, not left loose.
- **Sense checks.** Spans over a set limit, a column that stops under a lift core, a beam with no
  support, a column in a door line, or a missing column at a corner. These are checks by the rules in
  the app, not a design approval; the engineer decides.
- **Voice mode.** The engineer can also speak the changes over the photo: "Shift C5 half a metre east,
  make B7 230 by 600." The change is applied and logged with the voice note attached.
- **Revision memory.** A second photo of the same plan is compared with the first (Rev Merge): "3 columns
  moved, 2 beams resized", and the modeler's hand-fixes are kept.
- **Learn per engineer.** After a few jobs the tool knows one engineer writes "c/c" for spacing and
  circles columns, and another uses a filled square. It starts with the common marks and learns a profile.

### Implementation

- **Runs on the device.** Nothing is uploaded: the photo, the reading and the model stay in the browser,
  as the rest of cad2bim does (the repo rule: nothing leaves the device unasked).
- **Image steps (no new heavy stack):**
  - Sheet and grid detection and perspective fix with classic computer vision (OpenCV compiled to
    WebAssembly, Apache-2.0 licence).
  - Ink separation by colour and by difference from the architect's printed plan when it is known.
  - Marks classified by a small on-device model; handwriting read by a local OCR model (for example an
    Apache-2.0 or MIT one, run in WebAssembly or WebGPU). Licences to be checked one by one against the
    list in the app's CLAUDE.md before any model is shipped. No GPL.
- **Optional AI helper.** Where the person has chosen to use their own AI key (an existing cad2bim
  milestone), a cropped patch of one smudge, never the whole drawing, can be sent for a second opinion,
  and only after the person agrees.
- **Build step:** the result is a list of elements in the same input form the DXF to 3D pipeline already
  uses, so the Rust pipeline builds the columns and beams, assigns types, and writes IFC. No change to
  the v1 `Entity`; a new small adapter in front, like the v2 reader sits beside v1.
- **Data:** each element carries `Pset_C2B_Source` (ink crop id, photo id, confidence) and, where unclear,
  `Pset_C2B_Certainty` from Guess Gauge.
- **UI:** a "Red Pen" workspace: photo on the left, 3D on the right, a Questions list underneath, and an
  Export reply image button. Same design system, no new design language.
- **Agent:** MCP tool `read_markup(image) -> elements + questions`, so an AI agent can run the same job and
  must cite the ink crop for each element.
- **Cost:** medium to large. About 4 weeks for photo align, grid fit and printed-size reading; 4 more for
  handwriting and the question loop. Handwriting accuracy is the risk, so the question loop is the product.

### Risks

- Handwriting varies; a wrong confident reading is worse than a question. So: show confidence, and ask
  below a limit that the modeler can change. Count wrong reads in the first test.
- A markup is not a design. The tool builds what was marked and does not check strength. Its sense checks
  are modelling checks only; the engineer stays responsible.
- Photos with heavy shadow or a folded sheet may fail. Say so early ("sheet not found, retake") rather than
  produce a poor model.
- A heavy on-device model can be slow on an old laptop. Offer a "scan only the grid and the printed sizes"
  quick mode, and measure time on a low-end device.
- Privacy: markups are client documents. Keep everything local by default; the AI-helper crop is opt-in.

### Future update

- **Phase 1 (about 4 weeks):** photo straighten, grid fit, printed size notes, columns only, with the
  ink-crop link and the 3D view.
- **Phase 2:** beams, slab edge, the "unless marked" rules, and the question overlay with WhatsApp image.
- **Phase 3:** handwriting model, per-engineer profile, Guess Gauge tags and RFI Forge link.
- **Phase 4:** second-photo comparison (Rev Merge), voice changes, sense checks.
- **Phase 5:** foundations and wall marks, steel notes ("4-16 + 2-12") into rebar through Bar Reader's
  readers and the BBS Weaver path, so the first photo can also give a first bar schedule.
- **Phase 6:** tablet pen mode: the engineer draws straight on the architect's PDF in the browser and the
  3D model updates live. A Revit add-in button on the same job.
- **Test first:** run it on 10 real markups from the team, count right and wrong reads, and measure the
  time saved before building Phase 3.

### Sources (market check)

- Snaptrude Sketch to BIM (vendor blog) — https://snaptrude.com/blog/sketch-to-bim-space-layout-parametric-walls
- Higharc AI: 2D sketch to 3D BIM (AEC Magazine) — https://aecmag.com/ai/higharc-ai-delivers-3d-bim-model-from-2d-sketch/
- 2D to 3D tools WiseBIM, Plans2BIM, usBIM.planAI (AEC Magazine) — https://aecmag.com/bim/from-2d-to-3d-and-back/
- Sketch2BIM: hand-drawn floor plans to 3D BIM with a human in the loop — https://aitopics.org/doc/arxivorg:659C4DF4
- IFC model and structural layout from architectural sketch plans — https://soffeh.sbu.ac.ir/article_106785.html?lang=en
- TU Wien, From sketching to BIM — https://repositum.tuwien.at/handle/20.500.12708/187571?mode=full
- Fiverr "hand sketch to Revit model" (shows manual demand) — https://fiverr.com/robayat_bb/do-hand-sketch-to-revit-models
