# 20261026 Idea: Meeting Mind — the weekly BIM coordination meeting runs in cad2bim's viewer, the AI listens on the device, and every spoken decision ("shift this duct 150 up, MEP team, by Friday") becomes a BCF issue pinned to the exact element and camera that was on screen when it was said, with assignee, due date and a voice clip; the MoM writes itself, Revit jumps to each issue, and next week's meeting opens with a fly-through of only the open issues

## Entry 1

```
date-time:   2026-10-26 15:25 UTC (scheduled run)
idea name:   Meeting Mind — "Thursday, 4 pm, coordination call for the G+14 tower. The BIM
             coordinator shares cad2bim on the screen and presses Record meeting. They orbit to
             level 6, click the main duct that hits beam B6-14, and the MEP lead says: 'OK, we
             will drop the duct by 150 and reroute around the beam, Ravi's team, by Monday.' A
             small card slides in on the right: 'Issue #41 — Duct D-6-03 clashes beam B6-14 —
             action: drop 150 mm and reroute — owner: MEP (Ravi) — due: Mon 2 Nov — view saved —
             clip 0:00:18'. Nobody typed anything. Forty minutes and 23 issues later they press
             Stop. Out comes the MoM PDF (decision, owner, date, a picture of the exact view for
             each item), an Excel action list, and a .bcfzip. In Revit, Ravi opens the cad2bim
             button, sees his 9 issues, double-clicks #41 and Revit zooms to the same duct with
             the same section box. Next Thursday the meeting starts with 'Open issues tour': the
             camera flies from issue to issue, each one green if the model changed there and the
             clash is gone, red if not."
             One engine, four front ends:
               (1) cad2bim web app: a "Meeting" dock panel (record, live issue cards, MoM, tour);
                   speech to text runs on the device; issues are BCF 3.0 (the roadmap already
                   lists "Issues (BCF)" as planned; this builds it, with the meeting as the way
                   issues get made),
               (2) Revit (cad2bim bridge add-in + pyRevit AnonGee button): "My issues" list,
                   zoom to element / restore the BCF view, mark done with a comment, which goes
                   back into the same BCF file,
               (3) C2B / Structura / etabs-mcp: issues on a client DXF or an ETABS model are the
                   same BCF; for ETABS, "increase B6-14 to 300x600" can be pushed as a suggested
                   section change for the engineer to accept,
               (4) phone: the same panel as a PWA for site meetings: walk, point the camera at the
                   model on the phone, talk; issues land in the same list.
             Outputs: BCF 3.0 .bcfzip (topics, viewpoints with camera, selection, section box,
             snapshot), MoM PDF and Word-like doc, Excel action list (issue, element, discipline,
             owner, due, status), a per-owner WhatsApp / e-mail text summary, and a "what changed
             since last meeting" sheet.
potential:   Very high, because every project has this meeting every week and nobody enjoys the
             paperwork that follows it.
               - A typical mid-size project (G+10 to G+20, 4-6 disciplines) has one or two
                 coordination calls a week, 45-90 minutes each, 15-40 decisions per call
                 (estimated from how offices work, not measured).
               - After the call, the coordinator spends about 1.5-3 hours writing the MoM, taking
                 screenshots, making the issue list and sending it (estimated). Owners then spend
                 more time finding "which duct?" in their own Revit model.
               - Decisions said out loud and never written down are a known cause of rework: the
                 same clash comes back two weeks later because "nobody noted it".
             With Meeting Mind (target, to be measured in the pilot): MoM and issue list ready
             within 5 minutes of the call ending, about 2 hours saved per meeting for the
             coordinator, about 10-20 minutes saved per owner per week in finding issues, and
             every decision traceable to a voice clip and a view.
             Over a year: 2 meetings x 2 hours x 48 weeks = about 190 coordinator hours per
             project (estimate).
             Trend: "AI meeting notes" (Teams Copilot, Zoom AI Companion, Otter, Fireflies) and
             "AI agents in AEC" are among the most posted topics on LinkedIn right now, and
             BCF / openBIM coordination is a steady topic there. A screen recording where the
             speaker talks and issue cards appear on the model by themselves is a strong reel.
             Wow effect: you only talk; the issues pin themselves to the right element with the
             right camera; next week the model shows you, in green and red, which promises were
             kept.
             Market (to our knowledge, checked against what we know, not a full survey):
               - Revizto, BIMcollab Zoom/Nexus, Autodesk Construction Cloud (Issues), Trimble
                 Connect, Dalux, Newforma Konekt: strong BCF / issue tools, all paid per seat or
                 per project, and issues are typed by hand; some have AI features for search or
                 summaries, but none that we know links spoken words to the element and camera on
                 screen at that moment.
               - Generic AI note takers (Teams, Zoom, Otter, Fireflies): paid tiers, cloud
                 processing, and they know nothing about the model, so the notes say "this duct"
                 with no element.
               - Free BCF viewers exist (BIMcollab Zoom free tier, Bonsai/BlenderBIM's BCF, which
                 is GPL so cannot be reused here) but have no meeting capture.
             Nothing free, on device, model-aware and BCF-native that we know of.
```

### Description

**The pain (what happens today after every coordination call).**
- The call is on Teams / Zoom with someone sharing Navisworks, Revit or a viewer. Decisions are spoken fast:
  "drop it 150", "architect will shift the door", "keep the beam, MEP reroutes".
- One person tries to type notes during the call and misses half. Afterwards they replay the recording,
  take screenshots, write the MoM in Word, build the issue list in Excel and send both.
- The notes say "duct near grid C/5, level 6". The MEP modeller spends minutes finding which duct in their own
  model. The structural team never sees the item because it was filed under MEP.
- Next week nobody remembers what was closed. The same issue is discussed again; some are lost.
- BCF tools fix the "which element" part, but someone still has to type every issue during or after the call,
  and most of them are paid per seat.

**What Meeting Mind does.**
1. **Record with context, not just audio.** While recording, the viewer logs a timeline: camera, selected
   elements (IFC GlobalId), section box, visible levels, and the element under the pointer, every second (a few
   kB a minute). Audio is transcribed on the device, sentence by sentence, with timestamps.
2. **Find decisions.** A small language model (on device by default; a cloud model only if the office turns it
   on for a project) reads each window of transcript and pulls out action items: what, who, by when, which
   discipline, and the words that point at things ("this duct", "that beam", "the door on grid C").
3. **Pin them to the model.** "This / that" is matched to what was selected or under the pointer at that
   second; names said out loud ("beam B6-14", "grid C5", "level 6", "toilet 3") are matched against the
   model's marks, grids, levels and rooms. If two candidates fit, the card shows both and the coordinator taps
   one. Nothing is pinned silently when the match is weak; the card says "not pinned".
4. **Live cards.** Each issue appears as a card during the meeting so people can correct it out loud ("no, not
   Ravi, Priya") or with one click. Owners come from a project team list (name, discipline, e-mail, phone).
5. **Write the outputs.** On Stop: BCF 3.0 topics with viewpoints and snapshots, the MoM (attendees, agenda
   heard, decisions, actions table with pictures), the Excel action list, and one short text per owner.
6. **Close the loop next week.** "Open issues tour" flies the camera from one open issue to the next. For each,
   it compares the elements of the issue in the new model version with last week's (moved, resized, deleted,
   clash still there or gone, using the existing clash / QA checks) and suggests a status: likely fixed, not
   touched, changed but still clashing. People confirm in the meeting; the confirmation is itself recorded.

**Daily use cases.**
1. *Weekly MEP / structure / architecture coordination* (the main one).
2. *Client design review*: the client says "make this window bigger, remove this wall"; each comment is pinned
   and becomes a change list with a view, ready for the fee / variation discussion.
3. *Site meeting with the phone*: the site engineer walks with the model on the phone and talks; issues are
   pinned and sent to the BIM office the same evening.
4. *Internal QA review*: the team lead reviews a junior's model aloud ("tag missing here, wrong family here")
   and the junior gets a pinned to-do list.
5. *Structural review with ETABS*: "increase this beam to 300x600" becomes an issue plus a suggested section
   change in etabs-mcp for the engineer to accept or reject.

**Languages.** English first. Hindi and English mixed in one sentence ("isko 150 neeche karo") is normal in
Indian meetings; on-device speech models handle it with lower accuracy (to be measured on real meeting audio
in the pilot). The live cards are how mistakes get fixed.

**Privacy.** Audio and transcript stay on the device unless the office turns sharing on. People in the call are
told it is being recorded (a banner and a spoken line at the start); recording needs every attendee's consent
under most office policies and laws. Voice clips can be switched off, keeping only the text.

### Implementation

**1. Speech to text on the device.**
- Whisper models (OpenAI's weights are MIT) run in the browser: either whisper.cpp compiled to WebAssembly (MIT)
  or ONNX models through transformers.js (Apache-2.0) on WebGPU, falling back to WASM. Licences fit
  cad2bim's list. Models are served by cad2bim itself and pinned by SHA-256, like Python is today.
- Size: "base" or "small" multilingual (roughly 75-250 MB to download once, cached), to be chosen by measuring
  speed and accuracy on a normal office laptop. Voice activity detection cuts silence so it keeps up live.
- Audio source: the microphone, or the meeting app's audio through screen-share capture with audio
  (`getDisplayMedia` with audio works in Chrome for a tab or the whole screen on Windows).
- Speaker labels: optional; simplest version asks each person to say their name once, or uses "who is
  sharing" plus the team list. Good diarisation can come later.

**2. Context timeline.**
- A ring buffer in the viewer: every second (and on every selection or camera change) store camera position,
  target, selection GlobalIds, section box, hovered element. The current view link format (`viewLink.ts`,
  view token with camera, section box, selection and hidden elements) already serialises most of this, so a
  BCF viewpoint is a conversion of something that exists.

**3. Decision extraction.**
- A small instruction-tuned model on device (WebGPU; a 1-4 B model in 4-bit, licence checked to be on the
  allowed list before choosing) reads 30-60 s windows and returns JSON: `{action, owner, due, discipline,
  refs: ["this duct", "B6-14"], quote, t_start, t_end}`. JSON schema validated; anything invalid is dropped
  to a "maybe" list rather than guessed.
- Optional cloud model per project (off by default), only the transcript text, never the model file.
- Rule layer first for things a regex does better: dates ("by Monday", "next Thursday", "2nd"), sizes ("150",
  "300 by 600"), marks and grid names.

**4. Reference resolution.**
- Candidates: elements selected or hovered within ±5 s of the words, plus elements whose mark / name / grid /
  level / room matches a spoken name. Score by time distance and name match; the top one is pinned if it is
  clearly ahead, else the card asks. All matches by IFC GlobalId so they survive a re-export.

**5. BCF 3.0 in the engine.**
- Read and write `.bcfzip` (markup.bcf, viewpoint.bcfv, snapshot.png) in `@cad2bim/engine`; this is the
  roadmap's planned "Issues (BCF)" item (`roadmapItems.ts`, `exportTargets.ts` and `importSources.ts`
  already list it). Extra fields (voice clip offset, quote, meeting id) go into BCF labels and comments, so
  other BCF tools still open the file.

**6. Revit side.**
- The cad2bim Revit bridge (`revitBridge.ts`, `bridge/revit`) already keeps selection in step both ways; add
  "show issue" (select elements, set 3D view orientation and section box from the viewpoint) and "set status
  + comment". A pyRevit AnonGee button reads the same .bcfzip for offices not using the bridge.

**7. Next-week tour.**
- Compare issue elements between two model versions by GlobalId (geometry box moved, size changed, deleted)
  and re-run the clash check for the pair; suggestion only, people confirm.

**MVP (3-4 weeks).**
1. BCF 3.0 read / write with viewpoints in the engine and a simple issue panel (useful even without voice).
2. Record: microphone + on-device Whisper, transcript with timestamps, the context timeline.
3. Extraction with rules + a small model; pinning by selection time only (names later); live cards with edit.
4. MoM PDF + Excel action list. Pilot on 3 real internal coordination meetings; measure: share of decisions
   caught, share pinned to the right element, minutes the coordinator spent fixing, versus their usual time.

### Future update

- **Pinning by spoken names** (marks, grids, rooms) and by pointing on the phone camera (AR) for site meetings.
- **Promise tracker:** a dashboard of issues by owner and age; late items are coloured on the model, so the
  model itself shows who is behind.
- **Clash-to-agenda:** before the meeting, pick the top clashes (grouped, by level) as the agenda and the tour
  path, so the meeting starts with a plan (links to the IDS QA autofix and Sleeve Punch ideas).
- **Change Radar link:** "what changed since last meeting" uses the Change Radar idea's revision diff and BOQ
  delta, so cost impact of each decision is in the MoM.
- **Variation / RFI pack:** client-review issues become a numbered change list with BOQ delta (Bill Painter,
  Change Radar) for the variation order.
- **Model Reel link:** a 60 s video of the week's decisions, issue by issue, for the project WhatsApp group.
- **Agent follow-up:** through the cad2bim agent helper (MCP), "do the easy ones": e.g. lower a duct by 150 mm
  in Revit as a proposed change for the owner to accept, never applied silently.
- **Meeting search:** "when did we decide the lift pit depth?" jumps to the clip and the view.

### Improvement suggestions for existing tools in the repos

- **cad2bim:** build BCF 3.0 now (it is already on the roadmap and in the import / export lists); make the
  view token and the BCF viewpoint one shared type, so links, notes and issues all restore the same way.
- **cad2bim:** add a small "element mark index" (mark, name, grid intersection, level, room for every element)
  in the engine; Meeting Mind needs it, and so do search, the agent helper and QA reports.
- **pyRevit-addons (AnonGee):** a "BCF issues" pane (list, zoom to, status) that works without the bridge.
- **etabs-mcp:** an endpoint "suggest section change" that records a proposed change with a reason, for
  the engineer to accept, so issues from meetings can point at analytical members.
- **C2B:** export its Excel review workbook comments as BCF topics too, so client DXF review comments join the
  same issue list.
- **Structura / PlanWin AI Pro:** read and show BCF topics on their 3D views, so every tool in the kit shares
  one issue list.
