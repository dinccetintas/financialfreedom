# Roadmap: Financial Freedom

## Overview

The journey starts as small as possible: sign in and record one US stock purchase, but on a schema that is already correct for the five things research flagged as non-retrofittable — the append-only ledger, frozen FX rates, row-level tenant isolation, corporate-action typing, and exchange-qualified instrument identity. From there the ledger generalizes to every transaction type with fast bulk entry (because the owner setting up three other family members' portfolios in one sitting is an adoption requirement, not a nicety), market data and presentation modes come online together on the first real dashboard, multi-currency and corporate-action handling get proven on a real BIST TRY transaction, and only then does the "accurate picture" layer — returns, exposure, risk — get built on top of data everyone can already trust. Decision discipline (thesis, rules, alerts) follows, becoming the home screen's centerpiece. News, macro and the daily brief come next, sequenced immediately after the alert delivery they depend on rather than behind the agent stack — knowing what is happening across seventeen holdings is a stated core pain, and it should not wait on features it does not need. The Claude agent is deliberately sequenced last among the core layers: read access only once everything it could report on is trustworthy, write access only with validation/audit/reversal shipping in the same phase, and deep research/code execution only once basic write access is proven. Discovery — asking the family to trust the system's opinions about what to buy — is genuinely last, after the system has proven it can correctly show what they already own.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation — Sign In & First Position** - User signs in and records a US stock buy on a schema built right from the first migration
- [ ] **Phase 2: Full Ledger, All Transaction Types & Fast Bulk Entry** - Every transaction type plus rapid bulk entry to onboard other family members in one sitting
- [ ] **Phase 3: Market Data Sync & First Valuation** - BIST price-provider spike, then scheduled EOD sync and the first USD position value
- [ ] **Phase 4: Presentation Modes, Localization & Design System** - Turkish/English, Simple/Pro mode, 390px layout and the per-page design-prompt convention, established on the first real dashboard
- [ ] **Phase 5: Multi-Currency, Corporate Actions & Portfolio-Wide Valuation** - BIST TRY transactions convert correctly, bonus/rights issues apply correctly, full-portfolio P&L and value chart
- [ ] **Phase 6: Returns Engine — TWR, XIRR & FX Attribution** - Deposit-neutral and money-weighted returns, asset-vs-currency return split, verified against golden values
- [ ] **Phase 7: Exposure, Risk & BIST Fundamentals** - Allocation, concentration flags, volatility/drawdown/Sharpe/correlation, benchmarks, and KAP-sourced sector tagging
- [ ] **Phase 8: Thesis & Decision Rules** - Structured thesis, target/stop/max-size, portfolio-wide defaults with overrides
- [ ] **Phase 9: Rule Evaluation, Alerts & Attention-First Home** - Rule-breach evaluation, deduped Telegram alerts, periodic review, attention-first home screen
- [ ] **Phase 10: News, Macro & Daily Brief** - Holdings-filtered news, macro layer, analyst sentiment and the daily Telegram brief
- [ ] **Phase 11: Agent Core & Read Access** - Tenant-scoped natural-language Q&A over real holdings, with web search and RLS-backed tool isolation
- [ ] **Phase 12: Agent Write Access, Audit & Reversal** - Agent can write to the ledger/rules with deterministic validation, audit log and one-click reversal shipping together
- [ ] **Phase 13: Agent Deep Research & Code Execution** - Durable multi-step research and sandboxed code execution against a read-only data snapshot
- [ ] **Phase 14: Discovery** - Scored investment ideas across US and BIST, aware of existing exposure

## Phase Details

### Phase 1: Foundation — Sign In & First Position
**Goal**: A user on the family's fixed allowlist can sign in, create a portfolio, and record their first US stock purchase, seeing it appear in a transaction list — on a schema that is correct for tenant isolation, money precision, FX-rate freezing, corporate-action typing and instrument identity from the very first migration, since none of these can be safely retrofitted once real data exists.
**Mode:** mvp
**Depends on**: Nothing (first phase)
**Requirements**: [AUTH-01, AUTH-02, AUTH-03, AUTH-04, AUTH-05, AUTH-06, AUTH-07, AUTH-08, LEDG-01, LEDG-06, LEDG-07, LEDG-09, LEDG-10, INST-01, INST-03]
**Success Criteria** (what must be TRUE):
  1. A user on the fixed allowlist can sign in with email + password, the session persists across a browser refresh and across devices, and a forgotten password can be reset via an emailed link
  2. A signed-in user can create a named portfolio and record a buy transaction (date, instrument, quantity, price, fees, currency) for a US stock, and it appears in a filterable, sortable transaction list
  3. Editing or removing that transaction produces a new ledger row rather than mutating the original — the original stays visible in history
  4. A direct database-level query using a second family member's session returns zero rows for the first user's portfolio and transactions unless explicitly shared
  5. The instrument behind the transaction is stored keyed by exchange + symbol + ISIN (never a bare ticker), and every stored quantity, price and derived position uses NUMERIC/decimal arithmetic, never a float
**Plans**: TBD
**UI hint**: yes

### Phase 2: Full Ledger, All Transaction Types & Fast Bulk Entry
**Goal**: The owner can rapidly set up an entire family member's portfolio — every transaction type, many rows, one sitting — because manual-entry fatigue is the most likely real-world failure mode, and the highest-pain user has the least motivation to enter data himself.
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: [LEDG-02, LEDG-03, LEDG-04, LEDG-05, LEDG-08, LEDG-11, INST-02]
**Success Criteria** (what must be TRUE):
  1. User can record a sell transaction and the app resolves cost basis against existing lots using FIFO, showing which lot(s) were consumed
  2. User can record a deposit, withdrawal, fee, interest, a dividend receipt (including withholding tax), and an FX conversion between two currencies
  3. User can hold BIST equities, US equities/ETFs, gold, foreign currency and cash within the same portfolio
  4. User can enter a batch of transactions quickly enough that setting up another family member's full historical portfolio takes one sitting, not one form submission per transaction
  5. User can edit or delete a transaction they entered, and the correction is visible in history rather than replacing the original
**Plans**: TBD
**UI hint**: yes

### Phase 3: Market Data Sync & First Valuation
**Goal**: The system pulls end-of-day prices on a schedule and shows the current USD value of the position recorded so far — opened by a hands-on spike confirming which BIST EOD price provider actually works, since that choice is still unverified and must not be assumed.
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: [DATA-01, DATA-04, DATA-05, DATA-06, DATA-07, VAL-01]
**Success Criteria** (what must be TRUE):
  1. A BIST price-provider spike (EODHD and Twelve Data, pulled against real tickers such as GARAN.IS and THYAO.IS) is complete and a provider is committed before any sync code depends on the choice
  2. System syncs end-of-day prices for every held instrument on a schedule, respecting BIST and US trading calendars as distinct calendars
  3. A missing price bar is visibly recorded as missing/stale, never silently treated as zero or presented as a fresh price
  4. Adding a new instrument automatically triggers a backfill of its available price history
  5. A sync failure raises a visible alert rather than silently serving stale prices, and the user sees the current market value of their position, in USD
**Plans**: TBD
**UI hint**: yes

### Phase 4: Presentation Modes, Localization & Design System
**Goal**: The dashboard becomes usable by every family member — Turkish or English, plain-language or full analytics, phone or desktop — with the mode-toggle mechanism validated now, on the first real dashboard, while the app is still small enough that it isn't a retrofit; and a design-prompt convention is established that every later phase will follow.
**Mode:** mvp
**Depends on**: Phase 3
**Requirements**: [UI-01, UI-02, UI-03, UI-04, UI-06, DES-01, DES-02, DES-03]
**Success Criteria** (what must be TRUE):
  1. User can switch the interface between Turkish and English, and every number and date reformats to the correct locale convention (comma vs. period, date order)
  2. User can switch between Simple mode (plain language, few numbers, no jargon) and Pro mode (full metrics) on the same dashboard, seeing the identical underlying value presented two different ways
  3. Every screen shipped so far is fully usable at 390px width
  4. Every screen shipped so far has an accompanying written design prompt — specifying its data, states and hierarchy, suitable for pasting into Claude Design — and this is the standing convention every subsequent phase follows
  5. A design system (colour, type, spacing, chart styling) is defined once and applied consistently across the shipped screens
**Plans**: TBD
**UI hint**: yes

### Phase 5: Multi-Currency, Corporate Actions & Portfolio-Wide Valuation
**Goal**: A BIST TRY transaction converts to USD correctly — the rate frozen at transaction time drives cost basis, a live date-specific rate drives unrealized valuation — and the family sees total portfolio value, cost basis and P&L across every position, with BIST bonus and rights issues applied correctly rather than confused with each other.
**Mode:** mvp
**Depends on**: Phase 2, Phase 3
**Requirements**: [DATA-02, DATA-03, VAL-02, VAL-03, VAL-04, VAL-05, VAL-10, INST-04, INST-05, INST-06]
**Success Criteria** (what must be TRUE):
  1. System stores historical daily USD/TRY and gold rates sourced from TCMB EVDS, using a market FX rate at transaction-entry time and TCMB rates for historical backfill
  2. A BIST TRY buy transaction's cost basis stays fixed at the rate recorded on its trade date even after today's FX rate moves, while its unrealized value updates daily using a live, date-specific rate
  3. User sees unrealized and realized P&L and FIFO cost basis per position and per portfolio, plus portfolio value over time as a USD chart
  4. A stock split adjusts historical position quantities correctly
  5. A BIST bonus issue increases share quantity with zero change to total cost basis, while a BIST rights issue is recorded as a real cash outlay that adds to cost basis and counts as a cash-flow event
**Plans**: TBD
**UI hint**: yes

### Phase 6: Returns Engine — TWR, XIRR & FX Attribution
**Goal**: The family sees performance the correct way — deposit/withdrawal-neutral (TWR) and money-weighted (XIRR) shown side by side, with a TRY gain that was actually a USD loss made visible — verified against known golden-value test cases, because a wrong return figure is worse than a missing one.
**Mode:** mvp
**Depends on**: Phase 5
**Requirements**: [VAL-06, VAL-07, VAL-08, VAL-09]
**Success Criteria** (what must be TRUE):
  1. System computes time-weighted return so a deposit or withdrawal does not distort the reported performance figure
  2. System computes money-weighted return (XIRR) alongside TWR, both clearly labeled and explained in plain language
  3. A TRY-denominated position shows separate asset-return and currency-return figures displayed together, so a lira gain that was a dollar loss is visible as such
  4. Every return calculation (TWR, XIRR, FX attribution) is checked by an automated test suite against a documented set of golden-value cases with known correct answers, and the suite passes
**Plans**: TBD
**UI hint**: yes

### Phase 7: Exposure, Risk & BIST Fundamentals
**Goal**: The family sees the exposures no position-by-position view reveals — concentration by position, sector and currency, portfolio-level risk, and how they stack up against benchmarks — powered by BIST fundamentals parsed from KAP, which the spike confirmed is free and legally obtainable; the residual work is a parser against current KAP markup.
**Mode:** mvp
**Depends on**: Phase 6
**Requirements**: [EXPO-01, EXPO-02, EXPO-03, EXPO-04, EXPO-05, EXPO-06, EXPO-07, EXPO-08, EXPO-09, INST-07, DATA-08, DATA-09]
**Success Criteria** (what must be TRUE):
  1. System ingests BIST company financial statements from KAP filings (parsed against the `taxonomy-context-value` markup, ported to TypeScript) into structured fundamentals that tag each instrument's sector and industry, and a parse failure alerts rather than storing wrong figures
  2. User sees allocation broken down by asset class, sector, industry, geography and currency
  3. System flags when a single position, sector or currency exceeds a configured share of the portfolio
  4. User sees portfolio volatility, maximum drawdown, Sharpe ratio, and correlation between holdings, surfacing positions that move together
  5. User can compare portfolio performance against the S&P 500, BIST 100, gold and a USD deposit benchmark
**Plans**: TBD
**UI hint**: yes

### Phase 8: Thesis & Decision Rules
**Goal**: The family can write down, before emotion sets in, why they bought each position and exactly what would make them sell — the single feature most likely to change financial outcomes, and the mechanism that makes "when to cut losses" answerable at all.
**Mode:** mvp
**Depends on**: Phase 2
**Requirements**: [THES-01, THES-02, THES-03, THES-04, THES-05, THES-06, THES-07, THES-11]
**Success Criteria** (what must be TRUE):
  1. User can record a structured thesis for a position — why bought, target price, stop-loss, maximum position size, and what would prove the thesis wrong — as distinct fields, never free text
  2. User can set portfolio-wide default rules (max position %, max sector %, stop-loss %) that apply automatically to new positions
  3. User can override any portfolio-wide default on an individual position
  4. User can review and update a thesis, with previous versions retained and viewable
**Plans**: TBD
**UI hint**: yes

### Phase 9: Rule Evaluation, Alerts & Attention-First Home
**Goal**: The system actively watches every recorded rule against live prices and tells the family what needs attention this week and why — on Telegram and on a home screen that leads with what's breached rather than a list of holdings — with rule-derived facts made visually distinct from opinions before any opinion-generating feature ships.
**Mode:** mvp
**Depends on**: Phase 7, Phase 8
**Requirements**: [THES-08, THES-09, THES-10, THES-12, ALERT-01, ALERT-02, ALERT-04, ALERT-05, ALERT-06, UI-05, UI-07, UI-08]
**Success Criteria** (what must be TRUE):
  1. System evaluates every position against its rules on each data refresh and raises an alert stating which rule was breached and by how much, without re-alerting daily for the same unresolved breach
  2. User can link a Telegram account and receives rule-breach alerts there, with every alert also visible inside the app
  3. Each family member configures their own alert sensitivity independently of the others, and delivery runs behind a channel-agnostic interface so a second channel could be added without touching alert logic
  4. User sees a periodic review that names the specific positions needing attention and why, and the home screen leads with that list rather than with a list of holdings
  5. Rule-breach facts are visually and linguistically distinct from opinion-based content everywhere both will appear, and advice prescriptiveness (specific sized actions vs. gentle flags) is configurable per user
**Plans**: TBD
**UI hint**: yes

### Phase 10: News, Macro & Daily Brief
**Goal**: The family receives a daily Telegram brief and an in-app news feed covering only their own holdings — deduplicated, summarized, sourced from both Turkish and international coverage — with macro context always translated into what it means for what they own.
**Mode:** mvp
**Depends on**: Phase 9
**Requirements**: [NEWS-01, NEWS-02, NEWS-03, NEWS-04, NEWS-05, NEWS-06, NEWS-07, NEWS-08, ALERT-03]
**Success Criteria** (what must be TRUE):
  1. User sees news filtered to their own holdings, deduplicated across sources and summarized rather than shown as raw headlines, covering both Turkish and international sources
  2. System alerts on material events — earnings, guidance changes, large price moves, analyst rating changes — for held instruments
  3. User sees a macro layer (policy rates, inflation, TRY, commodities) and an economic calendar, always interpreted as what it means for their own holdings
  4. User sees aggregated analyst views and price targets for their holdings
  5. User receives a daily Telegram brief covering only their holdings: what moved, why, and what is coming
**Plans**: TBD
**UI hint**: yes

### Phase 11: Agent Core & Read Access
**Goal**: The family can ask the embedded Claude agent a question about their real holdings and get a correct, tenant-scoped answer — through narrow typed tools backed by Postgres RLS — sequenced only now that everything the agent could report on (ledger, valuation, returns, exposure, rules, news) has already been proven trustworthy.
**Mode:** mvp
**Depends on**: Phase 9
**Requirements**: [AGENT-01, AGENT-02, AGENT-03, AGENT-10, AGENT-13, AGENT-14, AGENT-15]
**Success Criteria** (what must be TRUE):
  1. User can ask the agent a natural-language question and receive an answer computed from their real holdings and transactions, never a number the model produced by its own arithmetic
  2. Every agent data access goes through a narrow typed tool that derives the portfolio/user identity from the verified session — never a general SQL tool, never a model-supplied identifier — and a direct database-level test confirms one user's agent session cannot retrieve another's rows
  3. Agent can search the web and fetch documents as part of answering, and any content it reads is structurally treated as data, never as instructions it can act on
  4. Agent answers in whichever language it was asked in, Turkish or English
  5. User sees their own agent token usage and cost, and conversation history persists across sessions
**Plans**: TBD
**UI hint**: yes

### Phase 12: Agent Write Access, Audit & Reversal
**Goal**: The agent can act on the family's own data — adding transactions, watchlists and rules — with the deterministic validation layer, the audit log and one-click reversal shipping in this same phase, never as a follow-up, because the first hallucinated or injected write can corrupt downstream calculations before a later phase starts.
**Mode:** mvp
**Depends on**: Phase 11
**Requirements**: [AGENT-08, AGENT-09, AGENT-11, AGENT-12]
**Success Criteria** (what must be TRUE):
  1. Agent can add a transaction, create a watchlist, and set a rule on the user's own data
  2. A deliberately malformed agent write attempt (unresolvable ticker, price wildly off market, negative resulting quantity, implausible date) is rejected or flagged, not silently accepted — every agent write passes the same deterministic validation a human write would
  3. Every agent-initiated change is recorded in an append-only audit log showing what changed and why, and it is surfaced to the user proactively rather than requiring them to go looking for it
  4. User can reverse any agent-initiated change in one click, and the reversal is itself a new, audited ledger entry rather than a silent rollback
**Plans**: TBD
**UI hint**: yes

### Phase 13: Agent Deep Research & Code Execution
**Goal**: The agent can go beyond a single Q&A turn — running multi-step research that survives longer than one request and executing code against a read-only snapshot of the family's data to produce charts and custom analysis — without the sandbox ever touching live database credentials.
**Mode:** mvp
**Depends on**: Phase 12
**Requirements**: [AGENT-04, AGENT-05, AGENT-06, AGENT-07]
**Success Criteria** (what must be TRUE):
  1. Agent can run multi-step deep research with streaming progress visible to the user as each step happens
  2. A deliberately long research session survives beyond a single request timeout by resuming from its last completed step, without losing prior progress
  3. Agent can execute code against a read-only, pre-fetched snapshot of the user's data to produce charts, scenario analysis and custom calculations on demand
  4. The code execution environment never holds live database credentials, even if the generated code is malicious or the sandbox is compromised
**Plans**: TBD
**UI hint**: yes

### Phase 14: Discovery
**Goal**: The system proposes new investment ideas across US and BIST, each with a written thesis and a value/quality/technical score, informed by what the family already owns and lacks — genuinely last, since asking the family to trust the system's opinions only makes sense once it has proven it can correctly show what they already own.
**Mode:** mvp
**Depends on**: Phase 13
**Requirements**: [DISC-01, DISC-02, DISC-03, DISC-04]
**Success Criteria** (what must be TRUE):
  1. System proposes new investment ideas across both US and BIST markets
  2. Each idea carries a written thesis explaining the reasoning, scored on value, quality and technical criteria
  3. Idea generation accounts for what the user already owns and which exposures they lack, rather than generating ideas in a vacuum
**Plans**: TBD
**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10 → 11 → 12 → 13 → 14

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation — Sign In & First Position | 0/TBD | Not started | - |
| 2. Full Ledger, All Transaction Types & Fast Bulk Entry | 0/TBD | Not started | - |
| 3. Market Data Sync & First Valuation | 0/TBD | Not started | - |
| 4. Presentation Modes, Localization & Design System | 0/TBD | Not started | - |
| 5. Multi-Currency, Corporate Actions & Portfolio-Wide Valuation | 0/TBD | Not started | - |
| 6. Returns Engine — TWR, XIRR & FX Attribution | 0/TBD | Not started | - |
| 7. Exposure, Risk & BIST Fundamentals | 0/TBD | Not started | - |
| 8. Thesis & Decision Rules | 0/TBD | Not started | - |
| 9. Rule Evaluation, Alerts & Attention-First Home | 0/TBD | Not started | - |
| 10. Agent Core & Read Access | 0/TBD | Not started | - |
| 11. Agent Write Access, Audit & Reversal | 0/TBD | Not started | - |
| 12. Agent Deep Research & Code Execution | 0/TBD | Not started | - |
| 13. News, Macro & Daily Brief | 0/TBD | Not started | - |
| 14. Discovery | 0/TBD | Not started | - |
