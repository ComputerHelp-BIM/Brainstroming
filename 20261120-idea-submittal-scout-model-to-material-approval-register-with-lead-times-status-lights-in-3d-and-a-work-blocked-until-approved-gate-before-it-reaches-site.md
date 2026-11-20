# 20261120 Idea: Submittal Scout — The 3D model writes the material approval register for you (every grade, make, bar, waterproofing, tile, door and pipe that needs sign-off), each approval gets a status light on its elements in 3D, a lead time that feeds the 4D, and a "work blocked until approved" warning before the gang reaches an element whose material nobody approved.

## Entry 3

```
date-time:   2026-11-20 (scheduled run, UTC)
idea name:   Submittal Scout (model to material-approval / submittal register, status lights in 3D,
             lead-time to 4D, blocked-work gate; in the browser, nothing uploaded)
potential:   High for contractors and PMC teams; medium-high for the BIM team's standing.
             Every site keeps a material approval register (MAR / submittal log) in Excel: cement,
             TMT steel, RMC mix designs, admixture, waterproofing, tiles, doors, MEP items. It is typed
             by hand from the BOQ and spec, it is not linked to the model, and nobody sees which
             elements are waiting on an approval. Common trouble: work starts on material that is not
             yet approved, or an approval takes 3 weeks and the order date is missed.
             Time saved: NOT measured.
             First test: take 2 real projects (kept out of git). Build the register by hand with a
             stopwatch; then with Submittal Scout. Compare (a) minutes, (b) items the hand register
             missed, (c) items Scout listed that the site says need no approval (false positives).
             Do not quote hours or accuracy until this is done.
             Market check (2026-11-20). One standard web search only. Found: vendor pages and blogs
             (Datagrid, Nomic, BuildSync) on AI-assisted submittal review, and one article saying AI
             agents can pre-fill fields from BIM metadata and link submittals to building models. These
             are vendor claims. A quoted "rejection rate 35% to 5%" is from a vendor and I found no
             independent check, so it is NOT used here. Not verified by me: the price of each product,
             and how far each links an approval to single 3D elements. Procore and ACC Build (both paid)
             are widely used for submittal workflow; I did not check their current features.
             What I did not find in this search: a free, browser tool that starts from the 3D model,
             builds the register, and shows approval status on the elements. "I did not find one",
             not "none exists".
             Trend: AI for submittals and preconstruction paperwork is a live topic in industry
             coverage (search above). I could not measure LinkedIn or Instagram counts, so this is
             not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Spec Sage: turns the spec into model rules and checks. Scout takes the same
                 material clauses and turns them into approval items with owners and dates.
               - Order Clock: order calendar from 4D and BBS. Scout adds the step before it: an
                 order cannot be placed until the item is approved, so approval lead time is added.
               - Ready Board: 3-week lookahead readiness. "Material approved" becomes one of its checks.
               - Bar Swap and Bid Mirror: a swap or a bidder's make is a new submittal. Scout opens it.
               - Handover Hub: approved makes and warranty data flow on into the O&M and COBie pack.
               - Issue Desk and Second Eyes: same pattern of register + gate + sheet, for materials.
             The wow: open the model, press "Scout". In seconds a register of 140 items appears
             (grouped by trade). The 3D turns red/amber/green by approval status. Press play on the 4D:
             when the date reaches a slab whose waterproofing is still amber, the slab flashes and a
             banner says "Terrace waterproofing starts in 12 days. Approval not yet received; vendor
             lead time 21 days. You are already 9 days late." One click drafts the reminder and the
             material approval request sheet, filled in from the model.
```

### Description

**The pain.**
- The MAR register is built from the BOQ by hand, once, and then goes stale.
- Nobody can answer "which floors are affected if the tile approval slips?" without a long search.
- Approval lead time (sample, test certificate, consultant review) is not in the 4D schedule, so
  orders go out late and work starts on unapproved material.
- A rejected make forces a change in the model, the BBS or the BOQ, and nothing tells the modeller.
- Modellers are asked "what is the make and grade of this?" all day and answer from memory.

**The idea.** Four moves.
1. **Build the register from the model.** Group elements by what needs approval: concrete grade and
   mix, rebar grade and diameter, couplers, formwork type, waterproofing system, blocks, tiles, doors,
   windows, pipes, cables. Each group becomes an item with quantity, the elements behind it, the
   spec clause (if a spec is loaded) and the usual documents needed (mix design, mill test certificate,
   sample, data sheet). A rule table decides what needs approval; it is editable and saved with the
   project, so a company's own habit lives in it.
2. **Track status.** Each item has a status (not raised, sent, approved, approved with comments,
   rejected, resubmit), owner, dates and attachments' names. Status is typed, pasted from Excel, or
   changed by a person. Nothing is sent anywhere by the tool.
3. **Colour the model.** 3D lights per status. Click an item to select all its elements; click an
   element to see its items ("this beam: M30 mix design approved; rebar Fe550D approved; coupler
   make rejected, resubmit").
4. **Put time and a gate on it.** Each item has a lead time (default per category, editable). The 4D
   start of the first element using it, minus the lead time, is the "raise by" date; minus ordering
   time as well, the "approved by" date. Items past either date are flagged. The "blocked" gate lists
   tasks that start in the next N days on an unapproved item; the Ready Board shows the same.

**What a user sees.**
- A "Submittals" panel: the register, filters by trade, status and "late", and a summary
  ("23 late, 41 due in 2 weeks").
- Per item: the 3D elements, a drafted request sheet (project, element list, quantity, spec clause,
  required documents), and a history of status changes with who changed them.
- When a make is rejected: a list of what changes (rebar diameter, door size, tile thickness) and a
  button to open the right tool (Bar Swap, or the type editor).

### Implementation

Inside the cad2bim app; the register lives in the project file (versioned like other project data).
Nothing leaves the device unasked.
- **Reading the model:** existing IFC reading in packages/engine (types, materials, property sets,
  quantities). Grouping rules are plain data: "all IfcReinforcingBar by grade and nominal diameter",
  "IfcDoor by type name and fire rating", and so on. Missing data (no grade on a column) is listed as
  "grade not stated" rather than guessed.
- **Rules:** a small editable table (category, grouping key, documents, default lead time, default
  review days). A starter table for Indian sites (IS grades, mill test certificate, cube test,
  third-party test report) is shipped as a template; a company can keep its own.
- **Spec link:** if Spec Sage has a spec loaded, clause text and numbers are attached to items. The
  link is by clause id and shows the source text, so a reader can check it.
- **4D link:** reads task start dates through the existing 4D data. A pure function computes
  "raise by" and "approved by" dates from task start, lead time and review days, tested with Vitest on
  small hand-checked cases.
- **Status in 3D:** colour overrides through the existing viewer selection and colour code; no new
  rendering.
- **Import and export:** register in and out as CSV or Excel (the Sheet Sync reader is reused), plus a
  one-page PDF request sheet per item through existing export code.
- **Agent access:** MCP tools in packages/mcp: "late submittals", "what blocks this task",
  "items for this element". An agent can answer "what is holding up level 5 slab?" with the same
  engine a person uses.
- **Build order:** (1) register from the model with the rule table and CSV out; (2) status and 3D
  colours; (3) lead time and the 4D gate; (4) request sheet PDF; (5) Spec Sage and Bar Swap links;
  (6) MCP.
- **Risks:** what needs approval differs per contract, so the rule table must be easy to edit and the
  first run must show why each item is on the list; a model with thin material data gives a thin
  register, and the page says so; the tool tracks status, it does not replace the consultant's
  approval and must say "record, not approval".
- **Check before release:** 2 real projects, the stopwatch test above, and a site engineer who
  reviews the list for false positives.

### Future update

- **Reviewer view:** a read-only one-file page (see Model Capsule) so a consultant can open an item,
  see the elements in 3D, and reply with a status file that lands back in the register.
- **First-pass document check:** read a vendor data sheet or test certificate PDF on the device and
  compare grade, size and standard number against the item; show the mismatch, never auto-approve.
- **Learning from rejections:** keep a company list of "rejected with reason" and warn when the same
  make is proposed again on another project.
- **Cost link:** a different make changes the rate; show the cost delta in Rate Rocket and the carbon
  change in Carbon Lens before the request is sent.
- **Handover:** approved items, makes, certificates and warranty periods flow into Handover Hub's
  O&M manual and COBie sheet with no retyping.
- **Portfolio view:** one board across a contractor's sites for items late on more than one project,
  so purchase can bundle and chase.

Sources used for the market check (vendor material, treat as claims):
[Datagrid, automate submittal management](https://datagrid.com/agents/for/automate-submittal-management),
[Nomic, AI agents reshaping submittal review](https://www.nomic.ai/blog/ai-agents-reshaping-submittal-review),
[Nomic, AI-powered preconstruction](https://www.nomic.ai/blog/rise-of-ai-powered-preconstruction).
