# 20261128 Idea: Odd Bits — the minor RCC members nobody drew (lintels, sill bands, kerbs, parapet and coping beams, chajjas, sunk-slab walls, plinth protection) found from the architect plan, modelled, reinforced and counted in BBS and BOQ, with a "model-complete" meter

## Entry 1

```
date-time:   2026-11-28 (scheduled run, UTC)
idea name:   Odd Bits — "The structure is modelled. The 6 to 10 percent of concrete and steel that sits in the small
             members is not. Find them, build them, count them."
             The architect's plan and the building's own rules imply dozens of small RCC members per floor that the
             structural drawings leave to "typical details" or to the site: lintels over every opening, sill bands,
             parapet and coping beams, toilet and balcony kerbs, chajjas, sunk-slab edge walls, lift-pit and tank
             curbs, plinth protection, RCC steps, AC-ledge slabs. Odd Bits reads the plan (or the open model),
             finds every place such a member belongs, builds it in 3D with the rebar the project's typical detail
             prescribes, adds it to the BBS and BOQ, and shows a "model-complete" meter: how much concrete,
             steel and shuttering the model had before and after, and which members were assumed.
potential:   High, because it closes a quantity gap that is in every project and is invisible until the bill.
             The daily pain: the modeller builds columns, beams, slabs and walls from the drawings. The model's
             concrete and steel totals come out lower than the site's by a few percent, every time, and nobody can
             say why. The usual answer is "add 5 percent for minor items" in the BOQ, a number nobody can defend
             to a client or an auditor. The small members are the reason, and they are also where site disputes
             start (a lintel without steel, a kerb poured as one with the slab, a parapet with no coping beam).
             My estimate, NOT measured: 6 to 10 percent of concrete and 4 to 8 percent of steel in a typical
             residential tower sit in members that the structural set does not draw one by one. The first test
             (below) is to measure that number on 3 of the team's own finished projects before anything is built.
             Time saved (estimate): 1 to 2 modeller-days per tower for the hand-modelling that is done late, plus
             the avoided argument over a "5 percent lump sum".
             Market: to my knowledge, not verified in this run, Revit has families for lintels, parapets and kerbs
             but places each by hand, rebar add-ons place bars in members the user has already modelled, and
             estimating tools add a percentage. I do not know of a tool that finds where the small members belong
             from the plan, builds them with their rebar and reports the model-complete number. Check before
             building: this is a thin search.
             Trending: "your BOQ is short and here is why" and "the 8 percent nobody models" fit the cost-and-
             quantity posts that engineers share on LinkedIn; treat this as a guess about interest, not a measured
             trend. The 3D before/after (the building with the odd bits lit up in one colour) is a good short clip.
             Extends: Wall Wizard (masonry-side lintels), Ratio Radar (flags missing quantity, cannot build it),
             Tag Tamer, Bar Optimizer and BBS Weaver (the BBS it feeds).
```

### Description

**The pain (a modeller's day).**
- The structural set draws 230 x 450 beams, 150 slabs, columns and footings. A note says "lintels over all openings
  as per typical detail T-7", "parapet 150 x 1000 with 8 mm @ 200", "kerb 100 x 150 at wet areas".
- The modeller models the major members. The small ones are "done later" and often never are, because there are
  hundreds, they are not on any sheet, and each is fiddly: height from the opening, bearing each side, a different
  size at a toilet door than at a 3 m window.
- The BOQ then carries a lump "add 5 percent". The client's QS has a different number. Both are guesses.
- On site the carpenter and the bar bender build them from experience, the quantity runs over, and the model is
  blamed for being short.

**What Odd Bits does.**

1. **Find the places.** From the open model (walls, openings, slab edges, level changes, wet rooms, terraces) or
   from the DXF/DWG the pipeline already reads, it lists every place a minor member is implied:
   - *Lintels:* above each door and window opening in a masonry wall that has no beam over it. Bearing, depth and
     width by span and wall thickness from the project's rule table.
   - *Sill bands:* under windows, where the rule table asks for them.
   - *Kerbs and sunk-slab edge walls:* at toilets, kitchens, balconies and terraces, from the finish and sunk levels.
   - *Parapets and coping beams:* at every exposed slab edge and terrace, height from the rule, with joints at
     expansion lines.
   - *Chajjas and AC ledges:* from the plan's window and AC-unit symbols.
   - *Pit, tank and shaft curbs, plinth protection, RCC steps and ramps' edge walls.*
2. **Show the rule behind each.** The rule table is plain data: "opening up to 1.2 m: bearing 150, depth 150;
   1.2 to 2.1 m: bearing 200, depth 200 with 2 x 12 mm", with the code or detail sheet it came from. The table is
   the project's own typical details (read from the notes, or typed once), not the tool's opinion. A rule the
   tool cannot find is shown as an open question, not guessed.
3. **Build them.** Each member is created in 3D as a real element (IFC class, type, mark, level, host wall or slab),
   with rebar made from its detail, using the same rebar and BBS machinery as the major members.
4. **Count them.** They join the BBS (bar marks, shapes, cut lengths, weights), the BOQ (concrete, shuttering
   area, steel) and the schedules. Each line says where it came from: "drawn", "from typical detail T-7",
   or "assumed".
5. **Model-complete meter.** One panel: concrete, steel and shuttering of the structure as drawn, the odd bits
   added, the new total, and the percent each odd-bit kind adds. A confidence word per line (drawn, rule-based,
   assumed). A before/after toggle lights the added members in one colour in 3D.
6. **Stay in step.** When a revision moves a door, the lintel moves with it, and the change is one line in the
   revision note ("W-12 moved 450 mm: lintel L-31 re-sized, 0.04 m3, 6 kg"). Members the person edited by hand are
   not overwritten; they are flagged as "hand-edited, check".

**What it is not.** It does not design. Sizes and bars come from the project's typical details and rule table. If a
member needs a design (a long cantilever chajja, a heavy parapet with a tank), it is flagged for the engineer, not
sized.

### Implementation

**Reuse what exists.** (Read for this run: `CLAUDE.md`, `docs/roadmap.md`, `docs/design/next-milestones.md` and
`docs/design/cad2bim-speed.md` in the cad2bim repository, and the existing idea files in this folder. The code was
not built or run, and I did not read the rebar or BBS code.)
- The Rust pipeline (`packages/pipeline`) already reads walls, openings (doors and windows hosted in a wall as Revit
  hosts them, 0.101.0), slabs, levels and sunk or stacked slabs from CH v2 drawings, and writes the IFC. Odd Bits'
  "find the places" step is a rule pass over the same analysis, so it can run before any IFC is opened.
- The engine already has BOQ and BBS, and the roadmap lists "legged shapes" in `docs/design/legged-shapes.md`; the
  odd bits' bars (stirrups, U-bars, straight bars) use the same shapes.
- The create API (columns, beams, slabs, walls and footings, 0.85.0) and the fast many-at-once creation (0.86.0)
  already put elements into the model in one update; lintels and kerbs are new element types on the same path.
- Revision (0.101.0) rebuilds only what changed and keeps IFC ids; odd bits rebuild with the host they hang on, so a
  moved opening moves its lintel and its id stays.
- QA findings already select and fly to elements; "opening with no lintel" and "exposed slab edge with no parapet"
  become QA checks even for people who do not build the members.
- The agent helper (MCP) can expose `odd_bits_scan(model)` and `odd_bits_build(kinds)`, so an agent can run it and
  summarise the meter.

**New pieces.**
1. **Rule table as data**, one JSON file per project, with the code or sheet cited per rule; a starter table for
   Indian residential practice, to be checked by the team's senior engineer, not trusted as shipped.
2. **Place finder** for each kind (openings, wet areas, slab edges, levels). The hard part is the host: which wall,
   which side of the wall, at which level. First version builds only where the host is certain and lists the rest.
3. **Element and rebar builders** for lintel, sill band, kerb, parapet, coping beam, chajja, curb. Each a pure,
   tested function from (place, rule) to (shape, bars).
4. **Model-complete panel**: totals before and after, per kind, with confidence; export to Excel next to the BOQ.
5. **Source tag** on every added element (a property: `OddBits.Source` = drawn, rule, assumed, plus the rule id) so
   it survives in the IFC and the BOQ can filter by it.

**Privacy.** Everything runs on the device. Nothing is uploaded.

**Test plan (what must be true before it is trusted).**
- *First, before building anything:* take 3 finished projects where the site's final concrete and steel are known.
  Compare them with the model's totals. Record the real gap and how much of it the small members explain. If it is
  under 2 percent, stop; the idea is not worth building.
- Run the finder on the same 3 projects. For 30 openings chosen at random, an engineer checks "lintel needed here,
  yes or no, and the size"; the tool must agree on at least 27, with each miss shown as a question, not a wrong
  member.
- Compare the tool's added concrete and steel with the site's actual quantities for those members (from the RA bills
  or the steel ledger). Record the error per kind. This is measured, not estimated.
- A 20-storey tower must scan in under a minute and build in under two on a normal laptop; measure and record.
- Show the meter to two QSs and one consultant and ask only: "would you put this number in a tender?". Keep the
  negative answers.

### Future update

- **Learn the team's practice:** the rule table improves from corrections (a lintel the engineer enlarged becomes a
  rule for that span), shown as a suggestion with the three projects it came from, never changed silently.
- **Link to Wall Wizard:** the masonry side (blocks, mortar, plaster) and the RCC side (lintel, sill band) agree on
  one opening, one height and one finish, so a moved window changes both bills.
- **Link to Ratio Radar:** its steel kg per m3 and concrete per m2 benchmarks get a second line, "with odd bits",
  so a project is compared on the same footing as its past projects.
- **Link to Pour Planner and Shutter Plan:** kerbs and parapets become pour items with their own joint and shuttering
  cycle, which is where the site's stoppages are usually hidden.
- **Link to Change Radar and Chit Edit:** the revision note includes the odd bits' change in the same cost delta.
- **Precast option:** a lintel can be marked precast, with a casting list and a count for the yard.
- **Other codes and regions:** the rule table is data, so a team in another country supplies its own typical details.

*References read for this run: the cad2bim repository's `CLAUDE.md`, `docs/roadmap.md`,
`docs/design/next-milestones.md` and `docs/design/cad2bim-speed.md`, and the existing idea files in this folder (to
avoid repeating them; Wall Wizard, Ratio Radar and Window Wright were read for overlap). No web search was run, so
the market and trend lines above are my own knowledge and guesses, not checked. Nothing in the source repository
was changed.*
