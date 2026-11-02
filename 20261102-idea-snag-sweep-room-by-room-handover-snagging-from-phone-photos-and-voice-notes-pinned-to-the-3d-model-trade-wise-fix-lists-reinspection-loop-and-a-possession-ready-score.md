# 20261102 Idea: Snag Sweep — walk the flats with a phone, say or snap each defect, and every snag lands pinned on the right room in the 3D model, grouped by trade, sent as a fix list, re-checked on a second walk, and rolled up into a "ready for possession" score

## Entry 1

```
date-time:   2026-11-02 15:44 UTC (scheduled run)
idea name:   Snag Sweep (Room-by-Room Handover Snagging, Pinned to the 3D Model, with Trade Fix Lists and a Possession-Ready Score)
potential:   High. Every project ends with snagging, and it is the stage where schedules slip and clients get
             angry. Today it is a paper checklist, a photo folder and a WhatsApp group; nobody can answer
             "how many snags are open in Tower B, and which trade is holding us up?" without a day of
             manual counting. Estimated saving (an estimate, not measured): 1 to 2 days per tower in
             collating and chasing, plus fewer repeat visits. It gives the BIM team a visible role in
             the last 10% of the project, where the model usually stops being used.
```

### Description

**The pain.**
- Possession and snagging are common topics in Indian real-estate and homebuyer content on LinkedIn and
  Instagram (buyers posting snag videos, independent snagging inspectors). Read this as a trend I saw in
  feeds, not a measured statistic.
- A site team lists defects in Excel or on paper ("B-1204 master toilet, tile hollow, wall corner"). Photos
  sit in a phone gallery. Which room? Which trade? Fixed or not? Everything is re-typed.
- The same snag is found twice by two people; some are never closed; the second walk starts from zero.
- The BIM model knows every room, level, finish and (with other ideas here) every trade's scope, but is
  not opened during snagging.

**What exists today (one thin search plan, to be re-checked before any "nothing like it" claim).**
- Paid snagging and field-issue apps exist (PlanRadar, Fieldwire, Procore, OpenSpace-style capture, and
  several local snagging apps). I did not verify their current prices or features here. They mostly pin to
  a 2D plan, not to the IFC model's rooms, and are subscription tools with files on their cloud.
- Free tools: spreadsheets and chat groups. Nothing I know of is free, in the browser, IFC-aware and keeps
  the photos on the device.
- In this repo: Audit Eye (structural cracks, existing buildings), Handover Hub (O&M and asset tags after
  handover), Site QR Pass (element cards), Meeting Mind (BCF from meetings). None does room-by-room
  finishing snagging with a re-inspection loop.

**The idea.** A **Snag** mode in cad2bim's web viewer, opened on a phone at site:
1. **Load the model (once, offline-capable).** The IFC opens from the device; rooms (IfcSpace) are listed
   by tower, floor and flat.
2. **Walk and capture.** Pick a room (or scan the room's QR from Site QR Pass). Tap a defect category
   (paint, tile, plumbing leak, door/window, electrical, carpentry, waterproofing, cleaning), snap a photo
   or hold-to-speak ("hollow tile near the door"). Speech-to-text and the category guess run on the device.
   Each snag gets room, level, trade, severity, photo, time and who.
3. **3D pin.** The snag pin sits on the room's element (wall, floor, door) in the 3D view, so the office
   sees where it is. Photos stay in the browser's local storage or a folder the user picked.
4. **Trade fix lists.** One tap makes a per-contractor list: grouped by trade and flat, photos attached,
   sorted by the best walking route. Export as PDF, Excel, or a WhatsApp-ready image set.
5. **Second walk.** The list reopens: Fixed? (photo of the fix), Rejected? (reason), New? Closed snags turn
   green; reopened ones turn red. A history shows how many walks a flat needed.
6. **Possession-Ready score.** Per flat: open snags by severity, a score, and a Ready / Not ready flag
   using the team's own thresholds. A tower heat map colours the 3D building by readiness.

**The wow 1: tower heat map on the 3D model.** Flats glow red to green as the walk progresses; a client or
director sees the whole handover state in one image, shareable as a still or the Model Reel video.

**The wow 2: "which trade is the bottleneck?"** A bar chart of open snags per contractor with ageing in
days, and a one-click polite chaser message per contractor listing exact rooms.

**The wow 3: repeat-defect finder.** If 60 flats show the same hollow-tile snag in the same bathroom type,
it flags a systemic cause (a method, a batch) instead of 60 separate fixes, and counts the cost.

### Implementation

Everything below is a plan; nothing was built or measured.

- **Place in the toolkit.** A new mode in `apps/web` (React) using the IFC viewer in `@cad2bim/engine`. No
  server: snags are stored locally (IndexedDB) and exported as a single project file. The CAD workspace
  is not involved.
- **Data model.** Snag = {id, guid of IfcSpace/element, trade, category, severity, status, photo refs, note,
  walk number, timestamps}. Exports map to **BCF** so any other BIM tool can read them (the repo already
  has a BCF path from the IDS/QA work).
- **Voice and categories.** Browser speech recognition where available, with a plain typed fallback. A
  small on-device keyword table guesses the trade. Check licences before bundling any model: MIT, Apache-2.0,
  BSD, ISC only (repo rule); no GPL.
- **Phone-first UI.** Large buttons, offline use, camera via the standard browser file input. Design uses
  the existing tokens and `packages/ui` components.
- **Rooms from IFC.** IfcSpace to flat grouping uses names/numbers, with a manual remap when the model has
  none. This depends on the model being modelled to room level; call this out in the UI.
- **Reports.** PDF fix lists and a readiness summary built in the browser (the repo already writes PDF/Excel
  for BOQ and BBS).
- **Privacy.** Photos of homes are personal data. They never leave the device unasked; sharing is an
  explicit export.
- **Risks.** Bad room names in models; phone storage size for photos (compress on capture); speech
  accuracy with Indian accents and mixed Hindi/English — keep typed edit one tap away.
- **First 2 weeks.** Room list + photo/voice snag + 3D pin + one trade PDF; test on one real tower's
  IFC, with a stopwatch against the team's Excel way. Record measured times in the next entry.

### Future update

- Re-inspection by QR scan: stand in a flat, scan, see only the open snags.
- Cost per snag from Rate Rocket rates and a contractor back-charge sheet.
- Link to Handover Hub: closed-snag flats flow straight into O&M packs and asset tags.
- Customer view: a read-only link showing their flat's snag status (no upload to others' servers unless
  the owner chooses to host).
- Learn from past projects: top-10 snag types by tower type, fed back to the site team before the next job.
- Photo "before / after" auto-pairing and a short Model Reel progress clip for the client.
