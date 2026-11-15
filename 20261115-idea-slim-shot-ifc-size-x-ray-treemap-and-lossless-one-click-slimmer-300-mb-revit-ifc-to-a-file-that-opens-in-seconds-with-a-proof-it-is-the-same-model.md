# 20261115 Idea: Slim Shot — a 300 MB Revit IFC that nobody can email, open or review. Slim Shot draws a size map of the file (which categories, types and repeated shapes eat the megabytes), slims it in the browser with a chosen profile (Review, Coordination, Client, Archive), and proves the slim file is the same model: same GUIDs, same counts, same quantities, same shapes

## Entry 4

```
date-time:   2026-11-15 (scheduled run, UTC)
idea name:   Slim Shot (IFC size X-ray + lossless slimmer with a "same model" proof, in the browser, nothing uploaded)
potential:   High, and it is a pain every BIM team has every week. A Revit IFC export of a real building is often
             hundreds of MB. It does not go by email or WhatsApp, the consultant's viewer takes five minutes
             (or runs out of memory) to open it, the cloud upload takes an hour on site internet, and the
             coordination meeting starts late because "the file is still opening". Today the modeller either
             re-exports with fewer categories and hopes (a 20 to 40 minute Revit job, then check again), or sends
             the whole thing and the other side suffers.
             Time and money saved: NOT measured here. A first honest test: take three real IFCs of different
             sizes (kept outside git), run each profile, and write down size before/after, open time in
             cad2bim and in one other viewer, and whether the proof passes. Do not quote "10x smaller" until
             that is done. (A published test cited below saw an IFC optimiser take one file to "almost 1/10",
             and another 560 MB file to 500 MB, so the result clearly depends on the file. That is exactly why
             the tool must SHOW where the bytes are before it promises anything.)
             Market check (2026-11-15, two standard searches, not exhaustive; verify before marketing):
               - Optimisers exist: Solibri's IFC Optimizer (a forum post asks whether it is still available),
                 Bentley's IFC Optimization utility (inside OpenBuildings / AECOsim), IfcPatch recipes
                 (Optimise, PurgeData, RecycleNonRootedElements) in IfcOpenShell, and a Revit-side advice list
                 (fewer categories, IFC4, purge first). Simplebim and similar desktop tools do model clean-up
                 too, as paid products; I did not check their current prices.
               - What I did NOT find: a free, no-install tool in the browser that (1) shows a size map of WHY the
                 file is big, (2) offers named profiles by audience, (3) keeps every GUID and writes a
                 machine-checked "this is the same model" proof, and (4) shows the 3D before and after side by
                 side. IfcPatch is a command line for people who write Python. The Bentley tool is inside
                 Bentley. This is two searches; "not found" is not "does not exist".
             Extends tools already in this repo (it does not repeat them):
               - Export Doctor (20261104): fixes the CONTENT of a Revit IFC (proxies, classes, storeys, psets).
                 Slim Shot changes the SIZE and keeps the content. They chain: doctor first, slim second.
               - Model Capsule (20261104): a one-file offline HTML viewer with its own size budget. Slim Shot's
                 output is a normal IFC that Revit, Solibri, Navisworks and BIMcollab open. A capsule can later
                 start from a slimmed IFC.
               - Very large models, Open again in seconds, Geometry paging (roadmap): make cad2bim itself cope
                 with big files. Slim Shot makes the file smaller for everyone else too, and its size map is the
                 same information those features need to decide what to keep in memory.
               - IDS idea (20261014): checks a model against what the client asked for. Slim Shot can use the
                 same IDS as a "keep only what is asked" list for the Client profile.
               - Mark Merge (Type Diet) (20261029): fewer mark TYPES for detailing. Different subject; the name
                 is the only thing close, so this idea is called Slim Shot to avoid confusion.
             Trend: the 2026 BIM trend pieces keep naming cloud collaboration, openBIM exchange (IFC, IDS, BCF)
             and mandated IFC deliverables. Every one of those assumes IFC files that move easily. A 300 MB file
             is the quiet blocker. Before/after "this file, then this file" is also a very clear LinkedIn or
             Instagram reel (see below).
```

## Description

### The problem, in a modeller's day

- 10:00 The coordinator asks for the latest structure IFC "before the 11:00 call".
- The export takes 25 minutes. The file is 340 MB. It will not go on email. The shared drive upload says 52 minutes.
- The architect's viewer opens it in six minutes, hides half the building, and the call starts without it.
- Afterwards nobody knows why it was 340 MB. Was it the 4,000 chairs, each a full copy of the same mesh? The
  imported CAD block that came in as 60 MB of triangles? The MEP fittings with five levels of detail each? The
  two thousand room-bounding spaces nobody asked for? Every property set repeated on every instance?

### What Slim Shot does, in four steps

1. **Size X-ray (the wow).** Drop the IFC. In a few seconds a treemap fills the screen: each box is a category,
   then a type, then a shape; box area is bytes in the file. Click a box and those elements light up in 3D. Typical
   findings written as plain sentences next to the map, for example:
   - "4,212 instances of 'Chair - Task' carry their own copy of the same geometry: 112 MB. Share one shape: 3 MB."
   - "One imported CAD symbol is 61 MB as triangles. Used by 14 elements."
   - "Properties: 18 identical property sets repeated on 22,000 elements: 40 MB. Share them where IFC allows."
   - "Spaces, grids' 2D lines and annotation: 9 MB. Not needed for a structural review."
   - "Float precision: coordinates written to 15 decimals. 7 MB. Millimetre is enough."
   Every line says "safe" (nothing visible changes) or "choice" (you decide). No number appears that was not
   counted from the file.
2. **Pick a profile.** Four starting points, each a list of rules the person can read and edit:
   - **Archive / Exact:** only the safe steps. Same shapes, same properties, same ids. Smaller because shared
     things are shared and text is tidied. (IFC also reads as zipped `.ifczip`; offer that as a last step.)
   - **Coordination:** also drops what coordination does not need (2D annotation, spaces if asked, very high
     detail on tiny parts like screws), keeps every element, GUID, type, material, level and the properties named
     in a list the person confirms.
   - **Review / Mobile:** a lighter geometry for phone and tablet and for the first open. Marks every changed shape
     as "simplified", and keeps the original shape reference so the full model can be put back.
   - **Client (uses IDS):** keeps only the properties the client's IDS file asks for, hides internal notes and
     costs, strips author and machine names from the header. (Pairs with the privacy step: a model sent to a
     client should not carry the consultant's file paths.)
3. **Slim, locally.** The slimmed IFC is written on the device. Nothing is uploaded. The tool shows a live
   estimate: "now 342 MB, after these steps 71 MB (counted, not guessed)". Steps can be switched off one at a
   time and the estimate moves.
4. **Same-model proof (the trust).** Before and after are compared automatically and the result is a one-page
   report that can go with the file:
   - GUID list: same set, none lost, none new (or a list of what was removed on purpose, by profile).
   - Counts per class and per type: equal, or listed differences.
   - Quantities: gross volume, area and length per class, before and after, within a stated tolerance.
   - Shape check: for every element, the bounding box and the volume of the new shape against the old one. The
     "Exact" profile must give 0 difference; "Coordination" and "Review" show the largest differences and click
     through to them in 3D.
   - A side-by-side 3D view with a slider (before on the left, after on the right) and a "difference" mode that
     lights only what moved by more than a set amount.
   If the proof fails for any element the tool says so and leaves that element as it was. It never hides a
   failure to make the size look better.

### Why a modeller would use it every week

- Before every issue to a consultant: one click, a file a fifth of the size, a proof sheet attached.
- When a client says "your file will not open": send the Review profile in two minutes.
- Before the model goes into a cloud tool with a per-GB cost or a file limit.
- As a diagnosis: the size map teaches the team which Revit habits make heavy IFCs (loose imported CAD, unused
  families, high-detail fittings), so the next export is lighter at source. A "send back to Revit" list is
  written: "Purge these 38 unused types. Switch these 12 families to coarse detail in IFC export."

### Honest limits

- Savings depend on the file. A well-made model with few repeated shapes will shrink little. The tool must say
  "this file is already lean: 8 % smaller" and not pad the result.
- "Same model" is a claim about the things the tool checks (ids, counts, quantities, shape bounds and volume). It
  is not a claim that every viewer will show every pixel the same. The report lists exactly what was checked.
- Sharing shapes (IfcRepresentationMap / IfcMappedItem) only works where the instances really are the same shape
  up to a move and rotation. The tool must detect that by hashing, as the viewer already does, and leave
  mirrored or scaled cases alone unless it can express them correctly.
- Some tools read shared shapes slowly or wrongly. The first release must be tested against Revit, Solibri,
  BIMcollab ZOOM, Navisworks and the That Open viewers with a real file and the result written down. I did not do
  this in this session.
- Property sharing and IFC schema details differ between IFC2x3 and IFC4. Check against the buildingSMART
  schema before building; I did not re-read the schema in this session.
- It does not repair a broken model. If it finds invalid references it reports them and does not guess (that is
  Export Doctor's job).
- Licences: reading with web-ifc and writing with cad2bim's own IFC writer keep this inside the allowed list.
  IfcOpenShell's IfcPatch is LGPL and its recipes are only a REFERENCE for what to look at; do not copy code or run
  it in the app (see the licence rule in the repo: no GPL or AGPL, "not even in a worker"; check whether LGPL is
  acceptable before even reading its source for ideas, and prefer writing from the IFC specification).

## Implementation

Where it sits in the repo (nothing in the source was changed in this session; this is a plan):

- **Engine (`packages/engine`).** The IFC reading and the writer already exist (`ifc/`, the IFC writer from 0.74).
  The viewer already keeps identical shapes once: `ifc/instances.ts`, `ifc/mappedPlacement.ts`,
  `model/synthetic.ts` and the notes in `docs/design/million-elements.md` ("meshes identical up to a move
  (hashed) are kept once"). Slim Shot reuses that hash to find shapes that can be shared in the FILE, not only in
  memory. New module, for example `packages/engine/src/slim/` with:
  - `measure.ts`: reads the IFC as a stream of entities, counts bytes per entity class, per product, per
    representation, per property set. This is the data for the treemap. (Pure counting; fast; no geometry needed.)
  - `plan.ts`: a list of steps, each with `id`, `safe | choice`, `bytesSaved` (computed), and `apply(model)`.
  - `apply.ts`: writes the new IFC by copying entities that stay, remapping ids, adding shared representation maps.
  - `proof.ts`: GUID set diff, per-class counts, per-class quantities, per-element bounding box and volume
    against the original, returned as data with `ok | differs`.
  - `profiles.ts`: the four profiles as data (JSON), so a team can edit and share their own.
- **Web (`apps/web`).** A "Slim Shot" panel in the file menu and in Export Doctor's result. Treemap with a small
  charting library already allowed by the licence list (MIT, BSD, ISC, Apache); check the dependency list before
  adding one, or draw it with the existing canvas code. Before/after slider using two viewer instances or one
  instance with two scenes. The proof report is a printable page (PDF through the existing Print / PDF path) and a
  JSON file.
- **Workers.** The work is I/O heavy. Run it in a worker and stream the file (read a slice, write a slice) so a
  400 MB file does not need 400 MB of heap. The geometry-paging and reopen-cache designs already deal with
  large buffers; reuse their approach.
- **Privacy.** No upload. No outside service. Proof report is made on the device. The Client profile removes author,
  machine and path text from the header on request.
- **Agent.** Expose `slim.measure`, `slim.plan`, `slim.apply`, `slim.prove` through `packages/mcp`, so a person can
  say "slim this to under 50 MB for the architect, keep every structural property" and the agent runs the same
  plan, shows the proof, and does not apply anything the proof fails.
- **Docs and versions.** Under the repo rules: a design note in `docs/design/`, a row in `docs/decisions.md`, a
  roadmap item, and a version bump with changelog, package README and `guide.ts` entry for each package changed.
  Dependencies (if any are added) must be checked against the licence list first.

### Build order (small steps, each useful on its own)

1. **Size X-ray only (about 1 to 2 weeks, my estimate, not measured).** Measure and draw the treemap, with the
   plain-sentence findings. Ships alone as a diagnosis tool. Already useful: it tells a team why their export is
   heavy, with no risk to any file.
2. **Safe steps with proof (2 to 3 weeks).** Share identical shapes, tidy text and precision, drop unreferenced
   entities and unused types. Proof report on. "Archive / Exact" profile only. Test on three real models in at
   least three other viewers.
3. **Profiles and the slider (2 weeks).** Coordination and Review profiles, side-by-side slider, difference mode.
4. **Client profile with IDS (1 to 2 weeks).** Needs the IDS reader from the QA roadmap.
5. **Round trip.** "Send back to Revit" suggestion list; a saved profile per consultant (what this architect's
   viewer can take).

### Tests to write first

- Round trip: slimmed Archive IFC read by web-ifc gives identical GUIDs, counts, quantities and per-element
  volumes. Zero tolerance for the Exact profile.
- A file with no repeats shrinks little and the tool says so.
- A file with broken references: reported, not fixed, no crash.
- A 400 MB file runs without exceeding a stated memory limit in the browser.
- Mirrored and scaled instances are not wrongly merged.
- Reading the slimmed file in two or three other viewers (manual check, written down).

## Future update

- **Slim at the source.** A Revit-side hint from the Bridge add-in: "these 38 types are unused, purge before
  export" and a saved IFC export setup that matches the size map's advice (docs/bridge/protocol.md).
- **Per-consultant profiles.** "This structural engineer's viewer chokes above 150 MB": remember it, and suggest
  the profile next time.
- **Diff of slims.** After a new revision, slim again and show only what changed since the last slimmed file
  (pairs with Change Radar, 20261014, and Issue Desk, 20261031).
- **Level-by-level and zone files.** Cut one IFC into one per level or per zone, each with the same GUIDs and a
  small index file, so a site engineer opens only their floor on a phone. (Takes the same proof.)
- **Capsule.** Feed a slimmed IFC to Model Capsule (20261104) so the one-file offline viewer starts small.
- **Geometry LOD files.** An extra "Review" level that keeps a low-detail shape beside the full one in the same
  IFC, so a viewer can pick. Check which viewers can use it before promising it.
- **Scheduled slim.** The agent slims the nightly export and posts the proof sheet where the team looks.

### Reel idea for LinkedIn / Instagram

15 to 20 seconds. Screen recording. A file called `STR_Tower_B.ifc` with a red label "342 MB" and a spinner "opening
… 5:41". The treemap blooms: one fat box labelled "4,212 chairs, same shape, 112 MB". Click, the chairs light up.
One click on "Coordination". The label drops to "71 MB". The slider drags across the 3D: you cannot see the
difference. A green stamp: "Same model: 0 GUIDs lost, volumes equal, 38 shapes simplified (listed)". The file opens
in two seconds. Caption: "Your IFC is not 342 MB. It is 71 MB and 271 MB of copies. Free, in the browser, nothing
uploaded." Posted as a before/after, which the feed likes, with an honest note under it saying which profile was
used and which files it was tested on.

### Sources (checked 2026-11-15)

- Solibri forum and Central Innovation tip on the IFC Optimizer and results: [Autodesk forum: IFC takes forever to open in Solibri](https://forums.autodesk.com/t5/revit-architecture-forum/ifc-exported-from-revit-takes-forever-to-open-in-solibri-model/m-p/7191682), [Central Innovation, IFC Tip Series Part 4](https://centralinnovation.com/?p=1631), [Solibri community post](https://society.solibri.com/post/14247)
- OSArch discussions of IFC optimising and the IfcPatch recipe (25 % safely removed in one example): [New IFC optimiser tool](https://community.osarch.org/discussion/comment/1800), [OSArch wiki: Revit and IFC geometry](https://wiki.osarch.org/revit-setup-for-openbim-revit-and-ifc-geometry/)
- IfcPatch recipes (Optimise, PurgeData): [IfcOpenShell ifcpatch source mirror](https://dev.glassvision.dedyn.io/Glassvision/IfcOpenShell/src/tag/bonsai-0.8.5-alpha2601272215/src/ifcpatch)
- Bentley's built-in IFC Optimization utility: [OpenBuildings Designer Help](https://docs.bentley.com/LiveContent/web/OpenBuildings%20Designer%20Help-v4/en/GUID-3CB7A5B3-5A2E-48E2-9116-E0B041AEF934.html)
- Research on IFC compression: [Liu, IFC Compressor (AIC 2015, PDF)](https://yushen-liu.github.io/main/pdf/LiuYS_AIC15IFCCompressor.pdf)
- 2026 trend reading (vendor and consultancy blogs, quality varies; the 27 % AI-use figure is not traced to a primary study and is not used above): [TechSoft3D, trends in AEC software 2026](https://www.techsoft3d.com/resources/blog/trends-in-aec-software-development-for-2026), [Novatr, State of BIM in 2026](https://www.novatr.com/blog/state-of-bim-adoption-trends-market-insights-future-outlook), [BD+C, BIM trends 2026](https://www.bdcnetwork.com/sponsored/article/55343350/the-bim-trends-that-will-define-aec-project-delivery-in-2026)
- No LinkedIn or Instagram post could be read in this session. The "trend" statement above comes from the blogs listed, not from the social feeds.
- In this repo: Export Doctor (20261104), Model Capsule (20261104), IDS + QA autofix (20261014), Change Radar (20261014), Issue Desk (20261031), Mark Merge (20261029); `docs/design/million-elements.md`, `geometry-paging.md`, `reopen-cache.md`, `ifc-writer-parameters.md` in cad2bim (read only).
