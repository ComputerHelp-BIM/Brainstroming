# 20261029 Idea: Steel Ledger — reconcile every kg of rebar: delivery challans vs BBS vs pour-by-pour use, with leak alerts and a recovery statement

## Entry 1

```
date-time:   2026-10-29 15:42 UTC (scheduled run, fourth idea of the day)
idea name:   Steel Ledger (Site Rebar Reconciliation). "Where did the other 6 tonnes go?"
potential:   High. Steel is 25-35 % of the structure's cost and the item most often argued over at the end of a
             job. Today the site store keeps a paper register, the BBS sits in Excel, and the 3D model (if
             any) knows the theoretical weight. Nobody joins the three until final billing, when a 4-8 %
             gap appears with no way to say which diameter, which pour, which month. Steel Ledger joins them
             from day 1, per diameter and per pour, and flags the leak while it can still be stopped.
             No free tool does this on the device. ERP/store modules and paid site apps record stock, but
             they do not know the 3D model's required weight per pour, so they cannot say what SHOULD
             have been used. cad2bim already holds that number (BBS Weaver, Bar Optimizer, Bill Painter).
```

### Description

**The pain.** The BIM modeller or planning engineer is asked, "Our steel consumption is 9 % over the BBS. Is it
wastage, over-lapping, or loss?" The only answer is a week of Excel on challans. The model could answer in
minutes if the site data were joined to it.

**Why now.** LinkedIn/Instagram AEC feeds keep showing "BIM to site", "quantity-to-cost" and "cost leakage"
posts. (Not verified by a live search in this run; judged from the existing ideas and general feed themes.)

**The idea.** A **Ledger** tab where steel moves are entered by phone or imported, and the model supplies the
"should be" side.

1. **Receipts in.** Photo of a delivery challan / mill test certificate / weighbridge slip. On-device OCR reads
   supplier, heat number, diameter, grade (Fe500D), weight. The modeller confirms and the lot is added.
   Excel/CSV import of the store register also works.
2. **Issues out, per pour.** Each pour from Pour Planner (or a simple pour list) gets "issued to site" entries,
   by diameter. A cut-and-bend yard slip photo can be attached.
3. **Theoretical per pour.** Pulled from the BBS: cut length x unit weight x number of bars for the elements in
   that pour, including laps and Bar Optimizer's planned offcut loss. This is the "should have been".
4. **The ledger.** Per diameter: received - issued = in store; issued - theoretical = excess; excess split into
   *planned wastage* (cutting plan), *rework/extra bars* (changes logged by Change Radar) and *unexplained*.
5. **Leak alerts.** Rules, editable: a pour issued 12 % over theoretical; a diameter whose store balance
   goes negative; a heat number with no mill certificate; a lot weight that differs from the challan
   total by more than the weighbridge tolerance; 16 mm bars issued while the pour needs only 12 mm.
6. **Substitution helper.** When 20 mm is short, suggests an equivalent 16 mm + 12 mm mix with the same
   area, flagged for the engineer's approval (not auto-applied).
7. **Recovery statement.** For a subcontractor on a "contractor supplies / client reimburses" or free-issue
   steel contract: allowed wastage % x theoretical = permitted; the rest is the recovery. A one-page PDF with
   the lot list, pours and the rate (rate from Rate Rocket if present).

**Outputs.** Weekly steel report (PDF/Excel), store balance by diameter, per-pour variance table, a 3D
heat-map of pours coloured by variance (green within 3 %, amber 3-6 %, red above), BCF issue for each
red pour, and a WhatsApp-sized summary for the project manager.

**Wow moment (reel-ready).** The 3D building with pours coloured by steel variance. Slider on the date. One
slab glows red: "Slab L4, 16 mm, +1.2 t over". Tap it: the three challan photos, the issue slips, the BBS rows
side by side. Total saved by catching it in week 3 instead of final billing is shown as a rupee counter.

### Implementation

- Reuse: the BBS tables and bar weights from BBS Weaver; pour grouping from Pour Planner; offcut plan from
  Bar Optimizer; revisions from Change Radar. New code is the ledger store, the OCR step, and the rules.
- Data: local-first (IndexedDB / file in the project folder). Nothing is uploaded. OCR in-browser (WASM,
  permissive licence only, per the licence rules). Export/import as a plain CSV so it also works without OCR.
- Maths: unit weight = d^2/162 kg/m (state it; allow IS 1786 tolerance table). Weighbridge tolerance and
  planned wastage are inputs, not constants.
- Phase 1 (2 weeks): manual entry + CSV import, per-diameter ledger, per-pour variance table, Excel/PDF.
- Phase 2: challan photo OCR, heat-number/MTC check, 3D variance colouring, leak rules.
- Phase 3: recovery statement, substitution helper, BCF and WhatsApp summary.
- Honest limits: OCR accuracy on handwritten slips will be uneven, so every read is confirmed by a person.
  The tool shows a gap and its likely cause; it never accuses anyone.

### Future update

- Same ledger for cement, RMC (m3 ordered vs poured vs model volume) and bricks.
- Multi-site view for a contractor head office: which site leaks most, per month.
- Barcode/QR bundle tags (Site QR Pass) so issue-out is a scan, not a form.
- Learn each yard's real wastage % across projects and pre-fill the allowed value.
- Tie into the carbon figures of Carbon Lens (kg CO2e of steel actually bought, not planned).

### Why it is unique / what exists

- Paid store/ERP modules track stock but have no 3D "should have been" per pour.
- Free spreadsheets can total challans but cannot link to model pours.
- This extends BBS Weaver, Bar Optimizer and Bill Painter rather than starting a new product.
