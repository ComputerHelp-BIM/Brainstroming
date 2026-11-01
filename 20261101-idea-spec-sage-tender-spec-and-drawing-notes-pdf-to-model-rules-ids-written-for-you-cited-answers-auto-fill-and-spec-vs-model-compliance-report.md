# 20261101 Idea: Spec Sage — drop the tender specification and drawing notes PDF, get the rules the model must follow (written as IDS for you), cited answers, auto-filled parameters and a spec-vs-model compliance report

## Entry 1

```
date-time:   2026-11-01 (scheduled run, UTC)
idea name:   Spec Sage (Specification and Notes PDF to Model Rules, Cited Answers, Auto-Fill and Compliance Report)
potential:   High. Every project has a 100-400 page specification, a "general notes" sheet and an
             employer's requirements document. The modeller is expected to know all of it. Nobody does.
             Wrong concrete grade, wrong cover, wrong fire rating, wrong finish code: found at site or at
             audit, then fixed at ten times the cost. Spec Sage reads those documents once, shows every
             rule it found with the page and clause it came from, and checks the model against them.
             Saving (an estimate, not measured): 1-3 hours per issue cycle on manual spec look-ups and
             checking, and fewer site rework items. It also gives the "wow" of asking a 300-page PDF a
             question and seeing the answer highlighted in the 3D model.
```

### Description

**The pain.**
- The numbers that matter are buried in prose: "Concrete for basement walls shall be M40 with
  crystalline admixture", "clear cover to footings 50 mm", "all internal partitions 150 mm AAC unless
  noted", "fire rating 2 hours for columns in parking". They sit on page 87 of a PDF and in a note on
  sheet S-001.
- The modeller reads it once at kick-off, models for six weeks, and the details drift. Review by the
  consultant then finds 40 comments that are all "as per spec clause X".
- Writing IDS or other rule files by hand (the checking route in Idea 1, IDS Gatekeeper) takes a specialist
  and is skipped on most jobs. So the check never happens, and the spec stays a PDF nobody queries.
- "Chat with your documents" is a recurring AI topic on LinkedIn and Instagram AEC posts; read this as a
  trend I have seen in discussions, not a measured statistic.

**What exists today (one thin search, to be re-checked before any "nothing like it" claim).**
- Rule-based model checkers (Solibri and others) are paid, and their rules are written by hand. General
  "chat with a PDF" tools exist, but they answer in words; they do not turn a clause into a check that runs
  on the IFC and points at the failing elements. buildingSMART's IDS is an open format for exactly such
  rules, but I am not aware of a free tool that *writes* IDS from a specification PDF with the source clause
  attached. I did not confirm this or any prices.
- Earlier ideas in this repo: IDS Gatekeeper runs rules that already exist. BEP Bolt writes rules from ten
  answers about the BIM Execution Plan. RFI Forge compares drawings with each other. None reads the
  *specification text*. Spec Sage is the missing front end for IDS Gatekeeper and the "spec side" of
  RFI Forge.

**The idea.** A **Spec** panel in cad2bim (IFC workspace, and the CAD workspace for drawing notes). Drop
one or more PDFs (specification, general notes, EIR, a client's checklist) and, if wanted, a text note:

1. **Read.** The PDF is read on the device (text layer first; scanned pages only with the person's say-so).
   Headings, clause numbers, tables of grades and covers are kept with their page numbers.
2. **Find the rules.** Each requirement becomes a *rule card* in a plain-English sentence plus a machine
   form: *what it applies to* (e.g. "columns in basement levels"), *what must be true* (grade = M40),
   *where it came from* (spec p. 87, clause 3.2.4, with the sentence quoted). Doubtful ones are marked
   **"confirm"** instead of silently guessed. Conflicting clauses (spec says M30, drawing note says M35) are
   put side by side as a **conflict card**, a ready-made RFI.
3. **Write the IDS for you.** Confirmed cards are exported as an `.ids` file (and run directly in the
   built-in checker). The person never sees XML unless they want it.
4. **Check the model.** Pass / fail per rule, grouped by clause. Click a failing rule: the elements light
   up in 3D, and the card shows "model says M30, spec p. 87 says M40".
5. **Fix or fill.** Missing values can be filled in one click from a rule ("set grade M40 on these 212
   basement columns", shown as a before/after list, undoable). Anything that is a design decision, not a
   data gap, is never filled automatically; it becomes an issue (BCF) for the engineer.

**The wow 1: ask the 300-page PDF a question, see the answer in the model.** Type or say: *"What is the
cover for the basement raft?"* The answer arrives with the clause, the page thumbnail with the sentence
highlighted, and (if the answer is a rule) the raft in 3D, green or red. It beats Ctrl+F because it knows
that "footing", "foundation" and "raft" are different things in this model.

**The wow 2: the spec diff.** Drop revision B of the specification next to revision A. Spec Sage lists
*rules that changed* ("cover 40 to 50 mm"), re-runs only those, and says how many elements are now
failing. A silent addendum no longer becomes a surprise at site.

**The wow 3: the project's rule library.** Rules confirmed on one project are saved (on the team's
device) as a library: "client X always wants fire rating on every column", "this consultant's cover table".
The next tender from the same client starts with 70% of the cards already known, with a "last seen on
project Y" note.

**The wow 4: a spec-vs-model compliance page.** One page for the BIM manager and the client: rules found,
confirmed, passing, failing, and unresolved conflicts, each with its clause. This is what turns "we followed
the spec" from a claim into evidence.

**Who uses it, when.**
- **BIM modeller:** at kick-off (what must I put in the model?), and before each issue.
- **BIM manager / coordinator:** to answer review comments with a clause, and to send conflict RFIs.
- **Structural / architectural lead:** to confirm the rule cards once, in 20 minutes, instead of
  explaining the spec to every new joiner.

### Implementation

**Guardrails first (they decide whether people trust it).**
- Nothing is uploaded: the PDF stays on the device, as for every file in cad2bim. Reading is local
  (a PDF text library; scanned pages go through local OCR only when switched on). Any online AI step is
  *off by default*, uses the person's own key (the same route as the planned AI chat with your own key),
  and sends only the clause text the person selected, never the model.
- A rule is never "true" because a model said so. The language step only *proposes*; a person confirms.
  Every card keeps the quoted source sentence so a wrong reading is seen in seconds.
- Unsure means "confirm". A tool that is right 95% of the time and does not say which 5% is worse than no tool.

**Steps.**
1. **Document reader (new, in the web app).** PDF text with positions and page numbers, outline/headings,
   table detection. Output: a clause tree `{page, clause, text}`.
2. **Rule extraction.** Two layers so it works without any AI:
   - *Pattern layer (offline, deterministic):* regexes and small vocabularies for the common forms:
     concrete grades (`M\d{2}`), steel grades (`Fe\s?\d{3}D?`), cover in mm, fire rating in hours, block
     thickness, finish codes, standard references (IS 456 and so on). Tables of "element, grade, cover"
     become many rules at once.
   - *Language layer (optional, own key):* turns the remaining free text into the same rule form, and
     answers questions. Its output goes through the same confirm step. Cost and quality are measured on
     a set of real specifications before it is offered; that number is not known today.
3. **Rule model.** `Rule { appliesTo (IFC class + filters: level name, space type, predefined type),
   requires (property set, property, operator, value), source {file, page, clause, quote}, status }`.
   Maps one-to-one to an IDS facet (`entity`, `property`, `attribute`) where IDS can say it, and to a
   cad2bim check where it cannot (for example "cover" read from rebar geometry).
4. **IDS writer and checker.** Reuse IDS Gatekeeper's runner. Spec Sage only adds the *source* link so a
   failure can open the PDF at the clause.
5. **Conflict finder.** Same subject + same property + different value across clauses or documents.
   Unit-aware (mm vs m, MPa vs N/mm2), so false conflicts are rare.
6. **Fill tool.** Writes properties through cad2bim's normal edit commands, so undo, version history and the
   project file work as for any edit. A dry-run list comes first.
7. **Ask panel.** Retrieval over the clause tree (keyword + synonyms for AEC words such as raft, footing,
   foundation, pile cap; embeddings only if run locally). The answer always shows the clause. No clause
   found means "not found in the documents", never an invented answer.
8. **Spec diff.** Compare clause trees A/B, rule cards A/B.
9. **Tests.** Start with 3 real specifications from past projects (kept outside git, since they are
   client documents) and write down, per document, how many rules were found, how many were right, and how
   many needed confirmation. Do not claim accuracy before that table exists.

**Effort (estimate).** Phase 1 (reader, pattern rules, rule cards, IDS export, run check, PDF jump): about
4-6 weeks for one developer. Phase 2 (ask panel, conflict cards, fill tool): 3-4 weeks. Phase 3 (spec diff,
library, compliance page): 2-3 weeks. Language layer on top of any phase, 1-2 weeks plus evaluation.

**Risks.**
- Scanned and badly structured PDFs: reading fails or is noisy. Show per-page reading quality, and say so.
- Specifications use "unless noted otherwise" and cross-references. First version flags such clauses as
  *needs a person*; it does not try to resolve them.
- Legal weight: the tool helps; it is not the contract. The compliance page says it covers the rules that
  were confirmed, and only those.

### Future update

- **Drawing-notes mode:** read the notes on a CAD sheet (the CAD workspace already reads text and tables) so
  "UNO" notes and schedule footnotes become rule cards too.
- **Link to Scope Scout:** a spec that demands LOD 350 or rebar detailing changes the hours; the quote page
  can say which clause drove which line.
- **Link to Mark Merge and Ratio Radar:** a rule library entry can set the allowed grades, so consolidation
  never proposes a grade the spec forbids.
- **Link to Issue Desk and Markup Mirror:** a review comment "as per spec clause 3.2.4" is matched to its rule
  card and its failing elements automatically, and the response sheet cites the clause.
- **Standards packs:** shipped, licence-checked starter packs for common national codes (only what we may
  redistribute; check each licence first, since many standards are copyrighted text).
- **Team scoreboard:** the rules that fail most often across projects, so the BIM manager can fix the
  template, not the model, next time.

### Notes for the owner

- This note changes no source files; it only adds this idea to the Brainstroming repo.
- Market statements above come from a quick, unverified look and my reading of discussions; verify them
  before using them in anything customer-facing.
