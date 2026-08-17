# Phase 1: Approved Package Set — Supply Chain Decisions

**Decided:** 2026-08-17
**Checkpoint outcome:** Task 1 of `01-01-PLAN.md` (`checkpoint:human-verify`, `gate="blocking-human"`) was answered by the developer with **"approved"** — no package swaps, no repins requested. The four `[SUS] too-new` packages were verified live against the npm registry before approval.

This file is the durable record every later plan's executor reads before running its first `npm install`, because each plan runs in a separate context window and cannot see the Task 1 conversation.

## Approved package set

| Package | Pinned version | Audit verdict | Status | Notes |
|---|---|---|---|---|
| next@16.3.1 | 16.3.1 | SUS (too-new) | APPROVED | Verified 2026-08-17: repository field points at github.com/vercel/next.js, weekly downloads 45,881,963. Flag is publish-recency only, not a hallucination signal. |
| better-auth@1.6.29 | 1.6.29 | SUS (too-new) | APPROVED | Verified 2026-08-17: repository field points at github.com/better-auth/better-auth, weekly downloads 4,675,506. |
| resend@6.20.0 | 6.20.0 | SUS (too-new) | APPROVED | Verified 2026-08-17: repository field points at github.com/resend/resend-node, weekly downloads 6,962,515. |
| react-email@6.9.2 | 6.9.2 | SUS (too-new) | APPROVED | Verified 2026-08-17: repository field points at github.com/resend/react-email, weekly downloads 3,073,073. NOT marked deprecated — use this unified package directly, never `@react-email/components`. |
| drizzle-orm@0.45.2 | 0.45.2 | OK | APPROVED | Matches Better Auth's declared peer dependency `^0.45.2`. |
| drizzle-kit@0.31.10 | 0.31.10 | OK | APPROVED | Companion to drizzle-orm; satisfies Better Auth's declared peer requirement `>=0.31.4`. |
| @neondatabase/serverless@1.1.0 | 1.1.0 | OK | APPROVED | HTTP/WebSocket Postgres driver for Vercel Functions. |
| decimal.js@10.6.0 | 10.6.0 | OK | APPROVED | Arbitrary-precision decimal arithmetic — non-negotiable per LEDG-09 and `.claude/CLAUDE.md`. |
| zod@4.4.3 | 4.4.3 | OK | APPROVED | Input validation on every server action/route handler. |
| vitest | (latest at install time — dev dependency, no exact pin required by 01-RESEARCH.md) | OK | APPROVED | Test framework, installed as a dev dependency (`npm install -D vitest`). |
| @react-email/components | (none — never install) | SUS (deprecated) | REMOVED | Deprecated on npm ("Package no longer supported"). All components and render utilities were folded into the unified `react-email` package. Use `react-email@6.9.2` instead. Do not reintroduce this package in any later plan. |

## Installation rule

Every later plan installs packages from the table above rather than re-deriving names or versions from prose elsewhere in `01-RESEARCH.md` or `01-CONTEXT.md`. Every version is a concrete pin — never `latest` — except `vitest`, which `01-RESEARCH.md` §Standard Stack lists without an exact pin as a dev-only test dependency. Adding any package to the project that is not listed in this table requires a new legitimacy check (registry page, official repository URL, download volume, deprecation status) and, if it is flagged `[SUS]` or `[SLOP]`, a fresh `checkpoint:human-verify` before install — this approval does not extend to packages introduced later that were never reviewed here.
