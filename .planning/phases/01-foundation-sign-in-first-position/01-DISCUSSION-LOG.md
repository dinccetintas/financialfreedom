# Phase 1: Foundation — Sign In & First Position - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-08-16
**Phase:** 1-Foundation — Sign In & First Position
**Areas discussed:** Migration scope, Allowlist mechanism, Sharing scope, Instrument lookup

---

## Gray Area Selection

Six gray areas were surfaced. The user selected four for discussion.

| Area | Description | Selected |
|------|-------------|----------|
| Migration scope | How much of the ARCHITECTURE.md schema lands in migration 001 | ✓ |
| Account layer | portfolios → accounts → transactions with account_id NOT NULL, but Phase 1 criteria mention only portfolio + transaction | |
| Correction model | Append-only means "edit" is void+reissue — raw ledger view vs netted view; one amending row vs two | |
| Allowlist mechanism | How the four accounts come to exist given AUTH-04's "no public registration surface" | ✓ |
| Sharing scope | AUTH-07 mapped to Phase 1 but criterion 4 only tests isolation — how much sharing ships | ✓ |
| Instrument lookup | INST-03 is Phase 1 but the provider spike is Phase 3 | ✓ |

**Notes:** Areas already settled by research were listed up front and deliberately not re-asked: Better Auth / Neon / Drizzle / Next.js 16, append-only ledger with `voids_transaction_id`, RLS with denormalized `portfolio_id`, `(exchange_mic, symbol)` + ISIN identity, frozen `fx_rate_to_base`, decimal-only arithmetic.

---

## Migration Scope

| Option | Description | Selected |
|--------|-------------|----------|
| Non-retrofittable core | Identity + ledger + market-data shape (13 tables); decision/agent/news layers deferred as purely additive | ✓ |
| Full schema now | All ~25 tables from ARCHITECTURE.md:92-274 in one migration; nothing to retrofit ever, but ~15 empty tables carrying RLS and indexes for months | |
| Phase 1 minimum only | 7 tables; smallest surface, but corporate_actions typing and fx_rates arrive later — exactly the retrofit the roadmap says to avoid | |

**User's choice:** Non-retrofittable core
**Notes:** Reasoning that drove the recommendation — the five things the roadmap calls non-retrofittable all live in the identity/ledger/market-data core; the decision, agent and news layers are additive and their requirements will have changed by Phases 8–13.

### Follow-up: reference data seeded in migration 001

| Option | Description | Selected |
|--------|-------------|----------|
| Synthetic instruments | CASH:USD, CASH:TRY, CASH:EUR, XAU:PHYSICAL with exchange_mic NULL, fixing the namespace convention before a conflicting row is hand-created | ✓ |
| market_calendar table | Add the table missing from the schema block but specified in prose; seeded with 2026 BIST + US holidays for Phase 3's "holiday vs sync failure" distinction | ✓ |
| TCMB FX backfill | Historical USD/TRY into fx_rates so fx_rate_to_base is sourceable for backdated entries — but EVDS integration is nominally Phase 5 | ✓ |

**User's choice:** All three
**Notes:** TCMB backfill was flagged during the discussion as pulling Phase 5 work forward. Resolved by scoping it to a one-shot script in Phase 1, with the scheduled EVDS sync job, gold rates and full DATA-02/DATA-03 integration remaining in Phase 5. Recorded as a scope boundary in CONTEXT.md D-05 rather than a phase expansion.

### Follow-up: migration authoring

| Option | Description | Selected |
|--------|-------------|----------|
| Drizzle schema + raw SQL | drizzle-kit generate for tables/indexes/FKs; RLS policies, CHECK constraints and seeds hand-written into the same numbered migration | ✓ |
| Hand-written SQL only | Maximum control and reviewability for money-critical DDL, at the cost of drift between the TS schema and the database | |
| Drizzle-kit only, RLS in app layer | Simplest tooling, but cannot satisfy criterion 4's direct database-level isolation test | |

**User's choice:** Drizzle schema + raw SQL

### Follow-up: enforcing LEDG-09 (no floats)

| Option | Description | Selected |
|--------|-------------|----------|
| CI guard + tests | information_schema introspection failing on real/double precision/float in money-named columns, plus decimal.js round-trip tests | ✓ |
| Convention + code review | Zero build cost; relies on every future migration being reviewed by someone who remembers, including agent-authored ones in Phase 12 | |
| Custom Drizzle column type | Strongest type-level guarantee; more upfront plumbing and a custom abstraction every contributor must learn | |

**User's choice:** CI guard + tests

---

## Allowlist Mechanism

| Option | Description | Selected |
|--------|-------------|----------|
| Seeded users, forced reset | Setup script inserts four users with a temp password, forced change on first login; no registration route ships at all | ✓ |
| Allowlist table + self-serve first login | Gated /register route; parents pick their own password with no temp-password handoff — but a registration surface does exist | |
| One-time invite tokens | No standing registration route and no insecurely transmitted temp password; costs token generation/expiry/revocation surface in Phase 1 | |

**User's choice:** Seeded users, forced reset
**Notes:** Chosen as the most literal reading of AUTH-04 ("no public registration surface exists"). Re-running the script to add a fifth person was accepted as fine at family scale.

### Follow-up: password-reset email delivery

| Option | Description | Selected |
|--------|-------------|----------|
| Resend | 3,000 emails/month free, first-class Next.js/Vercel integration, React Email templates for eventual TR/EN | ✓ |
| Amazon SES | Cheapest at volume and highly reliable; requires AWS setup, domain verification and sandbox exit for maybe two resets a year | |
| Owner-triggered reset, no email | Zero infrastructure, but AUTH-02 explicitly says "via an emailed link" — leaves a Phase 1 requirement unmet | |

**User's choice:** Resend

### Follow-up: session policy

| Option | Description | Selected |
|--------|-------------|----------|
| Long-lived, sliding | 30-day DB-backed session, httpOnly/secure/SameSite=Lax, refreshed on activity, unlimited devices | ✓ |
| Short-lived with refresh | Smaller window if a device is lost; more moving parts and more ways to strand a non-technical user at a login screen | |
| Long-lived + step-up on writes | Protects the ledger from an unattended phone, but adds friction to exactly the flow Phase 2 wants fast | |

**User's choice:** Long-lived, sliding

---

## Sharing Scope

| Option | Description | Selected |
|--------|-------------|----------|
| Schema + RLS + minimal UI | portfolio_shares, the RLS policy reading it, and a plain grant/revoke control — closes AUTH-07 without inventing notification or request flows | ✓ |
| Schema + RLS only, defer UI | Smallest Phase 1, but AUTH-07 sits half-done with no later phase mapped to finish it | |
| Full sharing with permissions UI | Most complete, but adds meaningful UI to the phase whose job is proving the schema is right, and write-sharing raises attribution questions Phase 2 is better placed to answer | |

**User's choice:** Schema + RLS + minimal UI
**Notes:** The table and policy were never really optional — the RLS predicate at ARCHITECTURE.md:330 reads portfolio_shares to implement private-by-default and opt-in sharing in a single place. Only the UI was genuinely in question.

### Follow-up: permission levels

| Option | Description | Selected |
|--------|-------------|----------|
| Column typed 'read'\|'write', only read grantable | Schema matches ARCHITECTURE.md:111 exactly; UI grants read only, so no migration when write-sharing arrives | ✓ |
| Read and write both grantable now | Close to what Phase 2's bulk-entry adoption problem needs, but decides ledger attribution semantics in the phase least equipped to reason about it | |
| Read-only, no permission column | Simplest table; adding the column later is a migration on a tenant table with live RLS depending on it | |

**User's choice:** Column typed 'read'|'write', only read grantable

---

## Instrument Lookup

**Correction made during discussion:** the initial framing suggested that wiring a data provider in Phase 1 would pre-empt Phase 3's spike. That was wrong — Phase 3's spike is specifically about BIST end-of-day prices (EODHD vs Twelve Data), whereas Finnhub's free tier is already committed in STACK.md for US profiles and news and is a separate decision. Phase 1 is US-stock-only. This materially changed the tradeoff and was surfaced before the question was asked.

| Option | Description | Selected |
|--------|-------------|----------|
| Finnhub free-tier US search | Real search by ticker or company name, cached into instruments on first add; free at 60 calls/min, already a committed dependency | ✓ |
| Seeded static list | Zero external dependency and deterministic tests, but adding an unlisted name means editing a CSV and redeploying — Phase 2's bulk entry hits that wall immediately | |
| Manual entry only | No provider or rate limits, but the parents will not know an ISIN and INST-03 says "search for" | |

**User's choice:** Finnhub free-tier US search

### Follow-up: ISIN sourcing

| Option | Description | Selected |
|--------|-------------|----------|
| OpenFIGI enrichment | Free, no key at this volume, already named at ARCHITECTURE.md:296; satisfies criterion 5 fully and populates the reserved figi column | ✓ |
| Nullable ISIN + manual override | Simplest, no new integration, but criterion 5 says "keyed by exchange + symbol + ISIN" — a null ISIN arguably fails the phase's own test | |
| Require ISIN at entry | Guarantees criterion 5, but hard-blocks the synthetic CASH:USD / XAU:PHYSICAL rows, which have no ISIN by nature | |

**User's choice:** OpenFIGI enrichment

### Follow-up: lookup failure handling

| Option | Description | Selected |
|--------|-------------|----------|
| Manual create, flagged for enrichment | Never blocks recording a real transaction on an external service being up; unenriched rows backfilled later | ✓ |
| Hard block until resolved | No junk in the instruments table, but a Finnhub outage stops data entry entirely | |
| Queue for owner review | Keeps the shared reference table clean, but adds an approval workflow in Phase 1 and makes the owner a bottleneck on his parents' data entry | |

**User's choice:** Manual create, flagged for enrichment

---

## Claude's Discretion

Two gray areas were surfaced but not selected for discussion. Resolved from research defaults and recorded as decided in CONTEXT.md so downstream agents are not left guessing:

- **Account layer (D-16):** `transactions.account_id` stays NOT NULL; creating a portfolio auto-creates one hidden default account; the multi-account UI belongs with Phase 2.
- **Correction model (D-17):** void + reissue as two rows; transaction list defaults to the netted view with per-row history expansion and a raw-ledger toggle.

These were offered again at the closing gate ("Revisit account layer or correction model") and the user chose to proceed to context instead.

## Deferred Ideas

- Write-permission portfolio sharing → Phase 2
- Scheduled TCMB EVDS sync job and gold rates → Phase 5
- Multi-account (broker grouping) UI → Phase 2
- Backfill job for unenriched instruments → Phase 3
- Registration route for a fifth user → deliberately not built; additive later
- `.claude/CLAUDE.md` §Conventions population → as Phase 1 precedents land
- ROADMAP.md Progress table orders phases 10–13 inconsistently with the Phases list → fix before Phase 10
