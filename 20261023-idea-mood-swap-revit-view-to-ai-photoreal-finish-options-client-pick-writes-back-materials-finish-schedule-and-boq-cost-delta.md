# 20261023 Idea: Mood Swap — one Revit view to six AI photoreal finish options in a minute, the client taps one, and the chosen finishes write back to Revit materials, the room finish schedule and a BOQ cost delta

## Entry 1

```
date-time:   2026-10-23 (scheduled run)
idea name:   Mood Swap — "Pick a 3D view of the lobby, living room or elevation. In about a minute you get
             six photoreal renders in different finish moods (Warm Teak, Italian Marble, Japandi, Industrial,
             Budget Smart, Client's Pinterest board), each one locked to the real model: same walls, same
             windows, same camera, and every surface tied to a Revit material and a priced catalogue item.
             The client taps the one they like on a phone, or taps a single surface ('floor from option 2,
             walls from option 5'), and one click writes those finishes back into Revit materials, the room
             finish schedule and the finish BOQ, with the rupee difference against the base design."
             One engine with four front ends:
               (1) pyRevit button (AnonGee tab, Essential panel): reads the active 3D view, camera, elements
                   and materials; sends passes to the render engine; later applies the chosen finishes,
               (2) cad2bim web viewer: the client "mood board" link. Swipe through the six renders,
                   before / after slider against the plain Revit view, tap a surface to see the product,
                   rate, quantity and cost, mix-and-match, approve,
               (3) C2B: early-stage mode on a client DXF plan (rooms extruded to a quick shell) so a mood
                   pack can go out before the Revit model exists,
               (4) structura / Wall Wizard / Bill Painter: the finish quantities and the cost delta.
             Outputs: six (or N) photoreal renders per view at 2K, a mask-true "material map" for each
             render, a finish selection sheet (surface, product, size, rate, area, amount), cost delta vs.
             base, client approval record (who, when, which option, per surface), and updated Revit
             materials / finish parameters / finish schedule.
potential:   Very high, on every architecture and interior project, every week. Finishes are the most
             argued-about decision with clients and the slowest loop between design, BIM and QS.
             Time today (one living room or one lobby, three finish options):
               - set up materials and lights in Enscape / Lumion / D5 / V-Ray per option: 2-4 hours each,
               - render, retouch, put in a PPT: 1-2 hours,
               - client picks "the floor from the second, walls from the first": re-render, another
                 half day,
               - then someone retypes the decision into the Revit materials and the finish schedule (often
                 never), and the QS prices it from a WhatsApp message: 2-4 hours, frequently wrong,
               - total: 1.5-3 days of visualiser + BIM + QS time per space, per round.
             With Mood Swap: 5-15 minutes per view including review; the client decision lands directly in
             the model and the BOQ. On a residential tower with 4-6 typical spaces (lobby, living, kitchen,
             master bed, toilet, elevation) and 3 rounds of changes, that saves 3-6 weeks of team time and
             closes finish decisions in one meeting instead of three.
             Quality saved: renders that do not match the model (the visualiser "fixed" a window); decided
             finishes never written into the model, so the finish schedule, the room data sheets and the
             BOQ disagree; surprise cost escalation when the client's marble is priced only after it is
             ordered; no record of which option the client actually approved.
             Trending angle: AI architectural rendering is one of the most shared themes on LinkedIn and
             Instagram in 2025-2026 ("sketch to render in 10 seconds", "Revit to photoreal with AI",
             before / after interior reels). Almost all of these stop at a pretty picture. The reel here
             goes one step further: grey Revit view -> six moods fan out like cards -> the client taps
             "Italian Marble" -> the Revit floor material changes live -> the finish schedule updates ->
             a counter shows "+₹4.8 lakh vs. base". 30 seconds, and every frame is a real model.
             Market: Veras (EvolveLAB, paid subscription, renders from Revit with AI but does not lock
             each surface to a Revit material or write choices back, no pricing); Enscape / D5 / Lumion
             AI features (paid, material-accurate only after manual material setup); LookX, mnml.ai,
             ArkoAI, PromeAI and similar (paid web tools, image-only, no BIM link, geometry often drifts);
             ComfyUI + ControlNet (free, but an expert workflow with no Revit, no materials, no cost). To
             our knowledge nothing free, or even paid, goes Revit view -> material-locked AI options ->
             client pick per surface -> Revit materials + finish schedule + BOQ cost delta in one loop.
```

### Description

**The pain (daily life of a BIM modeller / visualiser in an Indian architecture or interior office).**
- The client cannot read a finish schedule. They want to see the room. Every option means a half-day of
  material setup and rendering, so the office shows two options instead of six, and the meeting ends with
  "send me a few more".
- AI image tools are fast but they invent things: an extra window, a column gone, a ceiling twice as high.
  The architect cannot send that to a client without a disclaimer, and the BIM team cannot use it at all.
- When the client finally chooses, the decision lives in a WhatsApp chat and a PPT. Revit still shows the
  default "Generic - Floor" material, the finish schedule says "TBD", and the QS prices "Italian marble"
  at whatever rate they remember.
- Each round of "change only the wall colour" restarts the loop.

**The idea, in seven parts.**
1. **Model-true passes from the Revit view.** The pyRevit button takes the active 3D view (perspective
   or orthographic) and exports, with the same camera: a clean shaded image, a depth pass, a normal pass, a
   line / edge pass, and a **material ID mask** where every pixel carries the Revit element id, category,
   room and material. Geometry goes through the existing bridge to glTF; the passes are rendered headless
   in three.js (the cad2bim engine already renders IFC / glTF), so they are pixel-exact to the model.
2. **Surface groups.** The mask is grouped into editable surfaces: floor per room, skirting, walls (by
   wall type and by room face), feature wall, ceiling / false ceiling, doors, windows / frames, kitchen
   counter, railing, facade cladding. The modeller can rename, merge or lock a surface ("do not touch the
   structure", "keep the client's existing sofa").
3. **Mood packs and a priced catalogue.** A mood pack (YAML) is a set of surface -> catalogue-item choices
   plus a style prompt, e.g. Warm Teak: floor = engineered oak plank 190 mm, walls = warm white emulsion,
   feature wall = teak fluted panel, ceiling = gypsum with cove. Each catalogue item carries size, finish,
   texture swatch, unit, rate, wastage and lead time; the office keeps its own catalogue (Excel / YAML), with
   a starter pack of generic Indian items (vitrified 600x1200, Italian marble, granite, laminate, emulsion,
   texture paint, PU wood, ACP, HPL, glass, MS / SS railing). A "Client's Pinterest" mood is built from 3-5
   reference photos: an image model picks the nearest catalogue items for each surface.
4. **Mask-locked AI render (the unique part).** A diffusion model (SDXL / Flux class) runs with
   ControlNet-style conditioning on depth + edges + normals, so geometry and camera cannot drift, and with
   **regional prompts per surface mask**, so the floor gets the floor product and only the floor. Swatch
   textures are passed as image prompts per region (IP-Adapter style) so "Kajaria-like 600x1200 grey
   vitrified" looks like that tile, at the right joint size from the real tile module. A post check
   compares edges of the render with the line pass and rejects or re-seeds any option that moved a wall,
   window or door beyond a pixel tolerance. Same seed + same mood across views keeps the living room, the
   dining and the balcony consistent.
5. **Client pick, per surface.** The six renders go to a cad2bim web link (or the Revit dialog in the
   meeting). Swipe, compare against the plain Revit view with a before / after slider, tap any surface to
   see product, quantity (from the model), rate and amount. Mix-and-match "floor from 2, walls from 5"
   re-renders only those regions in seconds (inpainting on the mask). The client presses Approve; the link
   records name, time, option and the per-surface choices as an approval record.
6. **Write-back to Revit.** One click applies the approval: create or reuse Revit materials named from the
   catalogue (`FLR_ItalianMarble_Statuario_800x1600`), set appearance colour / texture from the swatch, set
   the finish parameters on rooms (`Floor Finish`, `Wall Finish`, `Ceiling Finish`, `Skirting`), paint wall
   faces per room where the wall is shared, swap the floor finish type, and update the finish schedule.
   Everything in one transaction group with a single undo, and a "selected finishes" legend on a sheet.
7. **Cost delta and downstream.** Quantities come from the model (room floor area, net wall area after
   openings from Wall Wizard, skirting length, ceiling area), times the catalogue rate + wastage. The sheet
   shows each option and the chosen mix against the base design: "+₹4.8 lakh, +6.2 %, 21-day lead time on
   marble". The quantities go to the structura BOQ and Bill Painter for later billing; Model Reel can turn
   the approved renders into a before / after clip; Drawing Autopilot puts the finish legend on the
   interior sheets.

**Why it fits this toolkit (extends what already exists).**
- **pyRevit-addons (AnonGee)** already has the WPF UI templates, the `bridge/` link to Python 3, an
  Export Schedule tool and parameter tools (Parameter Combine, Filter Parameter). Mood Swap reuses those
  to read views and write materials and finish parameters; it is a new button, not a new platform.
- **cad2bim web** already renders IFC / glTF with three.js in the browser and keeps files on the user's
  device; the passes (depth, normal, ID mask) are extra render targets, and the client mood link is a
  viewer plugin.
- **C2B** extrudes client DXF plans; a quick room shell is enough for early-stage mood packs before the
  Revit model is ready.
- **Wall Wizard** (finish BOQ, net plaster / paint areas), **Bill Painter** (RA bills), **structura** BOQ
  and **Model Reel** (video) all consume the approved finishes; **Markup Mirror** can treat a client's
  redline on a render ("darker floor") as a mood edit.

**Wow moments.**
- A plain grey Revit view becomes six photoreal moods in about a minute, and a toggle shows that every
  wall, window and column is exactly where the model has it.
- The client taps the floor in option 2 and the walls in option 5; only those regions repaint in seconds.
- Approve on the phone -> the Revit floor and walls change material live in the meeting, the finish
  schedule fills in, and the cost counter shows the rupee delta and the lead time.
- The same mood is applied to all six typical spaces with consistent materials, ready for the brochure.

### Implementation

**Phase 1 — Passes and surface map (2-3 weeks).**
- pyRevit button: read the active view camera (`View3D.GetOrientation()`, eye, target, up, field of view
  from the crop box), visible elements and their materials (`Element.GetMaterialIds`, face materials,
  painted faces via `Document.GetPaintedMaterial`), rooms per face (`SpatialElementGeometryCalculator`).
- Geometry to glTF through the bridge with element id + material id per mesh (reuse the cad2bim engine /
  IfcOpenShell path for IFC models).
- Headless three.js (Node + headless GL, or Playwright + Chromium) renders the shaded, depth, normal, edge
  and ID passes at the same camera; ID colours decode to element / material / room.
- Surface grouping rules in YAML (`surfaces_default.yaml`): category + room + face orientation -> surface
  name; editable in the WPF dialog with a click-to-merge preview.
- Tests: a sample room with known element ids, assert every mask pixel maps to a real element and the
  edge pass overlays the Revit image export within 1 px.

**Phase 2 — Render engine, mood packs, catalogue (3 weeks).**
- Render service with pluggable back ends: local ComfyUI (free, office GPU, a saved workflow with depth +
  canny + normal ControlNets, regional prompt masks and IP-Adapter swatches) or a cloud API (Replicate /
  fal / Stability) for offices without a GPU. Same job JSON for both.
- Mood packs and catalogue: `moods/*.yaml`, `catalogue.xlsx` (item, surface type, size, finish, swatch,
  unit, rate, wastage %, lead time, brand-neutral name); a starter catalogue with typical Indian finishes.
- "Pinterest mood": CLIP-style image embeddings of reference photos and catalogue swatches, nearest match
  per surface, shown for approval before rendering.
- Geometry guard: edge-map IoU between render and line pass per surface; reject or re-seed below the
  threshold; report "verified to model" on each image.
- Inpainting for per-surface mix-and-match using the same masks and seed.

**Phase 3 — Client link, write-back, cost (3 weeks).**
- cad2bim viewer plugin: option carousel, before / after slider, tap-surface card (product, area from the
  model, rate, amount), mix-and-match, approval with name and timestamp; export the approval as JSON + PDF.
- pyRevit write-back: create / duplicate materials, set appearance asset colour and texture from the
  swatch, set room finish parameters, paint wall faces per room, swap floor finish types by mapping, write
  a "Selected Finishes" legend and schedule. One `TransactionGroup`, undo, and a diff report of what
  changed.
- Cost engine: quantities from the model (room areas, net wall areas, skirting lengths, ceiling areas,
  door / window counts), catalogue rate x (1 + wastage), base vs. option vs. chosen mix, lead-time flag;
  Excel + PDF sheet; push lines to structura BOQ.

**Tech notes / risks.**
- AI images are design intent, not a material sample. Every render carries "AI visual, colours indicative;
  approve physical samples", and the approval record lists catalogue items, not pixels.
- Geometry drift is the trust killer: conditioning on depth + edges + normals plus the automatic edge check
  is mandatory; any render that fails is not shown.
- GPU cost and privacy: local ComfyUI keeps client designs in the office; the cloud back end is opt-in per
  project, with images only (no model file) leaving the machine.
- Revit material appearance assets are fiddly to edit across versions; start with colour + a generic
  texture and map to the office's own material library by name in a JSON config.
- Rates change often; the catalogue carries a date and the cost sheet prints it. Rates are office data, not
  shipped as truth.
- Model licences: use permissively licensed weights for commercial use and record the model and version
  in each render's metadata.

### Future update
- Video moods: a 10-second walkthrough per mood from the same passes along a camera path (Model Reel),
  with temporal consistency, for Instagram and the client WhatsApp group.
- Lighting moods: day / evening / night with the real Revit light fixtures and Ceiling Composer's lux
  plan, so the "cove light" render is backed by real fittings.
- Facade mode: cladding, ACP, glass tint and paint moods on elevations with the Sun Compass shadows, and the
  facade material BOQ.
- Supplier link: each catalogue item points to dealer availability and lead time; approved items become a
  purchase list grouped by vendor.
- Sales mode for builders: "choose your flat finish" upgrade packs with prices, picked by buyers on the
  web link and written to a per-flat customisation schedule.
- Site QR Pass: scan the room QR on site and see the approved finish render and product list, so the
  contractor lays the right tile.
- AI mode: "keep the budget under ₹12 lakh and make it feel warmer": the tool swaps catalogue items within
  budget, re-renders and shows the new delta.
