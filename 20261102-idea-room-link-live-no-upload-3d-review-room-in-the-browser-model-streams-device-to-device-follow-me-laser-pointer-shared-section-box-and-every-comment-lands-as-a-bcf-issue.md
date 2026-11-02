# 20261102 Idea: Room Link — a live 3D review room in the browser: the model goes straight from your device to the other people's devices (no upload, no account, no install), everyone sees what you see, and every comment becomes a BCF issue

## Entry 1

```
date-time:   2026-11-02 (scheduled run)
idea name:   Room Link — "The client is on a phone in a car, the structural consultant is on a laptop in
             another city, the architect is in the office. The BIM lead opens the tower in cad2bim and
             presses Open room. A link and a QR code appear. Each person taps it: no sign-in, no app, no
             plug-in. Within seconds the model loads on their screen, sent straight from the lead's
             machine (not through a server, not stored anywhere). The lead orbits to the level 6 transfer
             beam: everyone's camera follows (Follow me). The consultant switches to Free look, drags a
             section box through the beam, clicks it and types 'check cover here'. The lead sees the
             consultant's cursor as a coloured dot with a name, and a laser pointer when they hold a key.
             Every comment is pinned on the element and saved as a BCF issue with the view. Press End
             room: the guests' copies vanish from memory, the lead exports the issue list (.bcfzip + a
             one-page PDF with pictures), and nothing about the model is left on any server."
             Runs in the cad2bim web app first (the existing viewer and view link), then as a 'Start room
             from Revit' button in the add-in (exports the live model through the bridge into a room).
potential:   High, and it fits what cad2bim already promises: "files open from the person's disk and stay on
             their device". Every BIM team has the same call every week: someone shares a screen over
             Teams / Zoom / Meet, the picture is compressed and laggy, only one person can move the model,
             the client cannot point at the thing they mean, and afterwards someone must write down what was
             said. The alternatives are paid cloud tools that first need the model uploaded to a server
             and an account for every guest. Large developers, government clients and some consultants
             will not allow that, or the guest simply cannot get through the sign-in in time.
             "Figma for BIM" (live, multi-person, in the browser) and data control (who holds my model?)
             are both ongoing industry themes; a zero-upload live room speaks to both. (Stated as a
             general theme; no post counts were measured.)
             Time today (estimated from how offices work, not timed): preparing a review (exporting to
             NWC / PDF views, uploading, inviting, waiting for guests to get access): 20-60 minutes per
             meeting; during the call, "which beam?" / "go back" / "a bit left" clarifications: about
             5-10 minutes of a 40-minute call; writing the minutes afterwards: 30-60 minutes. With Room
             Link: opening a room takes about 1 minute, guests join in seconds, pinned comments make
             the issue list as you go. Estimate: 1-2 hours saved per meeting for the person who hosts;
             for a team with 3-5 reviews a week, 4-8 hours a week.
             Market: Autodesk Construction Cloud / Docs and Workshop XR, Trimble Connect, Revizto, Fuzor
             and Dalux Viewer give shared views or live sessions; they are paid or need an account, and the
             model lives in their cloud. Speckle's viewer is open but the model is first sent to a server.
             Bluebeam Studio Sessions is paid and for PDFs. BIMcollab ZOOM (free viewer) and Navisworks
             Freedom open a model on one machine only. A browser room where the model travels only
             between the meeting's devices, with BCF issues made as people talk, we did not find as a
             free tool. Not checked product by product; this is a review of what the teams around us use.
```

### Description

**The pain.** A model review today is a screen share. The picture is a compressed video of a 3D view. Only one
person can steer it. A guest who wants to say "this one, left of the core" can only describe it in words. The
alternative is a cloud viewer, which needs the model uploaded, accounts for each guest, and a company policy
that often says no. Afterwards the decisions are in someone's notebook.

**The idea.** One person (the host) holds the model. The others open a link and get the model sent straight to
their browser, device to device. Inside that room everyone shares:

1. **The camera, when they want it:** *Follow me* (the guest's view is locked to the host's), *Free look*
   (the guest moves alone; a *Back to host* button snaps to the host's view). The host can *Hand over*
   control to one guest for a minute.
2. **Pointers:** each person is a coloured dot with a name on the others' screens. Holding `L` draws a
   fading laser trail on the model, so "this one" means something.
3. **The section box and the selection:** the host's section box, hidden elements, colours (for example
   Colour by storey) and clicked element are shared in Follow me; in Free look they are local.
4. **Comments that become issues:** click an element, type or dictate a comment. It becomes a BCF 3.0 topic
   with the element's IFC GlobalId, the camera, the section box and a screenshot, visible to everyone at
   once. The host exports the list as `.bcfzip`, as an Excel list, and as a one-page PDF with pictures.
5. **No leftovers:** End room closes every connection. Guests' copies exist only in browser memory (never
   written to disk or the browser's storage unless the guest presses *Keep a copy*). If the host wants,
   guests are View-only and cannot export or download the model.

**Why it wows (the 20-second demo).** Host presses *Open room*; a QR code appears; a phone camera taps it; a
22-storey tower loads on the phone; the host orbits, and the phone follows; the phone user touches a beam, and
a green dot with their name lands on the host's screen with a comment. Nothing was installed, nobody signed in,
and the network panel shows no upload.

**Why it is useful every day, not only in demos.**
- Weekly coordination calls and client walk-throughs (the main case).
- Site engineers get a quick "look at this joint" from the office at their desk, on a phone, in a corridor.
- A senior reviews a junior's model remotely at the moment it is wrong, rather than by a screenshot chain on
  WhatsApp.
- Training: a lead walks a new modeller through a model with the shared pointer.
- Works with the repo's other ideas: Meeting Mind (voice to BCF) can listen to a Room Link session; Issue Desk
  and Rev Merge can read the issues; Model Reel can record a session as a video.

### Implementation

Written against what is in the cad2bim repository today (checked: `docs/sharing.md`, `apps/web/src/lib/viewLink.ts`,
`sharedModel.ts`, `exportTargets.ts`, `vercel.json`). It is a plan, not a measurement.

**What cad2bim already has and Room Link builds on**
- A view link that carries camera, section box and style (`viewLink` in `exportTargets.ts`). The room reuses
  this exact "view state" object as its message, so there is no new state model to invent.
- Encrypted sharing (AES-GCM 256, key in the `#` fragment, never sent to a server) in `sharedModel.ts` and
  `docs/sharing.md`. The room reuses the same encryption for chunks.
- The viewer (three.js) and the IFC reader (web-ifc) run fully in the browser, so a guest needs only a link.
- BCF is on the roadmap (`roadmapItems.ts`, `docs/design/actionable-qa.md`) but not built. Room Link builds the
  BCF 3.0 writer as a small module that Meeting Mind and the QA fixes can reuse (so this is not wasted work).

**Parts**
1. **Connection (WebRTC data channels).** The host and each guest connect directly. Two ways to start, both
   without storing the model:
   - *Link + code* (default): a tiny signalling step via a Vercel function that only passes the session offer and
     answer and forgets them after about 60 seconds (no model, no comments pass through it). A QR code carries
     the link.
   - *No-server fallback:* copy-paste the offer and answer text (for companies that allow no signalling
     server). Slower, but nothing at all touches our servers.
2. **The model's journey.** The host splits the IFC (or, better, the already parsed geometry plus properties
   in cad2bim's compact form; to be chosen after measuring) into 64 KB chunks, encrypts them, and sends them over
   the data channel. A guest that finishes receiving becomes a second source, so a 150 MB model going to 6
   guests does not need the host's upload speed six times (a small swarm; guests who have received a chunk can
   pass it on). Progress shows as a ring on each guest's join screen. Estimate to verify: with 20 Mbit/s upload,
   150 MB takes about 1 minute for the first guest; a *Light mode* sends only the visible storeys first, so the
   guest sees something in seconds.
3. **The room's messages (JSON, tiny).** `view` (camera, section box, hidden, colours), `cursor` (x, y, z or
   screen point), `laser`, `select` (GlobalId), `comment` (BCF topic), `role` (follow / free / driver), `leave`.
   Conflicts are simple: the host's state wins; comments are append-only with a client ID plus a timestamp.
4. **Roles.** Host (owns the model, can end the room, can make guests view-only), Driver (the person who has
   control for the moment), Guest. Guests can be marked *Can comment* or *Look only* at the door.
5. **BCF 3.0 writer** (zip of `bcf.version`, one folder per topic with `markup.bcf`, `viewpoint.bcfv`,
   `snapshot.png`), tested by opening the result in BIMcollab ZOOM and Solibri's free viewer, and with an
   import round trip in cad2bim itself.
6. **UI.** A "Room" button in the viewer toolbar; a small top bar showing the room name, people, Follow / Free
   toggle and a QR popover; a right dock panel for comments. Uses the design system's existing tokens
   and `StripTabs` / dock panels, not new styles (`npm run lint:css`).
7. **Revit side (later).** The add-in already exports the live model to the web app (Revit live). A "Start room
   from Revit" button sends the same data into a room, and an issue made in the room is sent back to Revit as
   a pinned note with its view (needs the BCF reader on the add-in side).

**Things that must be settled before building (honest risks)**
- **Content-Security-Policy.** `vercel.json`'s `connect-src` lists only `'self'`, localhost, Vercel Blob and
  GitHub. WebRTC needs a STUN server (and `wss:` for a signalling relay if one is used). Adding a STUN host is
  a security-gate item (`docs/security.md`, the `security-gate` skill) and a decision row in `docs/decisions.md`.
- **"Nothing leaves the device unasked."** A STUN server sees the guest's public IP address. The room must say so
  once and ask; a self-hosted STUN or the copy-paste fallback avoids it entirely.
- **Strict company networks** often block direct connections, and a TURN relay may be needed. The relay carries
  only encrypted chunks (it cannot read the model), but it is a server we or the customer must run. To be
  measured on two or three real office networks before promising "works everywhere".
- **Phones have little memory.** A 150 MB IFC may not load on a phone. Light mode (storeys on demand) and a
  hard limit with a clear message are needed, and a test on an ordinary mid-range Android phone.
- **Licences.** Any WebRTC helper library must be MIT / Apache-2.0 / BSD / ISC (CLAUDE.md licence rule); the
  browser's own RTCPeerConnection needs none. BCF 3.0 is an open buildingSMART standard.
- **Plans and accounts.** Decide whether Room Link is part of the free trial, a plan feature, or free for guests
  and paid for hosts (the repo's `auth.md` and the plans in `cloudProjects.ts` already separate who can share).
  A guest should never need an account.
- **Safe by default.** Rooms are unguessable (128-bit), end after 12 hours or when the host leaves, and show every
  person's name; the host sees a "Let in?" prompt for each new guest if the room is set to *Ask to join*.

**Suggested first slice (about one release, to check the idea is worth it)**
1. Two browsers on one machine: host sends a small IFC through a data channel, guest renders it (proves the
   transfer).
2. Follow me + cursors over the same channel (proves the "wow").
3. Comments as BCF export (proves the output).
Only after this, add the signalling function, the swarm, Light mode, roles and the Revit button.

**How to test.** Unit tests for chunking and re-assembly, encrypt / decrypt, the message schema and the BCF writer
(open the result in two external BCF tools). A Playwright test with two browser contexts joined to one room: host
changes the section box, guest receives it within a set time. A manual test over two different home / office
networks and one phone.

### Future update

- **Meeting Mind inside the room:** on-device speech to text turns what is said while pointing into issues
  automatically (the BCF topic already exists); the minutes PDF comes out at End room.
- **Issue tour for next time:** open the room, press *Open issues tour*, and the camera flies through last
  week's issues, marking each as fixed (the element changed) or still open (links to the Change Radar and
  Rev Merge ideas).
- **Record a room as a reel:** a local recording of the session (camera path + comments) that Model Reel turns
  into a short video for WhatsApp / LinkedIn.
- **Side by side:** two models (for example the architect's and the structure) in one room with a toggle; Flush
  Fit's clashes appear as shared markers.
- **Viewer-only kiosk:** a read-only room with a fixed tour for a client's sales office or site notice board,
  with the room ending at a set time.
- **Offline room on a site Wi-Fi:** a room between devices on the same local network with no internet (the
  copy-paste or QR signalling already allows this), useful on sites with weak mobile data.
- **Hand-over to AI:** the agent helper (`packages/mcp`) can join as a read-only guest to answer "what is the
  total concrete on this storey?" in the room.
