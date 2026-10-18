# 20261018 Idea: Chat-to-Model — drop in the site's WhatsApp group export, get every message, photo and "pour done" pinned to the right element in 3D, with an as-built timeline, DPR and RFI list

## Entry 1

```
date-time:   2026-10-18 (scheduled run)
idea name:   Chat-to-Model — "Your site WhatsApp group is already your project record. Link it to the model."
             Export the site group chat (WhatsApp "Export chat" zip: text + photos + voice notes), drop it into
             cad2bim / Structura. The tool reads messages like "C12 L3 shuttering ready", "B5-B9 slab L4 pour
             done 42 m3 6pm", "grid C/4 honeycomb at column, photo", "RFI: sleeve clash at B14", finds the element
             or zone (mark, grid, level, pour no.), pins the message + photo on it in 3D, and builds three outputs:
             (1) an as-built 4D timeline, (2) a Daily Progress Report (DPR), (3) an RFI / snag list (BCF).
potential:   Very high. On Indian (and most South-Asian / Middle-East) sites the real project record is a WhatsApp
             group, not the CDE. Photos, "pour done", "bar tying complete", RFIs and snags live there and are lost
             in 3,000 messages. The BIM modeller / planning engineer spends 30-60 min every day scrolling the chat to
             write the DPR and update the status; the model is never updated, so the 3D model is always behind the
             site. Chat-to-Model turns that unavoidable habit into data instead of asking site teams to learn a new
             app (adoption problem solved: the site changes nothing). 
             Saves ~45 min/day per project engineer (DPR + status), and finds old evidence in seconds ("show me all
             photos ever posted for column C12" when a quality dispute or a payment claim comes up).
             Trending angle for LinkedIn / Instagram: "WhatsApp to BIM", "your site group is a digital twin
             waiting to happen", "AI reads 3,000 site messages and paints the 3D model". A 15-second reel of a
             chat scrolling on the left while columns turn green on the right and a DPR PDF appears is instantly
             understood by engineers and owners, and it needs no explanation of IFC.
             Market: site-photo and progress apps exist (paid, per-user, cloud, need site staff to log in and tag
             each photo). Nothing known to us ingests the free-text WhatsApp export, links it to IFC elements
             offline in the browser, and returns DPR + 4D + BCF. (To be re-verified before launch.)
```

### Description

**The pain (daily life of a BIM modeller / planning engineer).**
- Site engineers post "slab pour L3 started 7am", "shuttering ready C4-C9", "rebar checking pending for B21" and
  photos in the group all day. Nobody types this into the model or the CDE.
- Every evening the engineer scrolls the chat and rewrites it into the DPR (Excel / Word) by hand, 30-60 min.
- Weekly: percentage-complete for billing, the 4D plan vs actual, and the "what did we promise the client" are all
  rebuilt from memory and chat scrolling.
- When a dispute arrives ("was the cover checked before that pour?"), the proof is a photo in a chat from 5 months
  ago. Search in WhatsApp is by text only, and the photo has no element name.
- RFIs raised in the chat ("sleeve clash at B14 - architect please confirm") get lost; they never become tracked issues.
- Asking site staff to use a new app for tagging fails: they will not. The chat is the habit, so use the chat.

**The idea.** A **Chat-to-Model** panel with five steps:

1. **Drop the export.** The WhatsApp "Export chat (with media)" zip, or the plain `_chat.txt`. Both Android and iOS
   formats, 12 h / 24 h clocks, English + Hinglish. Everything is parsed locally in the browser; nothing is uploaded.
2. **Understand each message.** A rule-based parser first (deterministic, no AI needed for 80% of lines), AI helper
   second for messy lines:
   - *Where*: element marks (C12, B5-B9, F3), ranges, grids ("C/4"), levels ("L3", "terrace", "plinth"), zones
     ("Block B", "wing 2"), pour numbers ("P-14"). Resolved against the model's marks / levels / grids.
   - *What*: activity (shuttering, bar tying, cover check, pour, deshuttering, curing, MEP sleeve, snag, RFI) from
     a small editable keyword list per language.
   - *How much / when*: quantity ("42 m3", "18 trucks"), time, who posted (role map set once: "Ravi = site engr").
   - Each match gets a **confidence** and a short reason; low confidence goes to a review queue instead of being
     guessed.
3. **Pin on the model.** Messages become pins on elements / zones; photos become thumbnails on the pin. Click an
   element -> the whole history list ("all messages and photos for C12, oldest first"). Click a message -> camera
   flies to the element. Unmatched messages ("rain stopped work", "cement short") stay as site-wide notes.
4. **Review queue (the trust step).** One screen: message, guessed element(s), activity, confidence; Accept / Edit /
   Reject; "apply to similar messages" learns the site's own words ("C-12" = C12, "first floor" = L1, "roof" = L9).
   Nothing changes status until accepted, except high-confidence lines if the user switches that on.
5. **Outputs.** From accepted entries:
   - **As-built status** per element (Not started / Formwork / Rebar / Cover OK / Poured / Cured / Stripped),
     colour-painted in 3D and written to a property set (`Pset_ComputerHelp_Status`) in the IFC.
   - **DPR (PDF + Excel + WhatsApp text)**: today's work by activity, quantities (m3 poured, kg steel tied), labour/
     material notes, issues, photo grid with element names. One tap to copy the text back to the group.
   - **4D timeline**: actual dates per element beside the planned sequence (Pour Planner link); slip in days.
   - **RFI / snag list**: lines tagged "RFI", "clash", "snag", "honeycomb" become BCF issues with the photo, element
     and viewpoint attached.
   - **Evidence search**: "photos of C12 before pour", "all rebar-check messages on L4 beams" in 2 seconds.

**Why it creates the wow effect.** The first run on a real 3-month chat is dramatic: 3,000 messages -> 400 matched
pins -> columns and slabs light up in the order they were built, with the real photos. Site people see their own
words and photos in the model and immediately understand the value, without any training.

### Implementation

Reuse what already exists; keep it browser-local and deterministic.

1. **Parser (pure TypeScript module, no DOM)** in `packages/engine/src/site-chat/`:
   - `parseWhatsAppExport(text)` -> `{ ts, author, body, attachments[] }`; handle the date formats
     (`dd/mm/yy, hh:mm am -`, `[dd/mm/yyyy, hh:mm:ss]`), multi-line messages, "<attached: IMG-001.jpg>", "(file attached)",
     system lines, deleted-message lines.
   - `extractMentions(body, modelIndex)`: regex + dictionary for marks, ranges (expand `B5-B9` using the model's
     mark list and grid order), grids, levels, zones, pour numbers; fuzzy-match with a distance limit and the model's
     own marks as the dictionary, so unknown names are rejected rather than hallucinated.
   - `classifyActivity(body, lang)`: keyword/phrase lists in JSON (English, Hindi-in-Roman, Marathi/Tamil optional),
     editable by the user and saved with the project.
2. **Model index**: build once from the loaded IFC (marks from `Name`/`Tag`, levels from `IfcBuildingStorey`,
   grids from `IfcGrid`, pour no. from the Pour Planner / Site QR Pass property). Same lookup as Sheet Sync and
   Bill Painter, so share the `ElementIndex` module.
3. **Optional AI helper** through the existing `@cad2bim/mcp` agent bridge: only for lines the rules marked
   "unsure" ("tower 2 first slab done, steel 6 t") and only returns structured JSON {where, activity, qty}; the
   quantities and the element resolution stay deterministic and must pass the same validators. Works with no AI at
   all (rules only) so there is no running cost and no privacy concern.
4. **Photos**: read from the zip, downscale to thumbnails in a worker, keep full images in IndexedDB; strip GPS EXIF
   by default (switch to keep if the site wants location proof). Voice notes: listed and linked to elements, with
   optional local speech-to-text later.
5. **Persistence**: `site-chat.json` (entries, accepted links, vocabulary, role map) next to the model; status goes to
   IFC property sets through the existing IFC writer; the log is append-only so a re-import of a longer chat only adds
   new lines (dedupe by timestamp + author + hash).
6. **Viewer**: pins and status painting reuse the overlay layer (three.js overlay in cad2bim, `overlays.js` in
   Structura); thumbnails in the Properties panel as a "History" tab.
7. **Outputs**: DPR builder reuses PDF (jsPDF is already in Structura) and Excel export code from BOQ / BBS; BCF
   issues reuse the IDS Gatekeeper BCF writer; 4D actuals feed the Pour Planner timeline.
8. **Privacy by design**: all processing local; chat contains phone numbers and personal talk. Show a "names/numbers
   hidden" toggle in reports, skip lines the user marks as private, never upload, and make the role map optional.
9. **Tests / fixtures**: synthetic chat files (Android + iOS, 24 h/12 h, Hinglish, wrong marks, ranges, duplicated
   exports) in `tools/fixtures/`; golden tests on parser output; confusion tests for "C1" vs "C12", "L1" vs "L10".

MVP (2 weeks): parse export, match marks / levels / ranges, review queue, pins + history tab, DPR text/PDF.
Then photo thumbnails and IFC status property (week 3), BCF RFI export and 4D actual-vs-plan (week 4), learning
vocabulary, evidence search, Structura read-only viewer link.

### Future update

- **WhatsApp bot / Telegram bot ingest** (opt-in): forward photos to a bot number so the log updates live, instead of
  exporting the chat weekly.
- **Voice-note to text** (local Whisper in the browser, Hindi/English) so "dictated" site updates are also linked.
- **Photo understanding**: detect "formwork / rebar / concrete / curing" from the photo to cross-check the message
  (flag "pour done" posted with a photo that shows bare rebar).
- **Bill Painter link**: accepted "completed" entries pre-paint the RA bill with quantities, the engineer only
  confirms; the chat becomes the audit proof attached to each billed item.
- **Scan Check / Site QR Pass link**: chat photos and QR inspection cards in one history for each element.
- **Weekly auto-summary for the owner** (one page, photos, planned vs actual, top 5 risks) as WhatsApp text + PDF.
- **Multi-site dashboard** for a contractor: late-activity heat map across projects from their chats.
- **Revit side**: pyRevit tool writes the same status / history to shared parameters and creates view-linked photo
  tags in the Revit model through the bridge.

### Market check

- Paid, per-user, cloud: site-photo and progress tools (360 photo walk apps, construction-management suites, BIM 360 /
  ACC issues, Procore, PlanRadar, and similar) need site staff to open an app and tag every photo to a location.
  In practice small and mid-size Indian sites do not do this; they keep using WhatsApp.
- Chat export analytics tools exist for generic text, but none that know BIM marks, grids, levels, pours, and return
  element-level status, DPR and BCF.
- A free, offline, browser-local **"WhatsApp export -> element-linked pins -> as-built status + DPR + RFI/BCF +
  evidence search"** workflow is not offered as far as we could tell (verify once more before launch). It extends
  existing pieces of this toolkit: IFC property writing and overlays in cad2bim, BOQ / PDF export in Structura,
  Pour Planner, Site QR Pass, Bill Painter, IDS Gatekeeper's BCF output.
