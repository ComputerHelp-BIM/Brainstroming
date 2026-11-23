# 20261123 Idea: Pour Proof — Photograph each RMC delivery challan as the truck arrives; the app reads it, adds it to that pour, and compares the cubic metres delivered with the 3D model's volume. It tells the foreman how much is still needed for the last truck, flags short loads and wastage, and writes the supplier claim sheet. On the device, nothing uploaded.

## Entry 1

```
date-time:   2026-11-23 (scheduled run, UTC)
idea name:   Pour Proof (cad2bim model + RMC challan photos or Excel to a per-pour concrete ledger:
             delivered vs modelled vs placed, a last-truck calculator, short-load and wastage
             alerts, and a supplier claim sheet)
potential:   High for Indian contractors, PMC teams and BIM teams that already hold concrete
             volumes in the model. Medium-high "wow": on pour night, a phone photo of the challan
             moves a bar on the 3D slab ("38.5 of 42.0 m3 delivered"), and the last-truck number
             appears before the pump is released. Concrete is bought by the cubic metre every
             week on every RC job, so the habit forms fast.
             Time saved and money saved: NOT measured. The pain below comes from public sources
             and team experience, not from a timed or costed study.
             First test: take 3 recent pours with their signed challans (kept out of git). Add up
             the challans by hand, compare with the model volume per pour, and note the gap and
             the time it took. Then run Pour Proof on the same pours and compare its gap, its
             read errors and its time. Do not quote savings until this is done.
             Market check (2026-11-23). Two standard web searches. Found: ready-mix and
             dispatch software for the PRODUCER side (Command Alkon, ConcreteGo, Linkoper and
             similar) with ticket reconciliation and invoice matching, and a construction app
             (Novade) that adds up delivery receipts per supplier. Guidance articles say a
             contractor can compare concrete placed so far with the volume still needed, to
             time the final order. One RMC quotation says the invoice is raised on the signed
             challan quantity, not on actual use; IS 4926 summaries say site water may be added
             only with the producer's representative present and recorded on the challan.
             What I did NOT find: a tool that compares the delivered volume with the BIM
             model's volume element by element, or that gives the last-truck number in 3D.
             "I did not find one", not "none exists". Vendor pages are marketing, so features
             and prices are not verified. I found no reliable RMC wastage percentage, so none
             is quoted here; the app uses the team's own history.
             Trend: "waste and cost leakage on site" and "AI reading site documents" posts are
             regular on LinkedIn. I could not measure counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Pour Planner (4D RMC, formwork cycle): plans trucks and pumps BEFORE the pour.
                 Pour Proof checks what ARRIVED and was placed, after and during it.
               - Steel Ledger: reconciles steel challans and weighbridge slips with the BBS.
                 Pour Proof does the same for concrete, which is bought per pour, not per bar.
               - Cube Clock: cube results and strike-ready map. Pour Proof links each challan
                 (grade, slump, time out of plant) to the cube set taken from that truck.
               - Raft Run: mass-pour roster and cold-joint clock. Pour Proof is the delivered
                 volume ledger that feeds its live board.
               - Bill Painter / Gang Pay / Measure Right: bill what was built. Pour Proof proves
                 how much concrete went in and what the supplier owes or is owed.
             The wow: open the model on a phone at the slab. Tap "Pour 4: Level 6 slab, 42.0 m3
             modelled". Photograph challan 1 of 7: the app fills date, plant, truck, grade M30,
             6.0 m3, slump, batch time, water-added note. After truck 6 the slab is 85% blue.
             The app says: "Delivered 36.0 m3. Modelled 42.0 m3. Last truck: 6.0 m3, plus 0.3
             m3 spare for pump line and edge forms (your rule, not a standard)." After the pour:
             one sheet with delivered 42.5, modelled 42.0, left over 0.5, and truck 3 flagged
             "challan 6.0 m3, batch-to-site time 2 h 40 min".
```

### Description

**The pain.**
- The foreman counts trucks on a notebook page. The challans pile up in a bag. At month end someone
  adds them by hand and argues about the gap with the supplier.
- The last truck is a guess: order too much and it is wasted (and the leftover is dumped);
  order too little and a cold joint appears.
- The modelled volume and the delivered volume live in different worlds. Nobody sees pour by pour
  where concrete went missing: openings, over-break, a sunk slab drawn wrong, or a short load.
- A short or late truck is noticed on site but not recorded against the challan, so the claim
  fails later.
- The supplier invoices from the signed challan. If the challan is wrong, the invoice is wrong,
  and the only proof is the photo in someone's phone.

**What Pour Proof does.**
1. **Pour list from the model.** Every pour (the Pour Planner or Joint Judge pours, or one the
   person draws) has its elements and a modelled volume, with the deductions the model already
   knows (openings, embedded items over a size the team sets).
2. **Challan in.** Photograph the challan, drop a PDF, or paste a supplier's Excel sheet. The
   reader fills a fixed set of fields: date, plant, truck, grade, volume, slump, batch time,
   arrival time, water added, signature present. Every field shows the part of the photo it came
   from, and the person confirms with one tap. Low-confidence fields are amber, never silent.
3. **Live pour ledger.** Delivered, modelled, placed and left over per pour, with the slab or beam
   set coloured by fill fraction in 3D.
4. **Last-truck calculator.** Modelled minus delivered, plus the team's own allowance (a number
   they set from past pours, not an app default and not a standard). It also shows the next
   truck's arrival gap so the pump is not released early.
5. **Alerts, all explained, none automatic accusations.**
   - Short load: challan volume below the truck's stated capacity, or the sum of challans below
     the order, shown as a question for the supplier.
   - Time and slump: the time between batch and arrival above the limit the team sets; slump
     outside the ordered range; water added without the producer's representative recorded.
   - Over-use: delivered above modelled by more than the team's tolerance, with the likely
     reasons listed from the model (over-break, thicker topping, a drawn-wrong sunk zone).
   - Duplicate or missing challan numbers in the sequence.
6. **Cube link.** Each challan can carry the cube set number, so Cube Clock gets the truck, grade
   and time without retyping.
7. **Supplier claim sheet.** For the month or a pour: ordered, challan, delivered (as the site
   confirmed it) and invoiced side by side, the flagged trucks with their photos, and a clear
   difference in cubic metres. It states what is proven and what is a question.
8. **Learning.** After a few pours the app shows the team's real over-use per element type
   (slab, beam, footing, wall) from its own history, so the allowance is a measured number.

**What the person sees.**
- A *Pour Proof* tab on the phone and on the desk: pour list, then one pour with a 3D fill view.
- A camera button, and an "add by Excel" button.
- Three numbers at the top: Delivered, Modelled, Left to order.
- A flag list under them, each flag opening its photo.
- Buttons: Pour sheet (PDF), Supplier claim (PDF + Excel), Month ledger (Excel).

**What it is not.** It does not measure the concrete on the truck and does not decide who is at
fault. It does not replace the engineer's pour clearance or the supplier's contract. It makes the
numbers and the evidence appear in one place, on time, so people can ask the right question.

### Implementation

- **Where it lives.** A new module in the IFC engine (`@cad2bim/engine`) for the ledger and the
  quantities (volumes already come from the model through the BOQ code); a phone-friendly panel in
  the web app. Nothing is uploaded: photos and challans stay in the browser's own storage on the
  device. Cloud sync, if ever added, follows the app's own accounts rules and is off by default.
- **Pour data.** A pour is a named set of element GUIDs with a modelled volume. Import from the
  Pour Planner or Joint Judge output when it exists; otherwise the person selects elements and
  names the pour. Volume comes from the model's own quantities; deductions use the BOQ rules.
- **Reading the challan.** Start with the cheapest reliable path: (1) a supplier's Excel or CSV
  import with a one-time column mapping saved per supplier; (2) a typed-in form, two taps per
  field with last values pre-filled; (3) photo reading on the device with a small OCR model, tested
  per supplier layout. No cloud OCR by default (the app's rule: nothing leaves the device unasked);
  if a cloud reader is ever offered, it is opt-in per photo with a clear notice. Licence check
  before adding any OCR library (MIT, Apache-2.0, BSD, ISC only; no GPL).
- **Ledger.** Plain records: challan, pour, element set, cube set, flag. Fill fraction per element
  is delivered volume shared by modelled volume in pour order, a display aid and labelled as such.
- **Last-truck rule.** `needed = modelled - delivered + allowance`; allowance comes from the team's
  settings, with the history table beside it. The app never sets it silently.
- **Flags.** Each flag is a small rule with a name, inputs and a plain sentence, kept in a rules
  file so a firm can edit limits. Limits for time, slump and water come from the project's spec or
  the person; none is hard-coded as a standard. Any limit taken from IS 4926 or IS 456 is cited
  from the current text before it ships.
- **Claim sheet.** Built from the ledger, with photo thumbnails; PDF through the app's existing
  sheet writer; Excel through the app's existing Excel export.
- **Agent helper.** Expose `pour_proof_status(pour)` and `add_challan(...)` through the MCP helper
  so an agent can answer "how much is left to order for Pour 4?". This follows `docs/agent/protocol.md`.
- **First version (smallest useful).** Manual and Excel challan entry, the pour list from the
  model, the three numbers, the last-truck calculator, the month ledger (Excel). Photo reading, 3D
  fill colours, flags and the claim sheet come second.
- **Test.** 3 real pours with signed challans, kept out of git; compare the hand total with the
  tool total and with the model; measure read errors per field for photo reading before trusting
  it; try with two supplier layouts; test on a phone in sun and with a poor network (offline use).
- **Version.** A new minor version of the engine and the app when built; changelog, README and
  guide entry as CLAUDE.md asks. Not done here; this repo holds ideas only.

### Future update

- A QR on the challan or on the truck's windscreen, scanned at the gate, opens the pour with the
  truck already added.
- Pump log: pump hours and line volume, so the "line and edge" allowance is measured.
- Link to Order Clock and Cash Curve: delivered volume turns into the supplier's bill and the
  cash-out the week it happens.
- Supplier scorecard from the team's own history: short loads, time to site, slump misses, with
  a month trend (kept private to the team).
- Leftover use: suggest a small non-structural pour (kerb, plinth filler) near the pump when a
  leftover would otherwise be dumped.
- Same ledger for other bulk items bought by volume or weight: sand, aggregate, blocks, tiles.
- Photo of the slab after the pour: check the finished level and volume against the model
  (with the Level Lens and Scan Check ideas).
