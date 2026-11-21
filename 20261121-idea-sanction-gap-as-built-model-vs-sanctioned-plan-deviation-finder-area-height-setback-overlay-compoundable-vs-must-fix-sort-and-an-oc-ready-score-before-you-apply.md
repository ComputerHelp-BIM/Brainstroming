# 20261121 Idea: Sanction Gap — Lay the as-built model over the sanctioned plan, find every deviation (extra area, height, setback, balcony, staircase headroom), sort them into "compoundable" and "must fix", and get an OC-ready score before you apply for the occupancy certificate.

## Entry 1

```
date-time:   2026-11-21 (scheduled run, UTC)
idea name:   Sanction Gap (as-built vs sanctioned plan: deviation finder + OC-ready score)
potential:   High for builders, PMC, architects and the BIM teams that serve them in India. Medium-high
             for the toolkit: it reuses the DXF-to-3D pipeline, the area statement and the overlay that
             are already planned, and it opens a use that is paid for in lakhs, not in hours.
             The pain: at the end of a job the building is rarely exactly the sanctioned drawing. A
             balcony grew, a staircase head room was lifted, a set-back wall moved, a mumty was added,
             a stilt got a room. The municipality compares the as-built drawing with the sanctioned
             plan at the occupancy certificate (OC) stage, and what the builder finds out then is
             expensive: compounding fee, a demolition notice, or a delayed OC. Today the check is a
             person flipping two PDFs. Nobody finds it while the slab is still on the drawing board.
             Why it matters in money (from the news search below, to be verified per city): deviation
             limits are thin (Bengaluru has treated more than 5 % as a violation, with a move to 15 %
             reported; Hyderabad compounds up to 10 %), registration and bank loans are tied to the
             sanctioned built-up area, and consumer courts have started ordering compensation when
             possession is given without an OC and beyond the sanctioned plan.
             Time saved: NOT measured. First test below.
             First test: take 2 finished projects with a sanctioned plan and an as-built set (kept out
             of git). A senior planner marks the deviations by hand. Run Sanction Gap. Report: found,
             missed, false alarms, and the minutes each way. Do not quote accuracy before this.
             Market check (2026-11-21), two standard web searches:
               - Found: OC rules and fee pages (HMDA, Odisha OBPS, BBMP); news that Karnataka is
                 moving from a 5 % to a 15 % deviation allowance (one source's body text did not
                 match its headline, so check the final notification); an NCDRC order of Aug 2026
                 reported by LiveLaw, a builder ordered to pay Rs 6 crore for possession without OC
                 and beyond the sanctioned plan; Tirupati consumer case where buyers found the
                 deviation by comparing plans after the fact.
               - Found: scan-to-BIM deviation tools (Hexagon Deviation Analysis, Avvir, Leica
                 Cyclone 3DR). They compare a scan with the design model for quality. They are paid
                 and I did not see them check FAR, set-backs or compounding limits.
               - Found: Inkers "Observance AI" and automated plan-approval services, which compare
                 as-built with approved design and set-backs. Closest to this idea. I did not see
                 a per-clause compoundable / must-fix sort, an OC-ready score, or a browser tool
                 where the file stays on the device. Paid suites I did not check feature by feature.
               - NOT found: a tool that takes a DXF/DWG/IFC as-built, overlays it on the sanctioned
                 drawing, and returns area, height and set-back deviations with the municipal limit
                 next to each. "I did not find one", not "none exists".
             Trend: OC delays and the deviation allowance are live news in Karnataka and Telangana
             (above). I could not measure LinkedIn or Instagram counts, so no trend there is claimed.
             Extends tools already in this repo and in the idea list (does not repeat them):
               - Area Statement Pro: model -> FSI / carpet-area sheet for APPROVAL. Sanction Gap is
                 the same sheet at OC: approved column vs as-built column, difference per line.
               - Twin Check, Scan Check: design vs BIM, and point cloud vs IFC for quality
                 tolerances (mm). Sanction Gap compares against the LEGAL drawing, in percent
                 and square metres, with the rule that applies.
               - Plan Tracer, Layer Whisperer, Rev Merge: they bring a drawing in cleanly. Sanction
                 Gap uses them for the sanctioned PDF/DXF and the as-built DXF.
               - Gap Guard, Access Pass, Exit Map: they check one rule. Sanction Gap does not redesign;
                 it reports only what changed against the sanction, then points at those tools.
               - Handover Hub, Stage Stamp: they use the finished model. Sanction Gap is the gate
                 before them.
             The wow: drop two files (sanctioned plan, as-built model). A ghost overlay shows the
             sanctioned outline in blue on the 3D model. Everything outside it glows: the 600 mm
             balcony that grew to 900, the extra room on the stilt, the terrace head room, the
             pent house. Top right a gauge reads "OC-ready 71 %": 9 deviations, 6 inside the
             allowance (compoundable, fee estimate), 2 over it (fix), 1 not compoundable (must
             remove). Drag the balcony back and the gauge moves live.
```

### Description

**The pain.**
- The sanctioned plan is a 2D PDF or DXF that was issued months or years before. The as-built model
  or drawing has gone through site changes, client requests and consultant revisions.
- No one reads the two against each other until the OC application, when the corporation does.
- Each deviation has a different rule: area (FSI or built-up), height, set-back, projection,
  staircase and lift headroom, parking, open space. Some can be compounded within a limit, some
  cannot. The builder learns which only from the notice.
- Late discovery is costly: compounding fees, security deposit forfeiture, delay in possession,
  buyers' compensation claims, and registration and loan limits tied to the permitted built-up area.

**The idea.** Five moves, all in the browser, files stay on the device.
1. **Two inputs.** The sanctioned plan (DXF/DWG, or a PDF through Plan Tracer) and the as-built (the
   cad2bim model, an IFC, or the as-built DXF). The user marks the plot corner and one known
   dimension on each; the tool aligns them and shows a residual error. If it cannot align well it
   says so and asks for another reference, it does not guess.
2. **Overlay and measure.** Per floor, the sanctioned outline is laid over the as-built outline.
   The tool measures: floor area, built-up area, FSI used, height of each block, set-backs from
   the plot line, balcony and chajja projections, staircase and mumty areas, stilt use.
3. **Rule sheet per authority (editable).** A small table per city: "deviation allowed up to X %",
   "set-back minimum Y m", "projection max Z m", "not compoundable: no sanction, residential-to-commercial
   change, set-back below N". Starter values are copied from the public notifications and marked
   "check the current notification"; every output prints the rule set, its source and its date. The
   user's own architect or liaison edits it.
4. **Sort.** Each deviation becomes one line: where (floor, grid), what, sanctioned value, as-built
   value, limit, status. Status is one of: inside the allowance (compoundable, estimated fee from
   the city's schedule, if loaded), over the allowance (fix before applying), not compoundable
   (remove), or "cannot tell, check the drawing". The OC-ready score is the share of the as-built area
   that is clear, with the open items listed first.
5. **Fix and re-score.** "Pull balcony B2 back 300 mm" moves it in the model and re-scores. The
   output is an as-built vs sanctioned statement (Excel + PDF), a deviation drawing (DXF with
   clouds, one layer), and a short letter list for the liaison. A signature block is left for the
   architect; the tool does not certify.

**What a user sees.**
- A "Sanction Gap" panel: two file slots, the alignment check, the city rule sheet, the gauge, a
  table of deviations sorted by status.
- A 3D view with the sanctioned outline as a blue ghost, and deviations glowing amber or red.
- Click a row: the camera goes there, with the sanctioned and the as-built value on a tag.

**Who gains.**
- BIM modeller: stops hand-comparing two PDFs for two days before submission.
- Site engineer and planner: sees at the next pour that a change will cost at OC, not after.
- Builder and PMC: finds the fee and the delay in the weekly meeting, not in the notice.
- Architect: signs a statement they can defend.

### Implementation

Built on what cad2bim already has:
- **Reading:** the DXF/DWG reader and the Rust pipeline give polylines, layers and levels from the
  sanctioned drawing. Plan Tracer (idea) covers a PDF. The as-built is the 3D model already built
  from the drawings, or an IFC read by the engine.
- **Alignment:** two-point + one-distance alignment, then a residual in mm. Same maths as Grid Lock
  (idea) uses for federated models; reuse it.
- **Measure:** floor outlines from slabs and walls (IfcSlab, IfcWall, IfcSpace where present); area by
  polygon union, set-back by distance to the plot polygon, height by top of the highest element
  per block. Balcony and chajja are found by their IFC type or the layer map in the CH template.
- **Rules:** a JSON rule sheet per authority (same idea as the IDS and Spec Sage rules, so the
  engine's existing check machinery is reused). Each rule has a clause text, a source link and a date.
- **Output:** the BOQ and area-statement writers already exist; add the two-column statement, a DXF with
  a "DEVIATION" layer (the DXF writer is in the repo) and a PDF.
- **Privacy:** nothing is uploaded. If a city rule sheet is fetched, it is a static file served by
  cad2bim itself.

Order of work: (1) alignment and overlay for one floor, with area only; (2) set-back, height and
projection; (3) rule sheet for 2 cities and the sort; (4) the score, fix-and-re-score, and the
statement; (5) the first test on 2 finished projects, and the numbers go in the README.

Risks, written plainly:
- Rules change often and differ by city and by year. The tool must show the rule's source and date,
  and never claim legal advice.
- A bad alignment gives false deviations. The residual is shown and a poor one blocks the report.
- A sanctioned drawing may be a scan or have no scale. Then it stops and says so.
- It finds geometry deviations only. Use changes, structural changes and fire changes need a person.

### Future update

- Run it on every revision during construction: a "gap since sanction" line next to each Change Radar
  report, so a change that creates a deviation is flagged the day it is drawn.
- Fee estimator per authority once a city's schedule is loaded and checked by the user's liaison.
- Revised-plan helper: when the deviation is large, produce the revised plan drawing set (through
  Drawing Autopilot) to apply for revalidation, with the changes cloud-marked.
- Photo / scan mode: use Scan Check's point cloud as the as-built when no model exists.
- Buyer-side mode (a separate, later product question): a flat buyer uploads the sanctioned plan and the
  builder's handover plan and gets the same gap list. Not for now; it changes who the user is.
- RERA link: the carpet area in the agreement vs the as-built carpet area (Stage Stamp and Area
  Statement Pro share the area engine).

### Sources (for the market check)

- [GBA / BBMP deviation allowance, ConstructionWorld](https://www.constructionworld.in/policy-updates-and-economic-news/gba-to-issue-occupancy-certificates-for-up-to-15-per-cent-deviation/91950)
- [Bengaluru 5 % to 15 % deviation, PropNewsTime](https://propnewstime.com/getdetailsStories/Mjg1NzQ=/bengaluru-moves-to-allow-up-to-15-deviation-in-building-plans)
- [NCDRC order, possession without OC, LiveLaw](https://www.livelaw.in/consumer-cases/failure-to-obtain-occupation-certificate-ncdrc-orders-builder-to-pay-6-crore-compensation-547237)
- [HMDA OC procedure](https://hmda.gov.in/procedure-for-occupancy-certificate)
- [Hexagon deviation analysis](https://multivistaservices.hexagon.com/service/deviation-analysis/)
- [Inkers compliance check](https://inkers.ai/services/compliance-check)
