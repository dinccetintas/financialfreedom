# Feature Research

**Domain:** Private family investment portfolio manager (BIST + US equities + gold/FX/cash) with thesis/exit-rule discipline and an embedded AI agent
**Researched:** 2026-08-16
**Confidence:** MEDIUM-HIGH (product features verified via multiple independent web sources including vendor docs, review sites, and user complaint forums; no direct product trials/screenshots taken, so exact UI mechanics are inferred from written descriptions)

## What Was Actually Studied

| Product | Category | What it concretely does |
|---|---|---|
| **Fintables** (fintables.com) | BIST-native fundamentals + portfolio | Portfolio menu shows tracked stocks in a persistent sidebar across pages. Detailed screener lets users query BIST with compound filters (e.g. "BIST100, metal ana sanayi, F/K<15, cari oran>1"). Watchlists support price alerts and per-stock news. Live USD/TRY chart with candlestick/line/Heiken-Ashi and technical indicator overlays. **Portfolio (cost-basis) tracking is a PRO-only, paid feature** (~420 TL/mo or ~4,200 TL/yr as of 2026) — the free tier lost this after originally being free, a recurring user complaint on Ekşi Sözlük/Şikayetvar. KAP (public disclosure) notification tracking is a stated feature. No evidence found of TWR/XIRR, thesis/stop-loss fields, or an AI agent — Fintables is fundamentals + news + a paid cost-basis view, not a decision-support tool. |
| **Sharesight** | Multi-currency cost-basis / returns reference | Automatic FX conversion with per-holding view in both trade currency and base currency; explicitly separates capital gain from currency gain (FX attribution) in reporting — the closest existing analogue to this project's FX-attribution requirement. Full dividend engine: DRIP/DRP handling, announced vs paid dividends, taxable income summaries, forward income report. Reports suite: performance, diversity, contribution analysis, multi-currency valuation, future income. Reports money-weighted return; TWR support is present but less emphasized in marketing than in Portseido/getquin. No thesis/stop-loss layer, no AI agent, no proactive alerting beyond price/news. |
| **Snowball Analytics** | Dashboard + rebalancing + dividends | Aggregates 1,000+ brokers into one dashboard. Rebalancing tool: user sets target % per category/holding, system computes exact buy amounts to return to target (not automatic execution — a "what to buy next" calculator). Goal feature: set a passive-income or net-worth target, see a forecast graph with adjustable assumptions (dividend growth, reinvestment, inflation, contribution cadence) and a "probability of reaching target" chart. Dividend calendar and payment history in table + chart form are the strongest feature per reviews. |
| **Kubera** | Net-worth aggregator, market + non-market assets | Connects 20,000+ institutions via Plaid/Yodlee/SaltEdge; manual entry for real estate (Zillow-linked auto-valuation in the US, manual elsewhere), vehicles, collectibles, domain names, private equity. Crypto/DeFi/NFT tracking across multiple chains. **Notable: ships an MCP-based integration so ChatGPT/Claude/Perplexity/Gemini can query live Kubera net-worth data directly, plus an "LLM-friendly markdown export" of the whole portfolio** — this is the closest existing product to "agent reads your portfolio," though it is read-only query, not a domain-specific analysis/write agent. |
| **getquin** | Mobile-first European tracker + AI | 500K+ users, €20B+ tracked. Broker/bank connections via API/open banking (Trade Republic, DEGIRO, Scalable Capital) with manual/CSV fallback. Benchmarking against MSCI World/S&P 500/NASDAQ-100/DAX/any ETF (Premium). True TWR is a paid-tier feature. **AI agent ("getquin AI") is embedded directly in the app with access to full portfolio data**: answers questions about performance drivers, dividend income, "is it time to harvest a loss," and which news is relevant to specific holdings; a dedicated Tax-Loss-Harvesting Assistant proactively watches the portfolio and flags harvestable losses. Community feed lets users share/critique portfolios (social feature — see anti-features). Reviews report real, unresolved complaints: broker-sync price mismatches persisting despite repeated support tickets, broken exchange connections, slow support gated behind an AI chatbot. |
| **Bistify** (bistify.net) | Turkish, BIST-only, statement import | Explicitly not a broker — pure tracking/analysis tool, so it never asks for IBAN/card/broker password (a trust-building anti-surveillance design choice worth copying). Bulk-import via Midas account statement, a prebuilt Excel template, or a Takasbank BES (pension) report. Tracks BIST stocks, funds, gold, FX and crypto together; shows total wealth, daily change, net performance in one screen. Advanced performance analysis, unlimited portfolios, import/export gated behind Bistify Pro. Validates that Turkish broker statement parsing (Midas specifically) is a solved, shippable problem — directly relevant to this project's "import-ready data model" decision. |
| **Portseido** | EU-friendly, import-only tracker | Deliberately has **no broker API connections at all** — CSV/manual/Google-Sheets import only, explicitly marketed as avoiding the need to hand over broker credentials. Strong on performance math: TWR + money-weighted return, risk metrics, benchmark comparison. Allocation breakdown by business, industry, sector, country, region, asset class, with filterable multi-portfolio views — closest existing match to this project's required exposure breakdown. Supports 85+ exchanges, multiple asset classes including bonds and forex. |
| **Simply Wall St** | Research/thesis presentation, not portfolio-first | The "Snowflake" is a 5-axis radar/polygon (value, future growth, past performance, financial health, dividends) that gives an instant visual shape for a stock — the size/shape/color communicate quality at a glance without reading a table. Behind it: a one-page plain-language report (rewards/risks, Fair Value estimate, analyst-consensus growth forecast, financial-health checks, dividend quality, insider activity). "Narratives" feature lets users/community attach a forward-looking thesis with an assumptions-driven fair-value model to a stock, which is the closest mainstream analogue to a structured, price-target-bearing thesis object — but it is analyst-authored/community-shared, not the user's own personal buy thesis with a stop-loss and invalidation trigger. |
| **Seeking Alpha (Quant Ratings)** | Idea generation via factor scoring | Computer-generated Strong Buy→Strong Sell rating from 100+ metrics graded into 5 factors (Value, Growth, Profitability, Momentum, EPS Revisions), each stock benchmarked against its own sector. PRO Quant Portfolio delivers 2-3 new stock ideas per week and maintains a running list of 30. Explicitly positioned by reviewers as best used as "an idea generator, a portfolio health check, and a confirmation layer — never the final decision" — i.e. even a mature product doesn't claim to replace human judgment, only to narrow the search space. |
| **Koyfin** | Screening / watchlist / news for pro users | Screener with 500+ metrics (fundamentals, technicals, performance, analyst estimates). Watchlist-level alerting: price moves, technical signals, valuation changes, earnings, analyst rating changes, insider trades, or freeform news keywords — set once per list rather than per ticker. News aggregated from 1,000+ sources with filtering. This "alert on the whole watchlist, not each ticker one at a time" pattern is directly reusable for this project's per-portfolio default rules. |
| **TIKR** | Screening / comps, stock-idea sharing | 100,000+ global stocks, ~335 metrics. Differentiator vs Koyfin per reviewers: TIKR leans toward shared "stock ideas" and side-by-side comparison rather than dashboards. |
| **Empower (formerly Personal Capital)** | Free net-worth aggregator with an "advisory nudge" layer | "Investment Checkup" compares current allocation against a recommended "Smart Weighting" alternative and gives specific reallocation tips — the most mainstream existing example of "here's what to change and why," aimed at a robo-advisory upsell. Fee Analyzer quantifies the long-run cost of fund/account fees in dollars, explicitly to trigger action. Both are free hooks for a paid advisory product — worth noting the incentive: Empower's "recommendations" exist to sell managed accounts, which is a trust problem discussed below. |
| **Wealthfolio** | Open-source, local-first, AI-embedded | Rust/Tauri desktop app; all data stored in local SQLite, no server unless the user opts into a paid sync add-on. Built-in AI assistant answers plain-language questions about the user's own portfolio and can run **locally via Ollama** for full privacy — proof that "embed an LLM directly against your own portfolio DB" is an already-shipped pattern, not speculative. Directly relevant precedent for this project's "agent that queries real holdings" requirement, even though this project is cloud/Vercel-hosted rather than local-first. |
| **Wealthfront / Betterment (robo-advisors)** | Automated advisory reference | Threshold-based automatic rebalancing (trade only when an asset class drifts past a set band, not on a fixed calendar) and automated tax-loss harvesting are the two "proactive" actions robo-advisors actually take without asking. Notably: both **execute** the recommendation automatically rather than just surfacing it — this project explicitly must NOT do this (advisory, not execution), but the trigger logic (threshold-based drift, harvest-on-loss) is directly reusable as an alerting rule. |
| **Delta (by eToro)** | Broad multi-asset tracker, crypto-first | Connects up to 10,000 brokerages/exchanges/wallets; strong on breadth (stocks, ETFs, crypto, NFTs, mutual funds, bonds, forex, commodities) and portfolio-level analytics (asset distribution, fee breakdown). No web app (mobile-only) is a repeated complaint. Nothing distinctive for thesis/advisory. |
| **Charles Schwab Portfolio Insights** | Incumbent brokerage + AI narrative | Recently shipped an AI feature that narrates a portfolio's day-over-day change, recaps news for the specific movers in the account, and surfaces "expert content" relevant to holdings — a mainstream broker validating the "daily narrated brief tied to your actual holdings" pattern this project wants. |

### Thesis/exit-rule specific products (deep dive — the project's top-priority differentiator)

Because this is explicitly the highest-leverage feature, three narrowly-focused products were found and studied specifically:

**Helm Terminal** (helmterminal.dev) — the closest working analogue to what this project wants to build:
- Model: user writes the "pillars" behind a holding (the specific, falsifiable reasons for owning it — e.g., "margin expansion continues," "no major customer concentration," "debt stays below 2x EBITDA").
- Monitoring: an agent scores SEC filings, earnings releases, news, and price against those pillars every hour markets are open, and flags "thesis drift" the morning a pillar weakens — with a verbatim, dated citation to the source document, so the user can verify the claim rather than trust it blindly.
- Cross-portfolio correlation: it explicitly flags when several holdings depend on the *same* pillar — "the hidden correlation that breaks a diversified-looking book all at once." This is a direct, already-validated version of this project's "you think you own 17 things but really own 4 bets" requirement, generalized from sector exposure to thesis exposure.
- Pricing/status: free core terminal, thesis-monitoring layer paid at ~$20/mo (as of mid-2026), live and shipping.
- Gap vs this project: US-only (no BIST), no portfolio manager/exit-rule-breach alerting (it flags thesis weakening, not "you hit your stop-loss"), no multi-user/family model.

**Thesis** (usethesis.com) — a "note-first" journal, explicitly NOT a monitor:
- Lets users log why they invested and record events against a position over time, tag/organize by theme, bookmark articles, sync holdings via Plaid (US-only).
- Push/email alerts are on price and news keywords, not on thesis-rule breach.
- Explicitly does not read filings itself — "a journal rather than a monitor." Confirms the market gap between passive journaling and active rule-breach monitoring; this project should sit on the monitor side, not the journal side.
- Has a social layer (share portfolio/thesis with community/friends) — matches this project's family-sharing need conceptually, but publicly, which this project explicitly must not do.

**MyThesis** — from ~$49.99/mo for 5 holdings, positioned similarly to Helm Terminal (automated monitor against sources) but priced for professional/serious retail use, not casual.

**Trading journal apps (Stonk Journal, TradesViz, StopLossTracker)** — a distinct, adjacent category aimed at active/day traders rather than buy-and-hold investors:
- Manual entry of setup, target price, stop-loss, and a "confidence level" per trade (Stonk Journal) — validates capturing target/stop/confidence as first-class fields at entry time, though these tools are trade-frequency oriented (win-rate stats, R-multiple, equity curve) which does not fit a 17-position buy-and-hold family portfolio.
- StopLossTracker is single-purpose: watches a stop-loss price across 70+ exchanges and sends end-of-day notifications — proves that a narrow, reliable "did we breach the rule" watcher is a viable standalone product, i.e. the core mechanism this project needs is not exotic to build.

**What "good" looks like, synthesized from all three:** a good thesis/exit-rule tracker (1) captures structured, falsifiable fields at entry time — not a free-text note — specifically: why bought, price target, stop-loss, max position size, and a stated invalidation condition; (2) re-evaluates those fields against fresh evidence (news, filings, price) on a schedule, not only when the user remembers to look; (3) cites the specific evidence for any flagged drift so the alert is verifiable, not a black box; (4) looks across the whole portfolio for shared pillars/correlated bets, not just per-position; (5) alerts through a push channel the user actually checks (this project: Telegram) rather than requiring the user to open the app to discover a rule was breached.

## Feature Landscape

### Table Stakes (Users Expect These)

Features nearly every competitor product ships. Missing these makes the product feel broken or incomplete compared to a free spreadsheet or the broker's own screen.

| Feature | Why Expected | Complexity | Notes |
|---|---|---|---|
| Manual transaction entry (buy/sell/dividend/deposit/withdrawal/fee/FX) | Every tracker studied supports this baseline; it's the raw data all analytics depend on | LOW–MEDIUM | Ledger-style (append-only transaction log) is the pattern that scales to splits/mergers correctly — see Q7 below |
| Current value, cost basis, unrealized/realized P&L per position | Universal across Fintables, Sharesight, Snowball, getquin, Portseido, Bistify | LOW–MEDIUM | Requires correct FX-on-transaction-date lookups for BIST/TRY legs |
| Multi-currency support with base-currency conversion | Sharesight, getquin, Portseido, Delta all do this; this project's FX-attribution goal goes further than table stakes (see differentiators) | MEDIUM | Sharesight is the reference implementation to study line-by-line |
| Dividend tracking with a forward calendar | Snowball, Sharesight, Portseido, getquin, Bistify all ship this; called out repeatedly as a "standout" feature across reviews | LOW–MEDIUM | Table stakes for a buy-and-hold, dividend-relevant family portfolio |
| Multi-portfolio / multi-account support | Bistify, Portseido, Snowball, getquin all support many portfolios per user | LOW | Directly needed: 4 family members × multiple broker accounts |
| Allocation breakdown by asset class / sector / geography / currency | Portseido, getquin, Simply Wall St all show this as a standard view | MEDIUM | Table stakes as a *chart*; the differentiator is surfacing hidden concentration (see below) |
| Benchmark comparison (index, ETF, custom) | getquin (Premium), Portseido, Sharesight, Koyfin all ship this | LOW–MEDIUM | This project's requirement (S&P 500, BIST100, gold, USD deposit rate) is a superset of what any single competitor offers together |
| Watchlists with price alerts | Fintables, Koyfin, TIKR, Delta all ship this | LOW | Baseline expectation before any "thesis" layer is credible |
| News feed scoped to holdings | getquin AI, Koyfin, Fintables, Charles Schwab Portfolio Insights all do some version | MEDIUM | Filtering to *relevant* news (not just ticker-mention) is where products differentiate — see Q5 |
| CSV/statement import | Bistify (Midas statement, Excel template, Takasbank), Portseido (CSV/Sheets), Delta | MEDIUM | Out of scope for v1 per PROJECT.md but the data model should accommodate it; Bistify proves Midas parsing is solved |
| Mobile-responsive or native mobile app | getquin, Delta, Bistify, Fintables all mobile-first or mobile-native | MEDIUM | This project requires 390px-width parity per PROJECT.md constraints |
| Dashboard "one number" — total portfolio value + daily change | Kubera, Delta, Snowball, getquin all lead with this | LOW | Universal first-screen pattern; must exist for both Simple and Pro modes |

### Differentiators (Competitive Advantage)

Aligned to this project's Core Value ("what should I do?" not "what do I own?"). None of the studied competitors combine all of these; several have one piece each.

| Feature | Value Proposition | Complexity | Notes |
|---|---|---|---|
| Structured per-position thesis (why bought, target, stop-loss, max size, invalidation condition) captured at entry | No mainstream portfolio tracker studied has this as structured fields — Sharesight/getquin/Fintables/Portseido have none of it; Thesis/Helm Terminal/MyThesis are the only precedents and none of them do multi-currency BIST+US portfolio math too | MEDIUM | Depends on: transaction entry (position must exist to attach thesis to) |
| Automated rule-breach monitoring against the thesis (price hits stop/target, thesis invalidation news) with proactive alert | Helm Terminal is the only product doing this well (pillar-drift detection with cited evidence); StopLossTracker proves the narrow stop-loss-only version is viable standalone | HIGH | Depends on: thesis capture, news ingestion, price feed, alert delivery engine |
| Cross-position "hidden correlation" / shared-thesis-pillar detection ("you own 17 things but 4 bets") | Helm Terminal is the only product found doing this (shared pillar detection); most exposure tools (Guardfolio, PortVisio-style) only do sector/ETF look-through concentration, not thesis-level correlation | HIGH | Depends on: thesis capture + exposure analysis; conceptually the hardest feature in the roadmap |
| Periodic proactive review naming specific positions needing attention, with why | Empower's "Investment Checkup" and Charles Schwab's Portfolio Insights are the closest mainstream analogues, but both stop at generic allocation nudges, not per-position sell/cut/hold calls with rationale; Wealthfront/Betterment only rebalance/harvest automatically, they don't narrate | HIGH | Depends on: thesis + exposure + rule-breach detection all feeding one synthesis step |
| FX return attribution (separate asset return from currency return) | Sharesight is the only mainstream competitor doing this cleanly; none of the BIST-specific products (Fintables, Bistify) appear to do it at all — this is a genuine gap in the Turkish market specifically | MEDIUM–HIGH | Foundation-level per PROJECT.md; needs historical FX rates per transaction date |
| TWR + XIRR side by side | Sharesight (MWR-emphasized), Portseido (does both explicitly), getquin (TWR paid tier) — no product studied cleanly presents both with a plain-language explanation of when each matters | MEDIUM | Standard formulas exist; correctness with irregular cashflows (family deposits/withdrawals) is the real work |
| Embedded AI agent with read+write access to the user's own portfolio, web research, and code execution | Kubera (MCP export, read-only), getquin AI (read-only Q&A + tax-loss harvest flagging), Wealthfolio (local LLM Q&A) are all read-only query assistants; none can write transactions, run scenario code, or do multi-step deep research with citations | HIGH | This is the single largest gap versus every competitor studied — none combine agentic write access + audit log + deep research |
| Sector/geography/currency exposure with "look-through" concentration detection | Portseido/getquin show static allocation charts; specialist tools (Guardfolio, PortVisio) do ETF look-through and ownership-overlap detection but are institutional-grade products, not consumer trackers | MEDIUM–HIGH | See Q3 below for visualization patterns |
| Simple mode / Pro mode toggle on identical underlying data | No portfolio tracker studied ships a true novice/expert toggle; Empower and Schwab lean fully narrative (novice-only), Koyfin/TIKR lean fully analytical (pro-only) — this project's dual-audience requirement has no single direct precedent | MEDIUM | See Q4 below; progressive disclosure is the established UX pattern to borrow, but no competitor applies it at the "two named modes" level this project needs |
| Configurable advice prescriptiveness per user (specific sized actions vs gentle flags) | Not found in any competitor — closest is Empower's generic "tips," far from personalized per-user tone/specificity | MEDIUM | Novel; depends on the advisory/review synthesis feature existing first |
| Daily AI-written brief scoped to holdings, delivered via Telegram | Schwab Portfolio Insights (narrated recap) and getquin AI (Q&A on demand) are partial precedents; none deliver a push brief outside their own app | MEDIUM–HIGH | Depends on: news ingestion/dedup + agent summarization + delivery engine |
| AI-curated stock ideas tied to what the user already owns/lacks | Seeking Alpha PRO Quant Portfolio (2-3 ideas/week, factor-scored) and TIKR (shared ideas) generate ideas in isolation; none condition idea generation on the user's actual current exposures/gaps | HIGH | Depends on: exposure analysis existing first (need to know what's "lacking" before suggesting it) |
| BIST + US + gold/FX/cash unified in one portfolio with correct native-currency accounting | Bistify is BIST+gold+FX+crypto but not US equities; Delta/Kubera are broad but weak on BIST specifically; no product studied does BIST+US+gold+FX with true FX attribution together | HIGH | This combination alone is close to a genuine market gap, independent of the AI/thesis layer |

### Anti-Features (Commonly Built, Deliberately Avoid)

| Feature | Why Competitors Build It | Why Problematic Here | Alternative |
|---|---|---|---|
| Social/community feed (share portfolio, follow other investors, public performance comparison) | getquin and Thesis both build this for growth/engagement/network effects | PROJECT.md explicitly rules out social features; private family data must never default toward any public or semi-public surface, and it adds real moderation/security surface for zero value to 4 known users | Family-only sharing is already an explicit requirement (share a specific portfolio with a specific family member) — keep sharing scoped to that, nothing broader |
| Automatic broker API sync as the primary data path | getquin, Kubera, Delta, Sharesight, Snowball all lead with "connect your broker" | PROJECT.md explicitly defers this — most Turkish brokers have no public API, and getquin's own reviews show sync reliability is a persistent, publicly complained-about failure mode ("portfolio data are absolutely incorrect," unresolved for months) even at a well-funded, mature European product | Manual entry first, with an import-ready data model (Bistify's statement-import pattern is the credible bridge for later) |
| Automated trade execution / auto-rebalancing | Wealthfront, Betterment, and Snowball's rebalancing tool all execute or come close to executing | PROJECT.md explicitly prohibits this — "the system proposes, humans decide"; also crosses into regulated investment-advice/execution territory this project must stay clear of for family-only use | Show the rebalancing math (what to buy/sell to hit target) as Snowball does, but require the human to place the trade at their broker |
| Crypto/DeFi/NFT tracking | Kubera, Delta, getquin, Bistify all support this for market breadth | Explicitly out of scope — nobody in the family holds any; adds 24/7 pricing, wallet-sync and custody-adjacent complexity for zero current value | Data model can stay asset-type-extensible without building the feature |
| Real-time streaming quotes | Koyfin, Delta, Fintables Pro all offer this | Explicitly out of scope — licensed BIST real-time feed costs hundreds of USD/month against a $50-150/month total budget; the family is buy-and-hold, not intraday | Delayed/EOD pricing; the daily brief cadence matches the actual decision cadence better than tick data anyway |
| Tax-loss harvesting automation / tax reporting | Wealthfront, Betterment (auto-harvest); Empower (fee/tax awareness); getquin (harvest-timing assistant) | PROJECT.md explicitly excludes tax reporting/capital-gains calculation — Turkish and US tax treatment differ per instrument/holding period and correctness here is its own project | If useful later, surface it as an informational flag only ("this position has an unrealized loss"), never a tax calculation |
| Gamified goal/streak mechanics | Snowball's "reaching the target" forecast is close to this line; some dividend trackers add streak/badge gamification | Not requested and risks trivializing a decision-discipline tool aimed partly at a less financially fluent user (the father) — gamification reads as patronizing rather than authoritative for a "Wall Street portfolio manager" framing | Goal/target tracking is fine as a plain forecast number, not a badge/streak system |
| AI chatbot as the *only* support channel | getquin's own reviews cite hours lost trying to get past an AI support bot to reach a human | Not a feature this project ships externally (no customer support org), but the lesson generalizes: don't let the embedded AI agent be an opaque black box for *thesis-breach reasoning* either — always show the cited source, per Helm Terminal's pattern | Every AI-generated alert/recommendation should show its evidence (filing, news article, price data point), never just an assertion |
| Paywalling the core "why you're here" feature after acquiring the user for free | Fintables moved cost-basis portfolio tracking behind a ~$120/yr paywall after offering it free, a top user complaint on Turkish review sites | N/A — no monetization in this project (family-only, no billing per PROJECT.md), but worth noting as a trust lesson: don't build a bait-and-switch pattern if this project ever gates any feature per PROJECT.md's "additive later" multi-tenant path | Keep this in mind only if a future non-family tier is ever built |

## Answers to the Specific Design Questions

### 1. Position thesis and exit rules — deep dive

No competitor combines all the pieces this project needs, but the pieces exist separately and validate the approach:

- **Structured fields at entry** (not free text) is the right model — confirmed by Stonk Journal (setup/target/stop/confidence) and implied by this project's own requirement list (why bought, target price, stop-loss, max position size, invalidation condition). Free-text-only tools (Thesis/usethesis.com) are explicitly criticized by their own market positioning as "a journal, not a monitor" — i.e., free text alone does not enable automated monitoring, which is the actual differentiator this project wants.
- **Monitoring cadence and evidence citation** is where Helm Terminal is the strongest reference: it re-scores every pillar against filings/earnings/news/price on an hourly cadence and cites the exact source and date for any flagged drift, "so you can check Helm's work, not just trust it." This project should adopt the same evidence-citation discipline for every rule-breach alert (price data point, news headline, KAP filing) — an alert with no visible source is not trustworthy for a family that already distrusts complexity.
- **Presentation**: the two closest models are (a) a per-position "thesis card" showing the original thesis text, target, stop, and current status (on-track / at-risk / breached) with the specific evidence, and (b) Simply Wall St's Snowflake-style at-a-glance visual, though Simply Wall St's version is a generic quality score, not a personal thesis-breach status — this project needs the latter, personal version, not the former, generic version.
- **Portfolio-wide default rules with per-position override** (as specified in PROJECT.md) has no direct competitor precedent found — Koyfin's "alert the whole watchlist at once" is the closest reusable pattern (set a rule at the list level, apply to every member, override per-item as needed).
- **Cross-position thesis correlation** ("4 bets, not 17 stocks") is uniquely Helm Terminal's territory among everything studied — it is the single most important feature to study further before designing this project's own version, since it is a solved but narrow problem (US equities only, no portfolio-manager framing).

### 2. Proactive advisory

- Actual working "here's what to do" precedents are thin. Robo-advisors (Wealthfront, Betterment) act rather than advise — they rebalance and harvest losses *automatically*, silently, without presenting a recommendation for a human to approve. That's the opposite of what this project wants (propose, don't execute) but the *triggers* they use are exactly right: threshold-based drift for rebalancing, unrealized-loss-with-no-wash-sale-conflict for harvesting.
- Empower's Investment Checkup and Schwab's Portfolio Insights are the two mainstream products that actually narrate specific, personalized findings in plain language rather than raw data — but both stop at portfolio-level allocation nudges ("you're overweight X sector, here's the Smart Weighting alternative") rather than per-position sell/cut/hold calls with a rationale. Neither names a specific holding and says "cut this, here's why."
- **What makes a recommendation trustworthy vs ignorable**, synthesized across sources: (a) it must cite verifiable evidence, not just assert (Helm Terminal's pattern); (b) it must be specific and named, not generic allocation-speak (the gap in every product studied except thesis-tracker niche players); (c) it should acknowledge the incentive behind it — Empower's checkup exists partly to sell managed accounts, which erodes trust once noticed; this project has no such conflict (family-only, no upsell) and should make that explicit ("this is your own rule you set, not a sales pitch") to build credibility with the father specifically; (d) frequency matters — Schwab's daily narrated recap and this project's own "periodic review... three-month cadence" framing both suggest a scheduled, bounded cadence rather than constant noise is what keeps a recommendation feeling considered rather than reflexive.

### 3. Exposure analysis and concentration

- The core insight validated across multiple sources: sector concentration research explicitly states that "a portfolio holding 20 technology stocks is not diversified, it is a concentrated technology bet with the appearance of diversification," and that investors who map ETF look-through exposure typically discover 2-3x more concentration than they believed — this directly validates the PROJECT.md framing ("you think you own 17 things but really own 4 bets").
- The mechanism that makes this "immediately obvious" per the tools studied: (a) look-through — decompose every ETF/fund into its underlying holdings before aggregating by sector/geography/currency, not just tagging the fund itself; (b) treemap visualization specifically called out as effective because "oversized positions become impossible to ignore" — a treemap's area-proportional-to-size property does the concentration-flagging visually without requiring the user to read numbers.
- No competitor studied (Portseido, getquin, Simply Wall St) does full look-through decomposition in their consumer allocation charts — they tag by the security itself, not underlying exposure. Specialist institutional tools (Guardfolio-style) do this but aren't consumer products. This is a genuine feature gap this project can fill, and it's lower-complexity for a 17-position, mostly-single-stock (not ETF-heavy) family portfolio than for a typical retail ETF investor.
- Recommended visualization mix given the dataviz literature and what's proven in the category: treemap for "size of the bet" (position/sector), a simple bar or stacked-bar for currency/geography exposure over time (trend matters more than a snapshot for FX), and a correlation/pillar-overlap view (table or network, not yet solved elegantly by any product studied — Helm Terminal states the finding in text, not a diagram) for thesis-level concentration. Sunburst/Sankey were mentioned in the question but no product studied actually ships them for this use case; treemap and bar remain the field-tested choices — worth validating with the actual design-prompt/Claude Design phase rather than assuming a fancier chart type adds clarity.

### 4. Simple vs Pro presentation

- Progressive disclosure is the established, named UX pattern for exactly this tension: "a beginner doesn't need power-user behavior... yet hiding it indefinitely would infuriate advanced users... when a user gets more comfortable, shortcuts/customization/detailed analytics can be uncovered." The recommended layering pattern from UX literature: lead with the one number the user came to see, give it visual dominance, and layer everything else behind progressive disclosure — "a transaction opens into its full record, a chart opens into a filterable view."
- No portfolio product studied ships a true two-named-mode toggle (Simple/Pro) on identical data — the market splits into either fully-narrative consumer tools (Empower, Schwab) or fully-analytical pro tools (Koyfin, TIKR), not one product doing both from a mode switch. This confirms the project's approach is a genuine differentiator, not something to copy from a specific competitor, though the progressive-disclosure *mechanics* (same data, different depth, escalate on demand) are directly reusable.
- Practical implication for this project: rather than "simple mode = fewer features hidden," design each screen so the Pro view is the Simple view *plus* progressive disclosure (expandable detail, drill-down), not two structurally different information architectures maintained separately — this keeps the two modes provably consistent (same underlying numbers) and halves the design/maintenance surface, which matters given the small team building this.

### 5. News and alerting

- Aggregation scale: Koyfin pulls from 1,000+ sources, TIKR/Seeking Alpha similar; the real work is *filtering to relevant* (not just ticker-mentioned) and *deduping* across sources reporting the same event — none of the sources found describe their dedup mechanism in public detail, so this remains a build-it-yourself problem, not a solved pattern to copy verbatim.
- "Material event" triggers, synthesized from Koyfin's alert taxonomy and the AI-briefing research found: earnings releases, guidance changes, analyst rating changes, insider trades, large price moves, and — specific to an AI-pipeline architecture described for institutional daily briefings — SEC-filing-type events (8-Ks) mapped to "material event type + affected entity." For this project's BIST side, the direct analogue is KAP özel durum açıklaması (special-situation disclosures) — KAP is the authoritative, real-time source Turkish institutional investors rely on for exactly this signal, and Fintables already surfaces KAP notification tracking as a feature, confirming it's accessible and expected by BIST-savvy users.
- What a good daily brief contains, per the closest real precedents (Schwab Portfolio Insights: narrative day-change summary + recap of news for specific movers + relevant expert content; the described institutional AI-briefing pattern: multi-agent pipeline, each source cited back to its origin — FRED series, SEC filing, internal thesis doc — "so the output can be verified... without additional research"): (1) what moved and by how much, scoped only to holdings; (2) *why*, sourced and cited, not a generic market recap; (3) what's coming (earnings dates, ex-dividend dates, economic calendar items relevant to holdings) — directly matching PROJECT.md's "what moved, why, and what is coming" requirement almost verbatim, which suggests that requirement is well-grounded in an existing, validated pattern rather than speculative.

### 6. Stock discovery / screening

- Seeking Alpha's Quant Ratings is the most mature "AI-curated idea generator" studied: multi-factor scoring (Value, Growth, Profitability, Momentum, EPS Revisions) benchmarked within-sector, producing a discrete rating plus a running list of ~30 ideas refreshed 2-3/week. Its own reviewers' framing — "best used as an idea generator, portfolio health check, and confirmation layer, never the final solution" — is a good calibration for how prescriptive this project's own idea-generation feature should present itself: a scored, sourced starting point, not a directive.
- Simply Wall St's Narratives (community/analyst-authored fair-value thesis with explicit assumptions) is the closest existing example of "a credible investment thesis output": states the assumptions driving a fair-value number, is attributable to an author, and is comparable against the market price — this is a good structural template for what this project's own AI-generated stock ideas should output (thesis text + explicit assumptions + a target/fair-value number + score), rather than a bare buy/sell signal.
- **Tying discovery to what the user already owns/lacks**: not found in any competitor studied — Seeking Alpha and TIKR generate ideas in isolation from any specific user's holdings. This is confirmed as a genuine, unclaimed differentiator for this project, but it has a hard dependency: exposure analysis (knowing what's lacking) must exist and be reliable before idea generation can be honestly "tied to" it.

### 7. Transaction entry UX

- Ledger-style (append-only transaction log: buy/sell/dividend/split/merger/FX-conversion as discrete dated events) is the pattern that correctly handles corporate actions, and it's what PROJECT.md already specifies. An editable "positions table" (just current shares × current cost) breaks the moment a split, partial sale, or dividend reinvestment happens, because it has no history to recompute from — this is confirmed indirectly by a real complaint found ("app replaces the existing position with new data when adding a transaction, requiring the user to manually re-adjust quantity," and a case where the running total silently desynced and the only fix was a full data wipe/reload) — both are symptoms of a positions-table (not ledger) implementation.
- Products with the most complete transaction-type coverage found: "Portfolio Trader" (Buy, Sell, Dividend, Dividend Reinvestment, Capital Return, Split, and more) — a useful checklist of transaction types to support at minimum: buy, sell, dividend (cash), dividend reinvestment (DRIP), stock split, spin-off/merger, capital return, deposit, withdrawal, fee, FX conversion.
- What makes people abandon manual entry, per the pain points found: (a) having to manually re-enter each stock instead of any import path at all (top complaint, repeated across review sites) — this project mitigates by keeping the door open for later statement import per PROJECT.md, and Bistify already proves Midas-statement parsing is viable; (b) silent data-integrity bugs (desync, overwrite-on-add) that force a full restart — the ledger/audit-log approach this project already committed to (immutable, reversible) directly prevents this class of bug; (c) multi-currency purchases specifically are under-addressed by every product studied except Sharesight — Sharesight's approach (store both trade-currency and base-currency value per transaction, using the FX rate on that transaction's date) is the correct reference implementation to copy for BIST TRY-denominated purchases against a USD base.

### 8. Anti-features and top complaints (cross-product synthesis)

Beyond the anti-features table above, the most consistently repeated complaints across products researched: (1) broker-sync data accuracy — getquin's unresolved, repeatedly-reported "portfolio valuation cannot be relied upon" complaints are the strongest argument for this project's manual-entry-first decision, since even a well-funded, mature product cannot reliably solve broker sync; (2) feature paywalling after free-tier bait (Fintables' portfolio-tracking paywall backlash) — irrelevant to monetization here but a trust lesson if any feature is ever gated later; (3) AI-support-as-only-channel frustration (getquin) — the generalizable lesson is that this project's embedded agent must never be the *only* way to understand why an alert fired; every alert needs a plain, static, inspectable reason shown outside the chat interface; (4) mobile-only or web-only gaps (Delta's no-web-app complaint) — reinforces PROJECT.md's explicit mobile+desktop parity requirement as correctly prioritized, since it's a recurring complaint category when neglected.

## Feature Dependencies

```
Manual transaction entry (ledger)
    └──requires──> Multi-currency + FX-rate-on-date lookup
                       └──requires──> FX return attribution (asset vs currency return)
    └──enables──> Cost basis / P&L per position
                       └──enables──> TWR / XIRR computation
    └──enables──> Per-position thesis capture (thesis attaches to an existing position)

Per-position thesis capture (why/target/stop/max-size/invalidation)
    └──requires──> Manual transaction entry (position must exist)
    └──enables──> Automated rule-breach monitoring
                       └──requires──> Price feed (for stop/target breach)
                       └──requires──> News ingestion + KAP/filing feed (for invalidation-condition breach)
                       └──enables──> Alert delivery (Telegram)

Exposure analysis (sector/geography/currency/asset-class breakdown)
    └──requires──> Position + classification data (sector/geography tags per holding)
    └──enables──> Concentration-risk flagging
    └──enables──> Cross-position thesis correlation ("4 bets not 17")──requires──> Per-position thesis capture

Proactive periodic review ("what needs attention this week")
    └──requires──> Rule-breach monitoring
    └──requires──> Exposure analysis / concentration flagging
    └──requires──> Portfolio-level risk metrics (volatility, drawdown, Sharpe)
    └──enables──> Configurable advice prescriptiveness (tone/specificity layer on top of the review)

News ingestion, filtering, dedup
    └──enables──> Material-event alerting
    └──enables──> Daily AI-written brief
    └──requires (for BIST)──> KAP disclosure feed access

AI agent (query + write + web research + code execution)
    └──requires──> All of the above as queryable data (thesis, exposure, transactions, news)
    └──enables──> Natural-language Q&A, on-demand analysis, agent-authored review/brief drafts
    └──requires (for writes)──> Immutable audit log + one-click reversal

Stock idea generation, tied to what's owned/lacking
    └──requires──> Exposure analysis (to know what's lacking)
    └──requires──> Scoring/screening data (value/quality/technicals per PROJECT.md)

Simple mode / Pro mode
    └──requires──> All underlying computation to exist first (mode is a presentation layer, not new data)
    └──enhances──> Every user-facing feature above

Delivery engine (Telegram, channel-agnostic)
    └──enables──> Rule-breach alerts, material-event alerts, daily brief delivery
    └──conflicts with──> Nothing structurally, but must be built before any "proactive" feature has real user-facing value (in-app-only alerts don't match the "proactive" framing)
```

### Dependency Notes

- **Thesis capture requires transaction entry**: a thesis is meaningless without a position to attach it to — this fixes phase ordering: ledger/positions must ship before the thesis/exit-rule feature.
- **Rule-breach monitoring requires both a price feed and a news/filing feed**: stop-loss/target breaches are price-driven; invalidation-condition breaches are event-driven (a specific fact changing). Building only the price-driven half is materially easier and could be a valid earlier phase, with news-driven invalidation-condition monitoring following once news ingestion exists (this mirrors Helm Terminal's own scope, which is filings+news+price together from day one — worth attempting together if feasible, since a stop-loss-only version undersells the "thesis" framing).
- **Cross-position correlation enhances but does not block basic thesis tracking**: per-position thesis capture and monitoring is valuable standalone (as StopLossTracker and Thesis prove as standalone products); the "hidden 4 bets" cross-position layer is a valuable second pass, not a blocking dependency — sequence it after single-position thesis monitoring is solid.
- **Proactive periodic review depends on nearly everything**: it's the synthesis layer PROJECT.md calls "the highest-leverage feature" downstream of thesis + exposure + risk metrics all existing — this should be sequenced late, as a capstone that reads from already-correct underlying data, not built in parallel with the data layers it depends on.
- **AI agent writes conflict with data integrity unless the audit log ships first**: PROJECT.md accepts unconfirmed agent writes only because of the audit-log/reversibility mitigation — the audit log is a hard prerequisite, not a nice-to-have, for any agent-write capability.
- **Simple/Pro mode is a late-stage layer, not foundational**: per the progressive-disclosure research, both modes should read from the same computed data; building this early (before the underlying metrics are stable) would mean redesigning both modes whenever a computation changes. Sequence the mode toggle after the analytics it presents are settled.

## MVP Definition

### Launch With (v1)

Minimum to validate the "manager, not tracker" thesis for one family:

- [ ] Manual ledger-style transaction entry (buy/sell/dividend/deposit/withdrawal/fee/FX-conversion) across BIST, US equities, gold, FX, cash — foundation for everything else
- [ ] Multi-currency valuation with FX-rate-on-transaction-date and USD base conversion, including FX return attribution — the single biggest correctness risk and this project's stated foundation
- [ ] Cost basis, unrealized/realized P&L, TWR, XIRR per position and portfolio — table stakes, but must be right before anything else is trustworthy
- [ ] Per-position thesis capture: why bought, target, stop-loss, max position size, invalidation condition — the top-priority differentiator; must ship in v1, not deferred
- [ ] Price-based rule-breach monitoring (stop-loss/target hit) with Telegram alert — the simplest, most mechanical slice of "decision discipline," ship before the harder news/filing-driven half
- [ ] Sector/geography/currency/asset-class exposure breakdown with concentration flagging — needed to make "17 stocks, 4 bets" visible even before cross-thesis correlation exists
- [ ] Simple mode / Pro mode toggle on the core dashboard — needed from day one given the father/owner audience split is a stated hard requirement, not an enhancement

### Add After Validation (v1.x)

- [ ] News ingestion scoped to holdings, dedup, and invalidation-condition-driven rule-breach monitoring — trigger: v1's price-only monitoring is validated as reliable and the family trusts the alerts
- [ ] Cross-position thesis correlation ("hidden bets") — trigger: enough theses recorded across the 4 users to make correlation detection meaningful (needs a critical mass of data)
- [ ] Daily AI-written brief — trigger: news ingestion/dedup is solid enough not to produce noisy or wrong briefs, since a wrong daily brief actively erodes trust faster than no brief
- [ ] Proactive periodic review synthesis ("what needs attention this week") — trigger: rule-breach monitoring + exposure analysis + risk metrics are all individually trustworthy, since this feature is a synthesis of all of them
- [ ] Portfolio-level risk metrics (volatility, drawdown, Sharpe, correlation) — trigger: enough transaction history exists to compute meaningful time-series risk stats (thin history makes these numbers misleading)
- [ ] AI agent with write access + audit log — trigger: read-only agent Q&A is validated as accurate first; writes are a strictly harder trust bar

### Future Consideration (v2+)

- [ ] AI-curated stock idea generation tied to owned/lacking exposure — defer until exposure analysis is mature and trusted, since idea quality depends entirely on exposure accuracy
- [ ] Statement/broker import (Midas and others) — defer per PROJECT.md's explicit v1 exclusion; data model should stay import-ready
- [ ] Macro layer (FED policy, inflation, economic calendar interpreted for holdings) — defer until the holdings-specific layers are solid, since macro-without-holdings-context is just another generic news feed
- [ ] Aggregated analyst/notable-investor sentiment on specific holdings — defer; lowest-leverage of the "staying informed" requirements relative to build cost

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---|---|---|---|
| Manual ledger transaction entry | HIGH | MEDIUM | P1 |
| Multi-currency + FX attribution | HIGH | HIGH | P1 |
| Cost basis / P&L / TWR / XIRR | HIGH | MEDIUM | P1 |
| Per-position thesis capture | HIGH | MEDIUM | P1 |
| Price-based rule-breach alerting | HIGH | MEDIUM | P1 |
| Exposure breakdown + concentration flagging | HIGH | MEDIUM | P1 |
| Simple/Pro mode toggle | HIGH | MEDIUM | P1 |
| News ingestion + dedup + invalidation-condition monitoring | HIGH | HIGH | P2 |
| Cross-position thesis correlation | MEDIUM-HIGH | HIGH | P2 |
| Daily AI-written brief | HIGH | MEDIUM-HIGH | P2 |
| Proactive periodic review synthesis | HIGH | HIGH | P2 |
| Portfolio risk metrics (vol/drawdown/Sharpe/correlation) | MEDIUM | MEDIUM | P2 |
| AI agent read-only Q&A | HIGH | MEDIUM-HIGH | P2 |
| AI agent write access + audit log | MEDIUM-HIGH | HIGH | P2 |
| AI-curated stock idea generation | MEDIUM | HIGH | P3 |
| Broker statement import | MEDIUM | MEDIUM | P3 |
| Macro layer | LOW-MEDIUM | MEDIUM | P3 |
| Analyst/investor sentiment aggregation | LOW | MEDIUM | P3 |

**Priority key:** P1 = must have for launch, P2 = should have, add when possible, P3 = nice to have, future consideration

## Competitor Feature Analysis

| Feature | Fintables | Sharesight | getquin | Helm Terminal | This Project's Approach |
|---|---|---|---|---|---|
| BIST support | Yes, native | No | No | No | Yes, native |
| US equities | No (BIST/fund focus) | Yes | Yes | Yes | Yes |
| Multi-currency + FX attribution | Not confirmed | Yes (best-in-class reference) | Partial | N/A | Yes, USD base, Sharesight-style attribution |
| TWR/XIRR | Not confirmed | MWR emphasized | TWR (paid tier) | N/A | Both, presented together |
| Structured per-position thesis (target/stop/max-size/invalidation) | No | No | No | Yes (pillars) | Yes, structured fields at entry |
| Automated rule-breach alerting with cited evidence | No | No | Partial (tax-loss flag only) | Yes | Yes, price + news/filing driven |
| Cross-position thesis correlation | No | No | No | Yes | Yes (v1.x) |
| AI agent: read | No | No | Yes | Yes (implicit) | Yes |
| AI agent: write with audit log | No | No | No | No | Yes (unique) |
| Simple/Pro mode toggle | No | No | No | No | Yes (unique) |
| Family/multi-user private sharing | No | Account-level only | No | No | Yes, explicit per-portfolio sharing |
| Telegram delivery | No | Email reports | No | Not specified | Yes |

## Sources

- Fintables: fintables.com, Ekşi Sözlük (fintables entry, pages 2-7), Şikayetvar fintables subscription complaints
- Sharesight: sharesight.com/us/features, sharesight.com dividend-tracker, sharesight.com blog (multi-asset tracker), multiple third-party reviews (thecfoclub.com, findmymoat.com, mycapitally.com)
- Snowball Analytics: help.snowball-analytics.com/features, snowball-analytics.com/portfolio-tracker, thecollegeinvestor.com review
- Kubera: kubera.com/net-worth-tracker, help.kubera.com (MCP config, AI chatbot integration articles), kubera.com/blog/stock-portfolio-trackers
- getquin: getquin.com/portfolio-tracker, getquin.com/getquin-ai, fighttofire.com review, matchmybroker.com review, medium.com (tax optimization case study), trustpilot.com/review/getquin.com
- Bistify: bistify.net, App Store listing
- Portseido: portfolioglance.com, quantroutine.com, findmymoat.com reviews
- Simply Wall St: support.simplywall.st (Snowflake explainer), simplywall.st, stockunlock.com review
- Seeking Alpha: help.seekingalpha.com (Quant Ratings FAQ), about.seekingalpha.com/quant-sell-ratings
- Koyfin: koyfin.com/features, koyfin.com/features/alerts, koyfin.com/features/watchlists
- TIKR: findmymoat.com/vs/koyfin-vs-tikr
- Empower: nerdwallet.com, choosefi.com, robberger.com reviews
- Delta: thecollegeinvestor.com, matchmybroker.com reviews
- Wealthfolio: github.com/wealthfolio/wealthfolio, wealthfolio.ai, wealthfolio.app/docs/guide/ai-assistant
- Wealthfront/Betterment: nerdwallet.com, forbes.com robo-advisor reviews
- Helm Terminal (thesis tracking): helmterminal.dev, helmterminal.dev/blog/thesis-tracking-apps, helmterminal.dev/best-thesis-trackers, helmterminal.dev/vela-alternative
- Thesis (usethesis.com): usethesis.com, helmterminal.dev/usethesis-alternative
- Trading journals: stockbrokers.com/guides/best-trading-journals, tradesviz.com, App Store StopLossTracker listing
- Exposure/concentration: guardfolio.ai/concentration-risk, genesis-rm.com hidden-concentration-risk blog, portvis.io/blog/concentration-risk
- Progressive disclosure: uxplanet.org, wandr.studio/blog/fintech-dashboard-design, uxpin.com
- KAP / BIST disclosures: borsaistanbul.com public-disclosure-platform, mkk.com.tr, unlumenkul.com
- Charles Schwab Portfolio Insights: pressroom.aboutschwab.com press release, schwab.com/legal/portfolio-insights-disclosure
- AI daily briefing patterns: v7labs.com/blog/ai-daily-market-updates-investment-teams, wealthtechtoday.com AI daily briefing article
- Confidence classification: gsd-tools classify-confidence (websearch provider, unverified=LOW / cross-checked=MEDIUM) applied throughout; product-specific factual claims corroborated across 2+ independent sources where possible before inclusion

---
*Feature research for: Private family investment portfolio manager (BIST + US + gold/FX/cash) with thesis discipline and embedded AI agent*
*Researched: 2026-08-16*
