# Architecture Research

**Domain:** Multi-currency personal/family investment portfolio manager with embedded AI agent
**Researched:** 2026-08-16
**Confidence:** HIGH (data model, returns math, FX attribution, event sourcing, RLS isolation — all corroborated by multiple independent sources and standard accounting/GIPS practice) / MEDIUM (BIST-specific data sourcing, exact provider choice — flagged for phase-specific research)

## Standard Architecture

### System Overview

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (Browser)                                │
│   Next.js UI — Simple/Pro mode, TR/EN — Portfolio views, Agent chat (SSE)   │
└───────────────────────────────┬──────────────────────────────────────────--┘
                                 │ HTTPS (REST/RSC + SSE stream)
┌────────────────────────────────▼─────────────────────────────────────────────┐
│                         APPLICATION LAYER (Next.js / Vercel)                 │
│ ┌──────────────┐ ┌───────────────┐ ┌──────────────┐ ┌──────────────────┐    │
│ │ Ledger API   │ │ Returns Engine │ │ Rules/Alerts │ │ Agent Runtime     │    │
│ │ (tx CRUD,    │ │ (TWR, XIRR,   │ │ Evaluator    │ │ (typed tools,     │    │
│ │ lots, CAs)   │ │ FX attrib.)   │ │ + dedup      │ │ RLS session,      │    │
│ │              │ │               │ │              │ │ sandboxed exec)   │    │
│ └──────┬───────┘ └───────┬───────┘ └──────┬───────┘ └────────┬─────────┘    │
│        │                 │                 │                  │              │
└────────┼─────────────────┼─────────────────┼──────────────────┼──────────────┘
         │                 │                 │                  │
┌────────▼─────────────────▼─────────────────▼──────────────────▼──────────────┐
│                     POSTGRES (RLS-enforced, single source of truth)          │
│  transactions(append-only) → lots → positions_snapshot(derived)              │
│  price_history · fx_rates · corporate_actions · agent_audit_log(append-only) │
└────────▲────────────────────────────────────────────────────────────────────┘
         │ nightly / on-demand ingestion writes
┌────────┴──────────────────────────┐   ┌──────────────────────────────────────┐
│  MARKET DATA SYNC (Vercel Cron)    │   │  NEWS/MACRO PIPELINE (Vercel Cron)   │
│  BIST EOD job · US EOD job         │   │  Ingest → dedup → entity match →     │
│  FX rate job · holiday calendars   │   │  LLM classify/summarize → store      │
└─────────────────────────────────┬──┘   └──────────────┬────────────────────--┘
                                   │                      │
                          External providers      Telegram delivery adapter
                    (BIST/TR source, US quotes,   (channel-agnostic interface)
                     KAP, news APIs)
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|-------------------------|
| Ledger API | Owns transactions, lots, corporate actions — the only writer of the immutable source of truth | Next.js route handlers / server actions, TypeScript, Zod-validated inputs |
| Positions/Valuation | Derives current holdings + USD value from ledger + prices + FX; fully rebuildable | Pure TS functions + a `positions_snapshot` table refreshed nightly and on-demand |
| Returns Engine | TWR, XIRR, FX attribution — correctness-critical, independently testable | TypeScript module with golden-value unit tests; runs in the same Next.js/Vercel deploy |
| Market Data Sync | Scheduled ingestion of EOD prices, FX rates, corporate actions, calendar-aware | Vercel Cron → serverless function, provider-adapter pattern |
| Rules/Alerts Evaluator | Evaluates thesis + portfolio rules against current state, dedupes, dispatches | Vercel Cron job + fingerprint-based dedup table + delivery adapter interface |
| Agent Runtime | Typed tool-calling agent scoped to the authenticated user/portfolio, streamed to browser | Anthropic SDK / AI SDK, RLS-backed DB session, append-only audit log |
| News/Macro Pipeline | Ingest, dedup, entity-resolve, cheap-model classify, LLM summarize | Cron job, pre-filter before any LLM call |
| Postgres | Single source of truth, tenant isolation via RLS, append-only ledger + audit log | Vercel Postgres (Neon) with RLS policies set from verified session |

## Recommended Project Structure

```
src/
├── app/                        # Next.js App Router — pages, route handlers, server actions
│   ├── (dashboard)/            # portfolio views, position detail, exposure, returns
│   ├── agent/                  # chat UI + SSE endpoint
│   └── api/
│       ├── ingest/             # cron-triggered market data / news endpoints
│       └── agent/              # agent run endpoint (SSE), reversal endpoint
├── lib/
│   ├── db/                     # Drizzle/Prisma schema, RLS policy SQL, migrations
│   ├── ledger/                 # transaction, lot, corporate-action logic (writers)
│   ├── valuation/               # positions derivation, cost-basis engine (FIFO etc.)
│   ├── returns/                 # TWR, XIRR, FX-attribution — pure, heavily unit-tested
│   ├── market-data/             # provider adapters, calendar logic, backfill
│   ├── rules/                   # thesis/rule evaluation, alert fingerprinting/dedup
│   ├── delivery/                # NotificationChannel interface + TelegramChannel
│   ├── news/                    # ingestion, dedup, entity resolution, summarization
│   └── agent/
│       ├── tools/                # one file per typed tool, each self-scoping to session
│       ├── audit/                # audit log writer + reversal (compensating events)
│       └── sandbox/              # sandboxed code-exec client (Vercel Sandbox/E2B)
└── i18n/                        # TR/EN strings
```

### Structure Rationale

- **`lib/ledger` vs `lib/valuation` vs `lib/returns` are separate modules** because they have different mutability and testing profiles: ledger is the only writer of truth, valuation is a rebuildable projection, returns is pure math that must be unit-testable in isolation against known golden values (Excel/GIPS reference cases).
- **`lib/agent/tools/` is one file per tool, never a generic SQL executor** — this is the tenant-isolation boundary; each file is small enough to audit for "does this scope by session-verified portfolio_id."
- **A single Next.js/Vercel deploy, no separate Python service** — at 4 users / ~70 positions / years of daily history, the returns math is well within TypeScript's comfort zone (see Returns Engine section); a second service would add deployment/ops surface without a performance justification.

## Data Model

### Core Schema (concrete)

```sql
-- ── Identity & tenancy ─────────────────────────────────────────────
users (
  id uuid pk, email text unique, password_hash text,
  locale text default 'en',            -- 'en' | 'tr'
  ui_mode text default 'simple',       -- 'simple' | 'pro'
  advice_prescriptiveness text default 'moderate',
  created_at timestamptz
)

portfolios (
  id uuid pk, owner_user_id uuid fk→users, name text,
  base_currency text default 'USD', created_at timestamptz
)

portfolio_shares (               -- opt-in sharing model
  portfolio_id uuid fk→portfolios, shared_with_user_id uuid fk→users,
  permission text,                -- 'read' | 'write'
  created_at timestamptz,
  pk(portfolio_id, shared_with_user_id)
)

accounts (                       -- broker account / purpose grouping, multiple per portfolio
  id uuid pk, portfolio_id uuid fk→portfolios,
  name text, broker text, account_type text, currency text
)

-- ── Reference data (global, not tenant-scoped) ────────────────────
instruments (
  id uuid pk,
  exchange_mic text,             -- 'XIST' (BIST), 'XNAS', 'XNYS', null for synthetic
  symbol text,                   -- exchange-local ticker, e.g. 'GARAN', 'AAPL'
  isin text,                     -- nullable but unique when present
  figi text,                     -- nullable, from OpenFIGI mapping
  name text, name_tr text,
  asset_class text,              -- 'equity' | 'etf' | 'fund' | 'gold' | 'currency' | 'cash'
  currency text,                 -- native trading currency
  country text, sector text, industry text,
  metadata jsonb,                -- asset-class-specific fields (gold form, fund TEFAS code…)
  unique(exchange_mic, symbol),  -- collision-safe: GARAN.XIST ≠ any US collision
  unique(isin)                   -- where isin is not null
)

instrument_aliases (             -- for news entity resolution
  instrument_id uuid fk→instruments, alias text, alias_lang text
)

price_history (
  instrument_id uuid fk→instruments, price_date date,
  open numeric, high numeric, low numeric, close numeric, volume bigint,
  currency text, source text, is_stale boolean default false,
  pk(instrument_id, price_date)
)

fx_rates (
  base_currency text, quote_currency text, rate_date date,
  rate numeric, source text,
  pk(base_currency, quote_currency, rate_date)
)

corporate_actions (
  id uuid pk, instrument_id uuid fk→instruments,
  action_type text,              -- 'split' | 'dividend' | 'merger' | 'spinoff' | 'rights_issue'
  ex_date date, record_date date, pay_date date,
  ratio_numerator numeric, ratio_denominator numeric,
  cash_amount numeric, cash_currency text,
  target_instrument_id uuid fk→instruments,  -- for mergers/spinoffs
  raw_data jsonb, applied boolean default false
)

-- ── The ledger (append-only, source of truth) ─────────────────────
transactions (
  id uuid pk, account_id uuid fk→accounts, portfolio_id uuid,  -- denormalized for RLS
  instrument_id uuid fk→instruments,      -- null for pure cash deposit/withdrawal
  type text,                              -- 'buy'|'sell'|'dividend'|'deposit'|'withdrawal'
                                           -- |'fee'|'fx_convert'|'corporate_action'
  trade_date date, settle_date date,
  quantity numeric, price numeric, price_currency text,
  fx_rate_to_base numeric,                -- SNAPSHOTTED at write time — see FX section
  fx_rate_source text,
  gross_amount_local numeric, fee_amount numeric, fee_currency text,
  external_ref text,                      -- for a void/reversal to point back to
  voids_transaction_id uuid,              -- self-referencing: marks this row as a reversal
  notes text, created_by uuid fk→users, created_by_agent boolean default false,
  created_at timestamptz
)
-- transactions are NEVER updated or deleted after creation.
-- corrections are new rows with voids_transaction_id set.

lots (                            -- opened by a buy transaction
  id uuid pk, account_id uuid fk→accounts, instrument_id uuid fk→instruments,
  opened_by_transaction_id uuid fk→transactions,
  open_date date, quantity_opened numeric, quantity_remaining numeric,
  cost_basis_per_unit_local numeric, cost_basis_currency text,
  cost_basis_per_unit_usd numeric        -- frozen at open using fx_rate_to_base
)

lot_closures (                    -- which lot(s) fulfilled a sale — supports FIFO/avg/specific-ID
  id uuid pk, lot_id uuid fk→lots, closing_transaction_id uuid fk→transactions,
  quantity_closed numeric, close_date date,
  proceeds_per_unit_local numeric,
  realized_pnl_local numeric, realized_pnl_usd numeric, fx_gain_loss_usd numeric
)

corporate_action_applications (   -- audit trail of how a CA adjusted each lot
  id uuid pk, corporate_action_id uuid fk→corporate_actions,
  account_id uuid fk→accounts, lot_id uuid fk→lots,
  quantity_delta numeric, cost_basis_delta_local numeric,
  generated_transaction_id uuid fk→transactions
)

-- ── Derived / rebuildable (never hand-edited) ─────────────────────
positions_snapshot (
  portfolio_id uuid, account_id uuid, instrument_id uuid, snapshot_date date,
  quantity numeric, cost_basis_usd numeric,
  market_value_local numeric, market_value_usd numeric,
  unrealized_pnl_usd numeric,
  pk(account_id, instrument_id, snapshot_date)
)

portfolio_daily_returns (
  portfolio_id uuid, date date,
  market_value_usd numeric, external_cashflow_usd numeric,
  twr_daily_factor numeric, cumulative_twr numeric,
  local_return_factor numeric, fx_return_factor numeric,   -- FX attribution legs
  pk(portfolio_id, date)
)

-- ── Decision layer ─────────────────────────────────────────────────
position_theses (
  id uuid pk, portfolio_id uuid, instrument_id uuid fk→instruments,
  thesis_text text, target_price numeric, stop_loss_price numeric,
  max_position_pct numeric, invalidation_criteria text,
  created_by uuid fk→users, created_at timestamptz, updated_at timestamptz
)

portfolio_rules (
  id uuid pk, portfolio_id uuid, rule_type text,     -- 'max_sector_pct'|'max_position_pct'|...
  scope text, threshold numeric, enabled boolean
)

alerts (
  id uuid pk, portfolio_id uuid, rule_id uuid, thesis_id uuid,
  alert_type text, severity text, message text,
  fingerprint text,                       -- hash(portfolio,rule/thesis,type,direction)
  status text,                            -- 'active'|'acknowledged'|'resolved'
  first_triggered_at timestamptz, last_triggered_at timestamptz, resolved_at timestamptz
)

alert_deliveries (
  id uuid pk, alert_id uuid fk→alerts, channel text, delivered_at timestamptz, status text
)

-- ── Agent layer ─────────────────────────────────────────────────────
agent_conversations (
  id uuid pk, user_id uuid fk→users, portfolio_id uuid, title text,
  created_at timestamptz, updated_at timestamptz
)

agent_messages (
  id uuid pk, conversation_id uuid fk→agent_conversations,
  role text, content text, tool_calls jsonb, created_at timestamptz
)

agent_audit_log (                 -- append-only, event-sourcing style
  id uuid pk, user_id uuid, conversation_id uuid,
  tool_name text, arguments jsonb, result_summary jsonb,
  affected_table text, affected_row_ids uuid[],
  reversal_of uuid,                -- points to the audit_log row this reverses
  reversed_by uuid,                -- points to the row that reversed this one
  created_at timestamptz
)

-- ── News/macro ────────────────────────────────────────────────────
news_items (
  id uuid pk, source text, url text, published_at timestamptz,
  title text, raw_content text, dedup_hash text, cluster_id uuid
)
news_entity_links (news_item_id uuid, instrument_id uuid, relevance_score numeric)
news_summaries (cluster_id uuid, summary_text text, model text, tokens_used int, created_at timestamptz)
```

### Transaction-ledger vs position-snapshot — why append-only wins

An immutable, append-only `transactions` table with all positions **derived** is standard practice in every correctness-focused financial system (double-entry accounting systems, Ghostfolio, Sharesight, brokerage back-offices) for one reason: **a mutable positions table has no history**. If you edit a position row in place:

- You cannot answer "what did I own on March 3rd" — the row only ever reflects "now."
- A bug in the valuation formula, once fixed, cannot be reapplied to the past — the wrong numbers are already baked in and unrecoverable.
- Corporate actions (a split, a merger) cannot be safely reprocessed — if the split was already smeared into a mutable quantity field, re-running the corporate-action job double-applies it or has no idempotent basis to check against.
- It is incompatible with the audit/reversal requirement: "every agent write must be recorded and reversible" is only implementable if writes are events, not overwrites.
- Reconciliation against a broker statement becomes guesswork — you're diffing two numbers instead of diffing two event streams.

The correct model: `transactions` is truth and is **never** updated or deleted (corrections are new rows referencing `voids_transaction_id`). `positions_snapshot` and `portfolio_daily_returns` are **projections** — fully deletable and rebuildable at any time by replaying the ledger. This is exactly the event-sourcing pattern research confirms is now standard for both financial ledgers and AI-agent audit trails: "the only way to update an entity or undo a change is to add a compensating event to the event store."

### Instrument modeling — one polymorphic table

Use a **single `instruments` table** with an `asset_class` discriminator and a `metadata jsonb` column for asset-class-specific fields, not separate tables per asset class (equity_instruments, gold_instruments, …). Rationale: transactions, positions, price_history, and cross-asset-class allocation queries ("show me % gold vs % equity") all need one uniform foreign key; splitting the table forces every downstream query to UNION across tables. This is the pattern used by Ghostfolio and other multi-asset trackers.

**Identifier strategy** — avoid ticker-only identity, which collides across markets (BIST `GARAN` is unrelated to any US symbol, but symbol-only lookups are still ambiguous the moment a second exchange enters the picture, e.g. a dual-listed ADR):

- **Primary uniqueness key: `(exchange_mic, symbol)`** — exchange-qualified, collision-safe by construction.
- **ISIN as secondary unique-when-present identifier** — used for corporate-action matching and reconciling data across providers (a split feed keyed by ISIN should resolve unambiguously).
- **FIGI as optional third identifier** — useful when mapping to a market-data provider that indexes by FIGI (OpenFIGI is a free lookup service for this).
- **Synthetic instruments** (cash positions, physical gold, generic currency holdings) get a synthetic namespace, e.g. `CASH:USD`, `CASH:TRY`, `XAU:PHYSICAL` — same table, `exchange_mic` null, `asset_class` = 'cash'/'gold'/'currency'.

### Cost-basis lot tracking — what the schema must support

Requirement is FIFO by default (industry default when no method is elected), with the schema staying open to average-cost or specific-identification later. The schema supports all three via the same two tables:

- `lots`: one row per opening (buy) event, with `quantity_remaining` that only ever decreases.
- `lot_closures`: one row per (sale, lot) pairing, recording exactly how much of which lot was consumed and the resulting realized P&L in both local currency and USD (with the FX gain/loss split out separately — see FX section).

The **lot-selection strategy** (which lot(s) a sale draws down) is a pluggable function (`selectLotsFIFO`, `selectLotsAverage`, `selectLotsSpecific`) that decides the split recorded into `lot_closures` at sale time — the data model does not change between methods, only which lots get chosen and in what order. This matches standard practice: "the ledger records which lots were matched against each sale — helpful for verifying the correct method was applied," and average-cost can be modeled as a single synthetic lot per instrument per account that is continuously re-weighted rather than discrete lots, if that method is ever selected.

### Corporate actions — keeping history correct

Corporate actions (splits, dividends, mergers, spin-offs, rights issues — all common on BIST) are **never** applied by mutating existing transaction quantities. Instead:

1. `corporate_actions` stores the raw event (ex-date, ratio, cash amount, target instrument for mergers/spin-offs).
2. An application job matches the action to every affected `lot` (by instrument + held-as-of ex-date) and writes one `corporate_action_applications` row per lot, which itself generates a **new, real transaction** (`type = 'corporate_action'`) that adjusts quantity/cost-basis going forward.
3. The original buy/sell transactions that created the lot are untouched — the historical record of "what actually happened" stays intact, while forward-looking quantity and cost basis are correct because of the new event, not a silent overwrite.

This makes the split/merger/spin-off idempotent and reprocessable (the `corporate_action_applications` table is the check for "have I already applied this"), and it means a report generated for last year still shows what was true last year even after a split happened this year.

### Multi-currency — where the FX rate lives, and why

This is the single most consequential FX design decision, and the research confirms the standard-practice split precisely:

- **On the transaction: freeze the rate at write time.** Every `transactions` row stores `fx_rate_to_base` — the FX rate in effect on the transaction's trade date, captured once and never recalculated. This is what freezes cost basis and realized P&L in USD forever. It mirrors standard multi-currency accounting practice (transaction-date rate for recording the event) and is why a TRY buy from three years ago always shows the same USD cost basis no matter what today's TRY/USD rate is — reports must be reproducible, not silently rewritten every time the FX rate table is updated.
- **For live/unrealized valuation: look up from the `fx_rates` table at read/snapshot time.** Current market value and unrealized P&L change every day even with zero transactions, purely from price and FX movement — so these must use a **spot/closing rate looked up by date**, not a value baked into a row at some earlier time. `positions_snapshot.market_value_usd` is computed nightly (and can be recomputed on demand) as `market_value_local × fx_rates.rate(currency, snapshot_date)`.

In short: **transaction-time rate for cost basis and realized events (frozen), read-time rate for unrealized valuation (live).** Get this backwards (e.g. recomputing historical cost basis with today's rate) and every historical report becomes non-reproducible and silently wrong every day the FX table updates.

### Multi-tenancy and row-level isolation

- Postgres **Row-Level Security (RLS)** is the enforcement layer, not just application-level filtering. Every tenant-scoped table (`accounts`, `transactions`, `lots`, `positions_snapshot`, `position_theses`, `portfolio_rules`, `alerts`, `agent_conversations`, `agent_audit_log`, …) carries a **denormalized `portfolio_id` column** (even where it could be derived via a join) specifically so RLS policies can be simple, fast, index-friendly predicates rather than join-based ones.
- A policy of the shape `USING (portfolio_id IN (SELECT portfolio_id FROM portfolios WHERE owner_user_id = current_setting('app.current_user_id')::uuid UNION SELECT portfolio_id FROM portfolio_shares WHERE shared_with_user_id = current_setting('app.current_user_id')::uuid))` naturally implements both private-by-default ownership and the opt-in sharing model in one place — no application code has to remember to check sharing separately.
- Reference/global tables (`instruments`, `price_history`, `fx_rates`, `corporate_actions`, `market_calendar`) are **not** tenant-scoped — they are shared read data, world-readable to authenticated users.
- This RLS layer is what makes the **agent tenant-isolation boundary** enforceable even if a tool implementation has a bug — see AI Agent Architecture below.

## Returns Engine

### Time-weighted return (TWR)

Standard method: **sub-period linking via daily valuation.** Break the return period at every day an external cash flow (deposit, withdrawal — not dividends reinvested inside the portfolio, not buys/sells that are internal reallocation) occurs, compute a holding-period return for each sub-period, and geometrically link:

```
r_t = (V_t - CF_t) / V_(t-1) - 1

TWR(period) = ∏_t (1 + r_t) - 1
```

Where `V_t` = portfolio market value (USD) at close of day *t*, `V_(t-1)` = market value at close of the prior day, `CF_t` = net external cash flow during day *t* (deposits positive, withdrawals negative). Because EOD prices are available daily (not just at cash-flow moments), the **daily valuation method** is exact enough to avoid Modified Dietz approximation — it "values the portfolio whenever cash flows happen" by construction, since every day gets its own sub-period. GIPS requires geometric linking of sub-period returns for TWR; this is precisely that.

### Money-weighted return (XIRR)

XIRR is the discount rate `r` that zeroes the NPV of the full cash-flow series (all buys/deposits negative, all sells/withdrawals/dividends positive, plus the current portfolio value as a final positive terminal flow):

```
Σ_i  CF_i / (1 + r)^(days_i / 365)  =  0
```

Solved via **Newton-Raphson** (the method Excel/Google Sheets use internally), starting from an initial guess (0.1 is standard). **Convergence failure cases** to guard against:

- Multiple sign changes in the cash-flow series can produce multiple valid IRR roots or none in a sane range.
- A missing terminal value (no final inflow representing current holdings) makes the equation unsolvable.
- Rates outside roughly (-99%, +1000%) indicate a degenerate case, not a real answer.

**Recommendation: use a battle-tested library** (e.g. an npm `xirr` package that already implements Newton-Raphson with a **bisection or Brent's-method fallback** when Newton fails to converge within N iterations) rather than hand-rolling this — it is exactly the kind of numerically finicky code where a well-tested library beats a bespoke implementation, and correctness here is explicitly called out as a project constraint ("a wrong return figure is worse than a missing one").

### FX attribution — the differentiator formula

Decompose total USD return into a local-asset-return leg and a currency-return leg:

```
1 + R_usd = (1 + R_local) × (1 + R_fx)

R_usd  ≈  R_local + R_fx + (R_local × R_fx)
```

Where `R_local` is the return of the asset in its own trading currency (e.g. TRY price return for a BIST stock), `R_fx` is the percentage change in the FX rate (local currency → USD) over the same period, and the cross-product term is the (usually small) interaction effect between the two — standard in currency attribution.

**Implementation, matching the TWR sub-period approach so the two reconcile:** for each daily sub-period, compute two parallel daily factors and store both on `portfolio_daily_returns`:

- `local_return_factor_t` — the day's return computed by valuing everything in local currency (i.e., holding the FX rate fixed at the start-of-period rate).
- `fx_return_factor_t` — the day's return attributable purely to the FX rate moving, holding local-currency value fixed.

Chain-multiply each leg across the sub-periods independently to get compounded `R_local` and `R_fx` over any requested window (a quarter, a year, since-inception), the same geometric-linking mechanism as TWR itself. `R_usd = (1+R_local)(1+R_fx) - 1` should reconcile against the directly-computed USD TWR (small residual is the expected interaction term). This is what makes concrete the family's stated need: *"a TRY gain that was a USD loss is visible as such"* — a position can show `R_local = +40%`, `R_fx = -35%`, `R_usd ≈ -9%` on the same screen.

### Where to compute, and at what cadence

At this scale — **4 users, ~70 positions each (many overlapping), years of daily history** — the data volume is trivial: roughly 4 portfolios × ~1,500 trading days × ~70 positions ≈ 400,000 position-day rows total for a multi-year history, and only ~6,000 rows for portfolio-level daily returns. This is well within a few milliseconds of Postgres query time even unindexed, let alone with the natural `(portfolio_id, date)` primary key index.

- **Precompute, don't compute-on-read for the daily series.** A nightly Vercel Cron job (running after the market-data sync completes) recomputes `positions_snapshot` and `portfolio_daily_returns` for the day just closed. This keeps every page load fast (read a row range, not recompute from the ledger) and keeps the correctness-critical math in one well-tested place that runs once a day, not on every page view.
- **Compute-on-read is fine for ad hoc aggregation** (e.g. "what was my return last quarter," which the agent might ask) — that's just summing/chaining a bounded number of already-computed daily rows, not re-deriving from the raw transaction ledger each time.
- **Materialized views / TimescaleDB continuous aggregates are unnecessary here** — they solve a "too much data to recompute cheaply" problem this project does not have, add an extension dependency that may not even be available on Vercel's managed Postgres (Neon), and a plain cron-refreshed table is simpler to debug (you can just look at the rows).
- **Implement in TypeScript, in the same Next.js/Vercel deploy — not a separate Python service.** The stack decision is Next.js + Postgres; the math involved (daily chaining, Newton-Raphson XIRR, a two-factor FX decomposition) is straightforward numerical code with no dependency on Python's scientific-computing ecosystem (no need for pandas/numpy-scale operations at this row count). A second service would add a deployment target, a network hop, and an ops burden with no performance justification. What it *does* need — because it is correctness-critical — is a dedicated, heavily unit-tested pure-function module (`lib/returns/`) with golden-value test cases checked against known Excel/GIPS reference results, so the math is validated independent of the database and UI.

## Market Data Ingestion

- **Scheduled EOD jobs, exchange-aware.** BIST (XIST) closes ~18:10 Turkey time; US markets close 21:00/22:00 UTC depending on DST. Two separate Vercel Cron schedules, each firing after its market's close plus a buffer for the provider to publish EOD data, rather than one global "fetch everything" job.
- **Market holiday calendars differ between BIST and US.** Maintain a `market_calendar(exchange_mic, date, is_holiday)` table seeded from a static yearly holiday list per exchange. The sync job checks the calendar before expecting a bar for a given date — a missing BIST bar on a Turkish public holiday is expected, not an error; the same date might be a normal US trading day.
- **Missing bars / provider gaps:** retry same-day with backoff; if still missing on the next run, carry the last known close forward into `price_history` with `is_stale = true` rather than leaving a gap — a stale-but-flagged price keeps downstream valuation and returns jobs from breaking on a null, while the UI can visibly warn "price as of [date], market data delayed" instead of silently presenting a stale number as fresh.
- **Backfill for a newly added ticker:** the first time a transaction references an instrument with no `price_history` rows, trigger an on-demand backfill fetching the provider's available history (bounded by tier depth), then fall into the normal nightly incremental sync going forward.
- **Caching is the `price_history` table itself.** Never re-fetch a date already stored; each sync run fetches only the delta since the last stored date per instrument. Combined with a tiny universe (4 users' overlapping ~70 positions is likely 40-60 unique instruments), this keeps API call volume trivially low — well under any reasonable free/cheap tier, and a world away from burning quota.
- **Rate-limit handling:** with such a small ticker universe, a simple sequential loop with a short delay between calls (or a provider's batch-quote endpoint if available) is sufficient — no job queue infrastructure (BullMQ/etc.) is warranted at this scale; a single Vercel Cron function iterating the list fits comfortably inside the function timeout.
- **FX rates** are fetched in the same nightly job for the 1-2 currency pairs actually held (TRY/USD at minimum, possibly EUR/USD).
- **Provider selection is the single largest technical risk** in this project (per PROJECT.md) specifically for **BIST fundamentals/pricing** — international providers serve US equities/ETFs/gold well within budget, but Turkish-market coverage typically requires a Turkish-specific source (KAP for disclosures, TEFAS for funds, and likely a scraped or unofficial BIST price source given the licensed real-time feed is explicitly out of scope). **This should be flagged as needing its own phase-specific research spike** rather than resolved here — see PITFALLS.md.

## AI Agent Architecture

### Tool design and the tenant-isolation boundary (the critical security question)

**Use narrow, typed tools — never a general-purpose SQL tool**, even a "read-only" one. The reasoning, confirmed directly by current research on AI-agent database isolation: a SQL tool is dangerous specifically *because* SQL is expressive within itself — a forgotten WHERE clause, a crafted subquery, or `information_schema` introspection can leak another tenant's rows with **no error and no signal that anything went wrong**. The exact failure mode called out in current guidance: *"if Tenant A's agent calls a data tool and the connection carries the wrong context, the query returns Tenant B's records with no error — the agent reasons on them, acts on them, and may write back to them."* For a family finance app where the whole value proposition depends on trust, this is not an acceptable risk surface.

**Two-layer defense, both required:**

1. **Application layer — tools are scoped functions, not pass-through SQL.** Each tool (`get_positions`, `get_transactions`, `add_transaction`, `set_thesis`, `search_news`, `run_backtest`, …) is implemented as a small TypeScript function that receives `portfolio_id`/`user_id` **from the authenticated session context set by the server before the agent loop starts** — never as a freeform argument the model supplies that could be manipulated (including via prompt injection embedded in ingested news/filing content the agent reads mid-run). The tool decides its own scope; the model only decides *which* tool to call and with what business-level arguments (e.g. an instrument symbol, a date range).
2. **Database layer — RLS as the backstop.** Every DB connection the agent runtime uses has `app.current_user_id` set from the verified session, and the same RLS policies described in the Data Model section apply. If a tool implementation ever has a scoping bug, RLS still blocks the cross-tenant read/write at the database engine level — defense in depth, not reliance on a single layer.
3. **Never treat prompt instructions as a security boundary.** "Only look at this user's data" in a system prompt is a suggestion the model can be steered away from (directly by the user, or indirectly by malicious content in a fetched news article/filing); it is not enforcement. Enforcement lives in (1) and (2).

Read tools return **typed, paginated, size-bounded JSON**, not full table dumps — this bounds both the leakage surface and the token cost of a single tool call. Write tools call the **same application-level ledger functions a human-triggered UI write would use** (so business rules — immutable ledger, lot selection — apply identically whether a human or the agent initiated the write), and every write additionally goes through the audit-log wrapper below.

### Streaming multi-step runs to the browser

- **SSE, not WebSocket**, for the primary token/tool-progress stream — the traffic is unidirectional (server → client), which is exactly what SSE is built for, it is simpler to operate through Vercel's infrastructure than a persistent bidirectional socket, and it is what the AI SDK's streaming primitives are built around. WebSocket only earns its complexity if the agent needs to interrupt mid-stream for structured user input (e.g. an approval gate) — not a v1 requirement here since writes are unconfirmed-but-auditable by design.
- **Serverless timeout is the real constraint, not the protocol.** A multi-step deep-research agent run (web search → fetch → code execution → synthesis) can outlive a single Vercel function invocation. The current standard pattern (and Vercel's own 2026 direction with Workflows/AI SDK durable agents) is **durable execution**: break the run into checkpointed steps, persist run state after each step to Postgres, and let the run resume from the last completed step across multiple invocations rather than needing one long-lived process. Concretely for this project: an `agent_run_events` table (append-only, one row per emitted event — token chunk, tool-call-start, tool-call-result, final-answer) backs both the live SSE stream and durable resumption.
- **Resumability:** every agent run gets an id; the SSE endpoint, on (re)connect, replays from the last-acknowledged event id in `agent_run_events` rather than restarting the run — a dropped mobile connection resumes mid-run instead of losing progress. This event log doubles as the source for rendering tool-call progress in the UI (each tool-call-start/result event maps directly to a UI progress row) and, secondarily, feeds the audit trail.

### Sandboxed code execution

Agent-generated Python (charts via matplotlib/plotly, backtests, custom calculations) runs in a **microVM-based sandbox** (Vercel Sandbox, given the project is already on Vercel; E2B is the alternative if longer sessions or broader language support are needed later) — **never inline `eval`/`exec` in the main application process.** The sandbox receives a **pre-fetched, read-only snapshot** of only the relevant portfolio data (passed in as CSV/JSON at invocation time), not live database credentials — so even a sandbox escape or a maliciously-crafted generated-code path cannot reach the database at all. This is a third isolation layer on top of the tool-scoping and RLS layers above, specifically because code execution is the highest-risk agent capability.

### Audit logging with one-click reversal (event-sourcing style)

Every agent write goes through a single wrapper that, before or alongside the actual mutation, inserts a row into `agent_audit_log` capturing: the tool called, its arguments, a summary of the resulting change, and the affected table/row ids. **Reversal is implemented as a compensating action, not a physical delete/rollback** — consistent with the append-only ledger: reversing an agent-added `buy` transaction inserts a **new** `transactions` row with `voids_transaction_id` pointing at the original (the ledger stays append-only; the position projection, once recomputed, simply nets to zero for that pair). The reversal action itself writes a new `agent_audit_log` row with `reversal_of` set to the original entry, and the original gets `reversed_by` set — so the reversal is itself audited and, if ever needed, re-reversible. This mirrors the standard event-sourcing guidance that "the only way to undo a change is to add a compensating event to the event store," and gives the literal "one-click reversal" UI a single, well-defined `reverseAgentAction(audit_log_id)` function to call.

### Conversation persistence and context management

`agent_conversations` + `agent_messages` (role, content, `tool_calls jsonb`) give straightforward session resumption across visits. For long-lived conversations that exceed a practical context-window budget, the standard pattern is a **rolling summary message** inserted into the conversation (summarizing older turns) rather than silent truncation — the user should never lose access to what the agent "forgot" without an explicit marker that summarization happened.

## Alerting and Rules Engine

- Rules are **data, not code**: `position_theses` (target/stop/max-size/invalidation per position) and `portfolio_rules` (max sector %, max position %, concentration, with per-position overrides) are plain rows the evaluation job reads — adding a new rule type is a schema/UI change, not new evaluation code per rule instance.
- A scheduled evaluation job (nightly, or triggered right after the market-data sync completes so alerts reflect the freshest prices) walks all enabled rules per portfolio and computes the current metric against its threshold.
- **Deduplication so the same alert doesn't fire daily:** each potential alert gets a deterministic `fingerprint = hash(portfolio_id, rule_id|thesis_id, alert_type, breach_direction)`. Before inserting, the job checks for an existing `alerts` row with the same fingerprint and `status = 'active'` — if found, only `last_triggered_at` is updated and **no new delivery is sent**; if not found, a new alert row is inserted and delivery is triggered. When a later evaluation finds the condition no longer breached, the alert transitions to `status = 'resolved'` — a fresh breach after resolution is a genuinely new alert (correct re-notification), which is the standard fingerprint + state-machine pattern used by production alerting systems (Alertmanager-style: "if an alert with the same fingerprint is already firing, do not send another notification").
- **Delivery abstraction:** an `alert_deliveries` table plus a `NotificationChannel` interface (`send(alert): DeliveryResult`) with `TelegramChannel` as the only concrete v1 implementation. This is what keeps the "channel-agnostic, Telegram is one adapter among possible others" requirement literally true — adding email or another channel later is a new adapter class implementing the same interface, with zero schema change.

## News and Macro Pipeline

- **Pipeline shape:** ingest (RSS/API/KAP) → normalize/dedup → entity-match against held instruments → cheap-model classify (materiality, sentiment) → LLM summarize only what passed the filter → store → serve to both the per-holding news feed and the daily Telegram brief.
- **Deduplication** happens at two levels: a `dedup_hash` on normalized title+source+date catches exact/near-exact republication of the same wire story, and a `cluster_id` (via simple similarity clustering) groups multiple outlets covering the same underlying event so only **one** summary is generated per event even if 20 outlets ran it.
- **Entity resolution / ticker matching is two-staged, and the order matters for cost:** first a cheap regex/keyword pre-filter against an `instrument_aliases` table (company name variants, Turkish name, ticker) builds a small candidate pool of "articles that mention something a user holds"; only for that filtered pool does an LLM call run classification and summarization. This ordering — **filter first, spend LLM tokens last** — is confirmed as the dominant cost-control lever in current guidance ("pre-filtering hard kills 90% of cost... a naive pipeline spends tokens before it knows whether the document matters").
- **Cost control specifics for this project's budget:** use a cheap/fast model tier for the high-volume classify+summarize step (most items, most days), reserving the flagship agent model for the user-facing conversational agent and the lower-volume daily-brief synthesis (which reads a handful of already-summarized items, not raw articles).
- Output (`news_items`, `news_entity_links`, `news_summaries`) is a shared store consumed identically by the in-app news feed, the daily Telegram brief generator, and (as read context) the agent's tools — one pipeline, three consumers.

## Build Order

### Dependency graph

```
Layer 0  Auth + multi-tenant scaffold (users, portfolios, portfolio_shares, RLS)
            │
Layer 1  Core ledger (accounts, instruments, transactions, lots) + manual entry UI
            │                                    │
            │                        Layer 2  Market data sync (price_history,
            │                                  fx_rates, corporate_actions,
            │                                  market_calendar) — parallel with L1 UI,
            │                                  only needs `instruments` to exist
            ▼                                    │
Layer 3  Position valuation & P&L (positions_snapshot, cost basis via lots) ◄────┘
            │
       ┌────┼─────────────────┐
       ▼    ▼                 ▼
Layer 4  Layer 5           Layer 6
Returns  Exposure/          Thesis & rules
engine   allocation          recording UI
(TWR,    views, concen-      (position_theses,
XIRR,    tration flags       portfolio_rules)
FX attr)                     — only needs L1
       │                          │
       │                          ▼
       │                     Layer 7  Alerting/rules evaluator
       │                              + Telegram delivery
       │                              (needs L6 + L2)
       ▼
Layer 8  News/macro pipeline — only needs `instruments` (L1+L2), parallel with L4-L7
       │
       ▼
Layer 9  Agent read tools (needs L3 minimum; L4-L6 make it actually useful)
       │
       ▼
Layer 9b  Agent write tools + audit log/reversal (needs L1 ledger writers + audit schema)
       │
       ▼
Layer 10  Agent deep research, streaming durable runs, sandboxed code execution
       │
       ▼
Layer 11  Discovery / idea generation (needs holdings + exposures from L3/L5 to be useful)
```

### What can be built in parallel

- **Layer 1 (ledger UI) || Layer 2 (market data sync)** — the sync job only needs the `instruments` table to exist, not the transaction-entry UI to be finished.
- **Layer 4 (returns) || Layer 5 (exposure/allocation) || Layer 6 (thesis/rules UI)** — all three depend only on Layer 3 (valuation) or Layer 1 (ledger), not on each other.
- **Layer 7 (alerting) || Layer 8 (news pipeline) || Layer 9 (agent read tools)** — each has a distinct, narrow dependency (L6+L2, L1+L2, L3 respectively) and none blocks the others.
- **Agent write tools (9b) should not start before the audit-log/reversal schema exists** — this is the one place where starting "agent capability" work early would create rework, since every write tool must be built against the audit wrapper from day one, not retrofitted.

### Rationale for the ordering

The core value ("an accurate USD picture first") is fully represented by Layers 0-5: nothing about exposure, returns, or FX attribution is trustworthy until the ledger and valuation layers are correct, which is why they gate everything else. The differentiator (thesis/rules + agent) deliberately sits on top rather than beside — Layer 6 (thesis/rules recording) only needs the ledger to exist and can start early since it's pure data entry, but Layer 7 (alert *evaluation*) needs live prices (L2) to mean anything, and the agent (L9+) is explicitly sequenced last among core layers because an agent with no real underlying data to query provides no value and would just be answering from static/mocked state — validating the tool-isolation design against real RLS-protected data is also only meaningful once real tenant-scoped data exists.

## Vertical Slicing

Given fine granularity and the stated preference for working software early, slice by **user-visible outcome that spans the full stack**, not by architectural layer. Each slice below is deployable and demoable on its own, and — importantly — each slice forces a real architectural decision to be made early rather than deferred to a "market data phase" or "FX phase" at the end:

1. **"I can log in and record one manual buy transaction for a US stock and see it in a table."** Touches auth, portfolios, accounts, instruments, transactions, minimal UI. Proves the ledger end-to-end with zero valuation yet.
2. **"I can see that position's current USD value."** Adds `price_history` for exactly one instrument, one EOD fetch, and a position view. Proves the market-data-sync → valuation pipeline end-to-end for a single case before generalizing.
3. **"I can add a BIST equity transaction in TRY and see it converted to USD."** Proves multi-currency end-to-end on a second instrument type — forces the `fx_rates` table and the transaction-time-vs-read-time rate distinction to be real, not deferred.
4. **"I see total portfolio value and simple P&L across all positions."** Generalizes valuation across the full holdings set (equities, gold, cash, FX).
5. **"I see TWR and XIRR for my portfolio over the last year."** First returns-engine slice, validated against golden-value test cases.
6. **"I see my TRY position's return split into asset return vs currency return."** First FX-attribution slice — directly validates the core stated requirement ("a TRY gain that was a USD loss is visible as such").
7. **"I can write a thesis with a stop-loss for one position and get a Telegram message when it's breached."** First alerting slice, end-to-end from rule entry through evaluation to delivery — validates the delivery abstraction with its one real adapter.
8. **"I can ask the agent 'what's my biggest position' and get a correct, tenant-scoped answer from real data."** First agent slice — read-only tools only, validates the RLS + typed-tool isolation boundary against real data before any write capability is added.
9. **"The agent can add a transaction for me, and I can reverse it with one click."** First agent-write slice — validates the audit-log/reversal pattern end-to-end on the simplest possible write.

Later slices layer in exposure/concentration views, portfolio-wide rules, news feed, discovery, and deep-research/sandboxed-execution agent capability — each following the same pattern of touching schema + API + UI together rather than building a full backend layer before any UI exists for it.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Mutable positions table as source of truth

**What people do:** Store current holdings as a table that gets UPDATEd on every buy/sell instead of deriving it from a transaction log.
**Why it's wrong:** Destroys history — "what did I own on March 3rd" becomes unanswerable; a valuation-logic bug, once fixed, cannot be reapplied to correct past numbers because they're already overwritten; corporate actions cannot be safely reprocessed or reconciled; the audit/reversal requirement becomes unimplementable because there's no event to reverse.
**Do this instead:** `transactions` is the only source of truth, append-only. `positions_snapshot` is a derived, fully rebuildable projection.

### Anti-Pattern 2: Recomputing historical FX conversions with today's rate

**What people do:** Store transactions in local currency only, and convert to USD on every read using the *current* FX rate.
**Why it's wrong:** Silently rewrites historical cost basis and realized P&L every single day the FX rate changes — a report generated today and the same report generated tomorrow disagree about a transaction from two years ago, which is both non-reproducible and actively misleading for a family whose entire premise is "don't let FX noise distort the picture."
**Do this instead:** Freeze `fx_rate_to_base` on the transaction row at write time for cost basis/realized events; only use a date-specific `fx_rates` lookup for *unrealized* valuation, which legitimately changes daily.

### Anti-Pattern 3: A general-purpose SQL tool for the agent

**What people do:** Give the agent a "read-only query" tool that accepts arbitrary SQL for flexibility.
**Why it's wrong:** SQL is expressive enough that a forgotten WHERE clause, a crafted subquery, or schema introspection can leak another tenant's rows with no error signal — this is the single largest risk in a shared multi-tenant family app, compounded by prompt-injection risk from ingested news/filing content.
**Do this instead:** Narrow, typed tools that self-scope from the verified session, backed by Postgres RLS as a second enforcement layer.

### Anti-Pattern 4: Ticker-only instrument identity

**What people do:** Use the raw ticker symbol as the instrument's identity/primary key.
**Why it's wrong:** Collides across exchanges (a symbol string alone is ambiguous the moment a second market enters the picture), breaks on ticker changes, and is inconsistent across data providers.
**Do this instead:** `(exchange_mic, symbol)` as the natural key, ISIN as a secondary unique identifier for corporate-action matching and cross-provider reconciliation.

### Anti-Pattern 5: Applying corporate actions by mutating existing transactions

**What people do:** On a stock split, directly multiply historical transaction quantities in place.
**Why it's wrong:** Destroys the as-executed record of what actually happened, breaks reconciliation against broker statements, and makes the split non-idempotent (reprocessing corrupts data further).
**Do this instead:** A corporate action generates new, real ledger transactions (via `corporate_action_applications`) linked back to the originating lots; the original buy/sell transactions are never touched.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| US equities/ETF price provider (e.g. Twelve Data / Financial Modeling Prep / EOD Historical Data-tier) | Nightly cron pull, delta-only fetch by last stored date | Budget-friendly tier sufficient given tiny ticker universe and no real-time requirement |
| BIST price/fundamentals source | Nightly cron pull, likely Turkish-specific source or scrape given poor international coverage | **Flagged as highest-risk integration — needs dedicated phase-specific research** |
| KAP (BIST disclosures) | Scheduled poll/scrape | Feeds both news pipeline and corporate-action detection for BIST names |
| TEFAS | Scheduled pull for Turkish fund NAVs, if funds are held | Separate from equity price feed |
| Telegram Bot API | Delivery adapter implementing `NotificationChannel` | No approval process, free — matches budget constraint |
| Vercel Sandbox / E2B | Agent code-execution client, invoked per agent tool call | Sandbox receives a pre-fetched read-only data snapshot, never live DB credentials |
| Anthropic API (Claude) | Agent runtime, typed tool-calling, streamed via SSE | Per-user token usage logged for the visible-cost requirement |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Ledger ↔ Valuation | Direct function call within the same deploy, valuation reads from ledger tables + price/FX reference tables | Valuation never writes to ledger tables |
| Valuation ↔ Returns Engine | Returns engine reads `positions_snapshot`/ledger cash flows, writes `portfolio_daily_returns` | One-directional; returns engine never mutates positions |
| Market Data Sync ↔ everything downstream | Writes only to `price_history`/`fx_rates`/`corporate_actions`; all consumers read | Sync job has no knowledge of portfolios/users at all — purely reference-data ingestion |
| Rules Evaluator ↔ Delivery | Evaluator writes `alerts`, delivery adapters read `alerts`/`alert_deliveries` | Decoupled via the `NotificationChannel` interface, not direct coupling to Telegram |
| Agent Runtime ↔ Ledger/Valuation/Returns | Only through typed tools, each independently RLS-scoped | This is the tenant-isolation boundary — never a direct/raw DB path from agent to tables |
| Agent Runtime ↔ Sandbox | Data passed in at invocation as a snapshot, results returned as artifacts (chart images, computed values) | No live credentials cross this boundary in either direction |

## Sources

- [Time-Weighted Return — Wikipedia](https://en.wikipedia.org/wiki/Time-weighted_return)
- [Time-Weighted Return — AnalystPrep CFA Level III notes](https://analystprep.com/study-notes/cfa-level-iii/time-weighted-return-2/)
- [nodejs-xirr — Newton-Raphson convergence issue discussion](https://github.com/RayDeCampo/nodejs-xirr/issues/2)
- [PhpSpreadsheet — additional XIRR method when Newton-Raphson fails](https://github.com/PHPOffice/PhpSpreadsheet/pull/3262)
- [Currency Movement on Portfolio Risk and Return — AnalystPrep CFA Level III](https://analystprep.com/study-notes/cfa-level-iii/currency-movement-on-portfolio-risk-and-return/)
- [Chapter 4: Foreign Exchange Markets and Rates of Return](https://2012books.lardbucket.org/books/policy-and-theory-of-international-finance/s07-foreign-exchange-markets-and-r.html)
- [Ghostfolio — GitHub (Angular + NestJS + Prisma + Postgres + Redis architecture)](https://github.com/ghostfolio/ghostfolio)
- [Ghostfolio Portfolio Management — DeepWiki](https://deepwiki.com/ghostfolio/ghostfolio/4-portfolio-management)
- [Crypto cost basis methods: FIFO, HIFO and more — CoinTracker](https://www.cointracker.io/blog/crypto-cost-basis-methods-explained)
- [Understanding Corporate Actions Data — Exchange Data International](https://www.exchange-data.com/corporate-actions-guide-everything-you-need-to-know/)
- [Event Sourcing Pattern — Microsoft Learn / Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
- [AI Agent Explainability: Why Your Infrastructure Needs to Remember — AxonIQ](https://www.axoniq.io/blog/ai-agent-explainability-event-sourcing-infrastructure)
- [Agent Rollback and Checkpoint Patterns — Digital Applied](https://www.digitalapplied.com/blog/agent-rollback-checkpoint-patterns-2026-engineering-reference)
- [Multi-Tenant AI Agent Data Isolation — CockroachDB](https://www.cockroachlabs.com/blog/multi-tenant-ai-agents-why-data-isolation-starts-at-the-database/)
- [Row-Level Security (RLS) in PostgreSQL for Multi-Tenant SaaS Apps — Medium](https://medium.com/@anand_thakkar/row-level-security-rls-in-postgresql-for-multi-tenant-saas-apps-ef8c324031d0)
- [Resumable token streaming: why AI streams break on reconnect — Ably](https://ably.com/blog/token-streaming-for-ai-ux)
- [AI Token Streaming: From SSE to Durable Sessions — WebSocket.org](https://websocket.org/guides/use-cases/ai-streaming/)
- [A new programming model for durable execution — Vercel](https://vercel.com/blog/a-new-programming-model-for-durable-execution)
- [The Agent Stack — Vercel](https://vercel.com/blog/agent-stack)
- [AI SDK 6 / AI SDK 7 — Vercel](https://vercel.com/blog/ai-sdk-6)
- [E2B vs Vercel Sandbox — Northflank](https://northflank.com/blog/e2b-vs-vercel-sandbox)
- [Build AI data analyst with sandboxed code execution — E2B Blog](https://e2b.dev/blog/build-ai-data-analyst-with-sandboxed-code-execution-using-typescript-and-gpt-4o)
- [Alert Deduplication and Correlation — Rootly](https://rootly.com/alert-management/alert-deduplication-and-correlation)
- [Alerting System Low-Level Design: Evaluation, Deduplication, Routing — techinterview.org](https://www.techinterview.org/post/3233469424/lld-alerting-system/)
- [Candidate Generation Drives AI Pipeline Cost — DZone](https://dzone.com/articles/candidate-generation-cost)
- [Identifiers (ISIN, CUSIP, FIGI) — TradingView](https://www.tradingview.com/support/solutions/43000734977-identifiers-isin-cusip-figi/)
- [Modern Security Master Architecture for Financial Data — Intrinio](https://intrinio.com/blog/modern-security-master-architecture-unifying-ticker-cusip-isin-and-figi-data-at-scale)
- [Financial Instrument Global Identifier — OMG / OpenFIGI](https://www.omg.org/figi/)
- [Continuous aggregates vs materialized views — TigerData Community Forum](https://forum.tigerdata.com/forum/t/continuous-aggregates-vs-materialized-views/302)

---
*Architecture research for: multi-currency portfolio management platform with embedded AI agent*
*Researched: 2026-08-16*
