# 20261014 Idea: IDS Gatekeeper — one-click IFC readiness check, auto-fix and BCF issues

## Entry 1

```
date-time:   2026-10-14 (scheduled run)
idea name:   IDS Gatekeeper — "Is this model ready to issue?" in 10 seconds
potential:   High. Cuts the daily "check, fix, re-export, re-check" loop (typically 1–3 h per issue/revision
             per project) to minutes. Directly sells to every BIM team that must meet client BIM requirements
             (EIR / ISO 19650 / NBC-CPWD / IS-code deliverables). Free alternative to paid Solibri, BIMcollab Zoom
             rule sets and Revit-only add-ins.
```

### Description

**The pain.** Before every issue, a modeller manually checks that: every column/beam/slab has a mark, grade,
level and fire/load-bearing property; nothing sits on the wrong layer/type; no duplicate or zero-volume elements;
rebar/BBS data is complete; names follow the project standard. Today this is done by eye, by Excel filters, or by
paid checkers (Solibri, BIMcollab Zoom) that are heavy and mostly report problems without fixing them.

**Why now (trending).** buildingSMART **IDS (Information Delivery Specification)** reached v1.0 and is the hot
topic on LinkedIn/BIM circles in 2025–26 together with **openBIM, BCF, "AI-ready data" and model QA automation**.
Clients and consultants are starting to ship `.ids` files with tenders. Most free tooling is Python/CLI
(IfcTester, IfcOpenShell) — not usable by an ordinary modeller. No free, browser-based tool does
**check → explain → fix → re-check → hand-off** in one screen.

**The idea.** Add an **IDS Gatekeeper** to the cad2bim bottom-panel QA tab (it already has a QA/BOQ/Activity panel
and a design doc for "actionable QA"). The user drops an `.ifc` (already loaded) and optionally a `.ids` file, or
picks a built-in rule pack, and gets:

1. **Traffic-light readiness score** per discipline / level / element class (e.g. "Columns 96 %, Beams 71 %").
2. **Plain-language failures**, clickable → the offending elements are isolated and highlighted in 3D.
3. **Auto-fix buttons** for the safe, boring cases:
   - fill missing Mark/Tag from grid + level (`C-B2-L03`),
   - infer missing grade/section from geometry or neighbours ("same as 14 similar columns"),
   - rename to the project naming rule,
   - delete exact duplicates / zero-volume solids,
   - snap near-miss levels (e.g. 2999 mm vs 3000 mm).
   Every fix is shown as a diff and is undoable (fits the existing edit-and-save-as-IFC flow).
4. **BCF export** of the remaining issues (viewpoint + element GUID), so the Revit user can open them via the
   existing cad2bim Bridge for Revit, or any BCF tool (BIMcollab, Trimble Connect, Navisworks).
5. **One-page PDF/Excel "Model Readiness Certificate"** to attach to the transmittal (date, revision, rule pack,
   pass/fail counts) — useful as audit evidence for the client.

**Built-in rule packs (no IDS authoring needed):**
- *Structural RCC minimum* — mark, grade, section, level, load-bearing flag, material.
- *BBS-ready* — bar mark, diameter, spacing, cover, shape code present (feeds the existing BBS-to-Excel export).
- *Naming & units* — layer/type naming, mm/m unit consistency.
- *Clash-lite* — column/beam/wall overlap and floating elements (cheap geometric checks, not full coordination).
- *Custom* — import any `.ids`, or build a rule with a simple form ("IfcColumn must have Pset_ColumnCommon.Reference").

### Implementation

Fits the existing stack (browser, web worker IFC load, React UI, three.js picking) — no server, files stay local.

1. **Rule engine** (`packages/engine/src/qa/`): parse IDS 1.0 XML (applicability + requirements: entity,
   attribute, property, material, classification, partOf) with `DOMParser` into a small internal rule AST.
   Evaluate against the already-loaded element model inside the IFC worker; return `{ruleId, guid, status, reason}`.
   Reference behaviour: IfcTester (IfcOpenShell) to cross-check results in unit tests with the same `.ids/.ifc` pairs.
2. **Built-in packs** as JSON/IDS files in `packages/engine/src/qa/packs/`, versioned and documented.
3. **UI** (`packages/ui`): QA tab shows score cards → rule list → failing element table. Row click isolates in
   viewer (reuse picking/selection sync already used by the Revit bridge).
4. **Fix actions**: small pure functions `(model, failures) => patch[]` that use the existing property-edit
   pipeline and "save edits as IFC" writer; always preview diff first; one undo step per fix batch.
5. **BCF 2.1/3.0 writer**: zip of `markup.bcf` + `viewpoint.bcfv` + snapshot (canvas screenshot) via `fflate`/JSZip.
6. **Certificate**: reuse the Excel/PDF export plumbing (BOQ/BBS) — summary sheet with rule pack hash and revision.
7. **Tests/fixtures**: extend `tools/fixtures` (`make_props_ifc.py`) to generate IFCs with known defects; golden
   results per rule; performance target ≤ 5 s on the 50k-element fixture (run in worker, chunked, cancellable).
8. **Docs**: add `docs/design/ids-gatekeeper.md` and list in the roadmap; mention in `docs/ifc-compatibility.md`.

MVP (2–3 weeks): IDS parser + 3 packs + score + highlight + Excel report. Then fixes, then BCF.

### Future update

- **PlanWin AI Pro link**: export a PlanWin design result as IFC with the *BBS-ready* pack already satisfied; and
  run the same checker against the PlanWin→cad2bim handoff to catch mapping gaps.
- **Structura link**: show ETABS-vs-IFC mismatches (section sizes, storey heights) using the same issue list.
- **AI assist** (optional, using the Claude/Ollama pattern already in PlanWin): "Write an IDS from this client
  EIR PDF" and "explain this failure to the modeller"; the rules remain deterministic, AI only drafts them.
- **Revit side**: Bridge command "Fix in Revit" writes corrected parameters back to the native elements.
- **Revision diff gate**: compare readiness between revisions and block issue if the score drops.
- **Team dashboard**: aggregate certificates across projects (modeller performance, recurring errors) via Excel/CSV.
- **Clash-lite → real clash**: add rebar-vs-rebar congestion and sleeve/opening-vs-beam checks for structural BIM.

### Market check

- Solibri: powerful but paid and desktop-only; BIMcollab Zoom / Navisworks: paid, mainly viewing/clash, limited
  auto-fix. IfcTester/IfcOpenShell and the buildingSMART IDS tooling: free but developer-oriented, no fixing, no
  3D click-through. A free browser **check + auto-fix + BCF + certificate** flow tied to a structural/BBS toolkit
  is not offered as a product today.
