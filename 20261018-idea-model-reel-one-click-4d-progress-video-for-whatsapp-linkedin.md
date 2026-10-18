# 20261018 Idea: Model Reel — one click turns the IFC model + pour plan into a branded 4D / progress / rebar video for the client, WhatsApp and LinkedIn / Instagram

## Entry 1

```
date-time:   2026-10-18 (scheduled run, second entry of the day)
idea name:   Model Reel — "The weekly client update that makes itself: a 30-second cinematic video of your model."
             Open any IFC / cad2bim model. Pick a template: (1) 4D Build-up (floors rise in pour order with the date
             and m3 counter), (2) Rebar Explode (slab -> beams -> bars fly into place, bar-mark labels), (3) Progress
             Week (this week's painted/poured elements glow, rest ghosted), (4) What Changed (Rev A -> Rev B with
             red/green), (5) Clash Tour (fly through every open BCF issue). The tool plans the camera itself, adds
             captions, a logo and a progress bar, and exports MP4/WebM 9:16 (Reels / Shorts / WhatsApp status),
             1:1 and 16:9 (LinkedIn / client mail), plus a tiny single-file HTML player and a GIF.
potential:   High, and it is the shareable "wow" for the whole toolkit. Every BIM team is asked for "a video for the
             client / the MD / our LinkedIn page", and today it takes 2-6 hours: set up a Navisworks/Synchro/Twinmotion
             scene, build the camera path by hand, screen-record, edit in a video app, add captions. Most teams
             skip it and send screenshots. Model Reel does it in under 2 minutes from the data already in the
             toolkit (Pour Planner dates, Bill Painter progress, Change Radar diff, IDS/BCF issues, BBS Weaver
             cages), so each new feature of the toolkit automatically gets a video output.
             Time saved: ~3 h per video, a weekly owner update dropping from half a day to ~5 minutes.
             Business value: clients and owners understand a moving model in 10 seconds, approvals and bill
             certification move faster; the team's own LinkedIn/Instagram posts get content for free
             ("this week on site: L4 slab, 42 m3, 2 days ahead").
             Trending angle: short vertical video of construction sequence and "build in 30 seconds" reels are
             among the most-shared BIM content on LinkedIn and Instagram (construction time-lapse, 4D simulation,
             rebar detailing "satisfying" animations). This tool lets any modeller make them without a render farm.
             Market: see Market check.
```

### Description

**The pain (daily life of a BIM modeller / planning engineer).**
- "Send a 4D video for tomorrow's client meeting." The modeller exports the model to a heavy desktop planner,
  links the schedule, tries camera views, records the screen, trims it in another app. Half a day, then the
  schedule changes on the same evening and it is redone.
- Weekly progress is reported as a PDF and photos. Owners and non-technical stakeholders do not read it;
  a 20-second video of what was actually built that week is understood immediately.
- Rebar detailers and structural modellers do beautiful 3D work that nobody outside the team ever sees,
  because making a presentable animation is a separate skill and licence.
- Marketing/BD asks the BIM team for material for tenders and social posts; the BIM team has no time.
- Videos rendered in desktop tools are large (200 MB+), cannot be sent over WhatsApp, and need a viewer to replay.

**The idea.** A **Model Reel** panel with a "director" that makes the decisions, so the user only picks a story:

1. **Pick a story (template).**
   - *4D Build-up*: foundations -> columns -> slabs by pour/level; date, day-count and cumulative m3 counter;
     planned vs actual colour (grey = not started, blue = in progress, green = done, orange = late) if actuals
     exist (Bill Painter / Chat-to-Model).
   - *Progress Week*: the last 7 days of painted/poured elements glow; everything else ghosted. Ends on a stat card
     (m3, kg steel, % complete, days ahead/behind).
   - *Rebar Explode / Cage Build*: the concrete goes translucent, the cage assembles bar by bar in BBS order
     (main bars, stirrups, then laps), bar-mark labels appear; a "BBS in 20 seconds" video for the bar-bending yard
     and for the client's rebar approval.
   - *What Changed*: Rev A -> Rev B morph with removed (red) and added (green) elements and the BOQ delta
     (Change Radar data), good for design-review meetings.
   - *Clash Tour*: auto camera visits each open BCF issue / sleeve conflict in priority order with the issue title.
2. **Auto camera director.** Pure geometry, no AI required:
   - Bounding-box based hero orbit, then smooth dolly to the elements that become active at each time step
     (centroid + size -> distance, eased with a spring), never inside geometry, keeps the ground and the building
     in frame.
   - Rule set per aspect ratio: for 9:16 the camera favours tall framing and tighter orbit.
   - Shot list is editable (reorder, delete, change duration) as a simple timeline strip, and "re-roll" gives a new
     camera plan with a seed.
3. **Captions and branding.** Automatic lower-third text from the data ("L3 slab — 42 m3 — Day 36"), project name,
   logo, colour theme (dark/blueprint/daylight), progress bar, optional music-free subtitle track. No fonts or
   assets leave the machine.
4. **Export.**
   - MP4 (WebCodecs H.264 when available) or WebM (MediaRecorder fallback), 1080x1920, 1080x1080, 1920x1080.
   - *WhatsApp-safe* preset: under 16 MB (adaptive bitrate), 30 fps, 20-30 seconds.
   - GIF / animated WebP for emails and docs.
   - **Single-file HTML player** (model streamed as glTF + camera script + captions, ~2-5 MB): a link the client
     opens, can pause, scrub the date slider and orbit; no login, no installation.
   - Poster image and a 1-line caption + hashtags text for LinkedIn/Instagram.
5. **Re-run in one click.** The reel is saved as a recipe (`reel.json`: template, date range, theme, seed). When
   the schedule or progress changes, press *Rebuild* and next week's reel is ready. Recipes can be shared across a
   project so the whole team's reels look identical.

**Why it creates the wow effect.** Pick "Rebar Explode" and the grey slab fills with glowing bars building up in
sequence; pick "4D Build-up" and the building grows floor by floor with a ticking date. The result appears in the
browser in seconds and can be posted from a phone. It is also an honest demo of the whole toolkit: every other
tool becomes a visible 30-second story.

### Implementation

Everything runs in the browser, reusing the viewer already in the toolkit.

1. **Scene source**: the existing three.js scene in cad2bim (`packages/viewer`) and Structura's overlay layer.
   Extract a `ReelScene` interface: `getElements()`, `setVisibility(set)`, `setColor(set, color)`,
   `setOpacity(set, a)`, `getBounds(set)`. Same interface used by Pour Planner and Bill Painter so every module
   can register a "story" that returns timed visibility/colour keyframes.
2. **Story engine** (`packages/engine/src/reel/`, pure TypeScript, no DOM, testable): a story is
   `{ steps: [{ t0, t1, show[], glow[], ghost[], caption, focus[] }] }`. Built-in stories:
   - `build4D(schedule)` from Pour Planner dates (element -> pour -> day), colour by status.
   - `progressWeek(statusLog, from, to)` from Bill Painter / Chat-to-Model accepted entries.
   - `rebarBuild(bbs)` using BBS Weaver cage order (bar by bar, by mark, by member).
   - `whatChanged(diff)` from Change Radar.
   - `issueTour(bcf[])` from IDS Gatekeeper / Sleeve Punch.
3. **Camera director** (`camera-director.ts`): pure functions `planShots(story, bounds, aspect, seed)` ->
   keyframes `{t, pos, target, fov}`; Catmull-Rom path, critically damped spring smoothing, clamp to avoid ground
   and clipping; unit tests for "target always in frustum", "no jump greater than N metres per frame".
4. **Deterministic renderer**: drive time with a fixed timestep (do not use wall clock), render each frame to an
   OffscreenCanvas; this makes exports reproducible and independent of device speed. Capture:
   - Preferred: **WebCodecs `VideoEncoder`** + a small MP4 muxer (e.g. `mp4-muxer`) for exact frames.
   - Fallback: `canvas.captureStream()` + `MediaRecorder` (WebM) for browsers without WebCodecs.
   - GIF through a worker (gif.js / WASM encoder), palette limited to 128 colours with dithering.
5. **Overlay**: captions, counters and the stat card are drawn on a 2D canvas layered on the frame so they are
   baked in; text layout tested for 9:16 safe areas (Instagram UI covers the top 14% and bottom 20%).
6. **HTML player export**: serialise visible geometry to glTF/GLB (draco optional), the camera keyframes, step
   timeline and captions into one HTML with a ~40 KB player (three.js tree-shaken) -> a share link or an email
   attachment. Optionally publish through the Artifact/static hosting the team already uses.
7. **Large models**: use instancing and merged static geometry for "ghost" elements, LOD for far views, and render
   at 1080p with 2x MSAA only; auto-lower the resolution if frame time exceeds budget. Progress bar + cancel.
8. **UI**: one right-hand panel "Reel": story dropdown, date range, theme, aspect, duration slider, *Preview*
   (real-time), *Export*. The timeline strip shows shots; a "Why this shot?" tooltip explains each camera move so
   the modeller trusts and can override it.
9. **Tests / fixtures**: sample IFCs (small tower, RCC frame with rebar), golden-file tests for keyframes and for
   story JSON; frame-hash test (same recipe + seed -> same first/last frame hash); CI export of a 3-second clip
   with headless Chromium (Playwright) as a smoke test.
10. **Privacy**: all local; no upload. Watermark option and "blur the project name" toggle for public sharing.

MVP (2 weeks): ReelScene interface, build4D story from Pour Planner, camera director, WebM export in 16:9 and 9:16,
captions and counter. Then rebar story and WhatsApp-safe size preset (week 3), progress-week + stat card, MP4
(WebCodecs) and GIF (week 4), HTML player export, recipes and templates, What-Changed and Clash Tour.

### Future update

- **Voice-over / auto-narration**: local text-to-speech reading the captions ("Level 4 slab, 42 cubic metres, two
  days ahead of plan") in English/Hindi; optional royalty-free music bed.
- **Photo sandwich**: split screen with the real site photo of the same week (from Chat-to-Model / Site QR Pass) next
  to the model state; "plan vs reality" reel for owners.
- **Weekly auto-reel**: a scheduled job that rebuilds the reel every Friday from the latest status and drops the
  MP4 plus a WhatsApp caption into the project folder or a share link.
- **AR mode**: the HTML player's glTF opens in phone AR (WebXR) at the site to place the model on the actual plot.
- **Sequence authoring by drag**: click a group of elements -> "this is activity 14, 3 days" to edit the 4D right in
  the reel timeline and write the dates back to the Pour Planner.
- **Estimate reel (5D)**: cost accumulating as elements appear (S-curve chart drawing itself at the bottom), RA bill
  figures from Bill Painter.
- **Revit side**: a pyRevit button exports the same story from a Revit model (visibility/colour overrides per
  step, view export per frame) so Revit-only teams get the same output.
- **Brand packs**: per-company themes (logo, colours, outro card) stored in a shared JSON for consistent outputs.
- **Accessibility**: burnt-in subtitles + alt-text description for each reel, auto-generated from the story steps.

### Market check

- Desktop 4D tools (Navisworks Timeliner, Synchro, Power Project BIM, Twinmotion, Enscape-based workflows) can
  export timeline videos, but they are paid/licensed desktop software, need the schedule linked by hand, and do not
  produce vertical social-ready or WhatsApp-sized clips. For example, Elecosoft Power Project BIM documents video
  export from the 4D tab ("Creating videos of timeline simulations and fly pasts").
- A free, browser-only alpha tool with a "film maker" that catches the 4D schedule has been mentioned on the
  OSArch community (still very early, no vertical/9:16 export confirmed); this validates the demand and shows
  there is no polished free option yet.
- Search on 2026-10-18 found no tool that, from an IFC model plus progress data, offers one-click templates
  (4D build-up, progress week, rebar explode, what-changed, clash tour) with auto-camera, captions, 9:16 / 1:1 /
  16:9 export, WhatsApp-size preset and a single-file shareable player. (Verify again before launch.)
- It extends the toolkit rather than competing with it: every existing tool (Pour Planner, Bill Painter, BBS Weaver,
  Change Radar, IDS Gatekeeper, Sleeve Punch, Chat-to-Model) becomes a data source for a story.

Sources:
- [Elecosoft: Creating videos of timeline simulations and fly pasts](https://help.elecosoft.com/powerproject_bim/english/help/Content/HTML%20Topics/Timeline_simulations/Videos.htm)
- [OSArch community: browser IFC tool with 4D film maker](https://community.osarch.org/discussion/comment/29782/)
