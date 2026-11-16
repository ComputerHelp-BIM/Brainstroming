# 20261116 Idea: Cut Check — "Can I drill or core here?" Tap a spot on the 3D model (or the site phone) and get SAFE / SHIFT 80 mm / ASK THE ENGINEER, worked out from the real rebar, beams, tendons and pipes, with a one-page core-cut permit card and a check of the cover-meter scan against the model

## Entry 1

```
date-time:   2026-11-16 (scheduled run, UTC)
idea name:   Cut Check (post-pour core, drill and chase checker on the 3D model, in the browser,
             nothing uploaded)
potential:   High, and it comes up every week. After the pour, MEP, fit-out and the client change their
             minds: a new floor drain, an AC drain pipe, a sleeve that was forgotten, a split-AC hole in a
             beam-side wall, a cable tray anchor, a lift-shaft bracket. Each one is a hole in concrete that
             already has steel in it. Today the answer is a phone call to the structural office ("is it ok
             if the core is 100 mm here?") and a guess from a drawing that may be an old revision. A wrong
             hole cuts a main bar, a stirrup group in a beam, or a PT tendon. Repair is costly and the
             structural engineer is blamed.
             Time saved: NOT measured here. First test: take 20 real past core-cut requests (keep them out
             of git), time the usual phone-and-drawing answer, time Cut Check on the same spots, and have
             the structural engineer mark each answer right or wrong. Do not quote hours or rupees until
             that is done.
             Market check (2026-11-16, two searches; not exhaustive, so "I did not find one", not "none"):
               - Before coring, sites scan with handheld rebar scanners or GPR (Screening Eagle, Nirixense
                 ReX 100, GP Radar articles). These find steel in the real slab. I did not find a 2026
                 product that checks a spot against the BIM model's own rebar and says safe or not.
               - A 2012-13 research line (NIST / ASCE paper) used 3D imaging data to mark safe drilling
                 zones for embeds in bridge decks. A research framework, not a product.
               - Vendor claims of cost per hit (about $12,000 per repair, about $30,000 per PT cable) are
                 from a service provider's page. Treat as rough and US-based.
               - A rebar MCP startup (ConstructMCP, private beta, about $50/month planned) counts and
                 schedules rebar. It is not a drill-position check. Not verified inside any paid product.
             Extends tools already in this repo (does not repeat them):
               - Tendon Tracer: a "do not drill" map for PT slabs only. Cut Check works for every member
                 (beams, columns, walls, slabs, footings) and reads the PT map when there is one.
               - Sleeve Punch: plans openings BEFORE the pour. Cut Check handles the ones that come AFTER.
               - Scan Check and Audit Eye: compare a scan with the model. Cut Check uses the same idea on
                 one small patch with a cheap cover meter reading.
               - Cage Manual, Joint Judge, Cover Cop: the rebar and cover data it reads.
               - Site QR Pass: the QR card on site; Cut Check adds a "cut here?" button to that same card.
               - Hanger Hero, Live Wire, Drain Line: they place new MEP. Cut Check tells them where a
                 late hole is safe, so their fixes stop moving back to the structure.
             Trend: AEC trade press this year is full of AI agents and MCP servers for BIM (Autodesk Digital
             Builder, Bentley's STAAD MCP server, July 2026). Almost all of it is design and quantities.
             Site-side "ask the model before you cut" is the quiet gap. I could not measure LinkedIn or
             Instagram counts; the sources are web pages, so this is not a social-media claim.
             The wow: stand on site, open the card on the phone, tap "Cut here". The 3D slab shows the
             real bars as thin lines, a red 100 mm circle where you pointed, and the answer: "Hits 2 bars
             of 16 mm (bottom, B1 layer). SHIFT 90 mm toward the grid 4 and it is clear, 40 mm away from
             the nearest bar, cover meter check advised." One more tap and a permit card (PDF with a QR)
             is ready for the engineer to sign from the phone.
```

### Description

**The pain.**
- Holes after the pour are a daily event on every building site, and nobody owns the check.
- The answer lives in the structural engineer's head, or in a drawing set that may be a revision behind.
- A site engineer cannot read a beam section at 11 pm; they either wait a day (the work stops) or drill and hope.
- Hitting steel is not only a repair. It can be a main bar in tension, a shear link group near a support, or a
  tendon. The risk is different in each place and the rules are known, but not at the spot where the hole is.
- Cover meters and GPR show what is in the slab, but they read a patch; they do not say whether losing that
  bar is acceptable, or where the nearest clear spot is.

**The idea in one line.** The model knows where every bar, beam zone, tendon and pipe is. Let a person point at a
spot, tell the hole size and direction, and get a clear answer with a proof, the nearest clear shift, and a
permit card for the engineer.

### How it works (user view)

1. **Pick the hole.** On the 3D model (desk) or on the Site QR Pass card (phone): tap the element, then tap
   the spot. Choose type (core, drill, chase, anchor, sleeve), diameter or width, depth, and direction.
2. **The answer, in three colours.**
   - **SAFE**: nothing but concrete inside the hole plus a margin (default 25 mm each side, a setting).
   - **SHIFT**: it hits a bar or pipe, but a clear spot exists within a small distance. The tool shows the
     shifted circle and how far, in mm and in which grid direction.
   - **ASK THE ENGINEER**: it hits something that cannot be waived on the tool's say: a PT tendon, a main bar in a
     beam bottom or top zone, a stirrup group near a support, a column or wall boundary element, a shear wall
     bar, a hole in the bearing zone of a footing, or a hole larger than the rules allow for that member.
3. **The proof.** A small 3D view shows the hole as a cylinder through the real bars, each bar it touches
   named ("bottom B1, 16 mm, mark B7-L4"), its distance, and the rule that fired.
4. **Check the model against the wall.** The person may enter two or three cover-meter readings (position, depth,
   bar spacing). The tool compares them with the model's bars at that patch: "model says 150 mm spacing at
   40 mm depth; your meter says 150 mm at 38 mm. Matches." Or "meter says 200 mm. The model may be out of date,
   the answer is held until the engineer looks." The tool never trusts a scan blindly and never trusts the model
   blindly. It tells you when they disagree.
5. **The permit card.** One A5 PDF: element mark, level, position by grid, hole size, the answer, the proof image,
   the meter readings, who asked and when. A QR opens the same view. The engineer approves or rejects from a
   phone, with a comment. Approved cards are kept as a register ("hole log") that goes to the handover file.
6. **Learns the office rule.** The engineer sets once, for example "no core above 100 mm within 1.5 d of a
   column face", and the tool uses those rules from then on. Rules are data, not code.

### Why a BIM team would use it

- The modeller stops being the phone line for holes: the answer is on the model.
- The structural engineer gets a card with the drawing, the proof and a yes/no button, not a call.
- The site loses no day waiting; a SAFE hole needs no approval, a SHIFT hole is a one-tap request.
- Every hole is logged, so the model can be updated: "as-built holes" become openings in the model.
- It is a visible, demo-able feature: the red circle sliding to the clear spot gets a reaction in every client demo.

### Implementation

- **Data.** Reads the rebar geometry already read for the BBS (bar centrelines and diameters), the member
  boxes, and where available PT tendons (Tendon Tracer) and MEP (IFC pipes, conduits, ducts). Works on the
  model as it is; no new file type.
- **Hole test.** A hole is a cylinder (core) or a prism (chase). For each bar, compute the shortest distance from
  the bar's centreline to the hole axis; a hit is distance less than (hole radius + bar radius + margin). Use a
  spatial index (an R-tree or grid) so a floor of 100,000 bars answers in milliseconds.
- **Shift search.** Test circles on a ring around the picked point, growing outward in 10 mm steps up to a limit
  (default 300 mm), and choose the smallest shift that is clear. Prefer shifts along a grid direction, away from
  a support, and to the span's middle third for slabs. All of these are scored in one small function.
- **Rules as data.** A JSON rule pack per member type: the hit class (bar main / stirrup / distribution / PT), the
  limit on hole size by member depth, the minimum clear distance from supports, columns, beam faces and edges.
  The first pack is a plain-language conservative set written with a structural engineer of the firm; the code
  citation and the engineer's name are on every card. The tool never invents a rule.
- **Scan match.** A small form for readings (x, y, depth, spacing, bar diameter if known). The patch is compared
  by spacing and depth within a tolerance; the result is MATCH, DIFFERENT or NOT ENOUGH DATA. Later: import a
  scanner CSV export (format to be checked against the specific scanner's own export; not assumed).
- **Output.** PDF card with the existing PDF path and a small QR library; the register is a table in the
  project file and an Excel export. Approvals use the same review flow as other cards.
- **Privacy.** All of it runs in the browser on the model that is already open. No model is uploaded to ask
  the question. The phone card is the Site QR Pass link, which carries only what the engineer chose to share.
- **Tests.** (1) Geometry tests: a hole just touching, just missing, and fully inside a bar's cover; hand-checked.
  (2) 20 real past requests, answered by the structural engineer first and by the tool second (kept out of
  git); report agreement, and every case where the tool says SAFE and the engineer says not (that count must be
  zero before release). (3) A cover-meter trial on one real slab and one beam with a borrowed scanner, to
  measure how often model and wall agree. (4) A timed trial before any time claim.

### Risks and limits (stated, not hidden)

- The model's bars are only as true as the model. A bar moved on site, a missing bar, or a revised design the
  model does not have will make a SAFE answer wrong. Hence the scan match, the "model date and revision" printed
  on every answer, and a rule that the tool always says "advised: confirm with a cover meter" for core cuts.
- It is a decision aid, not a design check. Anything beyond the rule pack goes to ASK THE ENGINEER by default.
- Tendons and post-tensioned members are never cleared by the tool alone; they need the engineer's signature.
- Cover and spacing differ between a drawn bar and the placed bar; the margin must be larger than the site's
  real placing tolerance, set by the engineer.

### Future update

- **Phase 1 (about 2 weeks):** hole test and shift search on slabs and beams, rule pack v0, 3D proof, PDF card.
- **Phase 2:** columns, walls and footings; tendons from Tendon Tracer; MEP clashes from the model.
- **Phase 3:** cover-meter readings form and the MATCH / DIFFERENT result; hole log register and Excel export.
- **Phase 4:** AR-style view on the phone camera with the bars drawn on the wall (needs the Scan Check alignment,
  so only after that is trusted).
- **Phase 5:** scanner CSV import, and "as-built holes" written back to the model as openings and to the BBS as
  cut bars, for the handover file.
- **Phase 6:** a "plan holes in bulk" mode for a fit-out: paste a list of 200 holes from the MEP contractor's
  sheet and get a coloured list (safe / shift / ask) before anyone drills a single one.
