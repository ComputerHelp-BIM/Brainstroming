# 20261117 Idea: Jacket Judge — Give it the weak columns and beams (from an audit, a scan, or an ETABS demand/capacity list) and it compares RC jacket, FRP wrap and steel jacket for each member on capacity gained, cost, days and how much of the building must be vacated — then draws the chosen jacket in 3D with dowels, writes the BBS and BOQ, and gives the chip-drill-pour sequence.

## Entry 1

```
date-time:   2026-11-17 (scheduled run, UTC; second idea of the day)
idea name:   Jacket Judge (retrofit and strengthening planner for existing RC members, in the browser, nothing uploaded)
potential:   High, and a growing niche. Old RC buildings are being audited, extended or re-used (structural
             audit rules, redevelopment, change of use). The audit finds "C14, C15, B22 are weak"; then the
             real work starts: which fix, how thick, how many dowels, what does it cost, which flats must be
             vacated. That step is done by hand, in Excel, from vendor brochures. Nothing draws the result.
             Time saved: NOT measured. First test: take 3 past retrofit jobs (kept out of git), run the tool
             on the same weak-member list, and compare the scheme, the quantities and the hours taken.
             Do not quote hours or rupees until that is done.
             Market check (2026-11-17, two searches; not exhaustive, so "I did not find one", not "none"):
               - Sika CarboDur software: free, cloud-based, FRP confinement for rectangular and circular
                 columns with biaxial bending. One product's FRP only. No BIM link seen in the results.
               - Seismosoft FRP Designer: commercial, FRP columns and beams, calculation report.
               - Academic GUI (Assiut Univ.): FRP thickness from required drift, a research tool.
               - Papers compare jacketing, bracing and shear walls (2025, Zone IV soft-storey frames), but
                 on one building, not as a tool.
               - I did not find a tool that compares RC jacket, FRP and steel for each member on cost AND
                 downtime, then models the winner in 3D with dowels and writes the BBS and sequence.
               - I could not find 2026 retrofit-demand or scan-to-BIM adoption figures for India in these
                 searches. No demand number is claimed here.
             Extends tools already in this repo (does not repeat them):
               - Audit Eye: finds cracks and spalls and prices a repair. Jacket Judge starts where a member
                 is too weak, not just damaged, and designs the strengthening.
               - Scan-to-Frame: gives the as-is model. Jacket Judge strengthens members of that model.
               - Load Path X-Ray and Basis Book: supply demand and the stability story. Jacket Judge takes
                 demand/capacity ratios (DCR) as input.
               - Demo Map: plans retain/remove/new. Jacket Judge adds a fourth state, "strengthen".
               - Cover Cop, Bar Swap, Joint Judge: share the cover, bar and joint rules.
               - Detail Drop and Cage Manual: the jacket cage reuses their detailing and fixing manual.
             Trend: re-use and retrofit of existing buildings is a standing AEC news theme (embodied carbon
             saved by not rebuilding). I could not measure LinkedIn or Instagram counts; this is not a
             social-media claim.
             The wow: a weak column glows red in 3D. Press "Judge". Three ghost jackets grow around it —
             RC (+100 mm each face), FRP (2 layers), steel angles-and-battens — each with a card: "DCR 1.31
             -> 0.92 | Rs 38k | 6 days | 2 flats vacated." Drag the "must stay occupied" slider and the
             RC option greys out. Click one, and the jacket is modelled with dowels and ties, and the
             "chip - drill - clean - epoxy dowel - tie - shutter - pour" sequence appears as a 4D strip.
```

### Description

**The pain.**
- The audit gives a list of weak members. The fix is chosen by habit ("jacket everything 100 mm").
- Each option moves different numbers: capacity, member size (lost room space, door fit), cost, days, noise,
  and whether people can stay in the building. These are never compared in one place.
- The jacket is drawn in 2D, once, for a typical column; the real drawing, bar list and dowel count per
  member are done by hand, so quantities are guessed.
- Site gets one typical detail and improvises.

**The idea.** One panel, four steps.
1. **Input.** A list of members with demand/capacity ratio (from an ETABS result file, an Audit Eye finding,
   or typed), taken from the open IFC. Members are coloured by ratio.
2. **Options per member.** RC jacket, FRP wrap (columns: confinement; beams: flexure or shear strips), steel
   jacket, and "add a wall" flagged for the engineer. Each is sized by a transparent rule (shown on the
   card), not a black box.
3. **Compare.** Capacity after, size added, cost (Rate Rocket rates), days, noise and dust class, and
   "flats to vacate" (from the model's room data). Sliders: occupied floors, budget, max size growth,
   available vendors.
4. **Build.** The chosen scheme is modelled as new elements in the IFC (jacket, dowels, ties, wrap strips),
   with BBS, BOQ, a per-member sheet and the work sequence for 4D.

**Smart parts.**
- **Size growth vs fit.** Checks the grown column against doors, beams, walls and the plot line (Flush Fit's
  fit check) — "this jacket blocks the staircase door by 40 mm."
- **Beam-column joint.** Flags where a jacket stops at a joint with no way to continue (the classic retrofit
  mistake) and suggests continuing through the slab with core holes (Cut Check decides if they are safe).
- **Do not strengthen the wrong thing.** If strengthening columns pushes the weakness to the footing or a
  beam, it shows the next weakest member (a column stronger than its beam is a warning, not a win).
- **Same detail grouping.** Members with the same jacket are grouped into one type, so the sheet shows eight
  types, not eighty members (Mark Merge logic).
- **Honest status.** Every card says "preliminary — engineer to confirm", shows the code clause used, and
  never prints "safe".

### Implementation

- **Reads:** IFC members, DCR table (CSV / ETABS export / typed), rates, room data. All in the browser.
- **Rules:** first version limits itself to rectangular and circular RC columns, and beams for shear strips.
  Rules are from IS 15988 (seismic evaluation and strengthening of existing RC buildings), IS 456, IS 13920
  and ACI 440.2 for FRP; each rule is a short data file with the clause, so a reviewer can read and change it.
  I have not checked these clauses against the standards; a structural engineer must do that before release.
- **Geometry:** jacket as a swept profile around the member; dowels on a spacing grid; ties; FRP as strips with
  overlap. Written as new IFC elements with their own type names (Jacket-C14).
- **Outputs:** BBS rows, BOQ rows, one A3 PDF per jacket type, CSV of quantities, 4D tasks for the sequence.
- **Fits cad2bim:** web-ifc for reading, the viewer's colour overlay for DCR, `packages/engine` for the
  rules and BBS, MCP tool "strengthen_member" so an AI agent can run it on a list.
- **Privacy:** nothing leaves the device.

### Risks

- It is structural design aid, not a design. The engineer of record must confirm every scheme. Rules must be
  reviewed by a licensed engineer before the tool is released.
- Existing as-built data is uncertain (concrete grade, steel). The tool takes an "assumed" grade with a
  confidence tag, and the numbers carry it through.
- FRP and steel jackets need vendor data; vendors differ. Vendor data is an input, not assumed.
- Retrofit has hidden things (old pipes in the column, bad concrete). A site check list is part of the sheet.

### Future update

- **Phase 1 (about 3 weeks):** columns only, RC jacket and FRP wrap, DCR input, comparison cards, BOQ.
- **Phase 2:** beams (shear strips, soffit plates), steel jacket, dowel and tie detail in 3D, BBS.
- **Phase 3:** the sequence as a 4D strip, vacate-flats logic, fit check.
- **Phase 4:** joints, footings (enlargement), and "add a wall" with the Core Cage tool.
- **Phase 5:** link to Scan-to-Frame and Audit Eye so a phone walk becomes a retrofit scheme with a cost.
- **Phase 6:** learn from the past jobs' real quantities; a reel showing "red column to wrapped column."

### Sources (market check)

- Sika CarboDur software — https://www.sika.com/en/knowledge-hub/sika-carbodur-software-frp-calculation.html
- Seismosoft FRP Designer — https://seismosoft.com/products/frp-designer
- Assiut Univ. GUI for FRP details — https://icce2021.conferences.ekb.eg/article_1203.html
- BIS retrofitting training material — https://www.services.bis.gov.in/php/BIS_2.0/trainings/dashboardnew/uploads/pwd_b_retrofitting_of_existing_buildings_updated.docx
- Seismic evaluation and retrofit of RC structures — https://impressions.manipal.edu/open-access-archive/11766
