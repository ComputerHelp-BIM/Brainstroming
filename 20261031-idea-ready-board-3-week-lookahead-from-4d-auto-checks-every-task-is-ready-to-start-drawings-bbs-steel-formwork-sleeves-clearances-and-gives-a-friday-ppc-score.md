# 20261031 Idea: Ready Board — a 3-week lookahead built from the 4D model that checks, task by task, whether the work can really start (drawing issued? steel delivered? sleeves cleared? formwork free?), turns the answer into a red/amber/green board and a WhatsApp-ready weekly plan, and scores the week (PPC) on Friday

## Entry 4

```
date-time:   2026-10-31 (scheduled run, UTC)
idea name:   Ready Board (Make-Ready Lookahead and Weekly Plan Score from the 4D Model)
potential:   High. Every site and every BIM team that does 4D knows the gap: the 4D shows what the plan
             SAYS will happen, but the Monday question is "can the gang actually start slab 7 tomorrow?".
             Today that answer lives in the site engineer's head and a WhatsApp group. A task that is not
             ready on Monday is a lost day for 10-30 people (my estimate). Last Planner System (LPS) is
             the lean-construction answer, but it is run on paper, whiteboards or generic boards that do
             not know the model. Earlier ideas here feed it (Pour Planner, Order Clock, Steel Ledger,
             Sleeve Punch, Issue Desk, Chat-to-Model), but none of them says "task ready: yes/no, and what
             is missing". Ready Board is that check.
             Estimated saving (an estimate, not measured): 1-2 lost gang-days avoided per week on a
             mid-size building, plus a weekly meeting that drops from 60+ minutes to about 20.
```

### Description

**The pain (daily life of a BIM modeller and planner).**
The planner exports the 4D, prints a bar chart and the site team nods. On Monday the shuttering gang is
standing at Level 7 because (a) the revised beam drawing was issued Friday evening and nobody told the
foreman, (b) the 16 mm bars came, but the 12 mm did not, (c) a plumber still has to fix sleeves in the slab,
(d) the formwork set is still on Level 6 until the strike on Tuesday. Each one is a *constraint*. All four
were knowable a week earlier, from data that already sits in the model, the BBS, the order calendar and the
drawing register. Nobody looked, because looking means opening five files.

**Why now (trending).** Lean construction, "percent plan complete" and "reliable promises" are steady LinkedIn
topics, and 4D is now a standard ask in tenders. Research on BIM with the Last Planner System repeatedly finds
that model-linked lookahead helps find and remove constraints earlier, and a 2014 IGLC paper tracked owner
constraints with 4D on a retail renovation
([IGLC paper 1085](https://iglc.net/Papers/Details/1085),
[Lean Construction Institute on lookahead](https://leanconstruction.org/?p=12934)). Read those as research
direction, not as a market claim.

**What exists today (checked thinly, one search).**
- LPS itself is well described (vendor guide: 6-8 week lookahead, weekly plan, PPC)
  ([Eleco](https://eleco.com/news/what-is-the-last-planner-system/)).
- Academic work links Revit and LPS (a 2018 Brazilian plug-in study)
  ([Ambiente Construído](https://seer.ufrgs.br/index.php/ambienteconstruido/article/view/73466)).
- I did **not** find, in this one search, a free tool that checks each task's readiness automatically from
  model, BBS, drawing and material data. Commercial planning suites exist but I could not confirm their
  features or prices. Check again before writing "nothing like it exists".

**The idea.** A **Ready** workspace in cad2bim. It reads the 4D schedule (already planned in cad2bim), cuts
it to the next 3 weeks, and for every task runs a **readiness checklist** that is filled automatically where
the data exists and by a one-tap answer where it does not:

| Constraint | Checked from | Automatic? |
|---|---|---|
| Drawing issued at the needed revision | Drawing register (Issue Desk) vs. the revision the model element was last changed at | yes |
| Design clash or open RFI on these elements | BCF / RFI list (RFI Forge, Meeting Mind) | yes |
| Reinforcement ordered and arriving before the task | BBS + Order Clock calendar + delivery log | yes |
| Rebar cut and bent | Bar Optimizer / BBS status | yes |
| Formwork and props free (not still under the previous floor) | Shutter Plan cycle | yes |
| MEP sleeves and inserts fixed before the pour | Sleeve Punch / Hanger Hero lists, status per element | semi |
| Predecessor really finished | Progress painted in 3D (Bill Painter) | yes |
| Access, crane slot, labour, permit, weather | Crane Canvas slots; the rest one-tap | semi |

Each task gets a colour: **green** (all clear), **amber** (a constraint exists but its due date is before the
start), **red** (a constraint will still be open on the start date). The 3D view colours the *elements* the
same way, so the whole building shows where next week is blocked.

**The wow: "Why is it red?" in one click.** Click a red slab, and a side panel says: *"Slab L7 pour, planned
Mon 9 Nov: red. (1) Beam B14 drawing is at rev B, the model is at rev C (issued Fri); (2) 12 mm bars: 3.1 t
ordered, arrival Wed 11 Nov, after the start; (3) 2 sleeve locations unconfirmed. Suggested: move the pour to
Fri 13 Nov (all three clear), or split: pour the green half on Mon."* The tool proposes a re-sequence using
the 4D logic and shows the knock-on in a before/after bar, so the planner decides in a minute, not in a
meeting.

**The second wow: Friday PPC with reasons.** On Friday the site engineer ticks done / not done per task on a
phone (or sends a photo with Chat-to-Model). Ready Board computes **PPC** (percent plan complete = tasks done
as promised ÷ tasks promised), and, more useful, a **reason-for-variance chart**: "Of 9 missed tasks, 4 were
material, 3 were design/drawing, 2 were labour." After four weeks, the contractor and the consultant both see,
with evidence, whose delays they are. It also prints the **weekly work plan** as a one-page PDF and a
WhatsApp-sized image per gang ("Shuttering gang: L7 slab, Mon-Wed. Needs: B14 rev C drawing (printed),
props from L6 (struck Tue)").

**Who uses it, when.**
- **BIM modeller / 4D planner:** Monday morning, 15 minutes instead of a chasing round.
- **Site engineer / foreman:** Friday on the phone, one tap per task.
- **Project manager:** the Thursday coordination meeting, from the board, not from memory.
- **Consultant / client:** a read-only link that shows reasons, not excuses.

**Differences from the earlier ideas.** Pour Planner schedules a pour; Order Clock schedules deliveries;
Steel Ledger reconciles after the fact. Ready Board is the *gate* between them: it reads their outputs and
answers one question per task: ready or not, and why.

### Implementation

**Stage 1: the board (4-5 weeks).**
- Read the planned 4D schedule (cad2bim already links tasks to elements) and cut a 3-week window.
- Data model: `Task { id, elements[], start, finish, gang }`, `Constraint { task, type, status, owner,
  dueBy, source, evidence }`. Constraints are a small table in the project file; the app never uploads it.
- Manual one-tap constraints first (labour, access, permit), plus the three that exist already in the
  packages: drawing revision (register vs. model), BBS delivery dates, predecessor progress.
- Colouring rule is a plain function, easy to test: `red if any open constraint dueBy > task.start`,
  `amber if open but dueBy <= start`, else `green`.
- 3D colouring reuses the viewer's per-element override used by QA (`packages/engine`).

**Stage 2: the "why red" and re-sequence (3-4 weeks).**
- Explanation = list of open constraints with their `evidence` link (jumps to the drawing, RFI or order).
- Re-sequence: earliest date at which all constraints clear, respecting the 4D predecessor links; if a task
  can be split by zone (pour in two halves), offer the split. Simple forward pass over the task graph; no
  solver needed for v1.
- Show the knock-on in a before/after bar chart.

**Stage 3: weekly plan and PPC (2-3 weeks).**
- Weekly work plan: one-page PDF and per-gang PNG (the same renderer Model Reel and Issue Desk use).
- Phone check-in page (static, works offline, data stays on the device until the person exports a small
  JSON file or sends it themselves). Nothing leaves the device unasked.
- PPC and reason-for-variance chart, stored week by week for a trend line.

**Stage 4: connectors (as the other ideas land).** Each earlier idea publishes a tiny "status per element"
feed; Ready Board only reads them. So it is useful on day one with manual constraints and gets smarter as
Issue Desk, Order Clock, Sleeve Punch and Shutter Plan arrive.

**Licences and rules.** No new heavy dependency: a task-graph pass, canvas/SVG for charts, the existing PDF
writer. Everything is open and MIT/Apache-compatible. No GPL code.

**Risks, said plainly.** (1) A readiness check is only as good as its inputs; if the register or the order
calendar is stale, a green is a false comfort, so every constraint shows its source and age ("order data
updated 9 days ago"). (2) Site people will not type much; v1 must work with one tap and a photo. (3) PPC can
be gamed; show the reasons, not only the number.

### Future update

- **Pull planning wall:** a collaborative sticky-note style wall where trades place their own tasks, which
  then become 4D tasks linked to elements (the pull-planning session, but with the model beside it).
- **Learning durations:** after a few projects, suggest task durations and gang sizes from this team's own
  history of planned vs. actual.
- **Weather and holiday aware:** shift pours when the forecast shows heavy rain, and skip festival weeks
  with a local calendar.
- **Contractor scorecard:** PPC and reasons per subcontractor over time, feeding the Steel Ledger recovery
  statement and the RA bill.
- **Voice check-in:** a foreman speaks "slab seven done, block work not started, no cement" and the board
  updates (same engine as Meeting Mind / Chat-to-Model).
- **IFC export of status:** write readiness and PPC as an IFC property set so any viewer can colour it.
- **Agent hook:** an MCP tool (`packages/mcp`) so an AI agent can answer "what is blocking next week?".
