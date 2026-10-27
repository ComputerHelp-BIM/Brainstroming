# 20261027 Idea: Pile Pilot: drop in the soil report PDF and the ETABS reactions, and cad2bim builds the ground in 3D from the bore logs, works out pile capacity to IS 2911, lays out the pile groups under every column, designs the pile caps, and writes the pile schedule, cage BBS and drawing. On site, the piling crew logs each pile on a phone (bored depth, rock socket, cage, concrete, as-built position). When a pile goes in off position, Pile Pilot rechecks that cap at once and tells the engineer what to change

## Entry 1

```
date-time:   2026-10-27 (scheduled run, UTC)
idea name:   Pile Pilot. "A G+12 job on filled ground near a creek, 6 bore holes, 42 columns, all on
             bored cast-in-situ piles. The engineer drops two files into cad2bim: the geotech report
             PDF and the ETABS reactions table (etabs-mcp 0.6.0 already reads them). Pile Pilot reads
             the bore logs from the PDF: layer, depth, SPT N, soil type, water table, core recovery
             and RQD for rock. The 6 bore holes appear in 3D as coloured sticks, and the layers
             between them are joined into a soil block: brown filled soil, yellow silty sand, grey
             stiff clay, dark weathered rock. The building's columns stand above it.
             'Design piles': 600 dia, cut-off at -1.8 m. For each column it uses the nearest bore
             holes and IS 2911-1-2 (static formula, skin friction plus end bearing, safety factors
             as the code gives them), and it shows the working: 'BH-3, 600 dia, socket 1.8 m into
             weathered rock, safe load 1,450 kN compression, 210 kN uplift, 95 kN lateral (to be
             confirmed by the initial load test)'. Then it groups: 'C14: 6,240 kN + moments, 5 piles
             in a pentagon, spacing 3D = 1.8 m, cap 1,400 deep, M30'. Every cap is drawn in 3D with
             its piles going down into the soil block to their founding level. Colour by
             utilisation: green below 80 %, amber 80-100 %, red above. Click C14 and the pile and cap
             forces are there, with the clause behind each number.
             Out come: the pile layout plan with pile numbers and coordinates (DXF + PDF), the
             pile-cap schedule and details, the pile schedule (length, socket, cut-off, founding
             level), the cage BBS (helical links, lap, cover blocks, IS 2502 shapes), the concrete
             volume per pile with over-break, the set-out points (feeds Set-Out Pack), and the IFC
             with IfcPile and IfcFooting so QA, BOQ and BBS work on them as they do on everything
             else.
             On site (the best part): the piling supervisor opens the pile layout on a phone (the
             site page Site QR Pass already has). Pile P-C14-3 turns blue: 'bored'. He types the
             bored depth, the rock found at (depth), the socket length, the cage number, the
             concrete quantity poured by the tremie and the measured as-built centre (from the
             surveyor). The pile in 3D moves to where it really is. It was put 180 mm off. Pile
             Pilot rechecks C14 within a second: 'Eccentricity 180 mm. P-C14-3 now takes 1,560 kN,
             108 % of the safe load. Options: (a) add 1 pile at (x, y), cap grows 600 mm on the east
             side, +4.2 m3 concrete, +0.38 t steel; (b) deepen the cap to 1,600 and add a tie beam
             to C15, +2.1 m3; (c) ask the geotech for a load test on P-C14-3.' The engineer picks
             one. The cap drawing, the BBS and the BOQ update and are stamped 'Rev B: as-built
             pile deviation'. Concrete poured against the theoretical volume is shown per pile:
             '+38 %: possible cavity or over-break. Flag for an integrity test.'
             For the client and the LinkedIn post: the soil block in 3D, the piles going down into
             it as they are poured, day by day (the same timeline as Model Reel), the rock line in
             dark grey, and at the end every pile is green."
             One engine, five front ends:
               (1) cad2bim web app: a "Piles" dock panel (Soil, Design, Layout, Site log, Revise),
                   the soil block and the piles drawn by the existing three.js viewer,
               (2) etabs-mcp: reads the reactions per load combination and, with the engineer's
                   OK, writes the pile springs (axial and lateral stiffness from the soil layers)
                   back to ETABS as point springs, so the model sits on its piles, not on pins,
               (3) cad2bim CH template v2: a pile layer `CH-Pile` (circle + mark `P1`) and a pile
                   schedule table (`CHT-Pile`: DIA, CUT-OFF, LENGTH) in the foundation plan; the
                   template reader builds IfcPile beneath the cap like it builds footings today,
               (4) Revit (bridge add-in + pyRevit AnonGee button): "Place piles" builds the pile
                   and cap families with structural rebar, numbered, with the pile schedule and
                   the site status as a shared parameter (Bored / Cage / Poured / Tested / NCR),
               (5) Structura: piles and caps show in its 3D, BOQ and code-check views; its
                   codecheck gets an IS 2911 group (pile capacity, group efficiency, cap shear and
                   punching, IS 456).
potential:   High. Every tall building on weak ground, most bridges, metro and industrial jobs in
             India are on piles, and the pile work is one of the most expensive and riskiest
             parts of the job.
               - Today: the geotech report is a 60-200 page PDF read by eye. The engineer copies
                 the SPT values into an Excel capacity sheet, picks a pile count per column by
                 hand, a draftsman draws the pile layout and the cap details in AutoCAD, the BIM
                 modeller builds them again in Revit, and the QS measures them again. On site the
                 pile log is a paper register or a WhatsApp photo. About 3-6 working days for the
                 design and drawings on a mid-size job, and 1-3 days of redesign each time a
                 batch of piles goes in off position (estimated from how offices work, not
                 measured).
               - Mistakes we hear about often (from experience, not counted): capacity taken
                 from the wrong bore hole; uplift and lateral load not checked; as-built
                 deviation found only when the cap is being shuttered, so the cap is redesigned
                 in a hurry or not at all; pile concrete never compared with the theoretical
                 volume; the ETABS model on pinned supports when the piles are flexible.
             With Pile Pilot (target, to be measured on a pilot job): about 30-60 minutes for the
             first design and drawing set, and under 5 minutes to recheck a cap after a deviation.
             Wow: the 3D soil block with the piles going into it is something people stop and look
             at; the "pile moved, cap rechecked in a second, three priced options" moment is what
             the engineer and the site will remember.
             Market (from what we know; to be checked before building): geotechnical tools such as
             GEO5, PLAXIS, Ensoft GROUP / LPILE and Allpile design piles but are paid desktop tools
             and do not make BIM, BBS or drawings. Digital pile-logging apps exist for rig
             telemetry on large foreign sites (paid, tied to the rig maker). Revit and Tekla model
             piles but do not design them or recheck a cap from the as-built position. We do not
             know a free or browser tool that goes from the bore log PDF to the pile design, the
             BIM model, the site log and the cap recheck in one place, to Indian codes.
             Trending: "digital twin of the ground", "AI reads the soil report" and "paperless
             site records" posts get a lot of reach on LinkedIn; a short reel of the soil block
             and piles is the kind of BIM post that is shared.
description: What it does, step by step:
               1. Soil: read the bore logs from the geotech PDF. Text PDFs: table extraction
                  (pdf.js text positions, columns found by their x positions). Scanned PDFs: the
                  same OCR path Plan Tracer would use, on the device. The engineer sees each bore
                  log as a table next to the PDF page and fixes any cell before it is used (no
                  silent guessing). Excel bore logs are also read (SheetJS, already in Structura).
               2. Ground model: bore holes placed at their coordinates (from the report's site
                  plan, or typed). Layers joined between bore holes by matching soil type, with a
                  simple interpolation (triangulated surfaces per layer top). Where two bore holes
                  disagree, the block is shown hatched: "uncertain here, ask for a bore hole".
               3. Loads: ETABS reactions per joint and per load combination (etabs-mcp), or typed
                  per column, or from cad2bim's own quick load takedown (the Load Path X-Ray idea).
               4. Capacity: IS 2911-1-2 for bored cast-in-situ piles (static formula; skin
                  friction per layer, end bearing, rock socket as the code and the report allow),
                  uplift, lateral (IS 2911 Appendix C fixed-head approach), group efficiency and
                  the block failure check for clay. Every number lists its formula, its inputs and
                  its clause. The geotech's recommended capacity, when the report gives one, is
                  shown beside ours, and the lower governs unless the engineer says otherwise.
               5. Layout: for each column (or wall, or group of close columns) choose the pile
                  count and pattern (2, 3, 4, 5, 6, 8, 9 ... from a standard library) so that the
                  worst pile is within its safe load under every combination; keep 3D spacing and
                  the edge distance; merge caps that overlap; keep piles off the lift pit and the
                  sump (reads them from the model).
               6. Caps: depth from one-way shear, punching and the bending (IS 456; the truss
                  method for deep caps), rebar both ways, pile embedment, the column starter bars.
                  Tie beams where a cap has one or two piles.
               7. Outputs: IFC (IfcPile with PredefinedType BORED, IfcFooting PILE_CAP,
                  IfcReinforcingBar), DXF/PDF layout plan and details, Excel pile schedule, BBS,
                  BOQ (boring per metre by soil type, rock socket per metre, concrete with
                  over-break, steel, chipping to cut-off, disposal of muck, integrity and load
                  tests), set-out points.
               8. Site log: per pile, a status and a form (bored depth, rock level, socket, cage,
                  concrete poured, time, as-built centre, photo). Works offline on the phone and
                  syncs when the page is open again. Nothing leaves the device unless the team
                  shares the job.
               9. Revise: when an as-built position or a founding level is entered, recheck that
                  cap and, if it fails, offer priced options. Every revision is kept with who,
                  when and why, and the drawing gets a revision cloud.
implementation:
             Where it lives:
               - New package `packages/piles` (TypeScript, pure functions: soil model, capacity,
                 layout, cap design). No GPL. The geometry (triangulated layer surfaces, pile
                 meshes) uses three.js, already in the engine. The PDF reading uses pdf.js
                 (Apache-2.0); OCR, when it is needed, the same on-device engine as the PDF path.
               - The web app gets a "Piles" dock panel and a "Soil" layer in the viewer (section
                 box and transparency already exist).
               - etabs-mcp: one new tool to write point springs back (axial and two lateral
                 stiffnesses per pile group); reactions are already read in 0.6.0.
               - CH v2: `CH-Pile` and `CHT-Pile` rows in the template (a change to the template
                 standard, so it goes through docs/design/ch-template-v2.md first).
               - Revit bridge: "Place piles" command; the pile status as a shared parameter so a
                 view filter colours the piles.
             Steps (each one useful on its own):
               v0.1  Typed bore logs + typed loads -> capacity per pile with the working, pile
                     count per column, Excel out. (Proves the engineering; 2-3 weeks.)
               v0.2  3D soil block from the bore holes, piles and caps in 3D, colour by
                     utilisation, IFC out. (The wow; 2 weeks.)
               v0.3  ETABS reactions in, pile-cap design, DXF/PDF drawings, BBS and BOQ.
                     (3 weeks.)
               v0.4  Site log on the phone, as-built position, cap recheck with priced options,
                     revisions. (3 weeks.)
               v0.5  Bore logs read from the geotech PDF; springs back to ETABS; Revit command.
             Checks:
               - Unit tests: capacity against three worked examples (one sand, one clay, one rock
                 socket) from a textbook or a past job, checked by hand by a structural engineer;
                 group layout against hand calculations for 2 to 9 piles with biaxial moments.
               - A past job (with the owner's OK, kept outside git): compare Pile Pilot's pile
                 counts and cap sizes with the issued drawings; write down every difference and
                 why.
               - The output always says "design aid: to be checked and signed by the structural
                 engineer and confirmed by pile load tests". Pile Pilot never replaces the load
                 test or the geotech's advice.
             Risks:
               - Bore log PDFs vary a lot: start with typed and Excel input, add PDF reading
                 report by report, always with the engineer's check.
               - Soil between bore holes is a guess: show the uncertainty (hatched), never a
                 confident smooth surface where the data is thin.
               - Codes and local practice differ (some offices use only the geotech's capacity):
                 make the capacity source a clear choice.
future update:
               - Driven precast and micro piles; secant and contiguous pile walls for basements
                 (joins Dig Deep), with the shoring in 3D.
               - Raft-pile foundations (piled raft) with ETABS or a simple plate model.
               - Pile integrity and load test results read from the test reports and pinned to
                 the pile; a pile with a bad result goes red and its cap is rechecked without it.
               - Rig telemetry (depth, torque) from rigs that export it, so the log fills itself.
               - Eurocode 7 / ACI and other countries' codes as separate modules.
               - A "foundation options" view: isolated footings vs raft vs piles for the same
                 loads and soil, with cost, time and carbon (joins Carbon Lens) side by side.
               - Gaussian-splat or photo capture of the piling site after each day, shown over
                 the 3D pile layout (joins Scan Check).
```
