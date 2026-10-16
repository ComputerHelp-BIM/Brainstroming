# 20261016 Idea: Site QR Pass — scan the bar tag or drawing, see that exact element in 3D on the phone, tick the inspection, and the model updates

## Entry 1

```
date-time:   2026-10-16 (scheduled run)
idea name:   Site QR Pass — a QR code on every drawing, bar-bending tag and pour card that opens that exact
             element (or pour, or bar mark) highlighted in a lightweight 3D viewer on the site phone, with a
             pre-pour checklist, photo proof and progress status that flows back into the model (4D / 5D)
potential:   High. The modeller's output dies at the printout. On site the same questions are asked all day:
             "which column is this?", "what is the bar mark 14 shape and where does it go?", "is this the latest
             revision?", "has the cover been checked?". Each one is a phone call to the BIM office. Site QR Pass
             answers them in 3 seconds at the point of work, with no app install, no login and no server.
             It closes the loop the existing ideas leave open: Drawing Autopilot, Bar Optimizer and Pour Planner
             all produce paper/PDF; this makes the paper "alive". Paid equivalents (Procore, Autodesk Build,
             Trimble Connect field apps, Dalux) need licences, accounts and cloud upload of the model; a free
             static, privacy-first version does not exist for small and mid-size Indian/Asian contractors.
             Strong LinkedIn/Instagram trend fit: "scan-to-see BIM on site", "paperless site", "QR-coded
             rebar tags", "BIM for the foreman, not just the engineer" — a 20-second phone-scan reel sells itself.
```

### Description

**The pain (daily life of a BIM modeller / site coordinator).**
- Printed drawings and BBS sheets go to site; the modeller then gets WhatsApp photos asking "this column, which
  floor?" or "bar mark 27 — which beam?".
- Revisions: the foreman is still working from Rev B while the office is on Rev D. Nobody notices until the
  concrete is poured.
- Inspections (cover blocks, bar spacing, shuttering level, pre-pour check) live on paper or in WhatsApp and are
  never tied to the model, so 4D progress and "what was checked" are re-typed by hand from photos.
- Heavy field BIM apps exist, but they need paid seats, logins, cloud model upload and good connectivity. The
  site foreman will not use them; he will scan a QR code.

**The idea.** A **Site QR Pass** export in cad2bim (browser, files stay on the PC). One button, "Make site
pack", produces:

1. **QR codes, stamped on the outputs the team already creates**
   - on each **drawing sheet** (Drawing Autopilot DXF/PDF title block): opens that level/plan, with the
     revision and a green "LATEST" or red "SUPERSEDED — Rev D exists" banner;
   - on each **BBS cut tag** and bar-mark row (from the BBS Excel / Bar Optimizer): opens the bar mark's shape,
     length, count and the *member(s) it goes into*, highlighted in 3D;
   - on each **pour card** (Pour Planner): opens the pour, its concrete volume, grade, the pre-pour checklist.
   - on **member labels** (print sheet of stickers): column C-B3-L4 and so on, for one-scan identification.
2. **A tiny, self-contained "site viewer" page** (static HTML + a stripped glTF/IFC subset, ~1–5 MB for a whole
   building after quantisation and per-storey tiling) that:
   - opens at the scanned element, camera flown to it, element highlighted, neighbours ghosted;
   - shows size, grade, level, bar schedule, concrete volume, revision and date;
   - works **offline after first load** (PWA: service worker + cached tiles) — site connectivity is bad;
   - has a **section/clip slider** and "show only this storey / only this pour".
3. **Checklist and proof in the same page**: configurable pre-pour / formwork / rebar-cover checklist per element
   type, a camera button to attach a photo, a stamped name and time. Status colours the element in the viewer
   (grey = not started, amber = in progress, blue = rebar checked, green = poured).
4. **Feedback to the model without a server.** Statuses and photos are saved on the phone and sent back one of
   three ways, so no backend is required: (a) a share-sheet "Send to BIM office" that produces a small
   `site-status.json` + photos (WhatsApp, email), (b) a Google-Form / Apps-Script endpoint URL set once in the
   pack, (c) later, the optional cad2bim cloud. The office drops the JSON into cad2bim and the status becomes a
   **progress layer** and drives the **4D timeline** (actual vs planned) and **5D** (earned value) automatically.

**Why it is unique / extensible.** It reuses what exists in the repos:
- cad2bim already loads IFC in a worker, picks elements and shows selections (`packages/engine`), exports
  quantities and BBS to Excel, and has 4D timeline and 5D cost on its roadmap — Site QR Pass is the field-facing
  input for exactly those milestones (actual dates and statuses on elements).
- Structura already has the BBS/BOQ and cutting tags (`boq.js`, `rebar.js`, `exporters.js`) and a three.js viewer
  (`viewer.js`) that can be reused for the stripped site viewer.
- It turns the other brainstorm ideas from "files" into a **workflow**: Drawing Autopilot (sheets) +
  Bar Optimizer (tags) + Pour Planner (pours) + Change Radar (what changed) all get a QR that points to the live
  truth. Revision-superseded banners come directly from Change Radar's revision hash.

**What makes it a "wow".** The demo is 15 seconds: print a drawing, scan with the phone, the 3D column appears and
the camera flies to it; tick "cover checked", take a photo; back at the PC the model column turns green and the
4D Gantt bar advances. No install, no login, no cloud upload of the model.

### Implementation

Stages that can ship independently, each useful on its own:

1. **Stable element IDs and a URL scheme (1 week).** Use IFC `GlobalId` for members, and deterministic IDs for
   bar marks and pours. URL like `.../site/#/e/<guid>?rev=D` — the hash fragment is never sent to a server,
   which keeps the privacy promise. Revision = short hash of the model's element table (shared with Change Radar).
2. **QR generation (1 week).** Client-side QR (`qrcode-generator`, MIT, tiny) to SVG so it stays vector in
   DXF/PDF. Add `QR` layer in the DXF writer; add QR images to the jsPDF output and to the Excel tag sheet
   (image or `HYPERLINK` formula as fallback). Sticker sheet export as PDF (Avery-style grid).
3. **Site pack exporter (2 weeks).** From the loaded IFC build: (a) a per-storey glTF with Draco/meshopt
   compression, simplified rebar as instanced line meshes (only bar centrelines + diameter, not full geometry);
   (b) a JSON index `{guid -> props, storey, pour, barMarks[]}`; (c) the static viewer bundle. Package as one ZIP
   to drop on any static host (Vercel/GitHub Pages/Netlify/a school-lab web server) or open from a phone via
   a local network. Size budget enforced with a warning ("Tower pack = 4.2 MB, OK on 3G").
4. **Site viewer (2–3 weeks).** Small three.js app (reuse engine picking and highlight code), URL-hash driven
   focus, clip plane, storey filter, PWA with service worker and tile caching, Hindi/English UI toggle (same
   spirit as other tools), large-tap-target UI for gloves and sunlight (high-contrast "Ink" theme from tokens).
5. **Checklists and proof (1–2 weeks).** Checklist templates in JSON (pre-pour, rebar, formwork, cover, curing),
   stored in IndexedDB; `<input type=file capture>` for the photo; resize photos to ~300 KB; export bundle
   `site-status.json` + `photos/`. Signed with name/time (no PII beyond the name typed).
6. **Import back into cad2bim (1 week).** "Import site status" reads the JSON, matches by GUID, writes a
   `Status`/`ActualDate` property on elements (cad2bim already edits properties and saves to IFC), colours the
   progress layer, and feeds the 4D timeline (planned vs actual) and the 5D earned-value roll-up.

Risks and mitigations: model leakage (QR points to a static host the user controls; optional password-protected
ZIP; everything stays client-side), outdated paper (revision banner), big models (tiling plus size budget),
poor light/damaged print (QR level H error correction, large module size, short URLs).

### Future update

- **Live sync**: optional lightweight backend (Supabase / Cloudflare D1) so several foremen update the same
  model in near real time, with offline queue and conflict rules.
- **AR overlay** (WebXR) to see the rebar cage or column in place at one-to-one scale on the phone.
- **Material tracking**: scan a delivery tag, mark the bar-mark lot "received" and compare with the Bar Optimizer
  purchase plan; auto-flag shortages before a pour.
- **Cube-test and RMC records**: attach batch-ticket photos and cube results to the pour; OCR the ticket
  (volume, slump, time) to a record.
- **BCF issues from the field**: foreman raises an issue with photo and pin, exported as BCF (the IDS Gatekeeper
  issue flow already uses BCF).
- **NFC tags and voice notes** for sites where QR is unreadable (dirty, wet); WhatsApp-bot ingest of photos.
- **Dashboards**: percent complete per storey/zone, "what is ready to pour tomorrow", weekly progress PDF for the
  client, auto-generated with one click (LinkedIn-ready progress video from the 4D playback).
