# Phase 1: Foundation — Sign In & First Position - Context

**Gathered:** 2026-08-16
**Status:** Ready for planning

<domain>
## Phase Boundary

A user on the family's fixed allowlist signs in, creates a named portfolio, records their first US stock buy, and sees it in a filterable, sortable transaction list — on a first migration that is already correct for the five things that cannot be safely retrofitted once real data exists: row-level tenant isolation, decimal money arithmetic, FX rates frozen at write time, typed corporate actions, and exchange-qualified instrument identity.

**Requirements in scope:** AUTH-01, AUTH-02, AUTH-03, AUTH-04, AUTH-05, AUTH-06, AUTH-07, AUTH-08, LEDG-01, LEDG-06, LEDG-07, LEDG-09, LEDG-10, INST-01, INST-03

**In scope:** email+password sign-in, password reset by email, session persistence, portfolio create/name, portfolio sharing (read-only grant + revoke), one US stock buy transaction, transaction list with filter/sort, append-only correction semantics, instrument search and add, RLS enforcement.

**Out of scope (owned by later phases):** sell/dividend/deposit/withdrawal/fee/FX-convert transaction types and FIFO lot consumption (Phase 2), bulk entry (Phase 2), scheduled price sync and valuation (Phase 3), i18n and Simple/Pro modes (Phase 4), applying corporate actions and multi-currency conversion (Phase 5).

</domain>

<decisions>
## Implementation Decisions

### Migration Scope

- **D-01:** Migration 001 creates the **non-retrofittable core only** — `users`, `portfolios`, `portfolio_shares`, `accounts`, `instruments`, `instrument_aliases`, `transactions`, `lots`, `lot_closures`, `corporate_actions`, `corporate_action_applications`, `fx_rates`, `price_history`, `market_calendar`. Every one of these carries a constraint that is painful to retrofit against live rows. The decision layer (`position_theses`, `portfolio_rules`, `alerts`, `alert_deliveries`), the agent layer (`agent_conversations`, `agent_messages`, `agent_audit_log`) and the news layer (`news_items`, `news_entity_links`, `news_summaries`) are **not** created in Phase 1 — they are purely additive and their real requirements will not be known until Phases 8–13. — **Reversibility:** costly — adding the deferred tables later is a plain additive migration, but changing the shape of any core table listed here means migrating live tenant rows while RLS policies depend on the columns being altered.
- **D-02:** `market_calendar(exchange_mic, date, is_holiday)` is added to the core migration even though it is absent from the `ARCHITECTURE.md:92-274` schema block. It is specified in prose at `ARCHITECTURE.md:331` and `ARCHITECTURE.md:395`, and Phase 3 needs it to distinguish "BIST holiday, no bar expected" from "sync failed". Treat the schema block as incomplete here, not the prose as speculative.
- **D-03:** Migration 001 seeds **synthetic instruments** — `CASH:USD`, `CASH:TRY`, `CASH:EUR`, `XAU:PHYSICAL` — with `exchange_mic` NULL, per the namespace convention at `ARCHITECTURE.md:297`. Fixing the convention now prevents a conflicting hand-created row in Phase 2.
- **D-04:** Migration 001 seeds `market_calendar` with 2026 BIST and US holiday dates from a static list.
- **D-05:** A **one-shot TCMB EVDS backfill script** populates historical daily USD/TRY into `fx_rates` during Phase 1 setup, so `transactions.fx_rate_to_base` is sourceable for backdated entries from day one. **Scope boundary:** the scheduled/recurring EVDS sync job, gold rates, and the full DATA-02/DATA-03 integration remain Phase 5 work. Phase 1 ships a script, not a job.
- **D-06:** Migrations are authored as **Drizzle schema + hand-written SQL in the same numbered migration file**. `drizzle-kit generate` produces tables, indexes and foreign keys from the TypeScript schema; RLS policies, `CHECK` constraints (non-zero quantity, non-negative price, valid enum values) and seed data are appended as raw SQL. Drizzle's schema DSL cannot express RLS policies, and app-layer scoping alone cannot satisfy success criterion 4. — **Reversibility:** costly — the authoring pattern set here is followed by every subsequent migration in the project.
- **D-07:** LEDG-09 ("never floating point") is enforced by a **CI guard plus tests**, not by convention alone. A test introspects `information_schema.columns` and fails if any column whose name matches money/price/quantity/rate/amount/basis/value patterns has type `real`, `double precision` or `float`. Paired with unit tests asserting `decimal.js` values round-trip through the Postgres driver without precision loss. This is the mechanism that catches the mistake in Phase 7 or in an agent-authored migration in Phase 12, when nobody is thinking about it.

### Authentication & Allowlist

- **D-08:** Family accounts are created by a **setup script that seeds the four users** with a random temporary password; each user is forced to set their own password on first sign-in. **No registration route ships at all** — this is the literal reading of AUTH-04 ("no public registration surface exists"). Adding a fifth person means re-running the script, which is acceptable at family scale. — **Reversibility:** reversible — adding a gated registration route later is additive and touches no stored data.
- **D-09:** Password-reset email (AUTH-02) is delivered via **Resend**. Free tier covers 3,000 emails/month against an expected volume of a handful per year, integrates directly with Next.js on Vercel, and React Email templates matter because reset emails must eventually exist in both Turkish and English. Cost against the $50–150/mo budget: $0.
- **D-10:** Sessions are **long-lived and sliding** — 30-day database-backed Better Auth session in an `httpOnly` / `secure` / `SameSite=Lax` cookie, refreshed on activity, unlimited concurrent devices. This satisfies AUTH-03's "persists across browser refresh and across devices" directly. Rationale: the data is financial but family-private and read-mostly; forcing re-auth on a parent who checks their portfolio monthly costs more in adoption than it buys in security. No step-up auth on writes — it would collide directly with Phase 2's requirement that bulk entry be fast.

### Portfolio Sharing

- **D-11:** Phase 1 ships **`portfolio_shares` + the RLS policy that reads it + a minimal share/revoke control** on portfolio settings — pick a family member, grant, revoke. The table and policy are not optional (the RLS predicate at `ARCHITECTURE.md:330` reads `portfolio_shares` to implement private-by-default plus opt-in sharing in one place); only the UI was in question, and a plain grant/revoke control closes AUTH-07 honestly. No notification flow, no share-request flow, no "shared with me" section.
- **D-12:** `portfolio_shares.permission` is typed `text CHECK (permission IN ('read','write'))` exactly as `ARCHITECTURE.md:111` specifies, but **the Phase 1 UI only ever grants `'read'`**. Write-sharing raises a ledger-attribution question — when the owner enters a transaction into a parent's portfolio, is `created_by` the actor or the portfolio owner? — that Phase 2 is better placed to answer alongside bulk entry. The schema needs no migration when that answer arrives. — **Reversibility:** costly — widening or narrowing the CHECK constraint later is a migration on a tenant table with a live RLS policy depending on the column.

### Instrument Identity & Lookup

- **D-13:** INST-03 search is backed by **Finnhub's free tier, US symbols only**, resolving a ticker or company name to `exchange_mic` + `symbol` and caching the result into the local `instruments` table on first add. Phase 1 is US-stock-only by its own success criteria, Finnhub is already a committed dependency in `.planning/research/STACK.md`, and 60 calls/min is far beyond need. **This does not pre-empt Phase 3's provider spike** — that spike is specifically about BIST end-of-day prices (EODHD vs Twelve Data) and is a separate decision. — **Reversibility:** reversible — put the lookup behind a `SymbolSearchProvider` interface so swapping providers touches one adapter.
- **D-14:** ISIN and FIGI come from **OpenFIGI enrichment at instrument-creation time** — free, no API key required at this volume, and already named for exactly this purpose at `ARCHITECTURE.md:296`. This is what makes success criterion 5 ("keyed by exchange + symbol + ISIN") true rather than nominally satisfied, and it populates the `figi` column the schema already reserves.
- **D-15:** When lookup returns nothing or the provider is unreachable, the user can **always hand-create an instrument** from `exchange_mic` + `symbol`; the row is marked unenriched (`enrichment_status` column, included in migration 001) so a later job backfills ISIN, FIGI and name. Recording a real transaction must never be blocked by an external service being down — the ledger is the source of truth, and Finnhub is not. Consequence: `instruments.isin` stays nullable with `unique(isin)` applying only where present, which is also required for the synthetic `CASH:*` / `XAU:PHYSICAL` rows that have no ISIN by nature.

### Claude's Discretion

The user did not select these two areas for discussion. Resolved from research defaults so downstream agents have a definite answer — treat these as decided, not open:

- **D-16 (Account layer):** `transactions.account_id` stays `NOT NULL` per `ARCHITECTURE.md:166`. Creating a portfolio **auto-creates one default account** (`name: 'Default'`, currency = portfolio base currency). Accounts are **not exposed in the Phase 1 UI** — the user sees portfolios and transactions only. The multi-account grouping surface belongs with Phase 2's broker-account work. — **Reversibility:** one-way — if a default account is later split into several real broker accounts, existing `transactions` and `lots` rows must be reassigned by migration, and `lot_closures` already reference them.
- **D-17 (Correction model):** An edit is implemented as **void + reissue: two new rows**. The correcting row sets `voids_transaction_id` to the original; a replacement row carries the corrected values. The original is never mutated or deleted, per LEDG-06 and `ARCHITECTURE.md:180`. The transaction list **defaults to the netted view** (corrected values shown, superseded rows hidden) with a per-row control to expand full history, plus a list-level toggle to show the raw ledger. Rationale: success criterion 3 requires the original stay "visible in history", not that it clutter the default view — and Phase 2 will multiply row counts. — **Reversibility:** one-way — the ledger's correction semantics are the foundation the audit log (Phase 12), returns engine (Phase 6) and corporate actions (Phase 5) all build on; changing the shape later invalidates every derived projection.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Schema and data model (primary — Phase 1 is largely a schema phase)
- `.planning/research/ARCHITECTURE.md` §Data Model, lines 92-274 — the concrete SQL schema this phase implements. Note lines 121-162 (reference/global tables), 164-203 (the ledger), and that `market_calendar` is **missing from this block** but specified in prose (see D-02).
- `.planning/research/ARCHITECTURE.md` §Transaction-ledger vs position-snapshot, lines 276-286 — why append-only wins; `transactions` is truth, `positions_snapshot` is a rebuildable projection (LEDG-07).
- `.planning/research/ARCHITECTURE.md` §Instrument modeling, lines 288-297 — single polymorphic table, `(exchange_mic, symbol)` primary uniqueness, ISIN secondary, FIGI/OpenFIGI, synthetic namespace (INST-01).
- `.planning/research/ARCHITECTURE.md` §Multi-currency, lines 318-325 — `fx_rate_to_base` frozen at write time vs read-time lookup for unrealized value. Phase 1 only writes the frozen leg, but the column must exist and be populated correctly from the first transaction.
- `.planning/research/ARCHITECTURE.md` §Multi-tenancy and row-level isolation, lines 327-332 — the exact RLS policy shape, the denormalized `portfolio_id` requirement, and which tables are global rather than tenant-scoped (AUTH-05, AUTH-08).
- `.planning/research/ARCHITECTURE.md` §Anti-Patterns to Avoid, lines 522-553 — specifically Anti-Pattern 1 (mutable positions table), 4 (ticker-only identity) and 5 (mutating transactions for corporate actions).

### Correctness constraints
- `.planning/research/PITFALLS.md` §1 Floating-point arithmetic for money, lines 19-40 — the basis for LEDG-09 and the CI guard in D-07.
- `.planning/research/PITFALLS.md` §5 Multi-currency errors: wrong rate, wrong date, wrong convention, lines 112-137 — the FX-rate direction convention must be typed explicitly in the schema, never stored as a bare number.
- `.planning/research/PITFALLS.md` §18 Mutable positions instead of an append-only ledger, lines 350-361.
- `.planning/research/PITFALLS.md` §19 Retrofitting multi-currency and i18n, lines 362-373 — why the currency columns must be right now even though Phase 1 is USD-only.
- `.planning/research/PITFALLS.md` §24 Postgres connection exhaustion from serverless functions, lines 430-441 — constrains how the RLS session variable (`app.current_user_id`) is set per request under pooling.

### Stack and provider decisions
- `.planning/research/STACK.md` — Better Auth, Neon Postgres, Drizzle ORM, Next.js 16 App Router, `decimal.js`, Finnhub free tier, TCMB EVDS. These are committed; do not re-litigate.
- `.claude/CLAUDE.md` §Technology Stack — the same decisions in their authoritative project-instruction form, including the explicit "What NOT to Use" table.

### Requirements and scope
- `.planning/REQUIREMENTS.md` lines 12-19 (AUTH-01…08), 23 (LEDG-01), 28-32 (LEDG-06/07/09/10), 37-39 (INST-01/03) — the requirement text this phase must satisfy.
- `.planning/ROADMAP.md` lines 32-44 — phase goal and the five success criteria that define done.
- `.planning/PROJECT.md` §Constraints and §Key Decisions, lines 115-149 — budget, multi-tenancy-from-day-one, FIFO as v1 cost basis, split FX sourcing.

</canonical_refs>

<code_context>
## Existing Code Insights

**This is a greenfield repository.** `.planning/` and `README.md` are the only tracked content; there is no application code, no `package.json`, no migrations, and no `.planning/codebase/` maps.

### Reusable Assets
- None. Every asset this phase needs is created by it.

### Established Patterns
- None in code. The binding patterns are documentary: the schema at `ARCHITECTURE.md:92-274` and the "What NOT to Use" table in `.claude/CLAUDE.md`.

### Integration Points
- **Neon Postgres** via the Vercel Marketplace integration — must be provisioned before migration 001 can run.
- **Resend** — API key for password-reset email (D-09).
- **Finnhub** free tier — API key for US symbol search (D-13).
- **OpenFIGI** — no key required at this volume (D-14).
- **TCMB EVDS** — free registered API key for the one-shot FX backfill (D-05).
- **Vercel** — already connected per `PROJECT.md:113`.

### Conventions this phase establishes for the whole project
Phase 1 is the first code in the repository, so it sets precedent that later phases inherit: the migration authoring pattern (D-06), the CI money-type guard (D-07), the provider-adapter boundary (D-13), and the void+reissue correction semantics (D-17). These should be written into `.claude/CLAUDE.md` §Conventions as they land — that section currently reads "Conventions not yet established."

</code_context>

<specifics>
## Specific Ideas

- Success criterion 4 must be verified by a **direct database-level test**, not an application-layer assertion: open a session as a second family member and confirm a raw query against the first user's `portfolios` and `transactions` returns zero rows. This is why app-layer scoping was explicitly rejected in D-06.
- Success criterion 5's "keyed by exchange + symbol + ISIN" is what drove D-14 (OpenFIGI enrichment) rather than a nullable-ISIN shortcut — the criterion should be demonstrably true for real exchange-traded instruments, with nullability reserved for synthetic and unenriched rows only.
- The `fx_rates` direction convention (is the stored number USD→TRY or TRY→USD?) must be explicit in the column naming or a typed enum, per `PITFALLS.md:112` and the Key Decision at `PROJECT.md:145`. Phase 1 writes the first rows, so it fixes this convention for good.

</specifics>

<deferred>
## Deferred Ideas

- **Write-permission portfolio sharing** — schema supports it (D-12) but the UI grants read only. Belongs in Phase 2, where the `created_by` attribution question can be answered alongside bulk entry.
- **Scheduled TCMB EVDS sync job and gold rates** — Phase 1 ships a one-shot backfill script only (D-05). Full DATA-02/DATA-03 integration is Phase 5.
- **Multi-account UI (broker account grouping)** — `accounts` exists and is populated with a hidden default (D-16); the user-facing surface belongs with Phase 2.
- **Backfill job for unenriched instruments** — D-15 marks rows for enrichment; the job that processes them fits naturally with Phase 3's market-data sync work.
- **Registration route for a fifth user** — deliberately not built (D-08). Additive if the tool ever leaves the family.
- **`.claude/CLAUDE.md` §Conventions population** — currently empty; should be filled as Phase 1's precedents land.

### Documentation defect noticed (not a Phase 1 concern)
- `.planning/ROADMAP.md` Progress table (lines 227-242) orders phases 10–13 differently from the Phases list (lines 24-27): the table has Agent Core at 10 and News at 13, the list has News at 10 and Agent Core at 11. Harmless now; should be corrected before Phase 10 planning.

</deferred>

---

*Phase: 1-Foundation — Sign In & First Position*
*Context gathered: 2026-08-16*
