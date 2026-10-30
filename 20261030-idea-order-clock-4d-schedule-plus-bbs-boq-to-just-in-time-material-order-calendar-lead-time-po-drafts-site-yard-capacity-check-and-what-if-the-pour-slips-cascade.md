# 20261030 Idea: Order Clock — turn the 4D schedule and the BBS/BOQ into "what to order, from whom, by which date", and see the cascade when a pour slips

## Entry 2

```
date-time:   2026-10-30 (scheduled run, UTC)
idea name:   Order Clock (4D + BBS/BOQ to a Just-in-Time Procurement Calendar). "The model knows what you will
             need and when. Let it tell the purchase desk before the site shouts."
potential:   High. On a mid-rise RCC job the site engineer and the purchase manager spend a large part of every
             week answering one question: "when does the next 16 mm come, and how much?". Steel, cement, RMC,
             blocks, plywood, props and ready-mix admixtures all have lead times; ordering late stops a pour,
             ordering early blocks cash and yard space and rusts steel. The numbers needed are already in the
             model (BBS by diameter and length, BOQ by element) and in the 4D plan (which element is built in
             which week). Nobody joins them. Free tools stop at quantity take-off; paid 4D/5D suites
             (Synchro, Navisworks Manage, Vico-class tools) show cost curves but not an order calendar with
             lead times, delivery slots and yard capacity, and they are priced per seat per year.
             Estimated saving (an estimate, not measured): 3-5 hours a week for a purchase/planning pair and,
             more importantly, fewer "no steel today" idle days.
```

### Description

**The pain (daily life of a modeller / planner / site engineer).**
The BBS gives the total weight of each bar diameter. The schedule says "Level 5 slab pour on 12 Nov". Between
those two facts a human does the arithmetic in Excel every week: how much 12 mm and 16 mm must be cut and bent
by when, how many trucks of RMC, how many bags of cement for the blockwork two floors below, what plywood
must be back from the floor that was just struck. It is done from memory and from the last WhatsApp message.
When the pour date moves, the order does not move with it, and the first anyone knows is a truck with nowhere
to go or a gang with nothing to tie.

**Why now (trending).** The 2025-26 LinkedIn and Instagram AEC feeds are about *5D BIM that actually saves
money*, *AI for construction planning*, *supply-chain delays and price swings* (steel and cement prices make the
news every quarter), and *lean / just-in-time site logistics*. The common complaint under those posts: "nice
model, but the purchase team still uses Excel". Order Clock is the missing hand-over from the model to the
purchase desk.

**The idea.** In the 4D/5D workspace (the roadmap's *4D timeline* and *5D cost*), press **Order Clock**. For
every material it builds a **time-phased demand** from the model, then subtracts lead times and delivery
rules to produce an **order calendar**.

1. **Demand by week, from the model.** Each task in the 4D plan owns elements; each element owns quantities
   (concrete m3, steel kg by diameter and by cut length from the BBS, formwork m2, blocks, plaster m2, ...).
   The demand table is simply quantity x task start, spread across the task's duration. Output: "Week of
   10 Nov: 16 mm 4.2 t, 12 mm 2.1 t, RMC M30 118 m3, cement 90 bags, ply 140 sheets".
2. **Lead-time rules per material (editable table).** Steel: mill order 10 days + cutting/bending 3 days +
   transport 1 day. RMC: order 2 days before, slot rules per plant (max 8 m3 per truck, one truck every
   12 minutes per pump). Blocks: 7 days. Ply: reuse cycle from the Shutter Plan idea. The rules are a small JSON
   per project and per supplier, so a team keeps its own.
3. **Order calendar.** Each demand line becomes an *order-by* date. The view is a calendar with three lanes:
   *Order by* (purchase), *Deliver on* (site/store), *Used in* (the task). Colours: green = comfortable,
   amber = less than 2 days of float, red = order date already passed.
4. **Yard and cash check.** Site storage is limited: a yard polygon with a capacity per material (say 18 t of
   bars, 6 pallets of blocks). If the plan delivers 35 t of steel in one week, the clock splits it into two
   deliveries and says why. The same demand, priced with the BOQ rates, gives a **cash-out curve** by week, so
   the accounts team sees the peak before it arrives.
5. **What-if: the pour slips.** Drag the Level 5 slab pour by +5 days. Every dependent order moves, the
   cascade is shown ("steel for Level 6 columns slips 5 days, ply back-order clears, RMC slot 14 Nov released,
   cash peak moves to the next month"), and the tool lists the **orders already placed that are now too early**
   (they will sit in the yard) and the ones that are **now too late**. Press *Accept* and the calendar and
   the PO drafts are re-issued.
6. **PO drafts.** One click makes a purchase-order draft per supplier and per week: item, specification
   (grade, diameter, cut-length list from the BBS), quantity, required date, delivery address, and a tolerance
   line (for example +/- 2 % on weight). Output as PDF, Excel, and a plain-text message for WhatsApp, because
   that is how orders actually go out. A cut-and-bend list goes to the bar-bending yard in the same run.
7. **Delivered vs ordered (light touch).** The purchase desk marks each delivery received; the clock shows the
   gap against the plan. (The detailed reconciliation of challans and weighbridge slips against the model is
   the Steel Ledger idea; Order Clock stops at "ordered, delivered, short".)

**The wow moment (a 30-second reel).** The 4D model builds floor by floor. Under it, a calendar fills with
coloured order tiles that "fly out" of each element a few weeks before it is built: a tile of 16 mm bars leaves
Level 5 and lands on a date three weeks earlier. Then the planner drags the Level 5 pour and **every tile
slides with it**, one turns red, and a counter shows "3 orders too early, 2 too late, cash peak moved to
Dec". Finally one click prints the week's POs. Planning work that takes an afternoon is shown in seconds, and
the "drag the pour and watch the orders move" moment is easy to understand without any explanation.

**Why this is unique / extends what is already here.**
- It uses the 4D timeline and 5D cost on the cad2bim roadmap, plus the BBS and BOQ the engine already makes.
  The new part is the *lead-time and delivery logic*, which no part of the toolkit has yet.
- It sits between existing ideas without repeating them: *Pour Planner* decides when to pour and how many
  trucks, *Bar Optimizer* decides how to cut bars from stock lengths, *Steel Ledger* checks what actually came
  on site, *Rate Rocket* prices the model. Order Clock is the *upstream* piece: what to order and when, and what
  happens to the orders when the plan changes.
- Free alternatives are spreadsheets. Paid ones (Synchro, Vico-type 5D suites, ERP modules for contractors)
  need a full licence and a clean data set; Order Clock works on the file in the browser and nothing leaves the PC.

**Day-to-day use cases.**
- Monday morning: the planner opens the clock, reads the next 3 weeks of orders, and sends the POs.
- After every progress update (from Bill Painter or the WhatsApp site diary idea): re-run, and the calendar
  re-plans from the actual dates.
- Before a monsoon or a festival break: stock up the right materials for the right days, not everything.
- Tender stage: a cash-out curve and a delivery plan are part of the bid, and show the client you have
  thought it through.
- Subcontractor and supplier meetings: share the calendar as a read-only link or a PDF, one agreed plan.

### Implementation

Browser only, a worker, no server; files stay on the device. cad2bim was read for reference; nothing there was
changed.

1. **Inputs (all existing or planned):** the model's quantities (`packages/engine` BOQ and BBS), the task list
   from the 4D timeline (task id, start, finish, element set), and rates from the 5D cost. If 4D is not yet
   built for a project, accept a CSV/Excel or an MS Project/Primavera export of tasks with an `element set` or
   level column; that alone is enough to start.
2. **Demand engine (new, TypeScript/Rust, small):** for each task, for each material line, spread quantity
   over the task days (linear by default; "front-loaded" for steel, "on pour day" for concrete), then sum by
   day and week. Pure functions, easy to test.
3. **Rules engine:** a JSON rule pack per material: `leadDays`, `batchSize`, `slotRules`, `minOrder`, `shelfLifeDays`
   (cement), `maxStayDays` (steel rusting), `yardCapacity`. Orders are placed by walking backwards from the use
   date and splitting by capacity. Deterministic first; an optional AI step (the user's own key, as decided for
   cad2bim) only drafts messages and reads supplier price lists, never decides dates.
4. **Cascade (what-if):** keep the order as a small dependency graph (task -> demand -> order). A drag changes the
   task date, the graph recomputes in milliseconds, and the diff against the saved plan is the red/amber list.
5. **Outputs:** the Excel writer (calendar, POs, cash-out curve), a PDF writer for POs, a text export for
   WhatsApp. The cash curve reuses the 5D chart. Cut lists reuse the BBS tables.
6. **UI (React, the design system):** a dock panel "Order Clock" with three tabs (Calendar, Orders, Cash);
   colours from tokens only; the calendar is a plain virtualised grid so a 30-month job stays smooth; a drag
   handle on the 4D Gantt row triggers the what-if. A Python/MCP tool `order_calendar()` for the AI agent.
7. **Tests:** synthetic projects with a known answer (one slab, one column, one wall) checking dates, splits and
   totals; a property test that total ordered quantity equals total demand plus the stated wastage. Report what
   was measured and what was estimated.

**Phases.**
- *Phase 1 (about 2 weeks):* demand by week from BBS/BOQ + tasks CSV, fixed lead times, order calendar, Excel
  and WhatsApp-text output. Already a useful, demo-able tool.
- *Phase 2:* yard capacity and splitting, cash-out curve, PO drafts as PDF, the what-if cascade with drag.
- *Phase 3:* supplier rule packs (RMC plant slots, mill cycles), delivered-vs-ordered marking, link to Steel Ledger
  and to the progress ideas so the plan re-plans from actual dates.

**Honest risks.** Lead times and market conditions change; the tool says "your rule table says 10 days", not
"the market says 10 days", and every rule has an owner and a date. Quantities are as good as the model: a missing
element means a missing order, so the Ratio Radar check should run first. The tool drafts orders; a person sends
them. Wastage and lapping rules differ by site and must be editable.

### Future update

- **Live price feed (opt-in):** pull the day's steel and cement rate from a user-supplied sheet or API and show
  "order now vs next week" with the price difference, a decision aid and not an advertisement for any vendor.
- **Multi-project pooling:** if three sites need 12 mm in the same week, suggest one consolidated order.
- **Supplier scorecard:** days late and quantity short per supplier, from the delivered-vs-ordered marks.
- **Weather-aware shifting:** use the monsoon calendar to pull RMC and steel earlier before the wet weeks and to
  hold back items that spoil in the rain.
- **Prefab and precast mode:** casting-yard lead times and truck loads (pairs with Kit Cutter).
- **Reel-ready export:** the "tiles fly out and slide when the pour moves" scene as a short video, like the
  Model Reel idea, so the tool markets itself on LinkedIn and Instagram.
- **Contractor-owner sharing:** a read-only link to the calendar and the cash curve for the client's finance
  team, to answer "when do you need the next payment?" with a chart instead of a call.
