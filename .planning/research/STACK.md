# Stack Research

**Domain:** Private multi-currency family investment portfolio management platform (BIST + US equities/ETFs + gold + FX + cash) with an embedded Claude AI agent
**Researched:** 2026-08-16
**Confidence:** MEDIUM-HIGH overall — HIGH on framework/database/AI-layer/financial-math choices, MEDIUM on exact market-data pricing tiers (provider pricing pages returned inconsistent numbers across sources and some blocked automated fetches), LOW-MEDIUM specifically on BIST fundamentals coverage, which is the single largest technical risk in this project and needs a hands-on spike before committing budget.

---

## 1. Market Data — the highest-risk decision

**Bottom line: a workable combination exists comfortably under budget, but BIST fundamentals coverage from any paid API is unverified and must be spiked (free trial signup + test query) before you build against it.** Nothing below claims certainty on that point where certainty isn't earned.

### Recommended combination (concrete, priced, fits budget)

| Source | Purpose | Cost | Confidence |
|---|---|---|---|
| **EODHD — EOD Historical Data, All World** | US + international end-of-day prices, splits, dividends. Ticker suffix `.US` for US, `.IS` for Istanbul (exchange code `IS`, MIC `XIST`) | **$19.99/mo** ($16.58/mo billed annually) | MEDIUM — US coverage confirmed; BIST price coverage well-evidenced (exchange code exists in their system) but not hands-on verified. **Spike: sign up, pull `GARAN.IS` and `THYAO.IS` EOD history before committing.** |
| **TCMB EVDS (Central Bank of the Republic of Türkiye)** | Historical USD/TRY daily rates + Istanbul Gold Exchange (TRY/USD gold) data — needed for cost-basis FX conversion on every past transaction | **$0** (free, register for an API key at evds3.tcmb.gov.tr) | HIGH — official central bank source, this is the correct authoritative source for historical TRY rates, not a third party's derived number |
| **Finnhub** free tier | US company profiles, basic fundamentals, company news, dividend calendar | **$0** (60 calls/min free tier — generous for ~17 US positions × 4 users) | HIGH — well-documented, widely used, free tier confirmed generous |
| **Claude's native `web_search` tool** (used by the embedded agent) | Turkish + international news synthesis, KAP disclosure lookups, analyst sentiment, "what happened and why" for the daily brief | **~$10 per 1,000 searches** (billed as part of agent usage) | HIGH for the mechanism, MEDIUM for actual monthly cost — depends on brief frequency; at 20–50 searches/day this is roughly $6–15/month |
| **borsapy** (Python) or direct KAP review via the agent | BIST fundamentals fallback where EODHD doesn't deliver acceptable quality | **$0** | LOW-MEDIUM — see BIST section below |

**Estimated total market-data cost: ~$20–35/month**, leaving $85–115/month of the stated $50–150 budget for hosting, database, and buffer — including headroom to add EODHD's Fundamentals tier ($59.99/mo) if the BIST spike confirms good coverage, and still land near the top of budget rather than over it.

### What is NOT obtainable at this budget — be explicit about this with the team

- **Real-time or licensed BIST streaming quotes.** A Borsa İstanbul Data Distribution Agreement plus a licensed vendor feed (Foreks, Matriks, İş Yatırım terminal data) runs into the hundreds of USD/month and requires a signed agreement with Borsa İstanbul. This is explicitly out of scope per PROJECT.md already — confirmed correct decision.
- **A guaranteed, complete BIST fundamentals API at this price.** No provider researched offers confirmed, verified BIST fundamentals coverage at a hobbyist price point. EODHD is the best candidate but unverified; Twelve Data and FMP require their most expensive tiers for non-US markets and neither confirms BIST fundamentals depth.
- **An official KAP API or RSS feed.** None exists. KAP (kap.org.tr) is a web platform with no public developer API or RSS feed documented anywhere found in this research.
- **An official TEFAS API.** None exists — TEFAS is a web platform only.
- **A single vendor covering both US and BIST fundamentals with confirmed completeness.** This is why the recommendation is a combination, with a documented fallback (agent-assisted web research) rather than a single "buy this and you're done" answer.

### US equities/ETFs — provider comparison

| Provider | Free tier | Cheapest paid tier | What you get | Verdict |
|---|---|---|---|---|
| **EODHD** | EOD, splits, dividends, limited calls | $19.99/mo (All World EOD) | Broad global EOD coverage, 70+ exchanges, official Python/JS client | **Recommended** — best price/coverage ratio for EOD data across both US and non-US markets in one subscription |
| **Twelve Data** | US equities/forex/crypto free (3 exchanges) | $29/mo (Grow — 20+ markets incl. Turkey per their own exchange list) | Real-time-ish quotes, credit-based rate limiting | Viable BIST alternative to EODHD at similar price — worth a side-by-side spike since Twelve Data explicitly lists Istanbul as covered under Grow, whereas EODHD's BIST inclusion is inferred rather than confirmed in this research |
| **Financial Modeling Prep** | 250 calls/day, US-only, EOD | Tiered Starter → Ultimate (exact current $ not verifiable from public docs during this research — pricing page blocked automated fetch; **verify directly before committing**) | Free tier is US-only; global/non-US coverage requires a paid tier, unclear if BIST is included at any tier | LOW-MEDIUM confidence on exact pricing — do not budget against unconfirmed numbers |
| **Finnhub** | 60 calls/min, US real-time quotes + company news | $49.99–$200/mo depending on data scope | Best free tier of any provider researched for US data + news | **Recommended for the free tier specifically** — use for US news/fundamentals without paying anything |
| **Tiingo** | 1,000 req/day, EOD US data | $10/mo (long-horizon EOD + fundamentals) | Clean, cost-effective EOD history for backtesting | Reasonable EODHD alternative for US-only if BIST is sourced separately |
| **Alpha Vantage** | 25 req/day (very restrictive) | $49.99+/mo | 50+ technical indicators | Not recommended — free tier too thin for daily sync of ~70 tickers |
| **Polygon.io** | 5 req/min | $99+/mo | High-quality real-time data, overkill for buy-and-hold EOD needs | Not recommended — priced for trading use cases this project doesn't have |

### BIST — the honest answer, provider by provider

| Provider | Covers BIST prices? | Covers BIST fundamentals? | Licensing / cost | Verdict |
|---|---|---|---|---|
| **EODHD** | Likely yes (exchange code `IS`, MIC `XIST` documented in their system) but not hands-on confirmed in this research | Unconfirmed — Fundamentals API is generically described for "70+ exchanges" but no source confirms depth for BIST specifically | $19.99/mo EOD, +$59.99/mo for Fundamentals | **Best bet, but spike before building** |
| **Twelve Data** | Yes — Borsa İstanbul explicitly listed as Level A / included under the Grow plan | Uncertain for fundamentals depth (their fundamentals endpoints are generally thinner than EODHD's) | $29/mo (Grow) | Good BIST price alternative; treat as a fallback if EODHD's BIST spike fails |
| **Financial Modeling Prep** | Free tier is US-only; unclear which paid tier (if any) includes BIST | Unconfirmed | Unclear — pricing page inaccessible during research | Do not budget against this until pricing/coverage is confirmed |
| **İş Yatırım** | Yes, via their retail brokerage screener/API (filters 40+ criteria, futures/options) | Available for retail account holders, not a public self-serve data API for third-party apps | Free with a brokerage account, but not designed as a general-purpose data API | If any family member already has an İş Yatırım account, worth investigating as a supplementary source, but not a primary integration target |
| **Foreks** | Provides BIST index/price data commercially | Unclear self-serve pricing (enterprise sales-driven, no public price list found) | Contact-sales, likely hundreds/month | Not viable at this budget |
| **Matriks** | Yes, professional terminal | Yes, professional terminal | ₺350–800+/month per data package (retail terminal pricing found) plus API contract required for programmatic access | Not viable at this budget for API access — the terminal itself is affordable but is a GUI product, not an API |
| **Fintables** | Yes (BIST-native, the strongest Turkish reference product per PROJECT.md's own competitive research) | Yes, extensive (their core value proposition) | API access is only offered via **corporate membership** with no public pricing; Trade/Pro/Evo tiers can additionally purchase a Borsa İstanbul data-distribution license as an add-on | **Not recommended for API integration** — no self-serve pricing exists; likely priced for businesses, not a $50–150/mo hobbyist budget. Worth a direct sales inquiry if EODHD/Twelve Data both fail the spike, but do not plan around it. |
| **borsapy** (open-source Python library, `pip install borsapy`) | Yes — screener with 40+ filter criteria, OHLCV history, sector/index data | Yes — balance sheets, P/E, PD/DD, ROE, dividend yield, foreign-ownership ratio | **Free**, but its own license states **personal and educational use only — explicitly prohibits commercial software products/services**; commercial use requires contacting Borsa İstanbul directly for a license | **This project's use case (private, non-commercial, 4-person family app) plausibly fits "personal use" as written**, which makes this a legitimate free option — but this is a legal reading, not a guarantee, and the library is an unofficial scraper wrapping other sources, so it can break without notice. Recommend as a **fallback/supplement**, not sole source of truth, with graceful degradation if it stops working. |

**Recommended BIST approach:** Use EODHD's All World EOD plan for BIST *prices* (spike to confirm coverage in week 1 of the relevant phase). For BIST *fundamentals*, do **not** commit to a paid fundamentals tier until the spike confirms depth — start with borsapy (free, personal-use license fits this project) supplemented by the embedded Claude agent's web search for anything borsapy misses, and only upgrade to EODHD's paid Fundamentals tier ($59.99/mo) if the spike shows it's meaningfully better. Given the buy-and-hold, ~3-month-cadence investment style stated in PROJECT.md, fundamentals do not need to be real-time — a periodic (weekly/monthly) refresh via any of these sources, with the agent filling gaps on demand, is sufficient and de-risks the whole category.

### KAP (Kamuyu Aydınlatma Platformu)

**No official API or RSS feed exists.** Confirmed by searching KAP's own site, the Borsa İstanbul public disclosure platform page, and multiple third-party sources — nothing publishes a documented public API or RSS endpoint. What exists instead:

- **`pykap`** (GitHub: cemsinano/pykap) — an unofficial Python documentation/wrapper project for KAP disclosures.
- **`KAP_Notifications`** (GitHub: alperaydyn/KAP_Notifications) — a community project demonstrating that scraping/notification-bot patterns against KAP are technically viable.
- **KAP Mobile app** — official but has no public API surface for third parties.

**Scraping viability:** Technically demonstrated as possible by multiple community projects. **Legally**, KAP's terms of service were not reviewed in this research and should be checked before production use — treat this as **best-effort, not guaranteed**, and design the system so KAP disclosure ingestion can silently degrade to "agent does a targeted web search for recent KAP filings on this ticker" without breaking anything else. This is the single item in this research most worth a dedicated legal/ToS review before the relevant phase, flagged as **LOW confidence, needs spike**.

### TEFAS

**No official API.** Multiple community options exist and are low-effort to integrate if/when fund holdings become relevant (not currently a stated v1 requirement — PROJECT.md lists BIST equities, US equities/ETFs, gold, FX, cash, not mutual funds):

- `Tefas-API` (GitHub: eneshenderson/Tefas-API) — client libraries in Python, TypeScript, Go, .NET, Java.
- Apify's `TEFAS API Scraper` — hosted scraper, usage-based Apify pricing.
- Several PyPI/GitHub scrapers (`tefas-crawler`, `tefas_scraper` with an MCP server variant).

**Recommendation:** Defer. Not needed for v1 scope. If added later, the TypeScript client from `Tefas-API` is the most direct fit for this stack.

### Gold and FX — the good news

This is the most solidly answered part of the market-data question:

- **USD/TRY historical daily rates and Istanbul Gold Exchange (TRY/USD) data: use TCMB EVDS.** It is free, official, requires only a registered API key, and is the authoritative source — not a third party's derived number. This directly solves the "historical FX for cost-basis conversion on every past transaction" requirement with HIGH confidence.
- **XAU (spot gold) international pricing:** EODHD and most equity data providers also carry commodities/forex including XAU/USD as a standard forex-style symbol, at no extra cost within the plans already recommended above.

### News APIs

| Option | Fit |
|---|---|
| **Claude's native `web_search` server tool** (used directly by the embedded agent) | **Primary recommendation.** Solves "news filtered to holdings, covering international and Turkish sources including KAP" without a dedicated paid news API — the agent searches live, in the language needed, at request time, for pennies per search. This is a genuine architectural advantage of building the news feature *inside* the agent rather than as a separate pipeline. |
| **Finnhub** free tier | Company-specific news for US holdings, free, structured JSON — good for a lightweight non-agent news list view distinct from the AI-written brief |
| **Marketaux** | Entity-first filtering, has a free tier; best when ticker/industry filtering matters more than depth. Reasonable fallback/supplement if agent-driven search proves too slow or costly for the daily brief. |
| **NewsData.io / mediastack** | Both explicitly document Turkey-region news coverage (Turkish sources including Anadolu Ajansı) with free tiers | Worth a look specifically for macro/country-level Turkish news distinct from stock-specific news |

**Recommendation:** Lean on the agent's own `web_search` for the AI-written daily brief (this is the natural fit and avoids a redundant subscription), and use Finnhub's free news endpoint only for a lightweight structured "recent headlines" list if a non-AI-generated feed view is wanted somewhere in the UI.

---

## 2. Framework — Next.js on Vercel

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Next.js | **16.x** (latest 16.3+ patch) | Full-stack React framework, App Router | Current stable major version as of Aug 2026. Cache Components (opt-in, no more implicit caching surprises), Turbopack dev server (up to 76% faster cold starts, near-instant HMR), React 19.2 features, Instant Navigations. Production builds still use Webpack in 16.x (Turbopack for production is in progress) — not a blocker for this project. |
| React | 19.2 (bundled via Next.js canary channel that 16 tracks) | UI library | Ships with Next.js 16; no separate version decision needed |
| Vercel | Pro plan | Hosting, deploy, Fluid Compute functions, Cron triggers | Owner's stated preference, already connected. Confirmed the right fit **with one important architectural caveat below** on long-running agent sessions. |

### Is Next.js on Vercel the right call given scheduled jobs + long-running agent sessions + streaming?

**Yes, with a specific pattern, not a naive one.** Here is what 2026 Vercel actually supports and how to use it:

- **Function timeout limits (2026):** With Fluid Compute (the current default execution model), the default function duration is **5 minutes** across all plans. On Pro/Enterprise, **800 seconds (13.3 min) is generally available** with function-level `maxDuration` configuration; **1800 seconds (30 min) is in beta**. Requests exceeding the configured duration return `504 FUNCTION_INVOCATION_TIMEOUT`.
- **For genuinely open-ended agent runs (multi-minute deep research), use Vercel Workflows** (GA as of April 2026, part of the Vercel/AI SDK ecosystem) — a durable execution primitive purpose-built for exactly this: code that can pause, resume, and survive redeploys/crashes for minutes to months, with a `DurableAgent` class specifically for building agents that maintain state across executions. This is the correct 2026 answer to "how do you run an agent for longer than a function timeout on Vercel" — it removes the timeout ceiling entirely rather than fighting it.
- **Practical architecture for this project:**
  - **Interactive chat turns** (the common case — most agent replies, most tool calls) run inside a normal Vercel Function with SSE streaming to the client. 800 seconds of headroom on Pro comfortably covers a Claude Opus 5 turn involving a handful of tool calls, web searches, and a code-execution pass.
  - **Genuinely long multi-step "deep research" requests** (the kind PROJECT.md explicitly calls out — "run multi-step deep research with streaming progress") should be handed off to a **Vercel Workflow** (or, equivalently, an **Inngest** durable function — see §6) that runs the agent loop outside the request/response cycle, persists progress events to Postgres as it goes, and lets the client subscribe to progress via a lightweight polling or SSE endpoint reading from that persisted state. This avoids ever needing a single HTTP connection to stay open for the full duration of a long-running task.
  - **Scheduled jobs** (daily EOD price sync, daily AI brief generation) are triggered by **Vercel Cron** hitting a thin API route, which immediately hands off to **Inngest** (or a Workflow) for the actual durable, retryable, multi-step work — this is the standard 2026 pattern (Cron as trigger, not as the execution engine) and avoids ever hitting a function timeout on a job that touches 70+ tickers and multiple external APIs.

**Confidence: HIGH** on the framework choice itself; **MEDIUM-HIGH** on the exact durable-execution primitive (Vercel Workflows vs. Inngest) since Workflows is a newer product (GA ~4 months as of this research) — recommend prototyping both in the relevant phase and picking based on DX, but either solves the underlying problem.

### What NOT to use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Pages Router | Legacy pattern, no Cache Components, weaker RSC support | App Router (the only reasonable choice for a 2026 Next.js project) |
| Trying to force a single HTTP request to stay open for a multi-minute agent research task | Will hit the 800s/1800s ceiling and users on flaky mobile connections will see it fail | Vercel Workflows / Inngest durable functions + progress polling, as above |
| Vercel Cron as the execution engine for multi-step jobs | No built-in retry/step-level idempotency; a partial failure mid-sync of 70 tickers either fails the whole job or silently duplicates work | Vercel Cron as trigger only, Inngest/Workflows as the actual execution engine |

---

## 3. Database — Postgres

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| **Neon Postgres** (via Vercel Marketplace integration) | Postgres 16/17 (Neon-managed) | Primary application database | **HIGH confidence, default recommendation.** Vercel deprecated its own white-labeled "Vercel Postgres" in mid-2025 in favor of the native Neon marketplace integration — Neon is now *the* first-party Postgres path on Vercel, not a third-party bolt-on. Serverless scale-to-zero, connection pooling built for exactly this serverless-function-fan-out pattern, instant branching (useful for safely testing schema/return-math changes), and 2026 pricing dropped meaningfully (storage $1.75→$0.35/GB-month, free tier compute doubled to 100 CU-hours/month). |
| Drizzle ORM or Prisma | latest | Type-safe query builder / migrations | Either is fine; Drizzle is the lighter-weight, more SQL-transparent choice increasingly preferred for serverless Postgres + Neon specifically (less connection/cold-start overhead than Prisma's engine binary); recommend Drizzle unless the team has strong existing Prisma familiarity |

### Alternative: Supabase

**Real, viable alternative — not a lesser choice, a different tradeoff.** Supabase bundles Postgres + Auth + Storage + Realtime; Pro plan is **$25/month** (8GB database, 100GB storage, $10/mo compute credit covering one micro instance, 250GB egress). Its compute does **not** scale to zero (you're paying for a running instance continuously), unlike Neon.

- **Choose Supabase if:** you want Supabase Auth bundled (rather than a separate Better Auth setup) and value Postgres Row-Level Security enforced automatically via their Auth+PostgREST integration for defense-in-depth on multi-tenant isolation.
- **Choose Neon (recommended default) if:** the application enforces tenant/user isolation at the application layer in Next.js server-side code (which this project's architecture — API routes, not direct-from-client PostgREST queries — naturally does anyway), and you want the cheaper, scale-to-zero serverless cost profile that better matches a 4-user app with bursty, low-continuous-load usage.

Note: **Postgres Row-Level Security is a native Postgres feature, not a Supabase exclusive** — it can be enabled on Neon too, if you want DB-layer isolation as defense-in-depth regardless of which provider you pick. Given PROJECT.md's explicit "no hardcoded assumptions" multi-tenancy requirement, enabling RLS on the `portfolios`/`transactions`/`positions` tables scoped by `user_id`/`family_id` is a worthwhile HIGH-value addition on either provider, independent of the provider decision.

### Should price history live in Postgres or a separate time-series store?

**Plain Postgres. No TimescaleDB, no separate time-series database, at this scale.** The math: 4 users × ~17 positions ≈ 70 unique symbols, daily bars, several years of history ≈ 70 × ~1,300 trading days ≈ **~91,000 rows** in a `price_history` table. This is trivially small for Postgres with a standard composite index on `(symbol, date)`. TimescaleDB's advantages (20x insert throughput, specialized time-bucket aggregation functions, automatic compression/retention policies) only start to matter at high-frequency or very-high-volume time-series workloads — none of which apply here. Adding TimescaleDB would be pure operational complexity with no measurable benefit at this data volume.

**Confidence: HIGH.**

---

## 4. AI Agent Layer

This is the section requiring the most careful primitive selection, because the wrong choice here compounds across the whole "agent" feature set (query portfolio, web search, code execution, write to user data, streaming, multi-step research).

### The four ways to build an agent on the Claude API — and which one fits

| Approach | What it is | Fit for this project |
|---|---|---|
| **Claude API + Tool Runner** (beta, part of the regular `@anthropic-ai/sdk`) | The SDK's automated tool-call loop: define custom tools (Zod schema + `run` function in TypeScript via `betaZodTool`), pass to `client.beta.messages.toolRunner()`, it loops calling the API → executing your tools → feeding results back until Claude is done. Supports streaming. | **Recommended primary approach.** This is the harness this project actually needs: a chat-shaped agent, custom tools you fully control (query portfolio DB, add transaction, set alert, run backtest), plus Anthropic's server-side tools (`web_search`, `code_execution`) declared alongside your custom ones in the same `tools` array. Runs inside your own Next.js API routes — you host it, you control the UI, you own the audit log requirement. |
| **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`) | Claude Code repackaged as a library — built-in Read/Write/Edit/Bash/Grep/WebSearch tools, full coding-agent harness | **Not recommended for this use case.** This is a coding/filesystem agent product, not an embedded product-feature agent. It ships tools (file read/write, bash) that are the wrong shape for "query a portfolio DB and answer questions about it." Do not confuse this with the Tool Runner above — they are genuinely different products despite similar names. |
| **Managed Agents (CMA)** | Anthropic hosts both the agent loop *and* a per-session sandbox container; you create a persisted, versioned Agent config and start Sessions against it via REST | Overkill for this project's shape. CMA is designed for server-managed, potentially long-running autonomous agents with Anthropic-hosted tool execution — genuinely useful for the "deep research" sub-feature in principle, but adds a second orchestration surface (agent/session/environment objects, vault-based credentials, webhook or polling event consumption) on top of an already-custom Next.js app. The Tool Runner + your own Postgres audit log covers the stated requirements with far less new surface area to learn and operate. **Worth revisiting later** specifically for the "deep research running minutes" sub-feature if the Tool Runner + Vercel Workflows pattern above proves awkward in practice — but not the v1 default. |
| **Manual agentic loop** | Hand-write the `while stop_reason == "tool_use"` loop yourself | Only worth it if the Tool Runner's per-turn hooks (which already support human-in-the-loop approval, error interception, result modification, streaming, and compaction) genuinely don't fit a specific control-flow need. Start with the Tool Runner; drop to manual only if you hit a documented limitation. |

**Confidence: HIGH** on "Tool Runner over Agent SDK for this shape of feature" — this is a documented, explicit distinction in Anthropic's own guidance, not a judgment call this research is making up.

### Vercel AI SDK — where it fits

Use the **Vercel AI SDK** for the **frontend/streaming plumbing layer**, not as a replacement for the Tool Runner's agent-loop logic: its `useChat` React hook, message-history state management, and tool-call rendering primitives are the standard, well-supported way to wire a streaming chat UI into a Next.js App Router page. The AI SDK's Anthropic provider increasingly supports passing through Anthropic's native server-side tools (`web_search`, `code_execution`) as well, so in practice you'll likely end up with: **AI SDK on the frontend + API route boundary, Tool Runner (or AI SDK's own `streamText`/`tools`/`maxSteps` loop, which is functionally equivalent) doing the actual multi-step orchestration server-side.** Either the Tool Runner or the AI SDK's native tool loop is a reasonable choice for the server-side orchestration itself — pick based on which integrates more smoothly with the rest of your AI SDK-based UI code once you're in the implementation phase; this is a low-stakes choice compared to the Agent SDK vs. Tool Runner distinction above.

### LangGraph / Mastra — when they'd matter, and why not yet

- **LangGraph:** low-level graph orchestration runtime, you own state/edges/model-provider-agnostic control. Valuable for genuinely complex multi-agent branching logic. This project's agent needs are a single conversational loop plus occasional delegated research — not complex enough to justify LangGraph's control-flow overhead yet.
- **Mastra:** TypeScript-first, frames agents as durable workflows (steps, retries, persistent state) — closer in spirit to Inngest/Temporal than to LangGraph, with built-in studio/evals/tracing. Genuinely appealing here because its workflow model directly overlaps with the "long-running deep research" durability problem this project needs to solve anyway (see §2). **Worth a serious look as an alternative to hand-rolling the Tool Runner + Vercel Workflows/Inngest combination**, but adds a framework dependency and its own learning curve for a small/solo team. **Recommendation: start with the simpler Tool Runner + Vercel AI SDK + Inngest/Workflows combination (fewer moving parts, most directly documented path); revisit Mastra specifically if the hand-rolled durable-research pattern proves painful in practice.**

### Code execution for charts, backtests, and scenario analysis

**Use Anthropic's native `code_execution` server tool as the primary and, for this project's scale, sufficient choice — do not stand up a separate sandbox service (E2B/Modal/Vercel Sandbox) unless a specific gap emerges.**

- Declare `{"type": "code_execution_20260521", "name": "code_execution"}` (or the non-beta `code_execution_20260120` variant, which additionally supports REPL persistence and programmatic tool calling) in the `tools` array — no separate infrastructure, no sandbox to provision or secure.
- Runs in an Anthropic-hosted, fully isolated container (1 CPU, 5GB RAM, 5GB disk, no internet access) with Python 3.11 and pandas/numpy/scipy/matplotlib/openpyxl pre-installed — exactly the toolset needed for portfolio charts, scenario analysis, and simple backtests.
- **Pricing is essentially free at this project's scale**: free when used alongside the `web_search`/`web_fetch` tools (which this agent needs anyway for the research features), otherwise $0.05/hour after **1,550 free hours/month per organization** — a 4-person family app will not come close to exhausting that.
- The sandbox has no internet access and no direct database connection by design — the pattern is: your custom `query_portfolio_db` tool fetches the relevant position/transaction/price data as JSON (server-side, in your own Next.js code, with proper user-scoping), and that JSON is what Claude passes into the code-execution container for the actual number-crunching. This is a clean separation and doesn't require exposing your database to Anthropic's infrastructure.

**When E2B / Modal / Vercel Sandbox would matter instead:** if the agent ever needs to run non-Python code, needs GPU access, needs a long-lived persistent dev-like environment across many turns, or needs direct outbound network access from inside the sandbox. None of these apply to this project's stated requirements. If a future need does emerge, **Vercel Sandbox** is the most naturally-integrated option given the rest of the stack (Firecracker microVMs, up to 45 min on Hobby / 24h on Pro), with **E2B** as the isolation-focused alternative.

**Confidence: HIGH** on recommending the native tool over a separate sandbox for this project's scope.

### Model choice and cost tracking

- **Default model: Claude Opus 5** (`claude-opus-5`) for the main conversational agent — $5/$25 per million input/output tokens. This is the correct default per Anthropic's current guidance for anything beyond trivial classification, and matches the quality bar this project needs (financial reasoning, portfolio-manager-grade advice).
- **Use Claude Haiku 4.5** (`claude-haiku-4-5`, $1/$5 per MTok) selectively for cheap, high-volume, low-complexity sub-tasks if the architecture ever delegates work — e.g., bulk-summarizing many news articles before the main agent synthesizes them. Not required for v1, but a lever worth knowing about given "no hard caps" is the stated policy and costs should still be proportionate.
- **Per-user usage/cost tracking** (a stated hard requirement — "System records per-user agent token usage and cost, visible to the user") is implemented by reading `response.usage` (`input_tokens`, `output_tokens`, `cache_read_input_tokens`, etc.) on every API call and persisting it to Postgres tied to `user_id` and a request/conversation ID. This is an application-layer implementation detail, not a separate product to buy — flag it as a required table (`agent_usage_log`) in the data model.

---

## 5. Financial Math

### Recommended approach: hand-roll in TypeScript, with rigorous test coverage — not a Python service, for v1

| Calculation | Recommendation | Confidence |
|---|---|---|
| **Money-weighted return (XIRR)** | Use the `xirr` npm package (RayDeCampo/nodejs-xirr) — 47K weekly downloads, MIT licensed, implements Newton-Raphson for irregular cash flows, the most established option researched. It hasn't shipped a new version in 12 months, but XIRR is a fully-specified, stable algorithm that doesn't need frequent updates — this is a case where "unmaintained" is not the same as "wrong." **Regardless of library choice, write property-based tests against known reference values (replicate Excel's XIRR() function on the same reference cash-flow examples) before trusting any implementation with real numbers.** | MEDIUM-HIGH — algorithm is well-understood; verify empirically before shipping either way |
| **Time-weighted return (TWR)** | No mature, widely-trusted JS library exists for this specifically. `@railpath/finance-toolkit` claims TWR/MWR support but is a small, unproven package — evaluate it, but do not trust it uncritically. **Recommend hand-implementing the standard chained sub-period method** (a TWR calculation that breaks the return series at every cash flow and geometrically links the sub-period returns) with test cases against known CFA-curriculum or Investopedia reference examples. This is the correct approach given the project's own stated principle: "a wrong return figure is worse than a missing one." | MEDIUM — the algorithm is well-documented in finance literature; implementation correctness must be verified by the team, not assumed from a library |
| **Drawdown, Sharpe ratio, volatility, correlation** | Hand-roll directly — these are simple, well-documented formulas (rolling maximum for drawdown, mean/stddev of returns for Sharpe/volatility, Pearson correlation for the correlation matrix) with no dominant, actively-maintained JS package worth the dependency risk. Unit test against a small hand-verified reference portfolio. | MEDIUM-HIGH — formulas are standard and simple enough that hand-rolling with tests is lower risk than trusting an unfamiliar small package |
| **Should there be a separate Python service (pandas/numpy/pyfolio/quantstats)?** | **Defer for v1.** Python's ecosystem (`quantstats`, `empyrical-reloaded`) is genuinely more mature for portfolio analytics than anything in JS, and Vercel does support deploying a Python (FastAPI) service alongside a Next.js app — as either a separate Vercel project the Next.js app calls over HTTP, or via Vercel's Python Functions runtime in the same repo. This is a legitimate, low-friction option if it's ever needed. But for v1: the actual math needed (TWR, XIRR, drawdown, Sharpe, correlation) is not so exotic that a well-tested TypeScript implementation is inadequate, and keeping everything in one language reduces operational surface for a small team. **Revisit this decision if Pro-mode analytics later need things quantstats does well and TypeScript doesn't** — Monte Carlo simulation, tearsheet-style multi-metric reports, or factor analysis. | MEDIUM — this is a real, revisitable tradeoff, not a closed question |

### Decimal precision for money — non-negotiable

**Use `decimal.js` for all monetary and FX arithmetic. Never use JavaScript's native `number`/float type for money, anywhere, at any layer.** This directly matches the project's own explicit constraint.

- **decimal.js**: arbitrary-precision decimal type, full TypeScript declarations, 3,951+ dependent packages on npm (this is an extremely widely-used, battle-tested library, not a niche choice) — **recommended.**
- **dinero.js**: conceptually well-suited (models money as integers in minor units specifically to avoid float issues) but **Dinero.js v2 is currently in alpha** — not production-appropriate for a project where "quality over speed" and correctness are explicit priorities. Dinero v1 exists but is effectively legacy/unmaintained.
- **currency.js**: lighter-weight, good for pure display formatting of fixed-precision currency values, but less suited to the multi-currency FX-conversion arithmetic and multi-year cost-basis compounding this project needs, which benefits from arbitrary (not fixed) precision.

**Database-layer implication:** store every monetary column as Postgres `NUMERIC`/`DECIMAL` (never `FLOAT`/`DOUBLE PRECISION`), with FX rate columns carrying extra decimal places (e.g., `NUMERIC(18,8)`) versus standard amount columns (e.g., `NUMERIC(18,4)`). Convert to/from `decimal.js` at every API/calculation boundary.

**Confidence: HIGH.**

---

## 6. Supporting Infrastructure

### Auth

| Option | Verdict |
|---|---|
| **Better Auth** | **Recommended.** As of September 2025, the Better Auth team took over stewardship of Auth.js/NextAuth (which is now in security-patch-only maintenance mode, no new features); in July 2026 Vercel acquired Better Auth outright, with the founding team joining Vercel to build agent-identity features on top of it — meaning it's now the Vercel-backed path forward, open source (MIT), free. Supports email+password credentials auth with a Postgres adapter (Drizzle/Prisma). |
| Auth.js / NextAuth v5 | Still functional (security patches only), but its own maintainers now point new projects toward Better Auth. Don't start a new project on it. |
| Clerk | Production-quality hosted UI, but billed per monthly active user — irrelevant cost-wise at 4 users, but adds a hosted-UI vendor and per-user pricing model this project doesn't need for a closed-allowlist family app. |
| Supabase Auth | Only makes sense if Supabase is also chosen as the database (see §3) — bundled, free on paid Supabase plans, and gets you RLS integration "for free." Skip if Neon is the DB choice. |

**Closed allowlist implementation note:** none of these tools natively support "signup closed to a fixed list of emails" out of the box — this requires a small custom check in the sign-up/sign-in handler regardless of which auth library is chosen (reject any email not on a configured allowlist). This is a code-level implementation detail, not a library gap.

### i18n (Turkish/English, App Router)

**next-intl** — recommended, HIGH confidence. Purpose-built for the App Router and React Server Components (translations load directly in server components with no hydration overhead), TypeScript autocompletion for translation keys, lighter bundle than a full i18next stack, and rapidly growing adoption (1.8M weekly downloads as of March 2026, ~4x growth over the prior 12 months, tracking App Router adoption directly). `next-i18next` only gained App Router support in March 2026 and is still described as having "rough edges" — not the safer choice for a fresh 2026 project.

### Charting

Financial data at 390px mobile width calls for two complementary libraries, not one:

- **Recharts** — for portfolio-level analytics: value-over-time line/area charts, asset-class/sector/currency allocation pie or donut charts, performance-vs-benchmark comparison lines, drawdown charts, correlation heatmaps. Highest adoption of any React chart library (48.9M weekly downloads), responsive sizing and animation built in, easy to get right quickly. This project's data volumes (a handful of series, a few thousand points per chart at most) are well within Recharts' comfort zone — no need for a canvas-heavy library meant for tens of thousands of live-updating points.
- **lightweight-charts** (TradingView's open-source library) — for individual-position OHLC/candlestick price history, which Recharts doesn't handle natively or well. Purpose-built for exactly this, canvas-based, extremely lightweight, and the de facto standard choice when a stock-style price chart is genuinely needed.

Both are lightweight enough to perform well at 390px on mobile.

### Telegram integration

**grammY** (TypeScript-native, actively maintained) over Telegraf for a serverless/webhook deployment — grammY's docs and API are explicitly designed around the "no persistent poll loop, webhook hits a serverless function" model this project needs, rather than assuming a long-running Node process. Set the Telegram webhook (one-time `setWebhook` call) to point at a Next.js API route (e.g. `/api/telegram/webhook`); verify Telegram's secret-token header on incoming requests for basic security. Use grammY's send-message API from the notification/delivery engine's Telegram channel adapter.

### Background job scheduling

**Inngest** — recommended default, HIGH confidence for this use case. Step-function model gives built-in per-step retry and idempotency, which matters directly for "sync ~70 tickers across multiple external APIs without one hiccup failing or duplicating the whole job." Generous free tier (25K function runs/month) is far more than a 4-user app's daily EOD sync + daily brief generation will consume. Pattern: **Vercel Cron triggers a thin API route → that route sends an event to Inngest → Inngest runs the actual durable, retryable, multi-step job** (this is the standard 2026 pairing, not an either/or).

- **Trigger.dev** is a credible alternative — runs on dedicated compute (not limited by Vercel function timeouts at all), excellent debugging/observability. Worth evaluating side-by-side with Inngest in the relevant phase if Inngest's DX doesn't click; either solves the underlying problem.
- **Vercel Cron alone** (no durable-function layer behind it) is adequate only for genuinely single-step, fast, idempotent triggers — not for the multi-ticker sync or multi-step brief generation this project needs.

---

## Installation

```bash
# Core framework
npx create-next-app@latest --typescript --app

# Database
npm install drizzle-orm postgres
npm install -D drizzle-kit
# (Provision Postgres via Vercel Marketplace → Neon integration)

# Auth
npm install better-auth

# AI agent layer
npm install @anthropic-ai/sdk ai @ai-sdk/anthropic

# Financial math
npm install decimal.js xirr

# i18n
npm install next-intl

# Charting
npm install recharts lightweight-charts

# Telegram
npm install grammy

# Background jobs
npm install inngest

# Dev dependencies
npm install -D typescript @types/node vitest
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|--------------------------|
| Neon Postgres | Supabase Postgres | You want bundled Auth + Storage + Realtime, or prefer Supabase's automatic RLS-via-PostgREST wiring over application-layer tenant isolation |
| EODHD (EOD + fallback fundamentals) | Twelve Data (Grow plan, $29/mo) | The BIST spike on EODHD comes back weak — Twelve Data explicitly lists Istanbul under its Grow plan and is a clean drop-in swap for BIST price data specifically |
| Better Auth | Supabase Auth | Only if Supabase is chosen as the database — then it's free and pre-integrated |
| Inngest | Trigger.dev | Inngest's step-function DX doesn't fit the team's preferences; Trigger.dev's dedicated compute avoids Vercel timeout concerns entirely |
| Claude API + Tool Runner | Managed Agents (CMA) | The "deep research running minutes" feature specifically proves awkward to build on the Tool Runner + Vercel Workflows pattern — CMA's hosted sandbox + session model is purpose-built for exactly that, at the cost of a second orchestration surface to learn |
| Hand-rolled TS financial math | Python microservice (quantstats/pandas) | Pro-mode analytics later need Monte Carlo simulation, tearsheet-style reports, or factor analysis that TypeScript genuinely can't do as well |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| JavaScript `number`/float for any monetary value | Silent precision loss compounds across years of transactions and multi-currency conversion; the project's own stated constraint explicitly forbids this | `decimal.js`, Postgres `NUMERIC` columns |
| Dinero.js v2 | Currently in alpha, not production-ready | `decimal.js` |
| Claude Agent SDK for the embedded portfolio agent | It's a coding/filesystem agent product (Claude Code repackaged) with the wrong built-in tool shape for a portfolio-query product feature | Claude API + Tool Runner |
| TimescaleDB / a separate time-series database | Pure operational overhead at ~91,000 rows of daily price data — solves a scale problem this project doesn't have | Plain Postgres with a `(symbol, date)` composite index |
| Real-time/licensed BIST streaming data feeds | Hundreds of USD/month, requires a signed Borsa İstanbul data agreement — explicitly out of scope per PROJECT.md | Delayed/EOD data from EODHD or Twelve Data |
| Fintables API as a primary BIST integration target | No public self-serve pricing; API access is corporate-membership only | EODHD/Twelve Data for prices, borsapy + agent web search for fundamentals fallback |
| Vercel Cron as the sole execution engine for multi-step jobs | No step-level retry/idempotency — a partial failure mid-sync either fails everything or duplicates work | Vercel Cron as trigger, Inngest/Trigger.dev/Vercel Workflows as the durable execution layer |
| Auth.js/NextAuth for a brand-new 2026 project | In maintenance-only mode; its own maintainers point new projects at Better Auth | Better Auth |
| next-i18next for a fresh App Router project | App Router support only landed March 2026, still described as rough | next-intl |

## Stack Patterns by Variant

**If the EODHD BIST spike fails (no usable BIST fundamentals at any tier):**
- Fall back fully to borsapy + agent-driven web search/KAP review for BIST fundamentals
- Keep EODHD for EOD prices only, or swap to Twelve Data's Grow plan for BIST prices specifically if EODHD's BIST price coverage is also weaker than expected
- This does not block v1 — buy-and-hold, 3-month-cadence investing does not require real-time or even weekly fundamentals refresh

**If the Managed Agents (CMA) surface proves worth adopting for deep research specifically:**
- Keep the Tool Runner for the main interactive chat (it's simpler and you already own the UI/audit-log integration)
- Use CMA sessions specifically for the "deep research running minutes" sub-feature, treating it as a specialized backend for one feature rather than the whole agent layer

**If Supabase is chosen over Neon:**
- Use Supabase Auth instead of Better Auth (bundled, free)
- Lean on Postgres RLS via Supabase's auth.uid() JWT integration for tenant isolation instead of (or in addition to) application-layer scoping

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| Next.js 16.x | React 19.2 (bundled) | No separate version pin needed — Next.js manages the React version via its canary channel |
| `@anthropic-ai/sdk` (Tool Runner, beta) | Claude Opus 5 / Sonnet 5 / Haiku 4.5 | Tool Runner is a beta SDK feature but stable in practice; works with all current-generation models |
| Vercel Workflows | Node.js/TypeScript and Python runtimes | GA as of April 2026; verify current SDK version at implementation time given its relative newness |
| Drizzle ORM | Neon serverless driver (`@neondatabase/serverless`) or standard `postgres` client | Use the Neon serverless driver specifically inside Vercel Functions for optimal connection pooling behavior; the standard `postgres` client works fine for local dev and non-serverless contexts |

## Sources

- EODHD pricing page (eodhd.com/pricing), exchange list (eodhd.com/list-of-stock-markets) — plan prices confirmed, BIST inclusion inferred from exchange-code documentation, not hands-on tested
- Twelve Data pricing (twelvedata.com/pricing) and Borsa Istanbul support article (support.twelvedata.com) — BIST explicitly listed under Grow plan
- TCMB EVDS official system (evds3.tcmb.gov.tr) and multiple third-party client libraries confirming free registration-based access
- borsapy GitHub repository and its documented personal/non-commercial license terms
- Borsa İstanbul official Data Dissemination and Data Distribution Agreement pages (borsaistanbul.com) — confirms licensed-vendor model for real-time/redistributed data
- Fintables corporate site and membership pages (fintables.com) — no public API self-serve pricing found
- Vercel official docs: Functions Limitations, Fluid Compute changelog, Vercel Workflows announcement blog and docs
- Neon vs Supabase 2026 pricing comparisons (multiple independent sources) and Neon's own 2026 pricing changelog
- Anthropic's own Claude API documentation (via the claude-api skill bundled with this environment) — Tool Runner vs. Agent SDK vs. Managed Agents distinction, code execution tool pricing and behavior, current model pricing table
- next-intl vs next-i18next adoption/download comparisons (i18nexus.com, locize.com)
- Better Auth's own announcement of the Auth.js stewardship transition and the subsequent Vercel acquisition (better-auth.com/blog, thenewstack.io)
- npm package pages and npm-trends/socket.dev for `xirr`, `decimal.js`, `dinero.js`, `currency.js` maintenance/adoption data

---
*Stack research for: private multi-currency family investment portfolio management platform with embedded Claude AI agent*
*Researched: 2026-08-16*
