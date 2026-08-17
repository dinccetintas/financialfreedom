# Walking Skeleton — Financial Freedom

**Phase:** 1
**Generated:** 2026-08-16

## Capability Proven End-to-End

A seeded family member signs in with email + password, creates a named portfolio, and sees it in their portfolio list on the deployed dev environment — with the row written and read back through a Postgres RLS policy that a second family member's session cannot see.

That single path exercises: project scaffold → build → routing → Better Auth session → Drizzle over Neon → migration 001 (the full non-retrofittable core) → transaction-scoped RLS session variable → server action write → server-scoped read → UI render → Vercel dev deploy.

## Architectural Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Framework | Next.js 16.3.1, App Router only (no Pages Router) | Committed stack (`.claude/CLAUDE.md` §Technology Stack); `01-RESEARCH.md` verified 16.3.1 live on the npm registry and confirmed Better Auth 1.6.29 declares `next: ^14 \|\| ^15 \|\| ^16` |
| Data layer | Neon Postgres (Vercel Marketplace) + Drizzle ORM 0.45.2 + `@neondatabase/serverless` 1.1.0 | Committed stack; Better Auth 1.6.29 pins `drizzle-orm: ^0.45.2` exactly, so the versions are chosen together, not independently |
| Two connection strings | `DATABASE_URL` (pooled, app runtime) and `DATABASE_URL_DIRECT` (unpooled, migrations only) | Neon documents that `drizzle-kit migrate` must run over a direct connection; running it over the `-pooler` string is an anti-pattern recorded in `01-RESEARCH.md` §Anti-Patterns |
| Tenant isolation | Postgres RLS as the enforcement layer; `set_config('app.current_user_id', $id, true)` as the first statement inside `db.transaction()` | Neon's pooled string runs PgBouncer in transaction mode and does not support session-scoped `SET`/`RESET`. `is_local = true` scope ends at COMMIT, so it cannot leak onto a recycled pooled connection. App-layer `WHERE` filtering alone cannot satisfy ROADMAP success criterion 4 |
| Identity table | Better Auth's generated `user` table IS D-01's `users` table, extended via `additionalFields` | Creating a second `users` table alongside Better Auth's generated one produces two identity sources and FK ambiguity for `portfolios.owner_user_id` (`01-RESEARCH.md` Pattern 4 / Pitfall 3). Do **not** hand-author a `users` table |
| Auth | Better Auth 1.6.29, email + password, database-backed session, 30-day sliding cookie (`httpOnly` / `secure` / `SameSite=Lax`) | D-10. No step-up auth on writes — it would collide with Phase 2's fast bulk entry |
| Registration | No sign-up page ships; `emailAndPassword.disableSignUp: true`; `databaseHooks.user.create.before` allowlist backstop; four users created by `scripts/seed-family-users.ts` | D-08 + AUTH-04's literal reading ("no public registration surface exists") |
| Migration authoring | Drizzle schema (TypeScript) → `drizzle-kit generate` → hand-written RLS policies, `CHECK` constraints and seed rows appended as raw SQL **in the same numbered migration file** | D-06. Drizzle's schema DSL cannot express `CREATE POLICY`. This authoring pattern is the precedent every later migration in the project follows |
| Money arithmetic | Postgres `NUMERIC` in the DB, `decimal.js` 10.6.0 in the app, string across every serialization boundary | LEDG-09 + `.claude/CLAUDE.md` "What NOT to Use". The driver returns `NUMERIC` as a JS string on purpose; coercing through `Number` at the RSC/JSON boundary silently loses precision before any `decimal.js` code runs |
| Validation | Zod 4.4.3 on every server action and route handler input | ASVS V5, `01-RESEARCH.md` §Security Domain |
| UI system | shadcn, `style: new-york`, `baseColor: neutral`, `cssVariables: true`; Radix primitives; lucide-react; Inter variable font | `01-UI-SPEC.md` §Design System. Preset fixed at plan time precisely because `npx shadcn init` is the first UI-touching command in a repo with no `package.json` |
| Deployment target | Vercel (already connected per `PROJECT.md:113`), preview deploy for the skeleton | Committed stack |
| Directory layout | `src/app/(auth)` · `src/app/(dashboard)` · `src/lib/{auth,db,ledger,instruments,money,fx,portfolios}` · `src/components` · `scripts/` · `tests/` | `01-RESEARCH.md` §Recommended Project Structure. No `[locale]` route segment in Phase 1 — i18n is Phase 4 scope, and an unused locale segment is speculative complexity |
| Test runner | Vitest against a real Neon Postgres branch (`.env.test`), never a mocked driver | RLS isolation cannot be proven against a mock — the proof must be a raw SQL query under a second session |

## Stack Touched in Phase 1

- [ ] Project scaffold — Next.js 16 + TypeScript + App Router + Tailwind + shadcn init + ESLint + Vitest
- [ ] Routing — `/sign-in` (public), `/portfolios` (session-gated), `/api/auth/[...all]` (Better Auth catch-all)
- [ ] Database — migration 001 applied to Neon; one real write (`INSERT` into `portfolios` through `withUserScope`) and one real read (server-scoped `SELECT` rendering the list)
- [ ] UI — the "Create portfolio" form submits to a server action and the new row appears in the list without a manual reload
- [ ] Deployment — Vercel preview deploy with `DATABASE_URL` / `DATABASE_URL_DIRECT` / `BETTER_AUTH_SECRET` set, plus a documented `npm run dev` full-stack local command

## Out of Scope (Deferred to Later Slices)

Explicit so later phases do not re-litigate Phase 1's minimalism:

- Sell / dividend / deposit / withdrawal / fee / FX-convert transaction types and FIFO lot consumption — Phase 2
- Bulk transaction entry — Phase 2
- Scheduled EOD price sync, backfill jobs, valuation and current market value — Phase 3
- Turkish localization, Simple/Pro mode, the project-wide design system and the per-page design-prompt convention — Phase 4
- Applying corporate actions (splits, bonus/rights issues) and read-time multi-currency conversion — Phase 5. The `corporate_actions` and `corporate_action_applications` tables are **created empty and correctly typed** in migration 001 (D-01); no application logic ships in Phase 1
- Scheduled/recurring TCMB EVDS sync and gold rates — Phase 5. Phase 1 ships a one-shot backfill **script**, not a job (D-05)
- Write-permission portfolio sharing — the `CHECK (permission IN ('read','write'))` constraint exists (D-12) but the Phase 1 UI only ever grants `'read'`
- Multi-account UI — `accounts` is populated with one hidden default per portfolio (D-16); the broker-account grouping surface is Phase 2
- A backfill job for instruments marked unenriched (D-15) — Phase 3
- A registration route for a fifth user (D-08) — additive if the tool ever leaves the family
- `positions_snapshot` and `portfolio_daily_returns` tables — not in D-01's list; LEDG-07 is satisfied in Phase 1 by a live derived query over `lots` (see the flagged assumption in `01-08-PLAN.md`)
- Dark mode — shadcn's default dark CSS variables are left un-customized, neither built nor explicitly disabled (`01-UI-SPEC.md` §Color)

## Subsequent Slice Plan

Each later phase adds one vertical slice on top of this skeleton without altering its architectural decisions:

- Phase 2: every remaining transaction type + FIFO lot consumption + fast bulk entry
- Phase 3: BIST EOD provider spike, scheduled price sync, first USD position value
- Phase 4: Turkish/English, Simple/Pro mode, 390px audit, design system
- Phase 5: multi-currency conversion, corporate actions, portfolio-wide valuation
- Phase 6: TWR / XIRR / FX-attribution returns engine
- Phase 7: exposure, risk, KAP-sourced BIST fundamentals
- Phase 8–9: thesis, decision rules, rule evaluation, Telegram alerts, attention-first home
- Phase 10: news, macro, daily brief
- Phase 11–13: agent read access, then write access with audit + reversal, then deep research + code execution
- Phase 14: discovery
