# 20261026 Idea: Carbon Lens — one click turns the BIM model into an embodied carbon heatmap: every column, slab, wall and bar shows its kg CO2e, a swap simulator tries low-carbon options (GGBS / fly ash / LC3 cement, AAC instead of red brick, leaner sections from ETABS, recycled or green steel) and shows carbon vs cost live, then writes the GRIHA / IGBC / LEED credit sheet and a one-page client carbon card

## Entry 1

```
date-time:   2026-10-26 (scheduled run)
idea name:   Carbon Lens — "The modeller opens the G+12 tower in cad2bim and presses Carbon. The model
             goes from grey to a heat colour: the raft and the ground-floor columns glow deep red, the
             upper slabs orange, the AAC partition walls pale green. A big number sits at the top:
             about 4,100 t CO2e, about 420 kg CO2e per m² of built-up area (indicative, with the
             default factors). A side panel says: concrete about 58 %, steel about 30 %, masonry about
             8 %, the rest about 4 %. The modeller drags three sliders: 'cement: OPC -> 30 % fly ash
             PPC', 'walls: red brick -> AAC', 'columns: ETABS sizes from the optimised run'. The model
             cools down while they drag. New total about 3,350 t (-18 %), cost +0.4 %. Press 'Client
             card' and a one-page PDF with the before / after 3D images, the % saving, the trees-
             equivalent line and the GRIHA / IGBC points it helps with comes out. All in the browser,
             nothing uploaded."
             One engine, four front ends:
               (1) cad2bim web app (IFC viewer + the CAD workspace's DXF -> 3D result): a new "Carbon"
                   colour mode in the viewer and a "Carbon" dock panel next to BOQ and QA; it reuses
                   the BOQ quantities and the BBS steel weights the engine already computes,
               (2) pyRevit button (AnonGee tab, Advance panel): run on the live Revit model, colour by
                   carbon with view overrides, write shared parameters (CL_Material, CL_kgCO2e,
                   CL_kgCO2e_per_m3, CL_Option) so schedules and tags show carbon next to quantity,
               (3) etabs-mcp / Structura / PlanWin AI Pro: pull the analytical sections and the
                   design steel (Ast) to get real rebar weights before any detailing exists, and run
                   "same building, two ETABS options" (e.g. M30 vs M40 columns, flat slab vs beam slab)
                   as a carbon + cost comparison,
               (4) C2B: carbon of a client's DXF on day one from the normalised JSON quantities, so
                   the first proposal already has a carbon number.
             Outputs: carbon heatmap view, a carbon BOQ (Excel: element, material, quantity, factor,
             kg CO2e, source of the factor), an option comparison table (carbon vs cost vs weight),
             a GRIHA / IGBC / LEED evidence sheet, a one-page client carbon card (PDF + image for
             LinkedIn / Instagram), and an IFC with carbon properties (Pset_EnvironmentalImpact-
             Indicators / a cad2bim Pset) for anyone downstream.
potential:   Very high, and rising every month. Embodied carbon is moving from "nice to have" to
             "asked for in the tender":
               - green rating systems (GRIHA, IGBC, LEED v4.1 / v5, EDGE) give points for reduced
                 embodied carbon / life-cycle assessment of the structure and envelope,
               - large developers and corporate tenants (data centres, offices, warehouses for
                 multinationals) now ask for an embodied carbon number per m² with their ESG
                 reporting (BRSR in India for listed companies),
               - the EU CBAM moves into its definitive phase in 2026 for steel and cement, and India's
                 Carbon Credit Trading Scheme covers cement and steel plants; "low-carbon cement" and
                 "green steel" are now product lines that suppliers market hard,
               - on LinkedIn, "embodied carbon", "net zero buildings", "LC3 cement", "green steel" and
                 "BIM + LCA" are among the most posted topics in AEC (heatmap and before / after
                 carbon posts get a lot of reach).
             Time today (one G+7 to G+15 building, estimated from how offices work, not timed):
               - export BOQ from Revit, map each line to a material, look up a factor, total it in
                 Excel: 1-2 days for a first LCA, and nobody trusts it after the model changes,
               - each option ("what if we use PPC / AAC / M40 columns?") is a new round in Excel:
                 0.5 day each, so usually only one option is shown,
               - consultant LCA reports for a rating: paid per project and take 1-3 weeks.
             With Carbon Lens: first result in under a minute; each option is a slider move; re-run
             after every model change. Expected saving 2-4 days per project for the BIM team, plus
             a new service the office can sell ("carbon-optimised structure" with every model).
             Wow effect: the model literally changes colour as you move a slider — red concrete
             cooling to green while the total drops — and the client gets a before / after card with
             their own building on it. That is a strong LinkedIn / Instagram post and a strong
             sales slide.
             Market: One Click LCA (paid, expensive per seat), Autodesk Forma embodied carbon and
             Revit's newer carbon analysis features (inside paid subscriptions, mainly early stage
             massing), Tally (Revit add-in, paid / limited), EC3 by Building Transparency (free,
             strong EPD database, but web-account based, mostly North American / European EPDs,
             not tied to a live IFC / DXF model with rebar), EDGE app (free, whole building level,
             not element by element), Cove.tool / Preoptima (paid). To our knowledge nothing free
             runs offline in the browser on an IFC / DXF / Revit model, includes the actual rebar
             from the BBS or ETABS, has Indian materials (PPC, PSC, LC3, AAC, fly ash bricks, Indian
             steel routes) and shows carbon vs cost live with a client card.
```

### Description

**The pain (daily life of a BIM modeller and the project lead).**
- The client or the green consultant asks: "What is the embodied carbon of the structure? Can we cut 20 %?"
  The BIM team has every quantity in the model, but turning it into carbon is a manual Excel job.
- Factors come from many places (ICE database, EPDs, India-specific studies) and units differ (per kg, per m³,
  per m², per tonne). Mistakes are easy and nobody writes down which factor came from where.
- Steel is a large share of carbon in an RCC building, but at the BOQ stage it is often a kg/m³ guess. The real
  rebar is in the BBS (cad2bim BBS) or in the ETABS design (etabs-mcp), not in the carbon sheet.
- Options are never compared properly: "PPC vs OPC", "AAC vs red brick", "M30 vs M40 columns", "flat slab vs
  beam slab" each take half a day, so the client sees one number, too late to change anything.
- Rating submissions (GRIHA, IGBC, LEED) need an evidence sheet: quantities, factors, sources, baseline vs
  proposed. It is rebuilt by hand for every project.

**What Carbon Lens does.**
1. Reads quantities the toolkit already has: concrete volume by element and grade, formwork area, masonry
   volume by block type, plaster and finishes, steel by bar diameter from the BBS (or kg/m³ from ETABS design
   Ast, or an office default, in that order, and it says which one it used).
2. Maps each element's material to a carbon factor from a factor library (a JSON / Excel table, versioned,
   every row with its source, year, scope A1-A3 / A4 / A5 and unit). The office fills and checks the values
   from the published sources before it ships; nothing is typed from memory.
3. Adds transport (A4) from a simple distance table (RMC plant, steel yard, block supplier, by city) and site
   waste (A5) as an editable % per material.
4. Colours the model by kg CO2e (total, per m³, or per m² of floor it supports) and shows the split by
   material, by level and by element type.
5. Runs options as "swaps" on top of the same model: a swap is a rule (e.g. "all M25-M40 concrete: OPC -> PPC
   30 % fly ash", "walls 230 red brick -> 200 AAC", "columns: use sizes and Ast from ETABS run B"). Each swap
   changes factors, quantities, or both, and also changes cost through the BOQ rates. Swaps can be combined.
6. Writes the outputs: carbon BOQ, option table, rating evidence sheet, client card, IFC properties.

Every number says what it is: "embodied carbon estimate, cradle to site (A1-A5), with the factors listed".
It is a design and comparison tool; a certified LCA for a rating can use its sheets as the input.

**Daily use cases.**
1. *Proposal stage*: from the client's DXF (C2B or the cad2bim DXF -> 3D), give a first carbon per m² with the
   fee proposal, plus "we can save about X % with these three swaps" — a differentiator in the pitch.
2. *Design meetings*: the structural engineer and architect move sliders together on the shared screen and
   agree on cement type, block type and grade zoning with carbon and cost in front of them.
3. *ETABS option compare*: two ETABS runs (etabs-mcp) side by side — beam-slab vs flat slab, M30 vs M40
   columns — with concrete, steel, carbon and cost for each.
4. *Rating submission*: export the GRIHA / IGBC / LEED evidence sheet with baseline and proposed, factor
   sources and quantities by element, instead of building it by hand.
5. *Model changes*: after every revision, Carbon Lens re-runs; the Change Radar idea's revision diff can show
   "this revision added 85 t CO2e (mostly the new transfer beams)".
6. *Marketing*: a monthly "carbon saved for our clients" post with real before / after models.

### Implementation

**1. Quantities (reuse, no new readers).**
- cad2bim: element quantities from `@cad2bim/engine` (the `ifc`, `bbs` and BOQ code), the DXF -> 3D output for
  drawings. Steel per element from the BBS when present.
- Revit: pyRevit collects material volumes per element (`GetMaterialVolume`) and rebar weights from rebar
  elements or the structural rebar schedule.
- ETABS: etabs-mcp's section, material and design results endpoints give member sizes, grades and required
  steel; the existing BOQ & 5D module there already turns them into quantities and costs.
- C2B: normalised JSON quantities (columns, beams, slabs, footings).

**2. Factor library (data, not code).**
- One table, `carbon-factors.json` with an Excel twin: material id, description, unit, A1-A3, A4 default,
  A5 waste %, density, source (database name, version, page / EPD id), region, year, notes.
- Starter set: concrete by grade and cement type (OPC, PPC fly ash, PSC slag, LC3), rebar by production
  route, structural steel, red brick, fly ash brick, AAC, solid / hollow concrete block, cement plaster,
  gypsum plaster, tiles, glass, aluminium. Values entered from the published sources (ICE database, Indian
  studies, supplier EPDs) by the engineer, checked by a second person.
- Concrete can also be built from its mix (cement, GGBS / fly ash, aggregates, water, admixture per m³) so an
  RMC supplier's mix design gives a project-specific factor.
- Supplier EPDs can be added as rows; the swap simulator shows them by name ("Supplier X green cement").

**3. Engine (TypeScript in a Web Worker).**
- `carbon(elements, factors, options) -> results`: pure function, unit-checked (kg, m³, m², t), each result
  keeps the factor row id so the report can cite it.
- Swaps are small rule objects: `{ match: { category, grade, material }, set: { factor | section | material } }`
  applied in order; combining is just a list. Cost uses the BOQ rate table already in the engine / etabs-mcp
  5D (INR default).
- Lives in `@cad2bim/engine` next to BOQ, so nothing leaves the device. The same function as a small Python
  module for pyRevit and C2B, with shared JSON test cases so the two stay in step.

**4. Viewer and panel.**
- three.js colour mode (same mechanism as QA colour-by-status): sequential palette by kg CO2e, legend, toggle
  total / per m³ / per m² supported. Smooth colour transition when a swap changes values.
- Panel: total, per m² of built-up area, donut by material, bar by level, option cards with carbon vs cost,
  "biggest 10 elements" list with zoom-to.
- Carbon vs cost chart: each option is a dot; the "best" frontier is drawn so the team sees which swaps give
  the most carbon for the least cost.

**5. Outputs.**
- Excel carbon BOQ and option table, rating evidence sheet (layout per rating system, filled from the same
  results), one-page PDF client card with before / after images, IFC with carbon properties per element.

**MVP (3 weeks).**
1. Factor library with 25-30 materials, filled from published sources and checked.
2. Carbon from cad2bim IFC quantities + BBS steel, heatmap and panel.
3. Three swaps: cement type, block type, concrete grade by element group; carbon and cost shown together.
4. Excel carbon BOQ and the client card PDF. Test on three past office models and compare with one
   consultant LCA if one exists for any of them (to see how close the numbers are).

### Future update

- **Auto-optimiser:** "cut 20 % at the lowest cost": try swap combinations (and ETABS section options through
  etabs-mcp) and propose the cheapest set that reaches the target.
- **Grade zoning helper:** suggest where higher grade concrete allows smaller columns (less concrete, less
  carbon) and where it does not pay, using ETABS axial ratios (links to the Load Path X-ray idea).
- **Rebar link:** use the Bar Optimizer idea's cutting-stock result to count real wastage instead of a flat %.
- **4D carbon:** with Pour Planner's sequence, show carbon "poured" week by week; with Model Reel, export a
  timelapse where the building grows and the carbon counter ticks up.
- **Operational carbon:** join with Sun Compass (RETV, daylight), Cool Load (HVAC) and Sun Roof (solar) ideas
  for a whole-life view: embodied + 30 years of operation − solar.
- **Supplier marketplace view:** compare RMC / cement / steel suppliers by their EPD and distance to site.
- **AI explain:** "why is level 2 so red?" answered from the results: "transfer beams B21-B26, 38 t CO2e, 9 %".
- **CBAM / export projects:** a sheet in the format foreign clients ask for steel and cement content.

### Improvement suggestions for existing tools in the repos

- **cad2bim:** keep a material id on every element in the engine's model (grade + cement type + block type), so
  BOQ, BBS, QA and carbon all read the same thing; today grade is in the mark or a property and block type
  is often only a layer name.
- **etabs-mcp:** add carbon to the existing BOQ & 5D module (one more column next to cost) with the same factor
  table; and an endpoint "quantities by option" for two loaded models.
- **pyRevit-addons (AnonGee):** a shared "colour by parameter" heatmap helper (also proposed for Load Path
  X-ray) so carbon, cost and load can all be shown in Revit the same way.
- **C2B:** add a carbon column to the Excel review workbook's quantity sheet, so a client DXF gets a carbon
  number on day one.
- **Structura:** a "colour by value" legend mode for any per-member number (carbon, utilisation, cost).
- **PlanWin AI Pro:** its AI assistant could answer "what if we use M30 instead of M25 in columns?" with steel,
  concrete, carbon and cost from its own take-down.
