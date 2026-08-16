# Financial Freedom

## What This Is

A private, family-only portfolio **manager** — not a tracker — for four people (the owner, mother, father, brother) holding Borsa İstanbul equities, US stocks and ETFs, gold, FX and cash. It gives an accurate USD-based picture of what each person owns, then adds the layer every consumer tracker is missing: the exposures you cannot see position-by-position, a written thesis and pre-committed exit rules for every holding, proactive Telegram alerts when something breaches those rules, and an embedded Claude agent that can query the portfolio, research the web, run analysis and act on the data.

The framing that drives every design decision: **it should behave like a Wall Street portfolio manager who happens to know your family**, not like a spreadsheet with a chart on top.

## Core Value

**Turn "I own 17 stocks and I'm lost" into "here are the three things that need your attention this week, and here's what to do about each."**

The accurate USD picture is the foundation and must be correct before anything else is trustworthy — but an accurate picture that produces no decisions is a failure. If the system tells you what you own but never tells you what to do, it has not solved the problem.

## Requirements

### Validated

(None yet — ship to validate)

### Active

**Foundation — the accurate picture**

- [ ] User can create an account and sign in with email + password; signup is closed to a fixed allowlist
- [ ] User can create multiple portfolios (by broker account or purpose) under their own login
- [ ] User can manually record buy, sell, dividend, deposit, withdrawal, fee and FX-conversion transactions
- [ ] User can hold BIST equities, US equities/ETFs, gold, foreign currency and cash in the same portfolio
- [ ] System values every holding in its native currency and converts to a USD base using the FX rate on the relevant date
- [ ] User sees current value, cost basis, unrealized and realized P&L per position and per portfolio
- [ ] System computes time-weighted return (TWR) and money-weighted return (XIRR) so deposits and withdrawals do not distort performance
- [ ] System separates return into asset return vs FX return (FX attribution), so a TRY gain that was a USD loss is visible as such
- [ ] Portfolios are private by default; a user can explicitly share a portfolio with another family member

**Seeing what you cannot see position-by-position**

- [ ] User sees allocation broken down by asset class, sector, industry, geography and currency
- [ ] System flags concentration risk when a single position, sector or currency exceeds a configured threshold
- [ ] User sees portfolio-level risk metrics: volatility, maximum drawdown, Sharpe ratio, and correlation between holdings
- [ ] User can compare portfolio performance against benchmarks (S&P 500, BIST 100, gold, USD deposit rate)
- [ ] User sees dividend income received, yield on cost, and an upcoming dividend calendar

**Decision discipline — the highest-leverage feature**

- [ ] User can record a thesis for each position: why bought, target price, stop-loss, maximum position size, and what would prove the thesis wrong
- [ ] System monitors every position against its recorded rules and raises an alert when a rule is breached
- [ ] User can set portfolio-wide default rules (max position %, max sector %, stop-loss %) that apply automatically and can be overridden per position
- [ ] System produces a periodic review that names the specific positions needing attention and why, rather than presenting undifferentiated data

**Staying informed**

- [ ] User sees news filtered to their own holdings, deduplicated and summarized, covering both international sources and Turkish sources
- [ ] System ingests BIST company financial statements from KAP filings, parsed into structured fundamentals that power sector tagging and screening
- [ ] User receives a daily AI-written brief covering only their holdings: what moved, why, and what is coming
- [ ] User receives real-time alerts for material events: earnings, guidance changes, large price moves, analyst rating changes, rule breaches
- [ ] Alerts and briefs are delivered to Telegram through a channel-agnostic delivery engine
- [ ] User sees a macro layer — policy rates, inflation, TRY, commodities, economic calendar — always interpreted as "what this means for your holdings"
- [ ] User sees aggregated analyst and notable-investor sentiment on their specific holdings

**The agent**

- [ ] User can ask the Claude agent natural-language questions and receive answers computed from their real holdings and transactions
- [ ] Agent can search the web, fetch filings and disclosures, and run multi-step deep research with streaming progress
- [ ] Agent can execute code against the user's portfolio data to produce charts, scenario analysis and custom calculations on demand
- [ ] Agent can write to the user's own data (add transactions, create watchlists, set alerts and rules) with every change recorded in an immutable audit log and reversible in one click
- [ ] Agent answers in the language it is asked in (Turkish or English)
- [ ] System records per-user agent token usage and cost, visible to the user as a spend dashboard (no caps, loop breakers or pre-run estimates — deliberate owner decision)

**Discovery**

- [ ] System proposes new investment ideas across US and BIST with a written thesis, scored on value, quality and technicals
- [ ] Idea generation accounts for what the user already owns and which exposures they lack

**Making it usable by everyone**

- [ ] User can switch the interface between Turkish and English
- [ ] User can switch between Simple mode (plain language, few actions, large numbers, no jargon) and Pro mode (full metrics and analytics)
- [ ] Advice prescriptiveness is configurable per user — specific sized actions for one member, gentler flags for another
- [ ] Interface works equally well on mobile and desktop

### Out of Scope

- **Crypto assets** — nobody in the family holds any; adds custody, wallet-sync and 24/7 pricing complexity for zero current value
- **Live broker API sync for v1** — most Turkish brokers expose no public API, and aggregators have poor Turkish coverage; the data model will accommodate it but v1 does not build it
- **Real-time streaming BIST quotes** — requires a licensed Borsa İstanbul data agreement costing hundreds of USD per month; delayed and end-of-day pricing is sufficient for a buy-and-hold family
- **Order execution / trading** — this system advises and tracks; trades are placed at the broker. Execution introduces regulatory and security burden far beyond the goal
- **Public signup, billing and subscriptions** — family-only for now; architecture stays multi-tenant-ready so this is additive later, not a rewrite
- **Tax reporting and capital-gains calculation** — Turkish and US tax treatment differ per instrument and holding period; correctness here is a project of its own
- **WhatsApp delivery for v1** — requires Meta business verification, per-message templates and per-message cost; Telegram delivers the same value in hours instead of weeks. The delivery engine stays channel-agnostic
- **Social features** — no sharing feeds, following, or public performance comparison; this is private family financial data
- **Continuous KAP material-event ingestion (özel durum açıklamaları) in v1** — carries the genuine scraping-frequency and legal-grey-area concern, for materially less value than the periodic statements; thesis invalidation uses price rules plus agent web search until the event feed earns its place
- **TEFAS fund data** — no official API and no family holdings in Turkish mutual funds today
- **Automated trade execution or auto-rebalancing** — the system proposes, humans decide

## Context

**The problem, in the owner's words:** The family manages everything manually through broker interfaces today. They buy a stock if they like its fundamentals, but have no view of industry exposure, no Sharpe ratio, no sense of whether it is a good time to buy. The father holds 17 stocks and they are "completely lost" tracking them — it is very hard to know what is happening across 17 names. They do not know when to sell, when to cut losses, what the strategy on a given stock is, what the best and worst case looks like, or how to adjust the portfolio for a given risk level. They want to know what is happening with the FED and what strategies experienced investors are advising.

**The critical insight:** Nobody is genuinely lost *tracking* 17 positions — a spreadsheet does that. They are lost *deciding*. Every competing product answers "what do I own and what is it worth?", which the family can already read off a broker screen. The unmet need is "given what I own, what should I do?" This is why the product is positioned as a manager rather than a tracker, and why exposure analysis and pre-committed exit rules rank above prettier charts.

**Why exit rules matter more than analytics:** By the time a position is down 30%, no amount of analysis helps — only a rule written while calm. Recording thesis, target, stop and invalidation conditions at purchase time is the only mechanism that makes "when to cut losses" answerable, and it is the feature most likely to change financial outcomes.

**Investment style:** Buy and hold, with openness to revisiting the portfolio for new opportunities on roughly a three-month cadence. Scale is around 17 positions per person across four people — hundreds of transactions, not hundreds of thousands. This means correctness matters far more than throughput.

**The audience is not uniform:** The owner is comfortable with analytics and wants specific, sized actions. The father needs everything to be immediately understandable, with suggestions and actions rather than a data dump. The same underlying data must be presentable two ways.

**Currency reality:** With TRY-denominated BIST holdings and a USD base currency, FX movement can dominate asset returns entirely. A position up 40% in lira may be flat or down in dollars. Reporting that does not separate asset return from FX return will actively mislead the family, which is why FX attribution is a foundation requirement rather than an analytics nicety.

**Competitive landscape (researched):** Fintables is the strongest Turkish reference — BIST-native, combines fundamentals with news and balance-sheet interpretation, and its portfolio view already charts value over time and supports foreign-currency display. Sharesight is the reference for correctness on multi-currency cost basis and TWR/IRR. Bistify proves Midas broker statements are parseable, validating the later import path. Kubera covers non-market assets like gold and cash. getquin is the mobile-first execution reference. None of them has an agent that can query your holdings, research the web and run analysis — that gap is this project's differentiator.

**UI workflow:** The owner will drive visual design through Claude Design. This project must therefore produce per-page design prompts — specifying what each screen shows, its data, states and hierarchy — as an explicit deliverable, rather than generating final visual design directly.

**Infrastructure already available:** Vercel is connected. Notion, Google Drive and GitHub are available as integrations.

## Constraints

- **Budget**: Approximately $50–150/month total running cost — Constrains market data provider selection; rules out licensed real-time BIST feeds and premium fundamentals APIs
- **Data availability**: BIST fundamentals come from KAP, the regulator's own platform, parsed from published filings — Confirmed obtainable free by hands-on spike (see `.planning/research/SPIKE-KAP.md`); the residual risk is parser maintenance against KAP markup changes, not data sourcing. BIST end-of-day prices still come from a commercial provider and remain unverified until the provider spike
- **Tech stack**: Next.js on Vercel with Postgres — Owner's preference; Vercel already connected; strong fit for the app, though the returns/risk math may warrant a separate compute path
- **Users**: Exactly four, closed signup, no public registration — Removes onboarding, billing and compliance surface from v1
- **Multi-tenancy**: Must be architecturally multi-tenant-ready from day one — "Family now, maybe more later"; proper user isolation with no hardcoded assumptions, but no billing or public onboarding built
- **Languages**: Full Turkish and English interface — Parents will not use an English-only finance application; retrofitting i18n costs far more than building it in
- **Devices**: Mobile and desktop equal priority — Portfolio checks happen on phones; analysis happens on desktop. Charts and tables must work at 390px width
- **Agent cost**: Shared Anthropic API key with no hard caps, but per-user usage must be visible — Owner's explicit decision; visibility is the compromise that costs nothing at write time
- **Data integrity**: Agent may write to user data without confirmation, but every write must be recorded in an immutable audit log and be reversible — Owner accepted the risk of unconfirmed writes; auditability is the non-negotiable mitigation
- **Regulatory**: Prescriptive buy/sell advice is acceptable for family use but becomes regulated investment advice if non-family users are ever admitted — The advice engine must stay cleanly separable so it can be gated later
- **Pace**: Steady, quality over speed — Money math must be correct; a wrong return figure is worse than a missing one

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Position as a portfolio *manager*, not a tracker | The family's stated pain is deciding, not tracking; every competitor already solves tracking | — Pending |
| USD as base currency with FX attribution | TRY inflation and depreciation make lira-denominated returns misleading about real purchasing power | — Pending |
| Include TWR and XIRR despite added complexity | Benchmark and risk comparisons are dishonest without them; deposits would otherwise register as performance | — Pending |
| Per-position thesis and exit rules as a first-class feature | The only mechanism that makes "when to sell" answerable; must be recorded before emotion, not after | — Pending |
| Manual entry first, import-ready data model | Zero integration risk, works for every market, ships immediately; Bistify proves Midas statements are parseable later | — Pending |
| Telegram over WhatsApp for alerts | Free, no approval process, hours of work versus weeks of Meta business verification and per-message cost | — Pending |
| Simple mode and Pro mode over a single interface | Father and owner need different presentations of identical data; one interface would compromise both | — Pending |
| Agent may write without confirmation, but all writes are audited and reversible | Owner explicitly chose speed over a confirmation gate; audit log preserves recoverability at no interaction cost | — Pending |
| Multi-tenant-ready without billing or public onboarding | Small cost now avoids a rewrite if the tool ever leaves the family | — Pending |
| No crypto, no order execution, no tax reporting in v1 | Each is a substantial project; none serves the family's stated need | — Pending |
| Per-page design prompts as a deliverable instead of direct visual design | Owner drives visual design through Claude Design and needs specifications to feed it | — Pending |
| FIFO as the v1 cost-basis method | Industry default when no method is elected, matches Turkish convention, simplest to explain; schema stays open to average-cost and specific-lot | — Pending |
| Split FX sourcing: market rate at transaction time, TCMB EVDS for historical backfill | Most accurate for new entries while using the free, official, authoritative source for the historical bulk. Rate direction convention typed explicitly in the schema, never a bare number | — Pending |
| Alert sensitivity configurable per family member | Consistent with per-person advice prescriptiveness; lets the owner run hot while the father receives only critical alerts, which is the specific defence against alert fatigue killing adoption | — Pending |
| KAP financial statements in v1; material-event stream deferred | Spike confirmed statements are free, official and machine-readable, and they unlock fundamentals, sector tagging and BIST screening. The continuous event stream carries the real scraping-frequency and legal-grey-area concern for materially less v1 value | — Pending |
| Own TypeScript KAP parser rather than depending on pykap | pykap's index calls work but its line-item parser is broken against KAP's Next.js rebuild; the working parts are thin HTTP wrappers, so porting avoids adding a Python service | — Pending |
| No agent cost controls beyond usage visibility | Owner's explicit decision, reaffirmed after being shown a $10–50 worst-case per pathological deep-research session. No loop breakers, no anomaly alerts, no pre-run estimates — a per-person usage dashboard only | ⚠️ Revisit |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-08-16 after initialization*
