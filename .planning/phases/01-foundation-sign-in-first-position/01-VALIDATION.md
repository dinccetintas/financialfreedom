---
phase: 1
slug: foundation-sign-in-first-position
# status lifecycle: draft (seeded by plan-phase) → validated (set by validate-phase §6)
# audit-milestone §5.5 distinguishes NOT-VALIDATED (draft) from PARTIAL (validated + nyquist_compliant: false) (#2117)
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-08-16
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.
> Seeded from `01-RESEARCH.md` § Validation Architecture. The Per-Task Verification Map is
> filled in once PLAN.md task IDs exist (`/gsd-validate-phase`).

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest (not yet installed — greenfield repo) |
| **Config file** | none — Wave 0 installs (`vitest.config.ts`) |
| **Quick run command** | `npx vitest run <affected-file>` |
| **Full suite command** | `npx vitest run` |
| **Estimated runtime** | ~30–90 seconds (integration tests hit a real Postgres/Neon branch) |

**Hard constraint:** RLS isolation tests require a genuine Postgres instance, not a mock. A mocked
driver cannot prove success criterion 4 (a database-level query by a second user returns zero rows).

---

## Sampling Rate

- **After every task commit:** Run `npx vitest run <affected-file>`
- **After every plan wave:** Run `npx vitest run`
- **Before `/gsd-verify-work`:** Full suite must be green, plus manual UAT of ROADMAP success criteria 1–5
- **Max feedback latency:** 90 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| *(pending — populated from PLAN.md task IDs by `/gsd-validate-phase`)* | | | | | | | | | ⬜ pending |

**Requirement → test mapping inherited from research (task IDs to be attached):**

| Req ID | Behavior | Test Type | Automated Command | File Exists |
|--------|----------|-----------|-------------------|-------------|
| AUTH-05 | Second family member's session returns zero rows for first user's data | integration (real Postgres, two simulated sessions) | `npx vitest run tests/db/rls-isolation.test.ts` | ❌ W0 |
| AUTH-04 | Signup route absent / allowlist hook rejects non-listed email | integration | `npx vitest run tests/auth/allowlist.test.ts` | ❌ W0 |
| AUTH-01, AUTH-03 | Sign-in succeeds for allowlisted user; session cookie set with correct `expiresIn` | integration | `npx vitest run tests/auth/sign-in.test.ts` | ❌ W0 |
| AUTH-02 | `sendResetPassword` invoked with correct user/url on `forgetPassword` | integration (Resend mocked) | `npx vitest run tests/auth/reset-password.test.ts` | ❌ W0 |
| LEDG-06 | Editing a transaction inserts new rows; original unchanged and still queryable | integration | `npx vitest run tests/ledger/correction.test.ts` | ❌ W0 |
| LEDG-09 | No money/price/quantity/rate/amount column has type `real`/`double precision`/`float` | schema introspection against `information_schema.columns` | `npx vitest run tests/db/money-types.test.ts` | ❌ W0 |
| LEDG-09 | decimal.js value round-trips through the driver with zero precision loss | integration | `npx vitest run tests/db/decimal-roundtrip.test.ts` | ❌ W0 |
| INST-01 | `unique(exchange_mic, symbol)` and partial `unique(isin)` reject duplicates | integration | `npx vitest run tests/db/instruments.test.ts` | ❌ W0 |
| LEDG-01 | Buy transaction with fees in USD produces exactly one `transactions` row + one `lots` row with correct decimals | integration | `npx vitest run tests/ledger/add-transaction.test.ts` | ❌ W0 |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `npm install -D vitest` — no test framework installed yet
- [ ] `vitest.config.ts` — with `.env.test` pointing at a real Postgres (Neon branch) test database
- [ ] `tests/setup.ts` — shared fixtures: seed 2+ test users via the setup-script pattern; helper to open a `withUserScope`-style scoped transaction for assertions
- [ ] `tests/db/rls-isolation.test.ts` — the single most important test in this phase; opens two transactions with two different `app.current_user_id` values and asserts zero cross-visibility at raw SQL level, not through application code
- [ ] `tests/db/money-types.test.ts` — schema introspection guard against float columns

*Optional, not required to ship Phase 1: a Neon database branch per test run to avoid RLS test pollution across parallel CI runs.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Session persists across a real browser refresh and across devices | AUTH-03 | Real browser cookie lifecycle across two devices is not meaningfully automatable at this phase's scale | Sign in on desktop; hard-refresh; confirm still signed in. Sign in on phone with same account; confirm both sessions remain valid. |
| Password-reset email arrives and its link completes a reset | AUTH-02 | Requires a real inbox and a live Resend send | Trigger reset for an allowlisted address; open the emailed link; set a new password; sign in with it. |
| Transaction list is usable and readable at 390px width | LEDG-10 | Visual/interaction judgement | Open the transaction list on a 390px viewport; confirm filter + sort are reachable and no horizontal scroll. |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 90s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
