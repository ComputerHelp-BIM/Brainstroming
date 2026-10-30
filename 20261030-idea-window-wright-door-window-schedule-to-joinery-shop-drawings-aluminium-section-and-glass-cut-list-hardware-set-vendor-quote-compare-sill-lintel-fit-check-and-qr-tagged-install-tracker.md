# 20261030 Idea: Window Wright — door/window schedule from the drawing to joinery shop drawings, section and glass cut lists, hardware, vendor quote comparison and a QR-tagged install tracker

## Entry 1

```
date-time:   2026-10-30 (scheduled run)
idea name:   Window Wright (Door and Window Joinery Autopilot). "D1, W3, W5 on the plan -> an order the
             fabricator can cut, in one click."
potential:   High. Doors and windows are small in volume but large in pain: hundreds of marks, every size
             slightly different, ordered by phone and WhatsApp, fabricated by a vendor who re-measures on
             site. Wrong sizes, wrong swing, wrong glass and "opening not ready" are daily rework. Every
             residential and commercial job has this package and it is usually done by hand from an Excel
             sheet. cad2bim already reads the door/window marks and schedules from the DXF (CH template v2)
             and builds them hosted in walls (pipeline 0.3.0). The one missing step is turning the modelled
             opening into a fabrication order.
             Market: window-design software (WinDoor-type tools, fabricator CAD) is paid, vendor-specific
             and does not read the project's own model. Revit door/window schedules list sizes but give no
             section cut list, glass sizes or vendor comparison. A free, on-device, vendor-neutral version
             does not exist as far as this run could tell (not verified by a live search).
```

### Description

**The pain.** The BIM modeller or site engineer is asked on Monday: "Send the window schedule to three
vendors by Wednesday and tell me who is cheapest." What happens today: copy marks from the plan to Excel,
re-type sizes, guess glass and track, send three different messages, get three differently formatted quotes,
and compare them by eye. Two months later: "W7 is 50 mm short, the sill was cast higher."

**Why now.** LinkedIn/Instagram AEC feeds keep rewarding "plan to schedule in seconds", "BIM to fabrication"
and "stop re-typing data" posts. A before/after of a window schedule turning into a priced order is exactly
that format. (Trend not verified by a live search in this run; judged from the existing ideas and general
feed themes.)

**The idea.** A **Joinery** panel. Select doors and windows (all, a level, a mark), pick a **system** (for
example 2-track aluminium sliding, UPVC casement, flush door with teak frame, fire door, steel louvre), and
Window Wright produces everything a vendor needs.

1. **Clean schedule, from the model.** One row per mark: width x height, sill and head level, wall thickness,
   swing/slide direction, quantity, level and flat. Counts and sizes come from the built openings, so they
   match the drawing, not a hand-typed sheet. Marks with the same size and type are merged; near-duplicates
   (W3 at 1195 and W4 at 1200) are flagged for the modeller to merge.
2. **Shop drawing per type.** Elevation with panels, mullions and transoms, a plan swing, and a section at
   the sill and head with the wall build-up. Dimensions, notes and a title block, as DXF and PDF. Auto-sheet
   per type, not per mark.
3. **Cut list (the wow table).**
   - Aluminium/UPVC: outer frame, shutter, interlock and bead lengths per mark from an editable system
     library (deductions per section, so a 1200 window gives exact shutter widths). Then a **6 m length
     optimiser** (the same cutting-stock idea as Bar Optimizer) gives bars to buy and the offcut %.
   - Glass: pane sizes after the rebate deduction, thickness by area and wind/height rule (editable
     table, with an "engineer to confirm" tag), toughened/laminated flags for sill < 1 m and for doors,
     and a **glass sheet nesting** list for the glazier.
   - Timber/flush doors: shutter, frame, architrave, ply/skin sizes.
4. **Hardware set.** Handles, locks, hinges, stoppers, rollers, mosquito mesh and grills per mark from
   the system's rule (a 1800 sliding window = 2 rollers x 2 shutters + 1 lock, etc.), as a counted list.
5. **Fit check before ordering.** Reads the structure and the finishes:
   - clear opening vs frame size after plaster (uses Wall Wizard's plaster thickness if present);
   - sill level, lintel/beam bottom and head clearance, a column or shear wall inside the opening width;
   - door swing against the next door, a wardrobe, or a bath; sliding window against a column face;
   - a window with its sill below 900 mm on an upper floor gets a "guard / toughened glass" flag.
   Every flag lands in the existing QA/BCF flow (IDS QA Autofix), pinned to the opening.
6. **Quote compare.** The RFQ is one Excel/PDF generated from the cut list. Vendors reply in their own
   sheets; the modeller pastes or imports the rates (per kg of section, per sq ft of glass, per piece of
   hardware). The tool recomputes each vendor's total on the same quantities, shows the lowest per type
   and the lowest basket, and warns when a vendor omitted an item. Rates can feed Rate Rocket.
7. **QR-tagged install tracker.** Each unit gets a mark + QR sticker sheet (as in Site QR Pass). On site
   the engineer scans: ordered, delivered, installed, glazed, snagged. A 3D view colours every opening by
   status. Opening-not-ready (no lintel, wrong sill) is raised before the vendor arrives.

**Outputs.** Door/window schedule (Excel), shop drawing PDF/DXF per type, RFQ pack, section/glass/hardware
lists, vendor comparison sheet, QR sticker sheet, status 3D view, and a one-page summary for the PM.

**Wow moment (reel-ready).** A plain plan with D/W tags. One click: the 3D model, then the same windows
"explode" into frame, shutter and glass. A cut-list table fills. The three vendor quotes drop in and the
cheapest basket lights green with the rupees saved. Last frame: the building coloured by install status.

### Implementation

- Reuse, not rebuild: mark, size, level and wall host from the DXF/pipeline reading (CH v2 schedules,
  `hosting` openings); schedule tables and Excel export from the existing schedule/BOQ code; cutting-stock
  solver from Bar Optimizer; QR from Site QR Pass; issues from the BCF flow. New code: the system library, the
  deduction rules, the fit checks and the quote compare.
- System library is **data, not code**: a JSON per system (sections, deductions, hardware rules, glass rules),
  with a few open generic systems shipped and a "make your vendor's system" editor. Vendors' exact catalogue
  sections are the user's to enter; the app ships none that it does not have rights to.
- Local-first: nothing is uploaded; RFQ and quote files are plain Excel/PDF that the person sends.
  No GPL code; PDF/DXF writing with permissively licensed libraries only (per the licence rules).
- Phase 1 (2 weeks): clean schedule from the model, merge of near-duplicate marks, Excel/PDF, fit check
  (clear opening, sill/head, swing clash).
- Phase 2: system library with section/glass/hardware cut lists, 6 m optimiser, shop drawing per type.
- Phase 3: RFQ pack and quote comparison, QR tracker and 3D status colouring, BCF flags.
- Honest limits: deductions and glass thicknesses depend on the vendor's system and the engineer's wind/
  safety check, so they are editable inputs that are shown, never hidden constants. The cut list is a
  planning list, and the vendor still measures on site; the tool says so on every sheet.

### Future update

- Curtain-wall and sliding-folding systems, with Skin Cutter handling the panelised facade.
- Photo of the vendor's quote read on the device into the compare sheet (confirmed by a person).
- Lead-time and delivery date per mark, pushed into the 4D timeline so a window is not ordered too early.
- Acoustic and U-value tags per system, a one-click energy/ECBC and sound-rating sheet.
- Shared vendor library across a company: one team keeps its approved systems and rates.

### Why it is unique / what exists

- Window fabrication CAD is paid, vendor-locked and starts from a blank drawing, not from the project model.
- Revit/IFC schedules list openings but have no cut list, glass nesting, hardware or vendor comparison.
- Excel templates work but are typed by hand and drift from the model.
- This extends cad2bim's DXF reading, hosted openings, BBS/Bar Optimizer cutting logic and QR tracking
  instead of starting a new product.
