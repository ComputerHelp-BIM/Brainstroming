# 20261119 Idea: Cash Curve — The 4D schedule and the 5D cost become one cash-flow S-curve: money going out to contractors and suppliers, money coming in from the client's payment slabs, and the funding gap between them. Drag a pour date and watch the money move.

## Entry 2

```
date-time:   2026-11-19 (scheduled run, UTC)
idea name:   Cash Curve (4D + 5D to a cash-flow S-curve and funding-gap check, in the browser, nothing uploaded)
potential:   High for the people who decide, and it makes the BIM team's 4D/5D work matter to them.
             Cost and schedule are usually shown apart. The question owners and contractors ask every
             month is different: "In which week do we run out of cash, and how much do we need?"
             Today that is answered in a hand-built Excel sheet that is out of date after the next
             schedule change. Time saved: NOT measured.
             First test: take 2 real projects (kept out of git) that already have a schedule and a BOQ.
             Build the cash curve by hand in Excel with a stopwatch, then with Cash Curve. Compare
             (a) minutes, (b) the peak funding gap and its week, (c) what the finance person says is wrong.
             Do not quote hours or accuracy until that is done.
             Market check (2026-11-19). I did NOT run a new web search in this run. What follows is from
             general knowledge and may be out of date; verify before using it in any pitch:
               - Primavera P6 and MS Project can load cost on activities and show a cumulative cost
                 curve. They show cost by schedule, not the client's payment slabs, and not the lag between
                 work done, bill, and payment. Paid.
               - 5D tools (Synchro, Vico, Bexel and similar) show cost over time from the model. I do not
                 know how far each goes into inflow vs outflow; I did not check.
               - Spreadsheet templates for construction cash flow are common and free, but they are not
                 connected to the model, so every revision means re-typing.
               - I did not find a free, browser-based tool that reads the 3D model, the 4D schedule and the
                 BOQ and returns inflow, outflow and funding gap together. "I did not find one", not "none".
             Trend: cash-flow and delayed-payment stress is a regular theme in construction posts; I could
             not measure LinkedIn or Instagram counts, so this is not a measured claim.
             Extends tools already in this repo (does not repeat them):
               - Rate Rocket: priced BOQ and a 5D cost curve. Cash Curve adds time-shifted money, in and out.
               - Order Clock: material order dates. Its dates become outflow dates here.
               - Crew Curve and Gang Pay: labour cost over time and pay sheets. They feed outflow.
               - Bill Painter: RA bills from painted progress. Its bill dates are the inflow trigger.
               - Delay Detective: when work slips, Cash Curve shows what the slip does to cash.
               - Pour Planner and Ready Board: the pour dates the user drags.
             The wow: a chart with two lines (cash out, cash in) and a red band where out is ahead of in.
             The banner says "Peak gap Rs 3.4 cr in week 31". The user drags the raft pour 10 days later
             on the 3D timeline. The red band shrinks and moves, the banner updates in about a second,
             and the 3D shows which pours cause the peak, lit orange. One more drag, "ask the client to
             release slab 4 before the slab 5 start", and the gap goes. A one-page PDF for the owner.
```

### Description

**The pain.**
- Schedule people talk in dates. Finance talks in weeks of cash. Nobody owns the link.
- Contractors pay suppliers and labour before the client pays them. The gap is real money, often
  borrowed, and it is found late.
- Developers selling flats under a construction-linked plan receive money by milestone (slab cast,
  brickwork, plaster and so on), not by work done. The curve of money in has steps; the curve of money
  out is smooth. They do not meet.
- Any schedule change means a rebuilt sheet. So the sheet is not rebuilt, and it is wrong.

**The idea.** Four moves.
1. **Money out, from the model.** Each 4D activity carries its cost from the 5D BOQ (material, labour,
   subcontract, plant). Each type of cost gets its own timing rule: supplier credit days, labour paid
   weekly, subcontractor on measured bill, steel on order. Defaults can be edited; nothing is hidden.
2. **Money in, from the contract.** The user picks or types a pattern: client RA bills with a
   retention percentage and an approval lag, or a construction-linked plan with milestones tied to
   named 3D events ("slab of floor N cast"). Each milestone is a model event, so it moves when the
   schedule moves.
3. **See the gap.** Two cumulative lines, a red band where out is ahead of in, the peak gap and its week,
   and a month table (out, in, net, running balance). Each part of the band can be clicked to see which
   activities and elements are behind it, lit on the 3D.
4. **What-if, live.** Drag a pour date, change a supplier credit period, move a payment slab, or add a
   delay of N days to a task. The curve and the peak update. Save any state as a named scenario and
   compare them side by side. Every number carries its source (activity, rule, milestone).

**What a user sees.**
- A "Cash" panel: the chart, the peak-gap banner, the scenario list.
- Click a point on the curve: the week's payments, in and out, with the elements behind each line.
- A short "to close the gap" list built from the changes tried in scenarios, with the gap each removes.

### Implementation

Inside the cad2bim app; the project file holds schedule, BOQ and rules. No server, nothing uploaded.
- **Inputs:** the existing 4D schedule (tasks with element links), the 5D BOQ and rate set, and a small
  "payment rules" table (kept in the project file, versioned). Missing inputs are named, not guessed.
- **Engine** (packages/engine, new module): pure functions. Input: tasks, costs per task, timing rules,
  inflow rules. Output: weekly and monthly out, in, net, running balance, peak gap. Pure code, so it is
  testable with Vitest on small hand-checked cases (one task, one rule; two tasks and a retention).
- **Milestone events:** a milestone is a query on the model ("last slab of level N reaches 100%
  progress", "first task of finishing on level N starts"). It uses the same element and level data as
  the 4D player, so a moved task moves the payment.
- **Chart and 3D link:** the chart is plain React with the design-system tokens; hovering a band
  selects the elements behind it through the existing viewer selection and colour overrides.
- **Scenarios:** stored as a diff over the base schedule and rules, so comparing is cheap and undo is
  free.
- **Output:** a one-page PDF and a CSV through the existing export code; an "assumptions" block on the
  page lists every rule used so the reader can challenge it.
- **Agent access:** MCP tools "peak gap", "run scenario", "explain week" in packages/mcp, so an AI
  agent can answer "what if we start the raft after Diwali?" with the same engine a person uses.
- **Build order:** (1) outflow from schedule plus BOQ with simple timing rules and the chart; (2) client
  RA-bill inflow with retention and lag; (3) milestone-linked inflow; (4) scenarios and drag-to-shift;
  (5) PDF and MCP.
- **Risks:** the numbers are only as good as the rate set and the payment rules, so the page says "plan,
  not accounting" and shows its assumptions; tax, interest and escalation are left out at first and
  said so; payment terms differ per contract and need an editable table, not fixed rules.
- **Check before release:** 2 real projects, the stopwatch test above, and a finance reviewer on the
  month table.

### Future update

- **Interest and funding cost:** price the gap (overdraft or loan rate) and show the cost of each week of
  delay in rupees.
- **Escalation:** apply steel, cement or labour index changes to future outflow, with a user-picked
  rate source.
- **Multi-project view:** a contractor with five sites sees the combined gap and where to move cash.
- **Actuals:** import the bank or accounts export, draw actual over planned, and flag the first week
  they separate (pairs with Delay Detective and Bill Painter).
- **Owner link:** a read-only one-file capsule (see Model Capsule) so the owner can try the sliders
  without the model.
