# 20261127 Idea: Pack Cutter — model to subcontract work packages with no gaps and no overlaps, scope sheets, an RFQ pack, and a "who owns this?" check

## Entry 1

```
date-time:   2026-11-27 (scheduled run)
idea name:   Pack Cutter — "Cut the model into subcontract packages. Find the scope nobody owns before the contract is signed."
             Pick the trades (excavation, PCC, rebar, formwork, concrete, masonry, plaster, waterproofing, MEP
             sleeves, and so on) or load your company's package list. Pack Cutter assigns every element in the 3D
             model, and every line in the BOQ, to exactly one package. It then shows what falls in a gap ("sunk
             slab waterproofing: no package") or in an overlap ("shuttering for the lift core: in two packages"),
             and writes a scope sheet and an RFQ pack for each package, with quantities that click through to the
             3D proof.
potential:   High. Splitting a job into subcontract packages is done in Excel, by hand, from the BOQ, by a
             planning engineer or QS, usually after the tender, under time pressure. The cost of a mistake is
             large and arrives late: a gap becomes a variation order priced by the one contractor who is
             already on site; an overlap is paid twice. The numbers below are my estimates, not measured:
             - Today: 2 to 5 days to split a tower BOQ into packages and write inclusions/exclusions, plus a
               rework round when the first bidder asks "is this in my scope?".
             - With Pack Cutter: first cut in minutes from the model, review in an hour, RFQ pack after every
               revision in seconds.
             Why it wins: every project needs it, nobody owns a tool for it, and the "gap" and "overlap" list is a
             visual hook (grey elements = "no owner", striped = "two owners") that a QS can screenshot into a
             tender meeting.
             Market: to my knowledge (not verified for this run), package splitting lives in paid estimating or
             ERP suites, or in a spreadsheet. Model-driven scope packages with a gap and overlap check exist
             at best as a paid add-on of enterprise 5D tools. A free, in-browser tool that works on IFC and
             keeps files on the device is not one I know of.
             Trending: "scope gap" and "variation order" stories are common on LinkedIn construction-claims
             posts; "AI for tendering" is a repeated topic. A before/after reel (grey "no owner" elements
             turning into coloured packages) fits the feed. Treat this as a signal of interest, not a measured trend.
```

### Description

**The pain (daily life of a BIM modeller, QS and planning engineer).**
- The model is full of quantities, but contracts are written by package: "rebar fixing", "shuttering and
  de-shuttering", "RMC supply and pumping", "blockwork", "waterproofing". The BOQ and the model are cut by element
  type or level, not by package.
- The split is done in Excel by tagging BOQ lines by hand. When a revision comes, the tags are lost or the new
  elements have no tag. Nobody notices until a bidder asks.
- Typical gaps: curing, chipping of cold joints, sunk-slab waterproofing, core-cutting, starter bars at pour
  breaks, blockwork around columns, temporary works. Each belongs to someone; the contract often says nobody.
- Typical overlaps: shuttering for shear walls in both the concrete and the formwork package; plaster in both the
  masonry and the finishing package.
- The RFQ goes out as a PDF and an Excel with quantities the bidder cannot check against the model.

**What Pack Cutter does, in four screens.**

1. **Packages.** A list of packages (a starter set for an RCC building; the user's own set saved per company).
   Each package is a small set of rules in plain words: "all `IfcBeam` and `IfcSlab` of grade M25 and above, levels
   below ±0", "rebar of all elements in zone B", "formwork of vertical elements only". Rules use the model's
   own properties (type, level, zone, grade, the BOQ line) and can be edited or typed in English ("put all
   basement waterproofing in one package").
2. **Coverage map.** The 3D model coloured by package. Three special colours: grey = no package (a gap), striped =
   more than one (an overlap), and amber = "ambiguous" (the rule matched, but a reviewer should look, for
   example, an element on a package boundary). A counter shows "98.4 % of BOQ value assigned; 37 gap items;
   12 overlap items". Clicking a number zooms to the elements.
3. **Interface and activity gaps.** Not every scope item is an element. Pack Cutter holds a checklist of
   "activities between packages" (curing, joint preparation, chipping, anchor bolts, sleeves cast in, testing
   cubes, dewatering, hoisting, scaffolding, site clearance after the pour). For each one it asks *which
   package owns it?* and flags the unowned ones. The list is the user's own and grows with each project.
4. **Outputs.**
   - **Scope sheet per package** (PDF and Excel): quantities by unit (m³, m², kg, nos) with the rule behind each,
     inclusions, exclusions, interfaces ("by others: …"), assumptions, and the drawing and model revision.
   - **RFQ pack** per package: the scope sheet, a priced-blank BOQ for the bidder to fill, a model view link or a
     self-contained 3D file (see *Model Capsule*) with only that package lit, and the commercial terms the user
     keeps as their own template.
   - **Package register**: one table, all packages, quantity, estimated value (from *Rate Rocket* rates when
     present), planned start (from the 4D when present), and status (draft, sent, bids in, awarded).
   - **BCF issues** for the remaining gaps and overlaps, so the QS and the BIM modeller work from one list.

**Revision proof.** When a new model revision comes (an IFC, or a rebuilt DXF), Pack Cutter re-runs the rules and
shows *new elements with no package*, *elements that moved package*, and *quantity change per package* with the
rupee or unit delta. The RFQ already sent can be re-issued as an addendum listing exactly what changed
(*Change Radar* supplies the diff).

### Implementation

Fits the existing stack: browser only, IFC read by web-ifc in a worker, one model for every derived view, files stay
on the device, no new server, no GPL code. This reads and writes through the one cad2bim API and never touches
source files of the app from outside.

1. **Package rules (`packages/engine`, new `packs/` module).**
   - A rule is `{id, name, match, exclude, unit, interfaces[]}`. `match` is a small predicate over the same
     properties the filters and colour-by already use (IFC class, type name, level, parameter, material, grade,
     BOQ line, rebar host). No new query language: reuse the filter engine.
   - Rules are evaluated in order, per element and per BOQ line. The result is a set of package ids per item.
     Zero = gap, more than one = overlap, one = clean. Pure TypeScript, tested on small synthetic models with known answers.
2. **Rule from English.** The existing plain-English command layer (0.81.0) gets a `pack` verb; with the user's own
   key, the AI chat can propose rules from a sentence and show the elements matched before anything is saved. The
   user always confirms. Without a key, the rule editor is a form.
3. **Coverage map.** A colour-by mode ("Package") in the viewer, with the grey and striped special cases drawn from
   the same material overrides colour-by uses today. The counters come from one pass over the model's element table.
4. **Activity checklist.** A JSON list of activities and who owns them, edited in a simple table. Starter list
   shipped as editable data, marked "starter, confirm with your own contract practice".
5. **Outputs.** Reuse the BOQ and BBS XLSX/CSV writers and the print-to-PDF path. The RFQ 3D view reuses the saved
   views and the one-file offline viewer idea. Register stored in the `.c2b` project file so it travels with the model.
6. **Revision.** Re-run on open; compare with the stored last result by GUID and by BOQ line; show deltas.
7. **Agent tool.** Add `package_coverage`, `package_scope` and `assign_package` to the MCP catalogue
   (`packages/mcp/src/tools.ts`) so an agent can answer "what is not in any package?".
8. **Testing.** Unit tests for gap, overlap and revision cases; a golden scope sheet; two or three real models in a
   scratch folder, never in git.
9. **Delivery order (small steps).**
   - Step 1: rules + coverage counters + the grey/striped map. About a week.
   - Step 2: scope sheet per package (Excel and PDF). About a week.
   - Step 3: activity checklist + interface gaps. A few days.
   - Step 4: RFQ pack + register. About a week.
   - Step 5: revision deltas and addendum. A few days.
   These are my estimates of effort, not measured.

**Risks and honest limits.**
- The tool finds what the model and the rules show. Scope that exists only in the specification or in the contract
  conditions needs the activity list; say so on every sheet ("model scope only").
- Wrong rules give confident wrong packages. Show the matched elements before saving a rule, and always print the
  rule beside each quantity.
- Not legal advice. The scope sheet is a draft for the QS and the contracts team to approve.
- Models with missing grades, levels or zones give more "ambiguous" items; the tool says so and leads to the fix first
  (a link to *Tag Tamer* territory).

### Future update

- **Learn from the last project:** import the company's awarded package list and exclusions; suggest the same cut and
  the exclusions that caused variation orders last time (links to *Repeat Offender*).
- **Bid return:** read the bidder's filled BOQ and match it to the model (links to *Bid Mirror*): quantity mismatch,
  missing lines, and "this bidder excluded curing".
- **Comparative statement:** side-by-side bids per package, with scope-adjusted totals so cheap-but-narrow bids are visible.
- **4D link:** each package gets start and finish from the schedule, and the register shows packages that must be
  awarded by a date to keep the programme (links to *Order Clock* and *Ready Board*).
- **Labour packages:** one click to turn a package into a per-unit work order (links to *Gang Pay*).
- **Variation guard:** during construction, any new element or change is tested against the packages and flagged
  as "inside package X" or "new scope", so a variation order starts with evidence.
- **Safety and permits:** carry the package's hazards and permits to the method statement (links to *Method Maker*).
- **Agent flow:** "split this tower into 12 packages and tell me what is missing", answered with 3D proof.
