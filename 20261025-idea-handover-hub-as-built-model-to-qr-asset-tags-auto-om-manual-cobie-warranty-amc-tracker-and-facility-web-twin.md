# 20261025 Idea: Handover Hub — turn the as-built Revit / IFC model into a facility-ready handover in one click: an asset register (COBie + Excel), a printed QR tag for every pump, DB, AHU, lift, valve and fire device, an auto-built O&M manual from the vendor datasheets, a warranty / AMC calendar, and a phone-friendly web twin where the facility team scans a tag, sees the asset in 3D and raises a service ticket

## Entry 1

```
date-time:   2026-10-25 (scheduled run)
idea name:   Handover Hub — "The building is done. Press Handover. Every maintainable asset in the model
             (pumps, DBs, panels, AHUs, FCUs, VRF indoor units, lifts, DG set, valves, sprinkler heads,
             detectors, fire extinguishers, water tanks, STP equipment, doors with access control) gets an
             asset ID, a QR sticker sheet ready to print, a row in the asset register, its datasheet / test
             certificate / warranty card pulled into a bookmarked O&M manual, and a place on the warranty
             and AMC calendar. The facility manager opens a link on their phone, scans the sticker on the
             pump in the basement, and sees where it is in 3D, what feeds it, its manual page, its warranty
             end date and its service history — and raises a ticket from the same screen."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, new "Handover" stack in the Essential panel): pick the
                   categories that count as assets, check that the required data is filled, fill the gaps
                   from Excel, number the assets, export the register, the QR sheet and the COBie file,
               (2) Excel / Sheet Sync (earlier idea): the MEP contractor fills make, model, serial number,
                   install date and warranty months in a locked template; the values flow back into the
                   model parameters,
               (3) Structura-style static web viewer ("Twin" mode): the IFC + register open in the browser,
                   assets coloured by warranty status (green = covered, amber = ends in 60 days, red =
                   expired), QR deep links open one asset, "what does this valve isolate?" highlights the
                   downstream system,
               (4) O&M builder: drops the vendor PDFs from a folder, matches them to asset types by make /
                   model (filename + text search, AI fallback), and binds one PDF manual with a cover,
                   index by system and floor, asset tables and the datasheets as bookmarked chapters.
             Outputs: asset register (.xlsx, one row per asset, one sheet per system), COBie 2.4 spreadsheet,
             IFC 4 with Pset_ManufacturerTypeInformation / Pset_Warranty / Pset_Asset filled, QR sticker
             sheet (A4 PDF, 24 / 40 labels per page, with asset ID, name, room, floor), O&M manual (PDF,
             bookmarked), warranty + AMC calendar (.ics + Excel), handover data-quality score per system,
             and a static web twin (folder that can be hosted on Vercel or opened from a USB stick).
potential:   Very high, and almost every project needs it. Handover is the last 5 % of a project that
             takes 20 % of the pain: clients, PMCs and FM companies (and government clients asking for BIM
             deliverables) now write "as-built model + asset data + O&M manuals" into the contract, and
             retention money is held until it is accepted.
             Time today (one G+12 residential / commercial tower, ~3,500 maintainable assets):
               - collecting make / model / serial / warranty from 8-12 subcontractors by email and
                 WhatsApp, then typing it into Excel: 2-3 weeks,
               - filling the same data into Revit parameters (if done at all): 4-6 days,
               - making QR or barcode labels in Word and sticking them by guesswork: 3-5 days,
               - binding the O&M manual (scan, rename, sort 400-900 PDFs, make an index): 1-2 weeks,
               - COBie export that fails the client's check and is fixed row by row: 3-5 days,
               - warranty dates tracked in someone's personal Excel and forgotten after a year.
             With Handover Hub: register, QR sheet, COBie and O&M manual in about 1 day once the vendor
             data is in the template; re-issue after a change in minutes.
             Expected saving: 4-6 weeks per project at handover, faster release of retention money, and
             fewer missed warranty claims (one missed pump or DG warranty claim can cost more than the
             whole handover effort).
             Wow effect: the model turns grey, then every asset lights up one by one with a QR icon floating
             over it; the register fills row by row beside it; a phone camera scans a printed sticker on a
             real pump and the browser flies to that pump in 3D, highlights the pipes it serves, and shows
             "Warranty ends in 43 days — book AMC?". That "scan a sticker, the building answers" moment is
             exactly the digital-twin / smart-building handover content that does well on LinkedIn and
             Instagram (ISO 19650-3, "golden thread of information", FM digital-twin reels).
             Market: Autodesk Tandem (paid, cloud, subscription per facility), Dalux FM, Plannerly,
             Archibus / Planon / IBM Maximo / Facilio (paid FM / CAFM suites, they expect the data to arrive
             clean — they do not build it from the model), the free Revit COBie Extension (exports a
             spreadsheet, does not collect vendor data, make labels, bind manuals or give a viewer). To our
             knowledge nothing free goes from a Revit / IFC model to vendor data collection -> asset IDs ->
             QR labels -> bound O&M manual -> COBie -> warranty calendar -> scan-to-3D web twin in one tool.
```

### Description

**The pain (daily life of a BIM modeller at handover).**
- The model is "as-built" in geometry, but the equipment in it is still a generic family named
  "Pump - 5 HP" with no make, model, serial number or install date.
- The client's BIM Execution Plan asks for COBie or an asset register with 20-40 fields per asset. The data
  sits with the plumbing, fire, HVAC, electrical, lift and DG vendors, in their own formats.
- The BIM team chases it by email, types it into Excel, and the Excel never goes back into the model. The
  model and the register disagree after the first change.
- The O&M manual is a stack of vendor PDFs bound in a hurry, without a link to the room or the floor where the
  equipment actually sits.
- Six months after handover, the facility team does not know which valve isolates which riser, or that the
  booster pump warranty expired last week.

**What Handover Hub does.** It treats each maintainable asset as one record — *what it is, where it is, what
system it belongs to, who made it, when it was installed, until when it is covered, and which documents
describe it* — and keeps that record in the model. Every handover output (register, COBie, IFC, labels,
manual, calendar, web twin) is generated from the record, so they never disagree.

**Daily use cases.**
1. *Residential tower handover*: pumps, tanks, DBs, lifts, fire devices and common-area fixtures registered,
   labelled and handed to the society or FM company with a web link instead of a cupboard of files.
2. *Hospital / data centre / IT park*: thousands of HVAC and electrical assets, strict PPM schedules; the
   register feeds the client's CAFM system directly (COBie / Excel import) instead of being retyped.
3. *Government / smart-city project*: the BIM deliverable check fails on missing fields; the data-quality score
   shows exactly which assets and which fields are missing, per subcontractor, before submission.
4. *Retrofit or existing building survey*: start from a scanned / traced model (Plan Tracer, Scan Check —
   earlier ideas), walk the building with a phone, scan or create tags, photograph name plates, and build the
   register as you go.
5. *During construction*: the same template collects material approvals and test certificates per asset, so
   handover is mostly done by the time the building is.

### Implementation

**1. Decide what an asset is.**
- Per-office YAML / Excel "asset rule set": which Revit categories and family types are assets (Mechanical
  Equipment, Electrical Equipment, Plumbing Fixtures above a threshold, Sprinklers, Fire Alarm Devices, Lighting
  Devices in common areas, Specialty Equipment, Doors with access hardware), which fields are required per class
  (e.g. pumps: make, model, serial, duty, kW, install date, warranty months, AMC vendor, isolating valve ID).
- Field set maps to COBie 2.4 (Facility, Floor, Space, Zone, Type, Component, System, Document, Attribute) and to
  IFC Psets so one fill feeds both.

**2. Number and locate (pyRevit, AnonGee tab).**
- Asset IDs from a pattern like `{Building}-{Level}-{System}-{Type}-{Seq}` (e.g. `T1-B1-PLB-PMP-003`), stable
  once issued (stored in a shared parameter; never renumbered after a change).
- Room / space, level, zone and grid reference resolved from the element location (room at point, nearest grid
  intersection) — reuses the location logic already used by tag / schedule tools in AnonGee.
- System membership from Revit MEP systems (piping / duct / electrical circuits) so "upstream / downstream" can be
  answered later.

**3. Collect vendor data (Excel round trip).**
- Generate one locked Excel per subcontractor with only their assets and only the editable fields, with
  dropdowns (make list, warranty months) and validation (dates, serial format).
- Import back: match on asset ID, write parameters, log every change (who / when / old / new) — the Sheet Sync
  engine (earlier idea).
- Data-quality score per system and per vendor: % of required fields filled, conflicts (same serial twice,
  install date after handover date), assets in the model with no vendor row and vice versa.

**4. Documents and O&M manual.**
- Vendor PDFs dropped in a folder; matched to asset types by make / model in the file name, then by text search
  in the PDF, with an AI classifier only for the leftovers (the person confirms each guess).
- Manual bound with a PDF library (pypdf / pdf-lib): cover, contents by system -> floor, one asset table per
  system, then the datasheets, test certificates and warranty cards as bookmarked chapters; each asset table row
  links to its chapter.

**5. Labels and calendar.**
- QR per asset encodes a short URL `…/twin#T1-B1-PLB-PMP-003` (works offline too: the ID is printed under the
  code). A4 label sheets for standard sticker paper; optional metal / weatherproof label CSV for vendors.
- Warranty end = install date + months; AMC / PPM intervals from the rule set; exported as `.ics` (one calendar
  per system) and as an Excel plan the FM team can print.

**6. Web twin (static, no server).**
- Same approach as Structura: static HTML + JS, three.js viewer, IFC loaded in the browser (web-ifc), register
  as JSON next to it. No database needed for v1; host on Vercel or open from a folder.
- Deep link from QR flies to the asset, isolates it, shows its card (data + document links + warranty status),
  and highlights its system upstream / downstream.
- "Raise ticket" in v1 = prefilled WhatsApp / email message with asset ID, location and photo; a proper ticket
  log comes later.

**Tech stack.** Python 3 core (rule sets, ID generator, COBie / Excel writers, PDF binding, `.ics`) shared by
the pyRevit button and a CLI; IFC 4 export with Psets via Revit's IFC exporter mapping file; static web viewer
in the Structura style (three.js, xlsx.js, jsPDF already vendored there); optional cad2bim web app hosting the
twin for teams that want accounts and sharing.

**MVP (4-5 weeks).**
1. Asset rule set + ID generator + room / level / system resolution in pyRevit for 4 classes (pumps, DBs,
   AHU / FCU, fire devices).
2. Excel round trip per vendor with validation and the data-quality score.
3. Asset register + COBie 2.4 + QR label PDF.
4. O&M manual binder (filename match only) with bookmarks.
5. Static web twin: load IFC + JSON, QR deep link, warranty colours. Demo on a sample building and record the
   "scan the sticker, the building answers" reel.

### Future update

- **Ticket and PPM log:** a small backend (cad2bim accounts) so the FM team logs services, readings and photos
  per asset; history shows on the asset card; overdue PPM turns the asset red in 3D.
- **IoT readings:** pull BMS / energy-meter values (BACnet / Modbus gateway, MQTT) onto the asset card — the
  step from "digital handover" to a live digital twin.
- **Phone survey mode:** photograph a name plate, OCR the make / model / serial into the register, and create
  the asset in the model if it is missing (retrofit and existing buildings).
- **Spare parts and consumables:** filters, belts, fuses per asset type, with reorder quantity across the
  building.
- **ISO 19650-3 / client EIR checker:** load the client's information requirements (or an IDS file — IDS QA
  idea) and score the handover against them before submission.
- **Golden-thread change log:** every change after handover (replaced pump, new DB) recorded against the asset
  with who / when / why — useful for fire-safety and insurance audits.
- **CAFM connectors:** direct import to common FM systems instead of spreadsheet hand-off.

### Improvement suggestions for existing tools in the repos

- **pyRevit-addons (AnonGee):** a shared "stable ID" utility (pattern-based, never renumbered, stored in a shared
  parameter) would serve Handover Hub, Site QR Pass and Bolt Up assembly marks alike; and the existing
  FilterParameter / Parameter Combine tools could gain a "required fields missing" filter mode per category.
- **Structura:** its static, no-upload, three.js + xlsx.js architecture is already the right base for the web
  twin; a generic "element card" panel (properties + document links) and URL hash deep links (`#id`) would make
  it reusable for any QR-to-3D idea.
- **cad2bim:** add COBie 2.4 (spreadsheet) and IFC Pset_ManufacturerTypeInformation / Pset_Warranty to the
  export roadmap, so models built on the web carry asset data from day one.
- **C2B:** the profiles / templates approach could ship a "handover profile" that pre-fills asset classes and
  required fields per building type (residential, hospital, office).
