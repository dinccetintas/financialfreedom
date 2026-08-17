# Requirements: Financial Freedom

**Defined:** 2026-08-16
**Core Value:** Turn "I own 17 stocks and I'm lost" into "here are the three things that need your attention this week, and here's what to do about each."

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Authentication & Tenancy

- [x] **AUTH-01**: User can sign in with email and password
- [ ] **AUTH-02**: User can reset a forgotten password via an emailed link
- [x] **AUTH-03**: User session persists across browser refresh and across devices
- [ ] **AUTH-04**: Signup is closed to a fixed allowlist — no public registration surface exists
- [ ] **AUTH-05**: Every tenant-scoped table enforces row-level security keyed to the authenticated session, so no query path can return another user's rows
- [ ] **AUTH-06**: User can create and name multiple portfolios under their own account
- [ ] **AUTH-07**: User can share a specific portfolio with a named family member, and revoke that share
- [ ] **AUTH-08**: Portfolios are private by default — sharing is always an explicit act

### Ledger & Transactions

- [ ] **LEDG-01**: User can record a buy transaction with date, instrument, quantity, price, fees and currency
- [ ] **LEDG-02**: User can record a sell transaction, with cost basis resolved FIFO against existing lots
- [ ] **LEDG-03**: User can record cash movements: deposit, withdrawal, fee and interest
- [ ] **LEDG-04**: User can record a dividend receipt, including withholding tax
- [ ] **LEDG-05**: User can record an FX conversion between two currencies
- [ ] **LEDG-06**: Transactions are stored in an append-only ledger — corrections create reversing entries rather than mutating history
- [ ] **LEDG-07**: Positions are derived from the ledger and can be rebuilt from scratch at any time
- [ ] **LEDG-08**: User can edit or delete a transaction they entered, with the correction visible in history
- [x] **LEDG-09**: All monetary values, FX rates and share quantities use decimal arithmetic end to end — never floating point
- [ ] **LEDG-10**: User can view, filter and sort the full transaction history for a portfolio
- [ ] **LEDG-11**: User can bulk-enter transactions quickly so one person can set up another family member's portfolio in a single sitting

### Instruments & Corporate Actions

- [ ] **INST-01**: System identifies every instrument by exchange and symbol plus ISIN, never by bare ticker
- [ ] **INST-02**: User can hold BIST equities, US equities and ETFs, gold, foreign currency and cash in the same portfolio
- [ ] **INST-03**: User can search for and add an instrument by ticker or company name
- [ ] **INST-04**: System records stock splits and adjusts historical positions correctly
- [ ] **INST-05**: System records BIST bonus issues (bedelsiz) as free shares that increase quantity without changing total cost basis
- [ ] **INST-06**: System records BIST rights issues (bedelli) as a real cash outlay that adds to cost basis and counts as a cash flow for return calculations
- [ ] **INST-07**: Instruments carry sector and industry classification for exposure analysis

### Market Data

- [ ] **DATA-01**: System syncs end-of-day prices for every held instrument on a schedule
- [ ] **DATA-02**: System stores historical daily USD/TRY and gold rates sourced from TCMB EVDS
- [ ] **DATA-03**: System uses a market FX rate at transaction entry time and TCMB rates for historical backfill
- [ ] **DATA-04**: System respects each market's trading calendar — BIST and US holidays are handled distinctly
- [ ] **DATA-05**: Missing price bars are recorded as missing, never silently treated as zero or carried forward without marking
- [ ] **DATA-06**: System backfills price history when a new instrument is added
- [ ] **DATA-07**: A data sync failure raises a visible alert rather than silently serving stale prices
- [ ] **DATA-08**: System ingests BIST company financial statements from KAP filings into structured fundamentals
- [ ] **DATA-09**: A KAP parse failure alerts rather than storing partial or wrong figures

### Valuation & Returns

- [ ] **VAL-01**: User sees current market value for every position and portfolio total in USD
- [ ] **VAL-02**: Every transaction stores the FX rate in effect at transaction time, freezing cost basis and realized P&L permanently
- [ ] **VAL-03**: Unrealized valuation converts at a date-specific live rate, so current value moves with the market
- [ ] **VAL-04**: User sees unrealized and realized profit and loss per position and per portfolio
- [ ] **VAL-05**: User sees cost basis per position, computed FIFO across lots
- [ ] **VAL-06**: System computes time-weighted return so deposits and withdrawals do not distort performance
- [ ] **VAL-07**: System computes money-weighted return (XIRR) alongside TWR, with both explained in plain language
- [ ] **VAL-08**: System separates total return into asset return and currency return, so a lira gain that was a dollar loss is visible as such
- [ ] **VAL-09**: Return calculations are verified against known golden-value test cases
- [ ] **VAL-10**: User sees portfolio value over time as a chart, in USD

### Exposure & Risk

- [ ] **EXPO-01**: User sees allocation broken down by asset class
- [ ] **EXPO-02**: User sees allocation broken down by sector and industry
- [ ] **EXPO-03**: User sees allocation broken down by geography and by currency
- [ ] **EXPO-04**: System flags when a single position exceeds a configured share of the portfolio
- [ ] **EXPO-05**: System flags when a sector or currency exceeds a configured share of the portfolio
- [ ] **EXPO-06**: User sees portfolio volatility and maximum drawdown
- [ ] **EXPO-07**: User sees Sharpe ratio for the portfolio
- [ ] **EXPO-08**: User sees correlation between holdings, surfacing positions that move together
- [ ] **EXPO-09**: User can compare portfolio performance against S&P 500, BIST 100, gold and a USD deposit benchmark

### Thesis & Decision Discipline

- [ ] **THES-01**: User can record why they bought a position, as structured fields rather than free text
- [ ] **THES-02**: User can set a target price for a position
- [ ] **THES-03**: User can set a stop-loss level for a position
- [ ] **THES-04**: User can set a maximum position size for a holding
- [ ] **THES-05**: User can record what would prove the thesis wrong
- [ ] **THES-06**: User can set portfolio-wide default rules that apply automatically to new positions
- [ ] **THES-07**: User can override any portfolio-wide default on an individual position
- [ ] **THES-08**: System evaluates every position against its rules on each data refresh
- [ ] **THES-09**: System raises an alert when a rule is breached, stating which rule and by how much
- [ ] **THES-10**: Repeated breaches of the same rule do not re-alert daily
- [ ] **THES-11**: User can review and update a thesis, with previous versions retained
- [ ] **THES-12**: User sees a periodic review naming the specific positions needing attention and why

### Alerts & Delivery

- [ ] **ALERT-01**: User can link a Telegram account to receive alerts
- [ ] **ALERT-02**: System delivers rule-breach alerts to Telegram
- [ ] **ALERT-03**: System delivers a daily brief covering only the user's holdings: what moved, why, and what is coming
- [ ] **ALERT-04**: Each user configures their own alert sensitivity independently of other family members
- [ ] **ALERT-05**: Delivery is channel-agnostic, so an additional channel can be added without touching alert logic
- [ ] **ALERT-06**: Every alert is also visible in the application, so nothing exists only in a message

### News & Macro

- [ ] **NEWS-01**: User sees news filtered to their own holdings
- [ ] **NEWS-02**: News is deduplicated across sources
- [ ] **NEWS-03**: News items are summarized rather than shown as raw headlines
- [ ] **NEWS-04**: System alerts on material events: earnings, guidance changes, large price moves, analyst rating changes
- [ ] **NEWS-05**: User sees a macro layer covering policy rates, inflation, TRY and commodities
- [ ] **NEWS-06**: User sees an economic calendar of upcoming events
- [ ] **NEWS-07**: Macro information is always presented with an interpretation of what it means for the user's own holdings
- [ ] **NEWS-08**: User sees aggregated analyst views and price targets for their holdings

### AI Agent

- [ ] **AGENT-01**: User can ask natural-language questions and receive answers computed from their real holdings
- [ ] **AGENT-02**: Agent reaches data only through narrow typed tools that derive identity from the verified session — never a general SQL tool, never a model-supplied user identifier
- [ ] **AGENT-03**: Agent can search the web and fetch documents as part of answering
- [ ] **AGENT-04**: Agent can run multi-step deep research with streaming progress visible to the user
- [ ] **AGENT-05**: Deep research survives beyond a single request timeout without losing work
- [ ] **AGENT-06**: Agent can execute code against a read-only snapshot of the user's data to produce charts and analysis
- [ ] **AGENT-07**: The code execution environment never holds database credentials
- [ ] **AGENT-08**: Agent can add transactions, create watchlists and set rules on the user's own data
- [ ] **AGENT-09**: Every agent write passes the same deterministic validation as a human write — instrument resolves, price within a sane band, quantity non-negative, date plausible
- [ ] **AGENT-10**: Content fetched from the web is treated as data, never as instructions the agent can act on
- [ ] **AGENT-11**: Every agent-initiated change is recorded in an append-only audit log showing what changed and why
- [ ] **AGENT-12**: User can reverse any agent-initiated change in one click
- [ ] **AGENT-13**: Agent answers in the language it is asked in
- [ ] **AGENT-14**: User sees their own agent token usage and cost
- [ ] **AGENT-15**: Conversation history persists across sessions

### Discovery

- [ ] **DISC-01**: System proposes new investment ideas across US and BIST
- [ ] **DISC-02**: Each idea carries a written thesis explaining the reasoning
- [ ] **DISC-03**: Ideas are scored on value, quality and technical criteria
- [ ] **DISC-04**: Idea generation accounts for what the user already owns and which exposures they lack

### Presentation & Accessibility

- [ ] **UI-01**: User can switch the interface between Turkish and English
- [ ] **UI-02**: User can switch between Simple mode and Pro mode
- [ ] **UI-03**: Simple mode presents plain language, few numbers, and clear actions with no jargon
- [ ] **UI-04**: Pro mode presents full metrics and analytics
- [ ] **UI-05**: Advice prescriptiveness is configurable per user, from specific sized actions to gentle flags
- [ ] **UI-06**: Every screen is usable at 390px width
- [ ] **UI-07**: Rule-based signals are visually distinct from AI-generated opinions everywhere both appear
- [ ] **UI-08**: The home screen leads with what needs attention, not with a list of holdings

### Design Deliverables

- [ ] **DES-01**: Each page has a written design prompt specifying its data, states and hierarchy, suitable for pasting into Claude Design
- [ ] **DES-02**: A design system defines colour, type, spacing and chart styling consistently across pages
- [ ] **DES-03**: Number formatting is consistent and locale-aware across Turkish and English

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Import

- **IMP-01**: User can upload a broker statement (Midas, İş Yatırım or similar) and have transactions parsed automatically
- **IMP-02**: User can import transactions from a CSV or Excel file with column mapping
- **IMP-03**: System detects and prevents duplicate imports

### Advanced Monitoring

- **MON-01**: System ingests KAP material-event disclosures continuously
- **MON-02**: Thesis invalidation is monitored against news and filings, not only price
- **MON-03**: System detects when multiple holdings depend on the same underlying assumption
- **MON-04**: Rule-breach alerts cite the specific dated source that triggered them

### Advanced Analysis

- **ADV-01**: User sees bull, base and bear scenarios per position with triggers
- **ADV-02**: System proposes risk-budgeted rebalancing given a target risk level
- **ADV-03**: User can backtest a strategy against their own holdings
- **ADV-04**: System tracks notable-investor position changes in the user's holdings

### Platform

- **PLAT-01**: Alerts can be delivered via WhatsApp
- **PLAT-02**: Broker API synchronization replaces manual entry where available
- **PLAT-03**: Public signup, billing and subscription management
- **PLAT-04**: Tax reporting for Turkish and US holdings

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Crypto assets | Nobody in the family holds any; adds custody, wallet sync and 24/7 pricing complexity for zero current value |
| Order execution / trading | The system advises; trades are placed at the broker. Execution adds regulatory and security burden far beyond the goal |
| Real-time streaming BIST quotes | Requires a licensed Borsa İstanbul agreement costing hundreds of USD monthly; unnecessary for buy-and-hold |
| Automated rebalancing | The system proposes, humans decide |
| Continuous KAP material-event ingestion in v1 | Carries the real scraping-frequency and legal grey area for materially less v1 value than periodic statements |
| TEFAS fund data | No official API and no family holdings in Turkish mutual funds |
| Social features | No feeds, following or public performance comparison — this is private family financial data |
| Separate Python service | At ~400K rows the maths fits comfortably in TypeScript; a second deployment target is unjustified |
| Time-series database | Plain indexed Postgres handles this data volume trivially |
| Job queue infrastructure | 40–60 instruments sync in a single scheduled function |
| Third-party code sandbox | Anthropic's native code execution covers this at no meaningful cost |
| Agent spending caps, loop breakers and pre-run cost estimates | Owner's explicit decision after seeing worst-case figures; usage visibility only |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 1 | Complete |
| AUTH-02 | Phase 1 | Pending |
| AUTH-03 | Phase 1 | Complete |
| AUTH-04 | Phase 1 | Pending |
| AUTH-05 | Phase 1 | Pending |
| AUTH-06 | Phase 1 | Pending |
| AUTH-07 | Phase 1 | Pending |
| AUTH-08 | Phase 1 | Pending |
| LEDG-01 | Phase 1 | Pending |
| LEDG-06 | Phase 1 | Pending |
| LEDG-07 | Phase 1 | Pending |
| LEDG-09 | Phase 1 | Complete |
| LEDG-10 | Phase 1 | Pending |
| INST-01 | Phase 1 | Pending |
| INST-03 | Phase 1 | Pending |
| LEDG-02 | Phase 2 | Pending |
| LEDG-03 | Phase 2 | Pending |
| LEDG-04 | Phase 2 | Pending |
| LEDG-05 | Phase 2 | Pending |
| LEDG-08 | Phase 2 | Pending |
| LEDG-11 | Phase 2 | Pending |
| INST-02 | Phase 2 | Pending |
| DATA-01 | Phase 3 | Pending |
| DATA-04 | Phase 3 | Pending |
| DATA-05 | Phase 3 | Pending |
| DATA-06 | Phase 3 | Pending |
| DATA-07 | Phase 3 | Pending |
| VAL-01 | Phase 3 | Pending |
| UI-01 | Phase 4 | Pending |
| UI-02 | Phase 4 | Pending |
| UI-03 | Phase 4 | Pending |
| UI-04 | Phase 4 | Pending |
| UI-06 | Phase 4 | Pending |
| DES-01 | Phase 4 | Pending |
| DES-02 | Phase 4 | Pending |
| DES-03 | Phase 4 | Pending |
| DATA-02 | Phase 5 | Pending |
| DATA-03 | Phase 5 | Pending |
| VAL-02 | Phase 5 | Pending |
| VAL-03 | Phase 5 | Pending |
| VAL-04 | Phase 5 | Pending |
| VAL-05 | Phase 5 | Pending |
| VAL-10 | Phase 5 | Pending |
| INST-04 | Phase 5 | Pending |
| INST-05 | Phase 5 | Pending |
| INST-06 | Phase 5 | Pending |
| VAL-06 | Phase 6 | Pending |
| VAL-07 | Phase 6 | Pending |
| VAL-08 | Phase 6 | Pending |
| VAL-09 | Phase 6 | Pending |
| EXPO-01 | Phase 7 | Pending |
| EXPO-02 | Phase 7 | Pending |
| EXPO-03 | Phase 7 | Pending |
| EXPO-04 | Phase 7 | Pending |
| EXPO-05 | Phase 7 | Pending |
| EXPO-06 | Phase 7 | Pending |
| EXPO-07 | Phase 7 | Pending |
| EXPO-08 | Phase 7 | Pending |
| EXPO-09 | Phase 7 | Pending |
| INST-07 | Phase 7 | Pending |
| DATA-08 | Phase 7 | Pending |
| DATA-09 | Phase 7 | Pending |
| THES-01 | Phase 8 | Pending |
| THES-02 | Phase 8 | Pending |
| THES-03 | Phase 8 | Pending |
| THES-04 | Phase 8 | Pending |
| THES-05 | Phase 8 | Pending |
| THES-06 | Phase 8 | Pending |
| THES-07 | Phase 8 | Pending |
| THES-11 | Phase 8 | Pending |
| THES-08 | Phase 9 | Pending |
| THES-09 | Phase 9 | Pending |
| THES-10 | Phase 9 | Pending |
| THES-12 | Phase 9 | Pending |
| ALERT-01 | Phase 9 | Pending |
| ALERT-02 | Phase 9 | Pending |
| ALERT-04 | Phase 9 | Pending |
| ALERT-05 | Phase 9 | Pending |
| ALERT-06 | Phase 9 | Pending |
| UI-05 | Phase 9 | Pending |
| UI-07 | Phase 9 | Pending |
| UI-08 | Phase 9 | Pending |
| AGENT-01 | Phase 11 | Pending |
| AGENT-02 | Phase 11 | Pending |
| AGENT-03 | Phase 11 | Pending |
| AGENT-10 | Phase 11 | Pending |
| AGENT-13 | Phase 11 | Pending |
| AGENT-14 | Phase 11 | Pending |
| AGENT-15 | Phase 11 | Pending |
| AGENT-08 | Phase 12 | Pending |
| AGENT-09 | Phase 12 | Pending |
| AGENT-11 | Phase 12 | Pending |
| AGENT-12 | Phase 12 | Pending |
| AGENT-04 | Phase 13 | Pending |
| AGENT-05 | Phase 13 | Pending |
| AGENT-06 | Phase 13 | Pending |
| AGENT-07 | Phase 13 | Pending |
| NEWS-01 | Phase 10 | Pending |
| NEWS-02 | Phase 10 | Pending |
| NEWS-03 | Phase 10 | Pending |
| NEWS-04 | Phase 10 | Pending |
| NEWS-05 | Phase 10 | Pending |
| NEWS-06 | Phase 10 | Pending |
| NEWS-07 | Phase 10 | Pending |
| NEWS-08 | Phase 10 | Pending |
| ALERT-03 | Phase 10 | Pending |
| DISC-01 | Phase 14 | Pending |
| DISC-02 | Phase 14 | Pending |
| DISC-03 | Phase 14 | Pending |
| DISC-04 | Phase 14 | Pending |

**Coverage:**

- v1 requirements: 110 total (corrected — the original 103 count in this section's prior draft did not match the 110 distinct requirement IDs actually listed under "v1 Requirements" above)
- Mapped to phases: 110 (100%)
- Unmapped: 0

---
*Requirements defined: 2026-08-16*
*Last updated: 2026-08-16 after roadmap creation*
