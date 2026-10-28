# 20261028 Idea: BEP Bolt: answer ten questions and get the whole BIM Execution Plan (ISO 19650 style): LOD/LOI matrix, naming and folder rules, responsibility matrix, model-check rules (IDS) and a Revit start kit; then the same plan checks every model upload and shows a live compliance scorecard, so the BEP stops being a PDF nobody reads

## Entry 1

```
date-time:   2026-10-28 15:23 UTC (scheduled run)
idea name:   BEP Bolt. "The client says: send your BEP before kick-off. Today a BIM
             coordinator copies last project's Word file, edits names and dates, builds the
             LOD matrix by hand in Excel, writes naming rules nobody follows, and three
             months later the model fails them. Here: the coordinator opens cad2bim, picks
             'New BEP', and answers ten questions: project type (residential tower,
             industrial, infra), who is who (architect, structure, MEP, contractor), the BIM
             uses wanted (clash, quantities, 4D, 5D, handover), the standard (ISO 19650,
             client EIR, CPWD / NHAI style, or the office's own), the software and IFC
             version, the coordinate system, the LOD/LOI per stage, the file naming style,
             the review rhythm. In a minute the app produces: (1) a BEP document (PDF + Word)
             in clean sections, (2) a LOD/LOI matrix per element class per stage, (3) a
             responsibility matrix (who models, who checks, who signs), (4) naming and
             folder rules for the CDE (WIP / Shared / Published / Archive), (5) an IDS file
             that encodes the matrix as checks, (6) a Revit start kit: shared parameters,
             view-template names and a worksets list. Then the wow: drop any IFC on the BEP
             and a scorecard lights up: 'Structure model v7: 91 % compliant. 14 columns have
             no fire rating at LOD 300. 6 file names break the rule. Level names differ from
             the architect's.' Click a red row and the elements glow in 3D. Press 'Weekly
             report' and a one-page PDF with the trend goes to the PM."
             The idea in one line: the BEP is generated from answers AND enforced from the
             same data, so there is one source of truth, not a PDF and a separate checklist.
potential:   High, and used at the start of every project plus every week after.
             - A BEP is asked for in almost every tender and in every ISO 19650 job. Writing
               one from an old Word file costs a day or more and is rarely checked later.
             - The pain is not the document, it is that the document and the model drift
               apart. A BEP that generates its own checks fixes that.
             - Junior modellers get the rules in the tool, in the words 'this column needs
               fire rating and a mark', not in a 40-page PDF.
             - It rides a trend: BIM mandates and ISO 19650 compliance are talked about all
               the time by BIM managers on LinkedIn (from general knowledge; I did not
               measure engagement in this run).
             Market (from knowledge, not verified this run; check before relying on it):
               - BEP templates are sold as Word/Excel files by consultants and BIM training
                 sites; they are static documents.
               - Model checkers (Solibri, BIMcollab, Autodesk Model Checker in ACC) are
                 paid and are set up by hand from rules; none I know generates the BEP and
                 the rules from the same answers.
               - IDS is an open buildingSMART standard, but writing an IDS by hand is
                 slow; few tools produce it from a LOD matrix.
             So the gap is 'one set of answers -> BEP + matrix + IDS + Revit start kit +
             live scorecard, in a browser, on the device'. If something close exists it is
             likely paid or tied to one platform.
             Wow factor: a client-ready BEP in a minute, and then the red/green scorecard
             on the real model. Reel idea: 'BEP in 60 seconds. Then I drop the structure IFC:
             91 %. Click red: 14 columns missing fire rating, glowing in 3D.'
description: BEP Bolt is a small rules engine plus templates.
             1. Questions to profile. A short wizard writes a JSON 'BEP profile'.
             2. Profile to matrix. A data table (element class x stage -> LOD, required
                properties) is chosen by the profile; the office can edit and save its own.
             3. Matrix to documents. Templates produce the PDF/Word BEP, the responsibility
                matrix and the CDE folder/naming rules from the profile.
             4. Matrix to checks. The same table is written as an IDS file; the existing QA
                (packages/engine QA, and 'IDS QA autofix' idea) runs it.
             5. Scorecard. Results grouped by discipline, class and rule, with a trend per
                upload and a click-to-select in 3D.
             6. Revit start kit. Shared-parameter file, workset list, view-template names
                from the profile, delivered through the existing bridge or as files.
             Honest limits: the app drafts the BEP; the BIM manager owns and signs it. Client
             EIR text is not copied from anyone's paid template; templates are the team's
             own, and the EIR can be loaded as a file to prefill answers.
implementation:
             Phase 0 (about 1 week): the profile JSON schema and the LOD/LOI matrix as a data
             file for a residential tower (structure only). Export the matrix to Excel and
             write the IDS from it.
             Phase 1 (about 2 weeks): the wizard in apps/web; BEP PDF and Word from templates
             with the writers already in the app; naming/folder rules page.
             Phase 2 (about 2 weeks): the scorecard: run the IDS through the engine QA,
             group and trend results, select-in-3D, weekly one-page PDF.
             Phase 3 (about 2 weeks): MEP and architecture matrices; responsibility matrix;
             file-name checker for a folder of uploaded files.
             Phase 4: Revit start kit through the bridge; agent tools ('make a BEP for a
             G+14 tower with 4D and 5D, then check this IFC').
             Where it goes: `packages/engine/src/bep/` (profile, matrix, IDS writer), a
             'BEP' dock panel in apps/web, a design note `docs/design/bep-bolt.md` first.
             No new dependency: documents use the existing writers. Nothing leaves the
             device. Proof: build a BEP for two finished projects and compare with the BEPs
             the office wrote by hand; report the time and the differences, measured.
future update:
- **EIR import:** read a client's EIR (PDF/Word) and prefill the profile and matrix, with
  every prefilled answer marked 'check this'.
- **Standards packs:** ISO 19650, UK, Gulf, India (CPWD/NHAI-style), each as a data pack.
- **Team scorecards:** per-firm and per-modeller trend (fair, opt-in), so coordination
  meetings start from numbers.
- **Auto-fix:** pair with IDS QA autofix: fill missing properties from rules and re-score.
- **MIDP/TIDP:** a delivery calendar generated from the stages, with due dates and owners.
- **Handover link:** the same matrix drives the COBie/asset data Handover Hub needs.
- **Agent:** 'which rules fail most this month, and who owns them?'

### Improvement suggestions for existing tools in the repos

- **cad2bim (QA):** let a QA rule set be loaded from an IDS file and saved as a named
  profile, with a stable rule id so a scorecard can trend each rule over time.
- **cad2bim (.c2b):** keep the BEP profile and score history in the project file
  (`docs/design/project-file.md`).
- **cad2bim (export):** a branded PDF report writer with a header, a trend chart and a
  table, reused by BEP, Area Statement Pro and Model Pulse.
- **Revit bridge:** a command to create shared parameters and worksets from a file, so
  the start kit is one click.
- **Model Pulse / Change Radar:** show the compliance score beside each revision.
```
