# 20261104 Idea: Model Capsule — turn the open model into one self-contained .html file the client double-clicks on any phone or laptop (no login, no install, no upload), with a client-safe profile, saved views and a tour, and a small reply file that brings their comments back as pinned BCF issues

## Entry 1

```
date-time:   2026-11-04 (scheduled run, second entry of the day)
idea name:   Model Capsule (One-File Offline 3D Review + Client-Safe Profile + Optional Password +
             Guided Tour + Reply File That Imports as BCF Issues)
potential:   High. Every BIM modeller sends "the model" to a non-BIM person every week: the client, the
             architect's boss, the site engineer, the approving authority. Today that means a PDF of
             screenshots (flat, out of date by tomorrow), a Revit or IFC file the receiver cannot open,
             or an account on a cloud viewer the receiver will not make. A capsule is a file: it goes on
             WhatsApp or email and opens with a double-click. Estimated saving (an estimate, not
             measured): 30 to 90 minutes per review round for the modeller (no screenshot sets, no
             "please install X", no re-explaining), and the review actually happens because opening it
             takes one tap. It also avoids the privacy question every team gets: "where did you upload
             our building?". Nowhere. The capsule is the file.
```

### Description

**The pain (what a modeller actually lives through).**
- The client wants "to see the model". The modeller exports 12 screenshots to PDF. The client asks for the
  view from the other side. Another 12 screenshots.
- The client has no Revit, no IFC viewer, and a phone. A cloud viewer link needs an account, a slow
  upload, and a conversation with IT about where the data goes.
- A 400 MB model cannot go on WhatsApp or email. So nothing goes, or a stale video goes.
- The reply comes back as a WhatsApp voice note: "the second floor, that wall near the lift, move it". The
  modeller has to work out which wall.
- Sending the real model also sends things the client should not see: rates, internal parameters, other
  disciplines' unfinished work, the notes in the properties.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this repo's idea branch: Room Link is a *live* review room streamed device to device (both people
  online at the same time). Model Reel makes a *video* of progress. Chat to Model and Snag Sweep bring
  site input back in. None is an *asynchronous, offline, single file the receiver opens alone*, and none
  has a client-safe stripping step.
- In cad2bim (from `docs/sharing.md`, `docs/design/million-elements.md`, `docs/design/actionable-qa.md`):
  the *view link* holds a view but never the model; *Share with the model* uploads an encrypted copy to
  server storage (needs the Vercel Blob set-up, an account on plans, and expires); the viewer already
  draws repeated geometry by GPU instancing and the QA panel already makes BCF 2.1 topics. The open
  parts are: a build that has no server at all, a size budget, and the return path.
- In the market (not exhaustively checked; treat as unverified): cloud viewers (Autodesk Viewer, Trimble
  Connect, Dalux and similar) need an upload and usually a sign-in; Navisworks Freedom is a Windows
  desktop program that I believe is no longer developed; Enscape and Twinmotion standalone exports are
  paid and mostly desktop; BIMcollab ZOOM is free but is an app to install. Open-source viewer libraries
  (That Open, xeokit, three.js) can show a model in a web page, but a modeller cannot get a finished
  single-file capsule with a client-safe profile and a reply loop from them without writing code. I could
  not confirm that a free, one-click, single-file tool with a return path exists; mark that unverified.

**The idea.** In cad2bim's web viewer, **File → Make a capsule**. One dialog, one output: `Project-Name_review_2026-11-04.html`
(a few MB to a few tens of MB). Inside is the cad2bim viewer's own code, the model prepared for sharing, the
saved views, the notes, and a small form to reply. Double-click it: the browser opens it from disk and it
works with the internet off.

1. **Prepare for sharing (the size budget).** The person picks a target ("fits WhatsApp: 16 MB",
   "fits email: 20 MB", "best quality") and sees a live estimate before building. The build keeps
   instanced families once, quantises positions, drops what is invisible in the chosen views (inside the
   slab, hidden by filters), merges small parts and simplifies rebar to centre lines or impostors at a
   chosen detail. A fidelity panel shows the cost honestly: "8,412 elements kept, 0 removed; geometry
   within 2 mm for walls and slabs, rebar shown as lines". Nothing silently disappears.
2. **Client-safe profile (the step teams need and forget).** A checklist with a preview: strip rates and
   BOQ columns, strip chosen property sets (everything except a whitelist by default), hide disciplines
   or categories, hide elements by filter or by view, remove file paths and author names from metadata,
   optional watermark with the recipient's name on every screen. The profile is saved for the project so
   the second capsule takes ten seconds.
3. **Views and a tour (the wow).** The sender saves 5 to 10 views and puts them in order with one line
   each: "1. Entrance and the new canopy", "2. Typical floor with the shifted lift". The capsule opens on a
   guided tour: tap Next and the camera flies to the next view, with the note and the elements involved
   highlighted. Plans, sections, a section box, a measure tool, an element tap that shows only the
   whitelisted properties, and, if the project has them, the 4D timeline scrubber. A non-BIM person
   learns the controls in one screen ("one finger rotates, two fingers move, tap a thing to see what it
   is").
4. **Reply, offline (the second wow).** In the capsule the client taps an element, writes a comment (or
   dictates it with the phone keyboard's own microphone), and it is saved as a pin on the 3D view with
   the camera position. "Send reply" makes a small file (`.c2reply`, a few KB: comments, views, element
   GlobalIds) that they send back on WhatsApp or email, or, on a phone, the share sheet. The modeller
   drops it into cad2bim and the comments appear as pinned issues, each one a click away from the
   exact wall, and can be exported as a BCF 2.1 file for Revit users. The question "which wall near the lift?" no longer exists.
5. **Optional password and watermark.** A password encrypts the model inside the file (same approach as the
   existing encrypted share: key derived from the password, AES-GCM). It deters casual forwarding, and
   the watermark names who it was given to. Honest limits on the card: **a file on someone's device
   cannot be recalled or made to expire**; a date can be shown ("prepared 4 Nov, superseded by newer
   capsules") but not enforced, and a password the receiver knows protects nothing from that receiver.
6. **Version stamp.** Every capsule shows its build time and a short model fingerprint. The reply file
   carries that fingerprint, so the modeller sees "this comment was made on the 28 October capsule; 3 of
   its elements have changed since" (this is where Change Radar and Revision Timeline plug in).

**LinkedIn / Instagram angle.** A 20-second screen recording: a phone with no signal (airplane mode on
screen), a WhatsApp attachment tapped, a building opening and flying through the tour, a comment pinned,
and the reply landing in cad2bim on a laptop. The hook, "your client does not need Revit, an account or
Wi-Fi to review your model", is easy to say and easy to see. Openness and data-privacy talk ("no cloud
upload of the client's building") is a common LinkedIn BIM theme; I cannot measure what is trending today
from here, so check the angle against the team's own feed before posting.

**Who benefits, day to day.**
- The modeller: stops building screenshot decks; one capsule per milestone; comments come back pinned.
- The BIM manager: a client-safe profile that is checked once and reused, so no rates leave by accident.
- The project manager and the client: a review they can do on a phone in a lift or on site with no signal.
- The site engineer: a capsule of one floor with the pour and rebar views, readable in the yard.
- The approving authority or consultant: a stable, dated snapshot they can open years later (the viewer code
  is inside the file, so an old capsule does not break when the app changes).

### Implementation

- **Packaging.** A build step in `apps/web` that produces one HTML file: the viewer bundle inlined
  (script, styles, fonts, icons as data), the prepared model as a compressed binary blob embedded in the
  page (base64 is about a third larger than binary; check a `data:` blob versus an embedded
  `<script type="application/octet-stream">`), and web workers created from Blobs, because a page opened
  from disk (`file://`) cannot load separate worker files. WebAssembly (web-ifc is not needed in the
  capsule if the model is already prepared) must be inlined or avoided; the capsule uses the engine's
  own prepared mesh format, so the 20 MB reading stack is not in the file. First task is a spike: open a
  built capsule from disk in current Chrome, Safari (iPhone) and Firefox, offline, and record what breaks
  (`file://` rules differ by browser).
- **Model preparation.** Reuse the engine's instancing (`ParsedModel.instances`, `docs/design/million-elements.md`),
  add quantised vertex positions and index compression. Candidate libraries, all on the repo's licence list
  to be confirmed against `docs/decisions.md` before use: meshoptimizer (MIT) for mesh compression and
  fflate or the browser's own `CompressionStream` for gzip. No GPL or LGPL code, as the repo rules say. Measure
  size and load time on three real models (kept outside git) and write the numbers in the next entry.
- **Client-safe profile.** A pure function from (model, profile) to (prepared model): property whitelist,
  category and discipline filters, metadata scrub. It is tested with a fixture IFC that contains a rate, an
  author name and a file path, and a test that searches the output bytes for them (the same idea as the
  repo's "nothing leaves the device" rule, checked mechanically). This is the safety-critical part; it
  ships with the build or not at all.
- **Tour and views.** Saved views already exist (`apps/web/src/lib/views.ts` and `viewLink.ts`); a tour
  is an ordered list of view ids with a note each, stored in the capsule's manifest as JSON.
- **Reply file.** A small JSON in a zip (or BCF 2.1 directly: the repo plans its own code for a zip of
  XML, `docs/design/formats-and-exchange.md`). The capsule writes it with the browser's download or the
  share sheet (`navigator.share` with a file, where the browser has it). cad2bim reads it back and
  creates issues pinned to the GlobalIds, with the camera, as in `docs/design/actionable-qa.md`.
- **Password.** PBKDF2 or Argon2-class key derivation in the browser, AES-GCM through WebCrypto (as in
  `apps/web/src/lib/sharedModel.ts`); the capsule's first screen asks for the password. A wrong password
  gives no hint about the contents. The card states that this stops a casual forwarder, not a determined one.
- **Where it lives.** The builder is part of the web app (a new action in the File menu and the command
  palette), the capsule viewer is a build target of the same viewer package, and the agent helper
  (`packages/mcp`) gets a `make_capsule` command so "send the client the typical floor and the new canopy
  as a capsule under 15 MB" can be one sentence later.
- **Tests.** (1) Build a capsule from a fixture model and open it in a headless browser from `file://` with
  the network blocked: it must render and the tour must advance. (2) Profile leak test as above.
  (3) Reply round trip: comment in the capsule, import the reply, issue pinned to the right GlobalId.
  (4) Size budget test: the build never exceeds the chosen budget or says it cannot.
- **Risks.** (1) A stale capsule is trusted as current: the date and fingerprint are on every screen, and
  the reply import warns when the model has moved on. (2) Large models do not fit a phone's memory:
  the size target and a "this floor only" option are the answer, with an honest warning when a model is
  too big. (3) A false sense of security from the password: the wording on the card. (4) Browser
  differences with `file://`: the first-week spike decides which browsers are supported, and the supported
  list is printed on the card. (5) Someone modifies the capsule and re-sends it: the reply carries the
  fingerprint, and a changed fingerprint is flagged, not trusted.
- **First 2 weeks.** The spike (a hand-made single file with one model, opened from disk on two laptops and
  two phones, offline); the preparation step with one size target and the instancing; the client-safe
  profile with the leak test; saved views as a tour; the comment pin and the reply file; the import into
  cad2bim as issues. No password, watermark or 4D in the first version. Record: size, load time on a
  mid-range phone, and what a real client managed without being told how.

### Future update

- Include the 4D timeline and 5D cost (cost only if the profile allows it) so a capsule can show "what
  gets built when" without a server.
- A *floor capsule* and a *trade capsule* in one click for site teams ("level 7 formwork and rebar").
- Capsule QR sheet: print a QR for a site notice board that points to a hosted capsule, when the team
  chooses to host one (the capsule stays a plain file; hosting it is the team's own decision, not the app's).
- Link with Room Link: open the same capsule in a live room when both are online, with the capsule's
  comments already pinned.
- Link with Issue Desk and Markup Mirror: replies become the response sheet, with the status of each
  comment tracked across capsule versions.
- Link with Change Radar: "what changed since the capsule you sent on 28 October" as a one-page note for the
  client, drawn on the model.
- AR on the phone (WebXR where the browser has it) to place a floor-plate or a staircase at site
  scale, once the base capsule is solid.
- A "capsule diff": the client opens two capsules side by side and flips between them.
