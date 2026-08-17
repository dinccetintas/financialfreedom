# Phase 1: Foundation — Sign In & First Position - Research

**Researched:** 2026-08-16
**Domain:** Next.js 16 auth scaffolding + append-only financial ledger schema on Postgres RLS (greenfield repo)
**Confidence:** HIGH on schema/architecture (corroborated by prior project research + this session's library-specific verification), MEDIUM on exact Better Auth + Neon RLS wiring (verified via docs this session but not hands-on tested), LOW on nothing load-bearing — the one genuine gap (Better Auth has no native forced-password-change) is flagged with a concrete workaround.

## Summary

This phase is almost entirely a schema and auth-scaffolding phase on a repository that currently contains nothing but `.planning/`. Two things this session's research changed or sharpened versus the pre-existing project-level research (`STACK.md`/`ARCHITECTURE.md`/`PITFALLS.md`):

1. **Neon's pooled connection string does not support session-level `SET`/`RESET`** — confirmed directly from Neon's own docs this session. This makes D-06's choice (hand-written raw-SQL RLS policies, app-set session variable) not just a stylistic preference but a *necessity*: Neon's declarative `crudPolicy`/`authUid` Drizzle helpers require Neon's own JWT-based Data API (`neon_auth`), which is incompatible with a Better Auth cookie-session app. The correct wiring is `set_config('app.current_user_id', $id, true)` (transaction-scoped, the `true` third argument) called inside a `db.transaction()` block over the pooled connection — this works because the scope ends automatically at transaction commit, so it never leaks across pooled-connection reuse, unlike a bare `SET`.
2. **Better Auth has no built-in "force password change on next login."** D-08's setup-script flow (seed 4 users with a random temp password, force them to set their own on first sign-in) is not a config flag — it requires a custom `mustChangePassword` field on the user record plus an app-level gate. This is a real implementation gap the planner needs a task for, not an assumption to carry forward silently.

Everything else in this phase — the ledger schema, RLS-as-backstop pattern, decimal/NUMERIC discipline, instrument identity, void+reissue corrections — was already correctly specified in the prior project research (`ARCHITECTURE.md` §Data Model is the canonical schema) and is reaffirmed here, not re-litigated.

**Primary recommendation:** Scaffold Next.js 16 + Drizzle + Better Auth + Neon exactly as `STACK.md` specifies; author migration 001 as Drizzle-generated DDL for tables plus hand-written raw SQL for RLS policies and CHECK constraints in the same migration file (D-06); wire RLS through `db.transaction()` + transaction-scoped `set_config`, never a bare `SET`; treat Better Auth's own generated `user`/`session`/`account`/`verification` tables as the identity substrate and extend `user` via `additionalFields` rather than creating a second, conflicting `users` table.

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Email+password sign-in & session | Frontend Server (SSR) | Database/Storage | Better Auth's route handler and middleware run server-side inside Next.js; the session row is persisted in Postgres (database-backed session per D-10) |
| Password-reset email delivery | API/Backend | — | `sendResetPassword` fires server-side via Resend; the reset token/URL must never be constructable or visible client-side |
| Allowlist enforcement (no public signup) | API/Backend | Database/Storage | No signup route exists at all (D-08); `databaseHooks.user.create.before` is a defense-in-depth backstop, not the primary control |
| Portfolio create/name, share/revoke | API/Backend | Database/Storage | Server actions/route handlers validate and write; RLS is the DB-side backstop on every read/write |
| Transaction ledger write (buy, correction) | API/Backend | Database/Storage | Ledger writer functions enforce append-only + decimal correctness before Postgres persists the row |
| Transaction list filter/sort | Browser | API/Backend | Client-side filter/sort of an already-fetched, server-scoped page of rows; the server, not the browser, decides which rows exist to filter |
| Tenant isolation | Database/Storage | API/Backend | RLS policy is the enforcement backstop (success criterion 4 requires a DB-level test); the app layer sets the session variable per request, it does not itself enforce isolation |
| Instrument search & OpenFIGI/Finnhub enrichment | API/Backend | — | External API keys (Finnhub) must never reach the browser; OpenFIGI/Finnhub calls happen server-side, results cached into `instruments` |
| Money/decimal arithmetic | API/Backend | Database/Storage | `decimal.js` at the app layer, `NUMERIC` at the DB layer; the browser only ever displays a pre-formatted string, never computes |

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| AUTH-01 | Sign in with email and password | Better Auth `emailAndPassword.enabled: true` — see Code Examples |
| AUTH-02 | Reset forgotten password via emailed link | Better Auth `forgetPassword`/`sendResetPassword` + Resend — see Code Examples |
| AUTH-03 | Session persists across refresh and devices | Better Auth database session, `session.expiresIn`/`updateAge` sliding config (D-10) — see Code Examples |
| AUTH-04 | Signup closed to fixed allowlist, no public registration surface | Setup script seeds 4 users (D-08); no signup route ships; `databaseHooks.user.create.before` backstop |
| AUTH-05 | Row-level security keyed to authenticated session on every tenant table | Hand-written `CREATE POLICY` SQL + transaction-scoped `set_config` — see Code Examples and Pitfall 1 |
| AUTH-06 | Create and name multiple portfolios | `portfolios` table (`ARCHITECTURE.md:104-107`), Ledger API writer |
| AUTH-07 | Share a portfolio with a named family member, revoke | `portfolio_shares` table + RLS policy reading it (D-11), minimal grant/revoke UI |
| AUTH-08 | Portfolios private by default | RLS policy default-denies; sharing is the only path to a second reader (D-11/D-12) |
| LEDG-01 | Record buy transaction: date, instrument, quantity, price, fees, currency | `transactions` table (`ARCHITECTURE.md:165-179`) |
| LEDG-06 | Append-only ledger; corrections create new rows, not mutations | Void + reissue pattern (D-17), `voids_transaction_id` self-reference |
| LEDG-07 | Positions derived from ledger, rebuildable from scratch | Computed at query time from `lots`/`lot_closures` in Phase 1 — see Open Questions (no `positions_snapshot` table in migration 001 per D-01) |
| LEDG-09 | All monetary values, FX rates, quantities use decimal — never float | `NUMERIC` columns + `decimal.js` + CI guard (D-07) — see Code Examples |
| LEDG-10 | View, filter, sort full transaction history | Server-scoped query + client-side filter/sort UI |
| INST-01 | Identify every instrument by exchange + symbol + ISIN, never bare ticker | `instruments` table, `unique(exchange_mic, symbol)`, `unique(isin)` where present (`ARCHITECTURE.md:122-135`) |
| INST-03 | Search for and add an instrument by ticker or company name | Finnhub `/search` (free tier) + OpenFIGI `/v3/mapping` enrichment (D-13/D-14) — see Code Examples |
</phase_requirements>

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| next | 16.3.1 [VERIFIED: npm registry] | App Router framework | Owner's committed stack (`.claude/CLAUDE.md`); current stable major |
| better-auth | 1.6.29 [VERIFIED: npm registry] | Email+password auth, session management, Drizzle adapter | Vercel-backed successor to Auth.js; declares `next: ^14 || ^15 || ^16` and `drizzle-orm: ^0.45.2` as peer deps, both satisfied [VERIFIED: npm registry `better-auth` peerDependencies] |
| drizzle-orm | 0.45.2 [VERIFIED: npm registry] | Schema + query builder | Committed stack; matches Better Auth's exact peer-dep pin |
| drizzle-kit | 0.31.10 [VERIFIED: npm registry] | Migration generation/execution CLI | Companion to drizzle-orm; declared peer requires `>=0.31.4` [VERIFIED: npm registry `better-auth` peerDependencies] |
| @neondatabase/serverless | 1.1.0 [VERIFIED: npm registry] | HTTP/WebSocket Postgres driver for Vercel Functions | Committed stack (`STACK.md`); needed for the pooled-connection RLS pattern below |
| decimal.js | 10.6.0 [VERIFIED: npm registry] | Arbitrary-precision decimal arithmetic | Non-negotiable per LEDG-09 and project CLAUDE.md |
| zod | 4.4.3 [VERIFIED: npm registry] | Input validation on every server action/route handler | Standard TypeScript validation choice; required by Security Domain V5 below |
| resend | 6.20.0 [VERIFIED: npm registry] | Transactional email (password reset) | D-09; free tier confirmed sufficient (see Environment Availability) |
| react-email | 6.9.2 [VERIFIED: npm registry] | Password-reset email template (TR/EN-ready) | **Use this unified package directly, not `@react-email/components`** — see Package Legitimacy Audit |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| pg (or Neon's driver in migration-only contexts) | — | `drizzle-kit migrate` needs a **direct, non-pooled** connection string, not the `-pooler` one [CITED: neon.com/docs/connect/connection-pooling] | Migration execution only, never app runtime queries |
| OpenFIGI REST API (no SDK) | v3 | ISIN/FIGI enrichment at instrument-creation time (D-14) | Server-side fetch call, no API key required at this volume, 25 requests/min and 10 jobs/request unauthenticated [VERIFIED: openfigi.com/api/documentation] |
| Finnhub REST API (no SDK) | v1 | Symbol/company-name search (D-13, INST-03) | Server-side fetch call, free tier, 60 calls/min |
| TCMB EVDS REST API (no SDK) | v1 (`evds2.tcmb.gov.tr/service/evds/`) | One-shot historical USD/TRY backfill script (D-05) | Registered API key sent via `key` HTTP header (not URL query, since a 2024 API change) [CITED: TCMB EVDS usage guides] |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Hand-written raw-SQL RLS policies (D-06) | Drizzle's `crudPolicy`/`authUid` declarative RLS helpers (`drizzle-orm/neon`) | Only works with Neon's own JWT-based Data API auth (`neon_auth`) [CITED: neon.com/docs/guides/rls-drizzle] — incompatible with Better Auth's cookie-session model. Not viable for this project; documented here so the planner doesn't rediscover this by trial and error. |
| `@neondatabase/serverless` pooled + `set_config(...,true)` in a transaction | Direct (non-pooled) connection for app runtime | Direct connections don't scale across concurrent serverless invocations at 4 users' burst pattern (Pitfall 24 in `PITFALLS.md`); pooled + transaction-scoped `set_config` is the correct choice |
| `react-email` (unified package) | `@react-email/components` | Deprecated — all components/rendering utilities were folded into the main `react-email` package [VERIFIED: npm registry, `deprecated: true`] |

**Installation:**
```bash
npx create-next-app@latest --typescript --app
npm install drizzle-orm@0.45.2 @neondatabase/serverless@1.1.0 better-auth@1.6.29 decimal.js@10.6.0 zod@4.4.3 resend@6.20.0 react-email@6.9.2
npm install -D drizzle-kit@0.31.10 vitest
npx @better-auth/cli generate   # generates Better Auth's own Drizzle schema (user/session/account/verification)
```

**Version verification:** All versions above confirmed live via `npm view <pkg> version` this session (2026-08-16). Better Auth's exact `drizzle-orm` peer pin (`^0.45.2`) was cross-checked against the installed `drizzle-orm` version to avoid a silent incompatibility — they match.

## Package Legitimacy Audit

| Package | Registry | Age (this version) | Downloads | Source Repo | Verdict | Disposition |
|---------|----------|---------------------|-----------|-------------|---------|-------------|
| next | npm | Published 2026-08-13 (days old) | 45.9M/wk | github.com/vercel/next.js | SUS (`too-new`) | Approved — reason is patch-recency on an extremely established package (46M weekly downloads, official Vercel repo), not a hallucination signal. `checkpoint:human-verify` before first install regardless, per protocol. |
| better-auth | npm | Published 2026-08-14 (days old) | 4.7M/wk | github.com/better-auth/better-auth | SUS (`too-new`) | Approved — same reasoning; 4.7M weekly downloads, official repo, Vercel-backed. `checkpoint:human-verify` before first install. |
| resend | npm | Published 2026-08-13 (days old) | 7.0M/wk | github.com/resend/resend-node | SUS (`too-new`) | Approved — same reasoning. `checkpoint:human-verify` before first install. |
| react-email | npm | Published 2026-08-07 (days old) | 3.1M/wk | github.com/resend/react-email | SUS (`too-new`) | Approved — same reasoning. `checkpoint:human-verify` before first install. |
| @react-email/components | npm | Published 2026-04-09 | 4.5M/wk | github.com/resend/react-email | SUS (`deprecated`) | **REMOVED** — do not install. Use `react-email` (unified package) instead; all components/render utilities moved there. |
| drizzle-orm | npm | Published 2026-03-27 | 13.6M/wk | github.com/drizzle-team/drizzle-orm | OK | Approved |
| drizzle-kit | npm | Published 2026-03-17 | 11.4M/wk | github.com/drizzle-team/drizzle-orm | OK | Approved |
| decimal.js | npm | Published 2025-07-06 | 68.9M/wk | github.com/MikeMcl/decimal.js | OK | Approved |
| @neondatabase/serverless | npm | Published 2026-04-17 | 2.9M/wk | github.com/neondatabase/serverless | OK | Approved |
| zod | npm | Published 2026-05-04 | 224.1M/wk | github.com/colinhacks/zod | OK | Approved |

**Packages removed due to [SLOP] verdict:** none.
**Packages flagged as suspicious [SUS]:** next, better-auth, resend, react-email — all flagged solely on version-publish recency (`too-new`), not on any hallucination signal (all have multi-million weekly downloads and matching official GitHub repos). The planner must still insert a `checkpoint:human-verify` task before each is first installed, per protocol, even though this research assesses the risk as effectively nil.

*`react-email` itself is approved; `@react-email/components` specifically is removed. Do not conflate the two — they are not the same package.*

## Architecture Patterns

### System Architecture Diagram

```
┌─────────────────────────── Browser ───────────────────────────┐
│  Sign-in / reset-password forms · Portfolio & transaction UI   │
│  (client-side filter/sort of an already-scoped page of rows)   │
└───────────────────────────┬─────────────────────────────────--┘
                             │ HTTPS (cookie: better-auth.session)
┌────────────────────────────▼──────────────────────────────────┐
│              NEXT.JS 16 APP ROUTER (Vercel Function)           │
│  ┌─────────────┐  ┌──────────────────┐  ┌───────────────────┐ │
│  │ Better Auth │  │ Server Actions /  │  │ Instrument search │ │
│  │ route + mw  │  │ route handlers:   │  │ (Finnhub, then    │ │
│  │ (sign-in,   │  │ createPortfolio,  │  │ OpenFIGI enrich)  │ │
│  │ reset, ...) │  │ addTransaction,   │  │                   │ │
│  │             │  │ shareGrant/Revoke │  │                   │ │
│  └──────┬──────┘  └────────┬──────────┘  └─────────┬─────────┘ │
│         │   every write/read wrapped in db.transaction():      │
│         │   1. set_config('app.current_user_id', id, true)     │
│         │   2. run the actual query via tx, never db           │
└─────────┼──────────────────┼────────────────────────┼──────────┘
          │                  │                        │
┌─────────▼──────────────────▼────────────────────────▼──────────┐
│         NEON POSTGRES (pooled connection, RLS-enforced)         │
│  user/session/account/verification (Better Auth-owned)          │
│  portfolios · portfolio_shares · accounts (RLS: owner ∪ shared) │
│  instruments · instrument_aliases (global, no RLS)               │
│  transactions (append-only, RLS via denormalized portfolio_id)  │
│  lots · lot_closures (RLS via denormalized portfolio_id)        │
│  fx_rates · market_calendar (global, no RLS)                    │
└───────────────────────────────────────────────────────────────┘
          ▲
          │ one-shot script, run manually during Phase 1 setup
┌─────────┴─────────┐        ┌─────────────────────────┐
│  TCMB EVDS backfill │        │  Resend (password reset │
│  (USD/TRY history)  │        │  email delivery)        │
└─────────────────────┘        └─────────────────────────┘
```

A reader tracing "user records a buy" follows: Browser form submit → Server Action → `db.transaction` sets `app.current_user_id` for that transaction only → INSERT into `transactions` (and `lots` if it's an opening buy) → Postgres RLS policy evaluates `portfolio_id` against the just-set session variable → row is written or rejected → transaction commits, session variable scope ends automatically.

### Recommended Project Structure

```
src/
├── app/
│   ├── (auth)/                 # sign-in, reset-password pages (public)
│   ├── (dashboard)/            # portfolio list, transaction list (session-gated)
│   └── api/auth/[...all]/      # Better Auth catch-all route handler
├── lib/
│   ├── auth/
│   │   ├── server.ts           # betterAuth() instance, databaseHooks, sendResetPassword
│   │   └── client.ts           # authClient (signIn.email, forgetPassword, ...)
│   ├── db/
│   │   ├── schema/
│   │   │   ├── auth.ts         # Better Auth-generated schema (user/session/account/verification)
│   │   │   ├── portfolio.ts    # portfolios, portfolio_shares, accounts
│   │   │   ├── instruments.ts  # instruments, instrument_aliases
│   │   │   └── ledger.ts       # transactions, lots, lot_closures, corporate_actions*, fx_rates, market_calendar
│   │   ├── client.ts           # Drizzle instance over @neondatabase/serverless, pooled connection string
│   │   ├── rls.ts              # withUserScope(userId, fn) — wraps db.transaction + set_config
│   │   └── migrations/0000_*.sql  # generated DDL + hand-written CREATE POLICY / CHECK (D-06)
│   ├── ledger/
│   │   ├── add-transaction.ts  # validated writer, decimal.js in/out
│   │   └── correct-transaction.ts  # void + reissue (D-17)
│   └── instruments/
│       ├── finnhub.ts          # search() — server-side only, FINNHUB_API_KEY never sent to client
│       └── openfigi.ts         # enrich() — ISIN/FIGI mapping at creation time
└── scripts/
    └── backfill-fx-rates.ts    # one-shot TCMB EVDS script (D-05), run manually, not a cron job
```

*`corporate_actions`/`corporate_action_applications` tables are created in migration 001 per D-01 but have no application logic yet — that's Phase 5. Creating the empty, correctly-typed table now is the point; do not build the application job in this phase.*

### Pattern 1: RLS session-variable wiring under a pooled connection

**What:** Every authenticated query runs inside `db.transaction(async (tx) => { ... })`, and the very first statement inside that transaction sets `app.current_user_id` as a **transaction-scoped** config value.
**When to use:** Every single read or write that touches a tenant-scoped table (`portfolios`, `accounts`, `transactions`, `lots`, `lot_closures`, `portfolio_shares`).
**Why this exact shape:** Neon's pooled (`-pooler`) connection string runs PgBouncer in **transaction mode**, and Neon's own docs state plainly that bare `SET`/`RESET` (session-scoped) statements are **not supported** through it [VERIFIED: neon.com/docs/connect/connection-pooling — "SET / RESET (session variables)" listed under "not supported with pooled connections"]. `set_config(setting, value, is_local)` with `is_local = true` is the transaction-scoped equivalent of `SET LOCAL` — its scope ends automatically at `COMMIT`/`ROLLBACK`, so it never leaks into the next tenant's transaction when PgBouncer recycles the physical connection. This is the one correct pattern for this exact stack; a bare `SET` here is a live cross-tenant leak waiting to happen the moment two requests share a recycled connection.

**Example:**
```typescript
// Source: pattern synthesized from Neon connection-pooling docs [CITED: neon.com/docs/connect/connection-pooling]
// and Drizzle's documented RLS transaction pattern [CITED: orm.drizzle.team/docs/rls]
import { sql } from 'drizzle-orm';
import { db } from '@/lib/db/client'; // pooled connection string

export async function withUserScope<T>(
  userId: string,
  fn: (tx: typeof db) => Promise<T>
): Promise<T> {
  return db.transaction(async (tx) => {
    // is_local = true -> transaction-scoped, cleared automatically at commit
    await tx.execute(sql`select set_config('app.current_user_id', ${userId}, true)`);
    return fn(tx);
  });
}
```

Hand-written RLS policy (appended as raw SQL in the same migration file as the Drizzle-generated `CREATE TABLE`, per D-06):
```sql
-- Source: pattern from ARCHITECTURE.md:330, translated to concrete SQL this session
ALTER TABLE portfolios ENABLE ROW LEVEL SECURITY;
CREATE POLICY portfolios_isolation ON portfolios
  USING (
    owner_user_id = current_setting('app.current_user_id', true)::uuid
    OR id IN (
      SELECT portfolio_id FROM portfolio_shares
      WHERE shared_with_user_id = current_setting('app.current_user_id', true)::uuid
    )
  );
```

### Pattern 2: Better Auth allowlist + no-signup-route enforcement

**What:** No `/sign-up` route ships at all (D-08 is the primary control). `databaseHooks.user.create.before` is a defense-in-depth backstop for any code path that could still call user-creation (e.g. an admin-plugin call, a future regression).
**When to use:** Configured once in the Better Auth server instance.
**Example:**
```typescript
// Source: pattern per Better Auth databaseHooks docs [CITED: better-auth.com/docs — user.create.before hook]
export const auth = betterAuth({
  emailAndPassword: { enabled: true },
  databaseHooks: {
    user: {
      create: {
        before: async (user) => {
          const allowlist = (process.env.FAMILY_ALLOWLIST ?? '').split(',');
          if (!allowlist.includes(user.email)) {
            throw new Error('Signup is closed to a fixed family allowlist.');
          }
          return { data: user };
        },
      },
    },
  },
  session: {
    expiresIn: 60 * 60 * 24 * 30, // 30 days (D-10: long-lived)
    updateAge: 60 * 60 * 24,      // refresh on activity, sliding
  },
});
```

### Pattern 3: Forced password change on first login (custom — no native Better Auth support)

**What:** Better Auth has **no built-in "force password change on next login" mechanism** [VERIFIED via WebSearch of Better Auth's own GitHub issue tracker: issue titled "Support for Forcing Password Change on next login" is an open feature request, not a shipped feature]. D-08 requires exactly this (setup script seeds a random temp password; each user must set their own on first sign-in).
**When to use:** Required for D-08 to actually work as specified.
**How:** Add a `mustChangePassword: boolean` field to the user record via Better Auth's `additionalFields`, set it `true` when the setup script creates each user, and gate the dashboard layout (server-side, in the session-reading layout/middleware) so any session with `mustChangePassword === true` redirects to a "set your password" page before reaching anything else. Clear the flag inside that page's own password-change handler.

```typescript
// Source: pattern per Better Auth additionalFields docs [CITED: better-auth.com/docs/adapters/drizzle]
export const auth = betterAuth({
  user: {
    additionalFields: {
      mustChangePassword: { type: 'boolean', required: false, defaultValue: false, input: false },
    },
  },
});
```

### Pattern 4: Reconciling Better Auth's generated tables with the ARCHITECTURE.md schema

**What:** `npx @better-auth/cli generate` produces its own `user`, `session`, `account`, `verification` tables (exact shape controlled by Better Auth, not hand-designed). `ARCHITECTURE.md`'s schema block lists a `users` table with `password_hash`, `locale`, `ui_mode`, `advice_prescriptiveness` — that table pre-dates the concrete auth-library binding and **must not be built as a second, competing table.**
**Resolution:** Treat Better Auth's generated `user` table as the canonical `users` table. Better Auth manages `password_hash` internally (inside its own `account` table, one row per credential provider — not a bare column on `user`). Add `locale`, `ui_mode`, `advice_prescriptiveness`, and `mustChangePassword` to `user` via `additionalFields`, then run `npx @better-auth/cli generate` again to regenerate the Drizzle schema file with the new columns included. Every foreign key in the rest of the schema (`portfolios.owner_user_id`, `portfolio_shares.shared_with_user_id`, `transactions.created_by`) points at `user.id`, not a separately-created `users.id`.
**Known adapter gotcha:** avoid the `fieldName` remapping option inside `additionalFields` — it has a documented bug where the Drizzle adapter throws "field does not exist in the schema" even when the column is genuinely present [CITED: github.com/better-auth/better-auth issue #4211]. Match the additionalField's property name to the actual Drizzle column name directly instead of remapping.

### Anti-Patterns to Avoid

- **Bare `SET app.current_user_id = ...` (no `LOCAL`, no `set_config(...,true)`, no transaction wrapper) over the pooled connection string.** Neon explicitly documents this as unsupported through pooled connections — it either silently no-ops or, worse, leaks into a different tenant's next transaction on a recycled connection. Always use `set_config(..., true)` inside `db.transaction()`.
- **Creating a second `users` table alongside Better Auth's generated `user` table.** This produces two conflicting identity sources and an FK ambiguity the moment `portfolios.owner_user_id` needs to point somewhere. Extend Better Auth's table via `additionalFields` instead.
- **Running `drizzle-kit migrate` against the pooled (`-pooler`) connection string.** Neon's own guidance is to run migrations over a direct connection [CITED: neon.com/docs/guides/rls-drizzle — "Run these migrations over a direct (non-pooled) connection string, not a pooled one"]. Keep two env vars: `DATABASE_URL` (pooled, app runtime) and `DATABASE_URL_DIRECT` (unpooled, migrations only).
- **Reaching for Neon's declarative `crudPolicy`/`authUid` RLS helpers.** They assume Neon's own JWT-based Data API auth; wiring them to Better Auth's cookie session is not a supported path and would require reinventing the JWT bridge Neon's Data API already does — not worth it when hand-written `CREATE POLICY` SQL (D-06) already works.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|--------------|-----|
| Password hashing, session token generation, CSRF-safe cookie handling | A custom auth layer | Better Auth's `emailAndPassword` + database sessions | Password hashing and session security are exactly the class of code where a subtle mistake is invisible until exploited; Better Auth is the Vercel-backed, actively maintained standard for this stack |
| Multi-tenant row filtering | Hand-added `WHERE user_id = ?` on every query, trusted to never be forgotten | Postgres RLS policies (this phase's D-06/success-criterion-4) | One missed `WHERE` clause anywhere is a silent cross-tenant leak; RLS makes the omission structurally impossible, not just unlikely |
| ISIN/FIGI lookup | A hand-maintained instrument-identifier mapping table | OpenFIGI's free `/v3/mapping` endpoint | OpenFIGI is the purpose-built, free, no-key-required service for exactly this; hand-maintaining identifier crosswalks is exactly the kind of "looks simple, is actually a data-maintenance project" trap `PITFALLS.md` Pitfall 7 warns about |
| Decimal-safe money arithmetic | Custom fixed-point integer math on `bigint` cents | `decimal.js` end-to-end | Already the committed project decision (LEDG-09); reinventing arbitrary-precision decimal math to save a dependency is pure risk for zero benefit |

**Key insight:** every "don't hand-roll" item in this phase is also a non-retrofittable foundation decision — a rolled-your-own auth bug, a missing RLS policy, or a bare-ticker instrument identity are exactly the classes of mistake `PROJECT.md` explicitly calls out as unaffordable to discover after real data exists.

## Common Pitfalls

### Pitfall 1: Bare `SET`/`SET LOCAL` for the RLS session variable under Neon's pooled connection
**What goes wrong:** A `SET app.current_user_id = '...'` call (or `SET LOCAL` outside of an explicit transaction) either silently fails to apply, or — worse — persists on the physical connection after PgBouncer recycles it to a different tenant's next transaction, since Neon's pooler runs transaction-mode pooling and does not support session-scoped `SET`/`RESET` through the pooled string.
**Why it happens:** `SET LOCAL` looks transaction-scoped by name, but if the code path executing it isn't genuinely inside a started transaction (e.g. it runs as a standalone statement before `BEGIN`), Postgres treats it as a no-op-outside-transaction or, depending on driver batching, as session-scoped.
**How to avoid:** Always call `set_config('app.current_user_id', $1, true)` (the `true` third argument, not `SET LOCAL` syntax) as the literal first statement inside a `db.transaction()` callback, never as a standalone query.
**Warning signs:** A test where two different simulated users, run concurrently, occasionally see each other's rows — this is the signature of a leaked session variable on a reused pooled connection, and it will be intermittent, not consistent, making it easy to miss in a single manual test run.

### Pitfall 2: Assuming Better Auth supports forced password change out of the box
**What goes wrong:** D-08's setup-script flow is designed around each family member being forced to set their own password on first login. If the planner assumes this is a Better Auth config flag, no task gets scoped for it, and the feature silently doesn't exist at ship time.
**How to avoid:** Scope an explicit task for the `mustChangePassword` additionalField + gate, per Pattern 3 above. This is real, non-trivial application code, not configuration.

### Pitfall 3: Creating a duplicate `users` table alongside Better Auth's generated `user` table
**What goes wrong:** `ARCHITECTURE.md`'s schema block (written before the auth library was chosen) lists a standalone `users` table. If migration 001 creates both Better Auth's generated tables *and* a hand-written `users` table, the app ends up with two disconnected identity records per person, and every FK in the rest of the schema has to pick one — inevitably inconsistently across different writers.
**How to avoid:** Per Pattern 4, extend Better Auth's `user` table via `additionalFields`; do not create a second table. Every reference in this phase's schema (`portfolios.owner_user_id`, etc.) points at `user.id`.

### Pitfall 4: OpenFIGI's unauthenticated rate limit (25 req/min, 10 jobs/request) throttling instrument enrichment
**What goes wrong:** D-14's enrichment happens at instrument-creation time; if the family bulk-adds several instruments in quick succession (more likely in Phase 2's bulk entry, but possible even in Phase 1 manual add), unauthenticated OpenFIGI calls can hit `429`.
**How to avoid:** Register a free OpenFIGI API key (raises the limit) even though D-14 states none is required at this volume — the "not required" and "not worth registering for" are different claims. Batch multiple lookups into a single `/v3/mapping` POST body (each request supports up to 10 jobs) rather than firing one request per instrument. On `429` or timeout, fall back to the D-15 unenriched-instrument path (never block transaction entry on this).

### Pitfall 5: NUMERIC values crossing the RSC/API boundary as JS `number`
**What goes wrong:** The Postgres driver (`@neondatabase/serverless`, like `node-postgres`) returns `NUMERIC` columns as JS **strings**, specifically to preserve precision [CITED: multiple Drizzle GitHub issues confirming this is intended behavior, not a bug]. If a server action or route handler does `JSON.stringify` after coercing that string through `Number(...)` — even implicitly, e.g. via a template literal or a naive serialization helper — precision is lost the moment it crosses into a JS float, and the resulting JSON payload the browser receives is already wrong before any `decimal.js` code runs client-side.
**How to avoid:** Keep every monetary/quantity/rate value as a string from the Postgres driver through the server action, through the JSON boundary, to the browser. Only construct a `Decimal` from that string for display formatting on the client; never round-trip through `Number`.
**Warning signs:** A displayed quantity or price that's off by a tiny fraction versus what was actually entered — the exact signature described in `PITFALLS.md` Pitfall 1, but occurring at the network-serialization boundary rather than the arithmetic layer.

## Code Examples

### Better Auth server instance (session config + allowlist + password-reset email)
```typescript
// Source: synthesized from Better Auth docs [CITED: better-auth.com/docs/authentication/email-password,
// better-auth.com/docs/concepts/session-management, better-auth.com/docs/concepts/email]
import { betterAuth } from 'better-auth';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { db } from '@/lib/db/client';
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

export const auth = betterAuth({
  database: drizzleAdapter(db, { provider: 'pg' }),
  emailAndPassword: {
    enabled: true,
    sendResetPassword: async ({ user, url }) => {
      // fire-and-forget per Better Auth's own timing-attack guidance;
      // on Vercel, wrap in waitUntil so the function doesn't exit before send completes
      await resend.emails.send({
        from: 'Financial Freedom <noreply@yourdomain.com>',
        to: user.email,
        subject: 'Reset your password',
        html: `<a href="${url}">Reset password</a>`, // replace with a React Email template
      });
    },
  },
  session: {
    expiresIn: 60 * 60 * 24 * 30, // 30 days, sliding (D-10)
    updateAge: 60 * 60 * 24,
  },
  user: {
    additionalFields: {
      mustChangePassword: { type: 'boolean', required: false, defaultValue: false, input: false },
      locale: { type: 'string', required: false, defaultValue: 'en', input: false },
    },
  },
  rateLimit: {
    // Better Auth's own defaults already give /sign-in 3 req/10s in production
    // [CITED: better-auth.com/docs/reference/security] — no override needed unless the default proves wrong
  },
});
```

### OpenFIGI enrichment call
```typescript
// Source: request/response shape per openfigi.com/api/documentation [VERIFIED: openfigi.com/api/documentation]
async function enrichByIsin(isin: string) {
  const res = await fetch('https://api.openfigi.com/v3/mapping', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' /* OPENFIGI-APIKEY header optional */ },
    body: JSON.stringify([{ idType: 'ID_ISIN', idValue: isin }]),
  });
  const [result] = await res.json();
  return result?.data?.[0]; // { figi, name, exchCode, ticker, ... } or undefined if unmapped
}
```

### Void + reissue correction (D-17)
```typescript
// Source: pattern per ARCHITECTURE.md:180-181, this session's SQL translation
await withUserScope(userId, async (tx) => {
  await tx.insert(transactions).values({
    ...originalRow,
    id: crypto.randomUUID(),
    voidsTransactionId: originalRow.id,
    quantity: '0', // the voiding row itself carries no new economic fact
    createdAt: new Date(),
  });
  await tx.insert(transactions).values({
    ...correctedFields,
    id: crypto.randomUUID(),
    createdAt: new Date(),
  });
  // originalRow is never UPDATEd or DELETEd
});
```

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|----------------|
| A1 | Better Auth's `databaseHooks.user.create.before` return-shape (`return { data: user }` to allow, throw to reject) is the current API surface | Pattern 2 | If the exact hook signature has changed in 1.6.x, the allowlist backstop silently fails to compile or fails to block — verify against `node_modules/better-auth`'s actual TypeScript types at implementation time before trusting this snippet verbatim |
| A2 | Better Auth's `rateLimit` defaults (100 req/60s global, 3 req/10s on `/sign-in`) are still the shipped defaults in 1.6.29 specifically (verified only via WebSearch summary of docs, not a direct fetch of the exact 1.6.29 changelog) | Code Examples, Security Domain | Low risk — even if defaults changed, this is a config-review item, not a security hole, since rate limiting exists in some form |
| A3 | `set_config(..., true)` scoped inside `db.transaction()` is fully compatible with `@neondatabase/serverless`'s driver behavior specifically (verified for Postgres/PgBouncer generally and for Drizzle's documented RLS pattern, but not hands-on tested against this exact driver+Neon pooler combination in this session) | Pattern 1, Pitfall 1 | If the serverless driver batches/pipelines statements in a way that breaks transaction boundaries, the RLS wiring could silently fail; the Phase 1 plan must include the direct database-level RLS test (success criterion 4) specifically to catch this before it ships |

**If this table is empty:** N/A — see above; none of these are compliance/retention/security-standard claims, all are implementation-detail risks with a concrete verification path (the RLS DB-level test itself is the check for A3).

## Open Questions

1. **Does `positions_snapshot` need to exist in migration 001, even empty?**
   - What we know: D-01's table list for migration 001 does not include `positions_snapshot` (valuation is Phase 3 scope); LEDG-07 ("positions derived from ledger, rebuildable") is nonetheless in Phase 1's requirement list.
   - What's unclear: whether LEDG-07 is satisfied in Phase 1 by a live query over `lots`/`lot_closures` (no stored snapshot needed yet, since there's no price data to value anything against) or whether the planner should create an empty `positions_snapshot` table now for forward-compatibility.
   - Recommendation: satisfy LEDG-07 with a computed query (e.g., `SELECT instrument_id, SUM(quantity_remaining) FROM lots WHERE account_id = ... GROUP BY instrument_id`) — this is genuinely "derived and rebuildable," costs no schema, and doesn't contradict D-01. Flag this reasoning explicitly in the plan so it isn't mistaken for an oversight.

2. **Should the App Router structure anticipate a future `[locale]` segment now, even though i18n is explicitly Phase 4 scope?**
   - What we know: CONTEXT.md's out-of-scope list explicitly defers i18n to Phase 4. Next.js 16 also renamed `middleware.ts` to `proxy.ts` (internals unchanged) [CITED: next-intl docs / migration guides] — a fact relevant to *whichever* phase adds next-intl's middleware-based locale routing.
   - What's unclear: whether laying out `app/(dashboard)/...` now versus `app/[locale]/(dashboard)/...` from the start meaningfully changes Phase 4's retrofit cost.
   - Recommendation: build Phase 1's routes without a `[locale]` segment, per CONTEXT.md's explicit scope boundary — introducing an unused locale segment now is speculative complexity for a phase that has no locale switcher to test it with. Note this as a known, bounded piece of Phase 4 rework (moving route folders under `[locale]/`), not a Phase 1 gap.

3. **Exact shape of `databaseHooks.user.create.before`'s return contract in Better Auth 1.6.29** — see Assumption A1. Verify against the installed package's TypeScript types at implementation time rather than trusting the WebSearch-summarized snippet verbatim.

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Neon Postgres (Vercel Marketplace) | All of Phase 1 — the entire schema | Must be provisioned before migration 001 runs | Postgres 16/17 (Neon-managed) | None — this is a hard blocker; the phase cannot start without it |
| Resend account + verified sending domain | AUTH-02 | Not yet provisioned (greenfield repo) | Free tier: 3,000 emails/mo, 100/day, 1 domain [VERIFIED: resend.com/docs/knowledge-base/account-quotas-and-limits] | None needed — free tier volume (a handful of resets/year for 4 users) is trivially within limits |
| Finnhub API key (free tier) | INST-03 | Not yet provisioned | 60 calls/min free tier (per prior project research, `STACK.md`) | None needed at this phase's volume (one US stock, manual add) |
| OpenFIGI (no key strictly required) | INST-03/INST-01 enrichment | Available unauthenticated | 25 req/min, 10 jobs/request unauthenticated [VERIFIED: openfigi.com/api/documentation] | Register a free key to raise the limit if bulk-add volume in later phases makes 25/min tight (see Pitfall 4) |
| TCMB EVDS API key | D-05 one-shot FX backfill | Not yet provisioned | Free, registration-based | None — required for D-05's backfill script; the script cannot run without it, but nothing else in Phase 1 blocks on it |
| Vercel project | Hosting/deploy | Already connected (`PROJECT.md:113`) | Pro plan | — |

**Missing dependencies with no fallback:**
- Neon Postgres provisioning must happen before any other Phase 1 work — this is Wave 0 of any plan for this phase.

**Missing dependencies with fallback:**
- Finnhub/OpenFIGI/TCMB EVDS/Resend accounts all need to be created during Phase 1 setup, but none blocks the schema work itself; they block only the specific features that call them (search/enrich, backfill script, reset email).

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest (not yet installed — greenfield repo) |
| Config file | none yet — see Wave 0 |
| Quick run command | `npx vitest run` |
| Full suite command | `npx vitest run --coverage` |

### Phase Requirements → Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|---------------------|-------------|
| AUTH-05 | RLS: second family member's session returns zero rows for first user's data | integration (real Postgres, two simulated sessions) | `npx vitest run tests/db/rls-isolation.test.ts` | ❌ Wave 0 |
| AUTH-04 | Signup route does not exist / allowlist hook rejects non-listed email | integration | `npx vitest run tests/auth/allowlist.test.ts` | ❌ Wave 0 |
| AUTH-01/03 | Sign-in succeeds for allowlisted user, session cookie set with correct `expiresIn` | integration (hits Better Auth route handler directly) | `npx vitest run tests/auth/sign-in.test.ts` | ❌ Wave 0 |
| AUTH-02 | `sendResetPassword` invoked with correct user/url on `forgetPassword` call | integration (Resend client mocked) | `npx vitest run tests/auth/reset-password.test.ts` | ❌ Wave 0 |
| LEDG-06/D-17 | Editing a transaction inserts two new rows; original row is unchanged and still queryable | integration | `npx vitest run tests/ledger/correction.test.ts` | ❌ Wave 0 |
| LEDG-09/D-07 | No column matching money/price/quantity/rate/amount/basis/value patterns has type `real`/`double precision`/`float` | schema-introspection unit test against `information_schema.columns` | `npx vitest run tests/db/money-types.test.ts` | ❌ Wave 0 |
| LEDG-09 | A decimal.js value round-trips through the Postgres driver (insert → select) with zero precision loss | unit/integration | `npx vitest run tests/db/decimal-roundtrip.test.ts` | ❌ Wave 0 |
| INST-01 | `unique(exchange_mic, symbol)` and `unique(isin)` (partial, where not null) constraints reject duplicates | integration | `npx vitest run tests/db/instruments.test.ts` | ❌ Wave 0 |
| LEDG-01 | A buy transaction with fees, in USD, produces exactly one `transactions` row and one `lots` row with correct decimal values | integration | `npx vitest run tests/ledger/add-transaction.test.ts` | ❌ Wave 0 |

### Sampling Rate
- **Per task commit:** `npx vitest run <affected-file>`
- **Per wave merge:** `npx vitest run` (full suite)
- **Phase gate:** Full suite green before `/gsd-verify-work`, plus the manual UAT of success criteria 1-5 from ROADMAP.md (sign-in/session persistence across a real browser refresh is not meaningfully automatable at this phase's scale and is better verified by hand per criterion 1)

### Wave 0 Gaps
- [ ] `npm install -D vitest` — no test framework installed yet
- [ ] `vitest.config.ts` — needs a `.env.test` pointing at a real (or Neon branch) test database, since RLS tests require a genuine Postgres instance, not a mock
- [ ] `tests/setup.ts` — shared fixtures: seed 2+ test users via the setup-script pattern, helper to open a `withUserScope`-style session for test assertions
- [ ] `tests/db/rls-isolation.test.ts` — the single most important test in this phase; must open two separate transactions with two different `app.current_user_id` values and assert zero cross-visibility at the raw SQL level, not through application code
- Consider a Neon database branch per test run (Neon's instant-branching feature) rather than a shared test database, to avoid RLS test pollution across parallel CI runs — not required for Phase 1 to ship, but worth scoping if CI parallelism becomes relevant later

## Security Domain

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|--------------------|
| V2 Authentication | yes | Better Auth `emailAndPassword` — password hashing, credential storage, and the closed-allowlist backstop are all Better Auth/app-layer, never hand-rolled |
| V3 Session Management | yes | Better Auth database session, `httpOnly`/`secure`/`SameSite=Lax` cookie (D-10), 30-day sliding expiry, session invalidation on password change |
| V4 Access Control | yes | Two-layer: Postgres RLS (primary, DB-enforced) + server action/route handler scoping (defense in depth) — never application-layer-only, per `PITFALLS.md` Pitfall 12 |
| V5 Input Validation | yes | Zod schemas on every server action/route handler input (transaction amounts, portfolio names, share grants) — reject before touching the DB, not after |
| V6 Cryptography | yes | Password hashing is entirely Better Auth's responsibility (never hand-rolled); TLS is Vercel/Neon's default — no custom crypto code in this phase at all |

### Known Threat Patterns for this stack

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|----------------------|
| SQL injection via raw string interpolation in a hand-written RLS-policy migration or a `tx.execute(sql\`...\`)` call | Tampering | Drizzle's `sql` template tag parameterizes values automatically when passed as template interpolations (as in the `set_config` example above) — never string-concatenate a user-supplied value into a raw SQL string |
| Cross-tenant data leak via a missed scoping filter or a leaked pooled-connection session variable | Information Disclosure / Elevation of Privilege | RLS as the DB-level backstop (Pattern 1); the direct database-level test in Validation Architecture above is the required proof, not just app-layer code review |
| Credential stuffing / brute force against the closed allowlist's 4 known emails | Spoofing | Better Auth's built-in rate limiter (3 req/10s default on `/sign-in`, `/change-password`) [CITED: better-auth.com/docs/reference/security] — confirm this default is actually active in 1.6.29 rather than assuming (see Assumption A2) |
| Password-reset token leakage via URL logging or referrer headers | Information Disclosure | Better Auth's reset tokens are single-use and time-boxed by default; never log the full reset URL server-side, and ensure the reset page doesn't trigger outbound requests (analytics, etc.) that could leak the token in a Referer header |
| CSRF against server actions / the Better Auth route handler | Tampering | Next.js Server Actions include built-in Origin-header verification; Better Auth's route handler additionally validates its own CSRF token on state-changing requests — no custom CSRF middleware needed for this phase |

## Sources

### Primary (HIGH confidence)
- [Better Auth — Session Management](https://better-auth.com/docs/concepts/session-management) — session `expiresIn`/`updateAge` config
- [Better Auth — Drizzle ORM Adapter](https://better-auth.com/docs/adapters/drizzle) — `additionalFields`, schema generation CLI
- [Better Auth — Security reference](https://better-auth.com/docs/reference/security) — built-in rate limiting defaults
- [Neon Docs — Connection pooling](https://neon.com/docs/connect/connection-pooling) — pooled connection SET/RESET unsupported (WebFetch-verified this session)
- [Neon Docs — Simplify RLS with Drizzle](https://neon.com/docs/guides/rls-drizzle) — `crudPolicy`/`authUid`/`neon_auth` scope and its incompatibility with non-Neon-Auth session models (WebFetch-verified this session)
- [OpenFIGI API Documentation](https://www.openfigi.com/api/documentation) — unauthenticated rate limit (25/min, 10 jobs/request), request/response shape (WebFetch-verified this session)
- npm registry (`npm view <pkg> version`) — all package versions in Standard Stack table, verified live this session

### Secondary (MEDIUM confidence)
- Better Auth GitHub issue #4211 — `fieldName` remapping bug in the Drizzle adapter (WebSearch, cross-referenced against the official repo)
- Better Auth GitHub issue #2324 — "Support for Forcing Password Change on next login" open feature request, confirming no native support (WebSearch, cross-referenced against the official repo)
- [Resend — Account quotas and limits](https://resend.com/docs/knowledge-base/account-quotas-and-limits) — free tier figures (WebSearch summary of official docs page)
- TCMB EVDS API usage guides (multiple third-party sources, cross-checked) — `evds2.tcmb.gov.tr/service/evds/` base URL, `key` HTTP header auth, `TP.DK.USD.A` series code

### Tertiary (LOW confidence)
- None used for load-bearing claims in this document — all technical claims above trace to Primary or Secondary sources; see Assumptions Log for the specific residual-risk items.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — every version verified live against the npm registry this session; peer-dependency compatibility (Better Auth ↔ Drizzle ↔ Next.js) cross-checked
- Architecture: HIGH on the schema itself (inherited from prior project research, unchanged), MEDIUM on the exact RLS wiring mechanics (verified via docs, not hands-on tested in this repo yet — the Phase 1 RLS integration test is the actual verification)
- Pitfalls: HIGH — the two most consequential findings this session (Neon pooled-connection SET/RESET limitation, Better Auth's missing forced-password-change) were both confirmed directly against official docs/issue trackers, not inferred

**Research date:** 2026-08-16
**Valid until:** 30 days (stable, well-documented libraries; re-verify package versions if planning is delayed past early-to-mid September 2026 given Better Auth/Next.js/Resend's active release cadence observed this session)
