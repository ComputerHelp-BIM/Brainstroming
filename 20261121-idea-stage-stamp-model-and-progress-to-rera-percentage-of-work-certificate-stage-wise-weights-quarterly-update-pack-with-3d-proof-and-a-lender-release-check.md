# 20261121 Idea: Stage Stamp — The 3D model and the site progress become the RERA percentage-of-work certificate: every stage (excavation, foundation, plinth, each slab, masonry, plaster, MEP, finishes) weighted as the state form asks, a quarter-on-quarter table, a 3D "this is what we certified" picture, and a check before the lender or the authority asks questions.

## Entry 1

```
date-time:   2026-11-21 (scheduled run, UTC)
idea name:   Stage Stamp (model + progress to RERA architect's percentage-of-work certificate and
             quarterly update pack, with 3D proof; in the browser, nothing uploaded)
potential:   High for architects, PMC and developer-side BIM teams in India; medium-high for the
             toolkit, because every residential project under RERA repeats this paperwork every
             quarter, per building or wing.
             The pain: the percentage of work is typed into a Form by hand from site visits and an
             Excel sheet. Two people give two numbers. The number is signed by the architect, filed
             with the authority, and used by the bank to release money, so a wrong or unprovable
             figure is a risk for the signer.
             Time saved: NOT measured.
             First test: take 2 real projects (kept out of git). For one past quarter, build the
             certificate table by hand with a stopwatch; then with Stage Stamp. Compare (a) minutes,
             (b) the percentage, (c) which stages the hand sheet could not back with evidence.
             Do not quote hours or accuracy until this is done.
             Market check (2026-11-21), two standard web searches:
               - Found: state RERAs ask for a quarterly update with an architect's certificate of
                 physical percentage of completion (Karnataka, Gujarat, Haryana, Bihar forms seen).
                 UP RERA is reported to make digitally signed project certificates mandatory from
                 September 2026 (news site, not checked against the circular). Maharashtra: the
                 regulations name Form 1 as the architect's certificate for fund release, Form 3 the
                 CA's, Form 4 the architect's completion certificate (a casemine copy of the
                 regulations; check the official gazette). A taxguru article says MahaRERA issued
                 over 9,177 show-cause notices over missing or wrong quarterly reports; that is an
                 advisory site quoting figures, I did not verify it.
               - Found (weak): a Bihar form lists plinth, stilt, and each superstructure slab as
                 separate lines with a percentage each; Haryana's lists sub-structure items.
               - NOT found: how each state weights the stages, whether a regulator accepts a
                 model-derived percentage, or any tool that builds the table from a BIM model with 3D
                 evidence. "I did not find one", not "none exists". Compliance suites and generic
                 RERA filing services exist and are paid; I did not check their features.
             Trend: RERA enforcement and digital certificates are live news in Indian real estate
             (search above). I could not measure LinkedIn or Instagram counts, so no trend is claimed.
             Extends tools already in this repo (does not repeat them):
               - Bill Painter (paint 3D progress to RA bill): same painted progress, but a different
                 output (a contractor's money bill by rate). Stage Stamp outputs a regulator's
                 percentage by stage weight, per building/wing, and keeps quarter history.
               - Area Statement Pro: RERA carpet area. This is the other half of RERA paperwork:
                 progress, not area.
               - Cash Curve and Delay Detective: money and delay from 4D. Stage Stamp adds the
                 regulatory snapshot and compares it with the plan.
               - Second Eyes: its signed-review pattern is reused for the architect's sign-off.
               - Photo Placer and Handover Hub: photos and as-built data feed the evidence page.
             The wow: open the model, press "Stamp quarter". Painted progress (done / part / not
             started) turns into a stage table: "Tower B: excavation 100 %, foundation 100 %, plinth
             100 %, slabs 7 of 14, masonry 20 %..." with the weighted total for the Form. Drag the
             quarter slider and the 3D shows what was certified in each past quarter, in a colour per
             quarter. A red flag says "Stage 'plinth' certified 100 % in Q2 but 3 columns are painted
             not done". One click makes the table, the proof sheet and an Excel the architect pastes
             into the state form.
```

### Description

**The pain.**
- The percentage is a signed statement. It is built from memory, site visits and an Excel sheet.
- Each state form has its own stage list and the weights are the architect's and the developer's
  call. Nobody keeps a record of why 40 % was chosen for the structure.
- A figure from last quarter cannot be shown against this quarter in one picture, so a jump or a
  fall (percentage going down because a slab was re-done) gets questions late.
- The lender's engineer and the authority see numbers, not the model. Arguments follow.

**The idea.** Four moves.
1. **Pick the form and the weights.** A small editable rule pack: the stage list of the chosen
   state form, each stage's weight, and how a stage is counted (by element count, volume, area or
   floors done). Starter packs are copied from public forms and marked "copy, check against the
   current form". Every output prints the pack name and version.
2. **Read progress from the model.** Done / partly done / not started per element, from the existing
   4D progress paint or a status column pulled in from Excel. Each stage's completion comes from its
   elements. A stage needing no model element (permits, utility connections) is typed.
3. **Make the certificate table and keep history.** Per building or wing: stage, weight, done %,
   contribution, total; the previous quarter's total; the change this quarter. Each quarter is saved
   as a frozen snapshot in the project file (versioned like other project data).
4. **Check before it is signed.** Warnings: a stage 100 % but some of its elements not done; a total
   that falls; a total that jumps more than the schedule allows; a stage certified before a stage
   that must come first (a slab above a slab not yet done). The tool never signs; it says "record,
   not certificate".

**What a user sees.**
- A "Stage Stamp" panel: building/wing picker, the stage table, a quarter slider, warnings.
- 3D coloured by the quarter in which each element was first certified; click a stage to light its
  elements; click an element to see which quarter counted it.
- A proof sheet (PDF) with the table, one 3D image per wing, and the list of elements per stage, and
  an Excel in the pack's column order for pasting into the state form or portal.

### Implementation

Inside the cad2bim app; nothing leaves the device. Nothing is filed with any authority.
- **Progress data:** the 4D timeline and element status (packages/engine, roadmap 0.119.0+); until
  that ships, an Excel status column read by the Sheet Sync reader, matched by GlobalId.
- **Stage grouping:** plain data rules over IFC class, level and building (IfcBuilding / zone):
  "IfcSlab on level N = slab stage N", "IfcFooting = foundation". Missing level data is listed as
  "not placed in a stage", never guessed.
- **Rule pack:** JSON (stages, weights, counting method, source URL and date). A pure function
  computes stage and total percentages; Vitest on small hand-checked projects, including the
  rounding rules each form states.
- **Snapshots:** each quarter stores the element statuses and the pack version; comparing two
  snapshots is a diff of two small tables.
- **Output:** table and proof sheet through existing export code (PDF, XLSX); 3D colours through the
  existing viewer colour overrides.
- **Agent access:** MCP tools in packages/mcp: "percentage of work for wing B", "what changed since
  last quarter", "which elements hold up stage X". An agent answers with the same engine.
- **Build order:** (1) pack format and the percentage function with Excel status in; (2) 3D colours
  and quarter snapshots; (3) warnings; (4) proof sheet and Excel out; (5) 4D progress link; (6) MCP.
- **Risks:** the stage weights and form layouts differ by state and change; the packs are copies and
  must show their source and date; a regulator may not accept a model-derived figure, so the output
  is evidence for the architect, who decides; thin progress data gives a thin table and the page says
  so. Check the current form with the project's architect and a lender's engineer before release.
- **Check before release:** 2 real projects, the stopwatch test above, and the signing architect's
  review of every warning for false positives.

### Future update

- **More states:** a pack library users can share inside a company; one pack per state, diffed when
  a state changes its form.
- **Photo evidence:** attach geotagged photos per stage using Photo Placer, in the proof sheet.
- **Lender release check:** compare the certified percentage with the loan's milestone table and say
  which tranche the quarter unlocks (a note for the developer, not a bank document).
- **Customer view:** a read-only Model Capsule for flat buyers showing the certified progress of
  their wing, if the developer chooses to share it.
- **Digital signature hand-off:** where a state portal accepts a digitally signed file, export the
  proof in the format the portal asks for; the architect signs outside the tool.
- **Portfolio board:** one view across a developer's projects: next filing date, last percentage,
  warnings open.

Sources used for the market check (not verified beyond these pages):
[Karnataka RERA QPR circular summary, TeamLease RegTech](https://teamleaseregtech.com/updates/article/57142/rera-karnataka-issued-circular-regarding-the-submission-of-quarterly-progress-reports-qprs-for-the-financial-year-2026-27-in-compliance-with-the-real-estate-regulation-and-development-act-2016/),
[UP RERA digital project certificates, The Realty Today](https://therealtytoday.com/news/news/up-rera-makes-digital-project-certificates-mandatory-from-september-2026-what-does-this-mean-for-developers-and-homebuyers/),
[Bihar RERA Form 7](https://rera.bihar.gov.in/images/Form-7RERA.pdf),
[Maharashtra RERA (General) Regulations 2017, casemine copy](https://www.casemine.com/act/in/5de8cab2eae8c7dd7fe2d274),
[IIHS policy brief, monitoring project progress by RERA](https://iihs.co.in/knowledge-gateway/wp-content/uploads/2023/06/Monitoring-of-Real-Estate-Project-Progress-by-RERA_Policy-Brief-2_27062023.pdf),
[Taxguru, RERA tech audit reality](https://taxguru.in/corporate-law/rera-tech-audit-reality-surviving-crackdown-forensic-reconciliation.html).
