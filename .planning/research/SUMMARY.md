# Project Research Summary

**Project:** Financial Freedom
**Domain:** Private multi-currency family investment portfolio manager (BIST + US equities + gold/FX/cash) with thesis-driven decision discipline and an embedded, write-capable Claude AI agent
**Researched:** 2026-08-16
**Confidence:** MEDIUM-HIGH overall — HIGH on data model, financial math, and core stack choices; MEDIUM-LOW specifically on BIST fundamentals/data-provider coverage, which every research file independently flags as the largest single technical risk in the project.

## Executive Summary

This is a correctness-first portfolio manager, not a tracker — the differentiator (per-position thesis/exit-rule discipline, exposure analysis that reveals hidden concentration, and a write-capable AI agent) sits entirely on top of a foundation that must be exactly right: an append-only transaction ledger with derived positions, decimal-precision money, FX rates frozen at transaction time but looked up live for unrealized valuation, and native handling of BIST-specific corporate actions (bonus/rights issues). No competitor studied combines BIST+US+gold/FX coverage, structured thesis capture, rule-breach alerting with cited evidence, and an agent with real write access — this combination is a genuine market gap, but only if the foundation math is trustworthy from day one, since a wrong number destroys trust faster than a missing feature.

The recommended stack is Next.js 16 on Vercel with Neon Postgres (no separate time-series DB, no Python service — the data volumes are trivially small at ~400K rows over years), Claude API + Tool Runner (not the Agent SDK) for the agent layer with narrow typed tools backed by Postgres RLS, decimal.js/NUMERIC for all money, and a combination of EODHD + TCMB EVDS + Finnhub + the agent's own web_search for market data, landing at roughly $20-35/month of the $50-150 budget. The single largest open risk is BIST fundamentals coverage — no provider has confirmed, verified depth at this budget, and this must be resolved with a hands-on spike (not more research) before committing to a data-provider phase.

Key risks to actively design against, not just test for: floating-point/imprecise money math, corporate actions applied against the wrong price-adjustment basis, FX rate confusion (today's-rate-for-history, double conversion, ambiguous rate direction), agent tenant-isolation failures in a family-trust-critical product, and the "lethal trifecta" of an agent that reads untrusted web/news content, holds private financial data, and can write to the ledger simultaneously. All five of these are non-retrofittable if gotten wrong in the phase-1 schema or the phase where agent write access ships.

## Key Findings

### Recommended Stack

Next.js 16 (App Router) on Vercel, with Neon Postgres via Drizzle ORM, Better Auth, next-intl for TR/EN, Recharts + lightweight-charts for visualization, grammY for Telegram, and Inngest (triggered by Vercel Cron) for durable background jobs. The AI agent layer uses the Claude API's Tool Runner (not the Claude Agent SDK, which is a coding-agent product with the wrong tool shape) with Anthropic's native `web_search` and `code_execution` server tools — no separate sandbox service needed. Money is decimal.js in TypeScript backed by Postgres `NUMERIC` columns, never JS floats. XIRR uses the `xirr` npm package; TWR and FX attribution are hand-implemented with golden-value tests, since no trustworthy JS library exists for either.

**Core technologies:**
- Next.js 16 + Vercel — owner's stated preference, already connected; use Vercel Workflows/Inngest for durable multi-step agent runs and cron jobs beyond the ~800s function ceiling
- Neon Postgres (plain, no TimescaleDB) — ~400K rows total at full scale, trivial for indexed Postgres; RLS enabled on all tenant-scoped tables regardless of provider
- Claude API + Tool Runner — typed custom tools + native web_search/code_execution in one array, hosted in your own Next.js routes, owns the audit-log requirement
- decimal.js + Postgres NUMERIC — non-negotiable for all monetary/FX/share-quantity arithmetic
- EODHD (EOD prices) + TCMB EVDS (free, authoritative FX/gold) + Finnhub free tier (US news/fundamentals) + agent web_search (news/KAP) — ~$20-35/month, well under budget, with BIST fundamentals unresolved pending a spike

### Expected Features

No competitor combines BIST-native coverage, true FX-attribution, structured thesis/exit-rule capture with cited-evidence rule-breach alerting, cross-position "hidden bet" correlation, and an agent with audited write access. The pieces exist separately (Sharesight for FX attribution, Helm Terminal for thesis-pillar monitoring, getquin/Kubera for read-only agent Q&A) but never together, and never for BIST.

**Must have (table stakes):**
- Manual ledger-style transaction entry across BIST/US/gold/FX/cash
- Cost basis, unrealized/realized P&L, multi-currency valuation with FX-on-transaction-date
- Multi-portfolio/multi-account support, dividend tracking with forward calendar
- Allocation breakdown by asset class/sector/geography/currency, benchmark comparison
- Mobile-responsive parity (390px), one-number dashboard

**Should have (differentiators — this project's core value):**
- Structured per-position thesis (why/target/stop/max-size/invalidation) captured at entry, not free text
- Automated rule-breach monitoring with cited evidence (price and, later, news/filing-driven)
- FX return attribution (asset return vs currency return) — Sharesight is the only mainstream precedent, no BIST product does this
- TWR + XIRR presented side by side with plain-language explanation
- Simple/Pro mode toggle on identical underlying data — no competitor studied does this
- Embedded agent with real write access + audit log — the single largest gap vs. every competitor studied (all others are read-only)

**Defer (v2+):**
- Cross-position thesis correlation ("4 bets not 17"), daily AI brief, proactive periodic review synthesis — all depend on the above being individually trustworthy first
- AI-curated stock idea generation, broker statement import, macro layer, analyst sentiment aggregation

### Architecture Approach

A single Next.js/Vercel deploy (no separate Python service, no TimescaleDB) built around an append-only `transactions` ledger as sole source of truth, with `positions_snapshot` and `portfolio_daily_returns` as fully rebuildable projections recomputed nightly. Postgres RLS enforces tenant isolation as a database-layer backstop beneath application-layer scoping — critical because the agent's tool calls must be structurally incapable of leaking one family member's data to another's session, even if a tool implementation has a bug.

**Major components:**
1. Ledger API — the only writer of transactions/lots/corporate actions; everything else derives from it
2. Returns Engine — TWR, XIRR, FX attribution, computed nightly and unit-tested against golden values, kept structurally separate from the ledger
3. Rules/Alerts Evaluator — reads thesis/rule rows as data, fingerprints alerts to dedupe, dispatches via a channel-agnostic delivery interface (Telegram is the only v1 adapter)
4. Agent Runtime — narrow typed tools (never a general SQL tool) scoped from verified session context, RLS as a second enforcement layer, sandboxed code execution with only a pre-fetched data snapshot (never live DB credentials), append-only audit log with compensating-event reversal

### Critical Pitfalls

1. **Floating-point money math** — use decimal.js + Postgres NUMERIC end-to-end, never a JS `number` between DB and UI; must be a phase-1 schema decision.
2. **Corporate actions applied against the wrong price basis** (the single most dangerous silent bug per PITFALLS.md) — BIST's routine bonus (bedelsiz) and rights (bedelli) issues must be modeled as distinct transaction types from day one; getting them confused corrupts cost basis silently on nearly every BIST holding within a year.
3. **FX rate confusion** — freeze `fx_rate_to_base` at transaction write time for cost basis/realized events; use a date-specific live lookup only for unrealized valuation. Getting this backwards makes every historical report non-reproducible.
4. **Agent tenant-isolation failure** — enforce via Postgres RLS, not just application-layer `WHERE` clauses, since this is a family-trust product where a leak between family members is not just a bug but a relationship-damaging failure.
5. **Prompt injection / lethal trifecta** — the agent has private data access, untrusted web/news content exposure, and write ability simultaneously; content read from the web must never be interpretable as an instruction, and every write (regardless of what prompted it) must pass the same deterministic sanity checks.

## Non-Retrofittable Foundation Decisions — Must Be Correct in the Phase 1 Schema

Every research document independently flagged a cluster of decisions that cannot be safely added after real transaction data exists. These are the highest-value output of this synthesis and must be treated as hard requirements of the first ledger/schema phase, not deferred or iterated on later:

1. **Append-only transaction ledger with fully derived positions.** `transactions` is the only writable source of truth (buy/sell/dividend/deposit/withdrawal/fee/fx_convert/corporate_action); `positions_snapshot` and `portfolio_daily_returns` are rebuildable projections, never independently mutated. This is also what makes the agent's audit/reversal requirement structurally trivial (reversal = a new compensating ledger row) rather than requiring a rewrite later.
2. **FX rate handling with two distinct semantics on the same row.** Every transaction stores `fx_rate_to_base` snapshotted at write time (freezes cost basis and realized P&L forever); unrealized valuation looks up a date-specific rate from `fx_rates` at read/snapshot time. Reversing this split makes every historical report non-reproducible and silently wrong every time the FX table updates. The rate convention (base/quote direction) must be typed, not a bare unlabeled number.
3. **Postgres Row-Level Security for tenant isolation**, keyed to the authenticated session, on every tenant-scoped table (`accounts`, `transactions`, `lots`, `positions_snapshot`, `position_theses`, `portfolio_rules`, `alerts`, `agent_conversations`, `agent_audit_log`). This must exist before any multi-user data does — it is dramatically cheaper to design before data volume grows, and it is the structural backstop that makes agent tool-scoping bugs non-catastrophic instead of catastrophic.
4. **Corporate-action transaction types, distinguished by type from day one.** A `corporate_action` transaction type (with `corporate_actions` and `corporate_action_applications` tables) must exist before the first real BIST transaction history is ingested — specifically distinguishing bonus/capitalization issues (bedelsiz: free shares, cost basis unchanged, no cash) from rights issues (bedelli: real cash outlay, a genuine new cost-basis addition and cash-flow event for XIRR). Retrofitting this after transactions exist means re-deriving every historical position by hand.
5. **Decimal arithmetic for all money, FX rates, and share quantities.** Postgres `NUMERIC`/`DECIMAL` columns (never `FLOAT`/`DOUBLE PRECISION`), decimal.js at every TypeScript boundary, `number` only at final display formatting. A schema migration to fix this after real data exists requires re-deriving every historical calculation.
6. **Instrument identity keyed by `(exchange_mic, symbol)` plus ISIN, never a bare ticker.** Tickers collide across exchanges and change on renames/delistings; retrofitting this after positions are tied to raw ticker strings means a data migration across every holding.

## The BIST Data Risk — Consolidated

This is the single largest technical risk identified across all four research files, and it is explicitly at-risk for several PROJECT.md requirements: BIST fundamentals for exposure/sector tagging, the discovery/idea-generation feature's BIST scoring, and (to a lesser extent) BIST corporate-action feed timeliness.

**Confirmed obtainable:**
- BIST end-of-day prices, plausibly via EODHD (exchange code `IS`/MIC `XIST` documented, but not hands-on verified) or Twelve Data (Istanbul explicitly listed under its Grow plan — the more confidently-confirmed option)
- Historical USD/TRY FX rates and Istanbul Gold Exchange data via TCMB EVDS — free, official, HIGH confidence, this directly and solidly answers the FX-attribution data need
- General news/KAP-adjacent research via the agent's own `web_search` tool, at low marginal cost

**Unconfirmed / not obtainable at this budget:**
- No provider researched has confirmed, verified BIST *fundamentals* depth at a hobbyist price point (EODHD's fundamentals tier is generically described for "70+ exchanges" with no BIST-specific confirmation)
- No official KAP API or RSS feed exists at all — scraping is technically demonstrated by community projects but legally unreviewed (flagged as needing a dedicated ToS/legal check)
- No official TEFAS API — not needed for v1 scope regardless
- Fintables (the strongest BIST-native competitor reference) has no self-serve API pricing — corporate-membership only, not viable at this budget

**Recommended provider combination and cost:** EODHD All World EOD ($19.99/mo) for prices, TCMB EVDS (free) for FX/gold, Finnhub free tier for US news/fundamentals, agent web_search (~$6-15/mo) for news/KAP synthesis — roughly $20-35/month total, leaving $85-115/month of headroom including room to add EODHD's Fundamentals tier ($59.99/mo) if it proves worthwhile.

**Fallback path if the spike fails:** `borsapy` (free, Python, personal-use license plausibly fits this non-commercial 4-person family app) supplemented by agent-driven web search/KAP review for fundamentals, with EOD prices staying on EODHD or swapping to Twelve Data's Grow plan if EODHD's BIST price coverage also proves weak. Given the buy-and-hold, ~3-month-cadence investment style, fundamentals do not need real-time refresh — this fallback does not block v1.

**The specific spike needed:** before any market-data-integration phase is planned in detail, sign up for EODHD (and ideally Twelve Data as a side-by-side comparison) and pull real BIST tickers (e.g. `GARAN.IS`, `THYAO.IS`) to confirm price coverage and fundamentals depth hands-on. This is a single-session validation task, not a multi-week research effort, and should gate the market-data phase's provider commitment rather than being resolved by further research.

**What's at risk in PROJECT.md if BIST fundamentals prove unobtainable:** the exposure-breakdown requirement's sector/industry tagging for BIST holdings, the discovery/idea-generation scoring requirement for BIST names specifically, and KAP-disclosure-driven invalidation-condition monitoring for thesis rule-breaches (falls back to agent web search, which is workable but less structured/reliable than a real feed). Price-based valuation, cost basis, TWR/XIRR, and FX attribution are NOT at risk — those depend only on EOD prices (well-evidenced) and TCMB FX data (confirmed, HIGH confidence).

## Agent Security — Reconciled Position

This project has all three legs of the "lethal trifecta" simultaneously by explicit design: the agent has full read access to real private financial data across four family members, it is exposed to untrusted external content (web search, news, filings), and it has unconfirmed write access to the ledger and to alerts/rules. The architecture and pitfalls research converge on one coherent position, not two competing ones:

**Layer 1 — Typed tools, never a general SQL tool.** Each tool (`get_positions`, `add_transaction`, `set_thesis`, `search_news`, `run_backtest`, ...) is a small function that receives `portfolio_id`/`user_id` from the server-verified session context, never as a model-supplied argument that could be steered by injected content. This is the primary defense and the only place business-rule consistency (lot selection, ledger invariants) is guaranteed identical between human and agent writes.

**Layer 2 — Postgres RLS as the backstop**, using the same policies that protect human sessions, so a tool-scoping bug or a maliciously-crafted agent-generated query still cannot return another tenant's rows at the database engine level. This is defense in depth specifically because a single-layer defense is not acceptable in a family-trust product.

**Layer 3 — Structural separation of "read untrusted content" from "propose a write."** Content fetched from the web/news/filings must be treated as pure data passed into a research/summarization step, never as instructions the agent loop can act on directly. Every resulting write — regardless of what prompted it, hallucination or injection alike — passes the same deterministic sanity checks (ticker resolves to a known instrument, price within a sane band of market, non-negative resulting quantity, plausible date range) before touching the ledger, since a confirmation gate was explicitly declined by the owner.

**Layer 4 — Sandboxed code execution with a data snapshot, not live credentials.** The agent's code-execution tool receives a pre-fetched, read-only JSON/CSV snapshot at invocation time; it never holds database credentials, so even a sandbox escape or malicious generated code cannot reach the database.

**What must ship in the same phase as agent write access, not later:** the deterministic write-validation layer (Layer 1 sanity checks), the audit log with proactive surfacing (not a passive log nobody reads) and one-click compensating-event reversal, and the structural read/act separation (Layer 3). None of these are safe to defer as "we'll add checks later" — the first hallucinated or injected write can corrupt downstream calculations before a follow-up phase starts. RLS (Layer 2) must exist even earlier, at the foundation/auth phase, before any multi-user data exists at all.

## Scope Reductions — What Research Eliminated

At this project's actual scale (4 users, ~70 overlapping positions, ~400K rows of multi-year daily history), several commonly-assumed components are unnecessary. The roadmap should not plan work for any of these:

- **No TimescaleDB or separate time-series database.** Plain Postgres with a `(symbol, date)` or `(portfolio_id, date)` composite index handles ~91K price rows and ~6K portfolio-daily-return rows trivially. TimescaleDB's advantages only matter at high-frequency/high-volume workloads this project doesn't have.
- **No separate Python microservice for financial math.** TWR, XIRR, drawdown, Sharpe, correlation, and FX attribution are all well within TypeScript's comfort zone at this row count; hand-rolled + heavily unit-tested code in the same Next.js deploy avoids a second deployment target, network hop, and ops burden. Revisit only if Pro-mode analytics later need Monte Carlo simulation or factor analysis.
- **No job-queue infrastructure (BullMQ etc.) for market-data sync.** A tiny ticker universe (40-60 unique instruments) fits comfortably in a single sequential Vercel Cron function with short delays between calls.
- **No Claude Agent SDK.** It's a coding/filesystem agent product (Read/Write/Edit/Bash) with the wrong tool shape; the Claude API + Tool Runner is the correct, lighter-weight primitive for a portfolio-query product feature.
- **No Managed Agents (CMA) for v1.** Adds a second orchestration surface; the Tool Runner + Vercel Workflows/Inngest pattern covers the stated requirements with less new surface area. Revisit only if the deep-research sub-feature proves awkward.
- **No materialized views or continuous aggregates for returns.** A nightly cron-refreshed plain table is simpler to debug and sufficient — the "too much data to recompute cheaply" problem these solve does not exist at this scale.
- **No WebSocket for agent streaming.** SSE is sufficient since traffic is unidirectional (server → client); WebSocket would only earn its complexity if the agent needed to interrupt mid-stream for structured approval, which is explicitly not a v1 requirement given unconfirmed writes.
- **No real-time/licensed BIST data infrastructure** — already correctly ruled out in PROJECT.md, reconfirmed independently by all four research files as both budget-incompatible and unnecessary for a buy-and-hold, 3-month-cadence family.

## Build Order — Reconciled

The architecture document's dependency-graph/vertical-slice ordering and the features document's MVP/P1-P2-P3 prioritization agree closely and can be reconciled into one sequence. Where they differ slightly (see note below), the architecture document's ordering is recommended because it forces real architectural decisions — the FX rate split, the RLS boundary, the audit-log pattern — to be validated early against real data rather than deferred.

**Recommended phase-level sequence** (fine-grained vertical slices within each, per the architecture document's slicing list):

1. **Auth + multi-tenant scaffold** — users, portfolios, portfolio_shares, RLS policies. Must exist before any multi-user data does (Pitfall 12).
2. **Core ledger + manual entry** — accounts, instruments, transactions, lots, minimal UI. Proves the ledger end-to-end with zero valuation, single US-stock case first.
3. **Market data sync (parallel with the tail of ledger UI work)** — price_history, fx_rates (TCMB EVDS), corporate_actions, market_calendar; only needs `instruments` to exist. **BIST spike must happen before this phase's provider commitment is finalized.**
4. **Multi-currency + position valuation & P&L** — proves the FX-rate-on-transaction-date vs live-lookup split on a real BIST TRY transaction before generalizing; cost basis via lots (FIFO default).
5. **Returns engine (TWR/XIRR) and FX attribution** — validated against golden-value test cases; directly proves the "TRY gain that was a USD loss" requirement on real data.
6. **Exposure/allocation views** — sector/geography/currency/asset-class breakdown with concentration flagging (parallel-safe with returns engine, both depend only on Layer 3/4 valuation).
7. **Thesis capture + price-based rule-breach alerting** — structured thesis fields, portfolio-wide default rules with per-position override, Telegram delivery adapter. This is the simplest, most mechanical slice of the core differentiator — ship before the harder news/filing-driven half.
8. **Simple/Pro mode toggle** — a presentation layer over already-stable computation; both research documents agree this is late-stage, not foundational, despite being a stated hard requirement (design once the underlying metrics are settled to avoid redesigning both modes on every computation change).
9. **Agent read tools** — narrow typed tools, RLS-validated against real tenant-scoped data; explicitly sequenced after real data exists so tool-isolation testing means something.
10. **Agent write tools + audit log/reversal** — must not start before the audit-log/reversal schema exists; every write tool built against the audit wrapper from day one, never retrofitted.
11. **News ingestion + invalidation-condition monitoring, cross-position thesis correlation, daily AI brief, proactive periodic review synthesis** — each depends on the prior layers being individually trustworthy first (a wrong daily brief erodes trust faster than no brief).
12. **Agent deep research / durable streaming / sandboxed code execution, discovery/idea generation** — genuinely last; idea generation specifically requires exposure analysis to be mature and trusted first, and discovery should not compete with decision-discipline features for early trust-building attention (Pitfall 20).

**Where the two documents' orderings would differ, if at all:** the features document's P1 list places the Simple/Pro toggle as P1 ("needed from day one given the father/owner audience split is a stated hard requirement"), while the architecture document treats it as a late presentation layer. **Recommendation: side with the architecture document** — build the toggle mechanism/pattern early enough to validate on the first dashboard slice (so it doesn't become an afterthought retrofit), but do not block core ledger/valuation phases on getting both modes fully polished; the underlying data must be correct before either mode's presentation is worth finishing.

**Onboarding/import note (from Pitfalls research, not in the architecture build-order):** because three of four users did not ask for this tool and the highest-pain user (father) has the least motivation to do initial data entry, a fast bulk-entry or lightweight import path for the owner to onboard the other three family members should be prioritized early — not full broker-statement import (correctly deferred), but making the owner's one-time bulk setup of others' portfolios fast is a near-term adoption risk, not a v2+ nicety.

## Research Flags

Needs deeper research or a hands-on spike during planning:
- **BIST market-data provider phase** — needs a hands-on spike (sign up, pull real BIST tickers) before finalizing the provider, not more desk research; also needs a KAP scraping ToS/legal review before relying on it for anything beyond best-effort.
- **Agent core / tool-architecture phase** — the read/act structural separation and write-validation sanity-check layer are novel enough (no direct competitor precedent for "agent with real financial write access") to warrant careful design-time research into current prompt-injection mitigation patterns at implementation time.
- **Corporate-actions handling for BIST bonus/rights issues** — the schema pattern is well-specified by architecture research, but the actual detection/ingestion mechanism (which feed reports bedelsiz/bedelli events reliably) needs validation alongside the BIST data-provider spike.

Standard, well-documented patterns (skip deep research-phase):
- **Ledger schema, RLS multi-tenancy, decimal money handling** — standard, well-corroborated financial-system and Postgres patterns.
- **TWR/XIRR/FX-attribution formulas** — standard, CFA/GIPS-documented math; the risk is implementation correctness (test against golden values), not algorithm research.
- **Next.js/Vercel durable execution (Workflows/Inngest), SSE streaming, Telegram delivery** — current, well-documented 2026 patterns with clear official guidance.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH on framework/DB/AI-layer/financial-math choices; LOW-MEDIUM on BIST data pricing/coverage specifics | Provider pricing pages returned inconsistent numbers across sources; BIST coverage inferred, not hands-on tested |
| Features | MEDIUM-HIGH | Verified via multiple independent web sources (vendor docs, reviews, complaint forums); no direct product trials, so exact UI mechanics are inferred |
| Architecture | HIGH on data model, returns math, FX attribution, event sourcing, RLS; MEDIUM on BIST-specific data sourcing | Data-model patterns corroborated against standard accounting/GIPS practice and existing open-source multi-asset trackers (Ghostfolio) |
| Pitfalls | HIGH for money-math/corporate-action/serverless pitfalls; MEDIUM for AI-agent-specific pitfalls (fast-moving field); MEDIUM for Turkish regulatory specifics | Regulatory read is informational, not legal advice — confirm with a Turkish securities lawyer only if the user base ever expands beyond family |

**Overall confidence:** MEDIUM-HIGH. The foundation-level technical decisions (schema, money math, RLS, FX handling) are HIGH confidence and consistent across all four documents. The one genuine open risk — BIST fundamentals/data coverage — is consistently flagged as LOW-MEDIUM across all four documents and is explicitly a spike-not-research problem, not a gap in the research itself.

### Gaps to Address

- **BIST fundamentals/price coverage** — resolve with a hands-on provider spike (EODHD and Twelve Data, real tickers) at the start of the market-data phase, not through further research.
- **KAP scraping legality** — no official API exists; community scraping is technically viable but legally unreviewed. Design the system so KAP ingestion degrades gracefully to agent web search if it must be dropped, and get a lightweight ToS/legal read before relying on it in production.
- **Exact FMP/EODHD Fundamentals tier pricing** — pricing pages were inaccessible to automated fetch during research; verify directly before budgeting against unconfirmed numbers.
- **Vercel Workflows vs. Inngest for durable agent execution** — Workflows is GA but relatively new (~4 months as of this research); prototype both in the relevant phase rather than committing from research alone.

## Open Decisions for the User

These require a product-owner call, not a technical one, and should be resolved before or during the relevant phase's planning rather than left implicit:

- **Cost-basis method default.** FIFO is recommended (matches general BIST/Turkish tax convention and industry default when unspecified), with the schema staying open to average-cost or specific-lot-ID later — confirm FIFO is acceptable as the v1 default.
- **Canonical FX rate source and convention.** Recommend a consistent market-rate provider (not TCMB's once-daily official rate) for transaction-time conversion, with TCMB EVDS as the free/authoritative source specifically for historical backfill — confirm which source is authoritative for which purpose, and lock the rate-direction convention (`TRYUSD` vs `USDTRY`) explicitly in the schema.
- **Materiality thresholds for alerts.** Research recommends starting conservative (under-alert initially, tune up) to avoid alert fatigue killing the core proactive-value differentiator — confirm initial default thresholds and whether they should be pre-set per family member from launch or left to be configured later.
- **Whether to attempt KAP-based invalidation-condition monitoring in the same phase as price-based rule-breach alerting, or defer it** — Helm Terminal (the closest competitor precedent) ships both together from day one, but doing so depends on the BIST data-provider spike resolving KAP access; confirm whether the price-only slice should ship standalone first if KAP proves unreliable.
- **Confirmation of the "no hard cap" agent-cost policy in light of realistic worst-case cost estimates** (a single pathological deep-research session could plausibly cost $10-50) — research recommends loop-breakers, context compaction, and real-time anomaly alerting (not caps) as mitigations; confirm this mitigation approach is acceptable given the stated no-caps decision.

## Sources

See individual research files for full source lists:
- `.planning/research/STACK.md` — EODHD/Twelve Data/TCMB EVDS provider docs, Anthropic API docs, Vercel/Neon official docs, npm package maintenance data
- `.planning/research/FEATURES.md` — Fintables, Sharesight, getquin, Helm Terminal, Bistify, Portseido, and 15+ other competitor products; UX/progressive-disclosure literature
- `.planning/research/ARCHITECTURE.md` — GIPS/CFA return-calculation references, Ghostfolio open-source architecture, Postgres RLS multi-tenant guidance, event-sourcing patterns
- `.planning/research/PITFALLS.md` — financial-math correctness literature, AI-agent security (lethal trifecta) current guidance, Turkish capital-markets regulatory context (informational only)

---
*Research completed: 2026-08-16*
*Ready for roadmap: yes*
