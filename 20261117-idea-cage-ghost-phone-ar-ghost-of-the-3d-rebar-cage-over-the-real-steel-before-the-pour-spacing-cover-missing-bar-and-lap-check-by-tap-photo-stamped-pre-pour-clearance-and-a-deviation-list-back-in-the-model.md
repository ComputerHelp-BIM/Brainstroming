# 20261117 Idea: Cage Ghost — Point the phone at the real steel on the slab, beam or column and see the 3D rebar cage as a ghost over it. Tap a bar to check spacing, cover, lap length and count against the model. Missing or wrong bars glow red. Sign the pour clearance with photo proof, and every deviation lands back in the model as a BCF issue. Runs in the phone browser; nothing is uploaded.

## Entry 1

```
date-time:   2026-11-17 (scheduled run, UTC; third idea of the day)
idea name:   Cage Ghost (browser AR pre-pour rebar check against the 3D model)
potential:   High, and it is the last gate before concrete goes in. After the pour, a missing bar
             cannot be seen or cheaply fixed. Today the pre-pour check is a site engineer with a
             tape, a printed drawing and a checklist. It is slow, and the drawing and the model can
             be a revision apart. Time saved: NOT measured. First test: on one real slab, time the
             usual check against Cage Ghost on the same pour, and count the errors each one finds
             (the steel is kept out of git; only the numbers are written down). Do not quote hours,
             rupees or "errors prevented" until that is done.
             Market check (2026-11-17, two searches; not exhaustive, so "I did not find one", not
             "none"):
               - GAMMA AR: overlays IFC/Revit/Navisworks on site and flags plan vs built. A paid
                 app (listings show about USD 86 per user per month on the Basic tier; indicative,
                 confirm with the vendor). A native app. I did not see rebar-specific checks.
               - Trimble Connect AR: AR tiers on a Trimble Connect plan, QA/QC clash spotting, a
                 headset option. Price not shown in the results.
               - NEXT-BIM Explorer and FieldAR: listed as free apps that overlay BIM models. Native
                 apps, not a browser page. No rebar checks seen.
               - A 2024 Bachelor thesis (OST, Switzerland) shows WebXR + glTF in a browser for
                 underground utility lines on Android tablets. A prototype, not rebar.
               - I did not find a free browser AR tool that checks rebar spacing, cover and laps
                 against the model and writes a signed pre-pour record. I did not verify Dalux.
             Extends tools already in this repo (does not repeat them):
               - Cage Manual: the step-by-step fixing manual and ITP checklist. Cage Ghost is the
                 inspection that comes after the fixing, on the same element, using the same
                 hold-point list.
               - Site QR Pass: scan a QR, see the 3D pour card. Cage Ghost uses the same QR to open
                 the element and to anchor the overlay.
               - Scan Check: point cloud vs IFC as-built. Cage Ghost needs no scanner, only a phone,
                 and is for steel before the pour, not concrete after.
               - Joint Judge, Cover Cop, Bar Swap: supply the expected cover, joints and any
                 approved substitutions, so the ghost shows the approved bars, not the old ones.
               - Steel Ledger: reconciles steel by weight after the fact. Cage Ghost checks the
                 steel in place, before the pour.
               - Pour Planner and Ready Board: a "pour clearance signed" status feeds the gate
                 "ready to pour".
             Trend: AR on site, "scan and check on your phone" and AI inspection are standing AEC
             posts. I could not measure LinkedIn or Instagram counts; this is not a social-media
             claim.
             The wow: stand at the edge of the slab, scan the QR on the pour tag, hold up the phone.
             The 3D bars appear over the real steel, lined up. One bar mat is shifted: its ghost
             lines glow red and a card says "Top bar T12 @150 modelled; measured 210 at gridline C,
             3 bars short over 2.1 m." Tap "Count bars": the phone counts the ghost bars the camera
             can see on the real steel and says "Matched 41 of 44." Tap "Clear": the photos are
             stamped with element id, time and position, a pre-pour card is filled, and the pour
             gate turns green on the Ready Board.
```

### Description

**The pain.**
- The last check before the pour is by eye and tape, against a printed drawing that may be a
  revision old.
- Missing bars, wrong spacing, short laps and thin cover are only found if someone looks at the right
  place. Everyone is in a hurry, because the concrete truck is booked.
- The sign-off is a paper tick and a few photos in a WhatsApp group, with no link to the element
  and no way to find them later.
- If a problem is found, nobody can say exactly which bars, at which gridline, are wrong.

**The idea.** One page, opened on the phone by QR, in four steps.
1. **Open the element.** Scan the QR from Site QR Pass. The page loads this element's rebar from the
   model (cached on the device; no upload).
2. **Anchor.** Tap two known points on the formwork (for example two column corners or a gridline
   intersection marked on site) so the ghost snaps to the real element. Re-anchor in one tap if it
   drifts as the person walks.
3. **Look and check.**
   - Ghost cage over the real steel, with a slider from "model only" to "camera only".
   - **Measure by tap:** tap two real bars to read the spacing and compare it with the model value.
     Tap a bar end to read the lap length and cover against the rule.
   - **Missing-bar finder:** compares the ghost bar lines with edges in the camera image and marks
     ghost bars with no matching real bar. Shown as a hint with a confidence colour, never as a
     verdict.
4. **Clear or hold.** Each hold point from Cage Manual is a tick with a photo. "Hold" lists the
   deviations. "Clear" writes the pre-pour card and sends the status to the Ready Board.

**What it is not.** It does not replace the engineer's inspection and does not certify anything. It
points the person at the likely problems and keeps the record. The signature on the card is the
person's.

### Implementation

- **Overlay.** WebXR immersive-ar in Chrome on Android (the browser asks for camera permission). Where
  WebXR is not available, a fallback shows the model over the live camera view with manual alignment
  (two taps), and the measure-by-tap tools still work. iPhone Safari does not offer WebXR AR in the
  versions I know of; this must be checked on the test devices before any promise is made. A recent
  iPhone with a USDZ Quick Look file is a possible but weaker route (no live checks).
- **Model to AR.** The element's bars come from the app's own rebar generator (the same data as the
  BBS), turned into simple tubes in three.js. No IFC to glTF conversion is needed on a server.
- **Anchoring.** Two-point anchor first (simple, honest). Later: ArUco-style printed markers at two
  corners of the pour for hands-free anchoring. GPS is not used; it is too coarse inside a building.
- **Measuring.** WebXR hit-test and depth (where the device gives depth) for the tap-measure tools. The
  page shows the measurement error it expects (for example "plus or minus 10 mm on this device") from
  a calibration step against a tape on the first use, and refuses to call a bar "wrong" when the
  difference is within that error.
- **Missing-bar hint.** Edge lines from the camera frame, run in a web worker, compared with the
  projected ghost bar lines. It is a hint with a confidence value. First version: spacing and count on
  a flat mat (slab, raft) only; columns and beams in a later step.
- **Records.** Photos are stamped (element id, time, device pose) and kept in the project file on the
  device. A deviation is written as a BCF issue pinned to the bars. Export the pre-pour card as PDF.
- **Privacy.** Camera frames stay on the device. Nothing is uploaded unless the person shares the
  card.
- **Test first.** (1) On a printed A1 sheet of bar lines on the floor, measure how far the ghost is
  off after anchoring, on three phones. (2) On one real slab, compare Cage Ghost's count and spacing
  with a tape on 20 spots. (3) Seed 5 deliberate errors in a mock-up and count how many it finds and
  how many false alarms it gives. Publish the numbers, even if poor.
- **Risks.** Drift of AR tracking; glare and dirt on bars; dense double mats where the camera cannot see
  the lower layer; iPhone support; the risk that people trust the green tick too much. The page shows
  the error band and the words "hint, not certificate".

### Future update

- Marker-based hands-free anchoring and a head-mounted version for the foreman.
- Column and beam cages: stirrup spacing, hook angle and lap zones.
- Cover blocks and chairs counted against Cover Cop's rule.
- MEP sleeves and embeds shown as ghosts too, so a missed sleeve is seen before the pour (with Sleeve
  Punch).
- A "same check, last pour" comparison, so a gang that repeats the same error is shown the pattern.
- Voice commands on site ("count top bars", "mark here").
- Feed the deviation counts per gang into Gang Pay and per element into Ratio Radar, as a quality
  score. Only after the false-alarm rate is known.
