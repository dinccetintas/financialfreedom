# API Coverage — Phase 1

> Full coverage by default. Opt-outs are explicit, reasoned decisions.
>
> Phase 1 integrates four external HTTP services: **Finnhub** (INST-03 symbol search, D-13),
> **OpenFIGI** (ISIN/FIGI enrichment, D-14), **Resend** (AUTH-02 password-reset email, D-09) and
> **TCMB EVDS** (the one-shot USD/TRY backfill script, D-05). Each is enumerated against its own
> full capability surface below. A second integration against the same need in a later phase
> starts from the same full-coverage baseline — do not inherit these opt-outs silently.

## Finnhub (free tier) — REST v1

| capability | decision | reason |
|---|---|---|
| symbol-search (`/search`) | INTEGRATE | |
| stock-symbols (`/stock/symbol`) | OPT-OUT | not needed yet — Phase 1 resolves one ticker at a time on demand; a full exchange listing is only useful for the offline instrument catalogue, which no requirement asks for |
| company-profile2 (`/stock/profile2`) | OPT-OUT | not needed yet — name/exchange come back on the search result; sector/industry classification is INST-07, explicitly Phase 7 |
| quote (`/quote`) | OPT-OUT | explicitly out of scope — price data is DATA-01, Phase 3, and Phase 3 opens with a provider spike that may not select Finnhub |
| stock-candles (`/stock/candle`) | OPT-OUT | explicitly out of scope — price history is DATA-06, Phase 3 |
| basic-financials (`/stock/metric`) | OPT-OUT | explicitly out of scope — fundamentals are DATA-08, Phase 7 |
| company-news (`/company-news`) | OPT-OUT | explicitly out of scope — NEWS-01, Phase 10 |
| market-news (`/news`) | OPT-OUT | explicitly out of scope — NEWS-05, Phase 10 |
| recommendation-trends (`/stock/recommendation`) | OPT-OUT | explicitly out of scope — NEWS-08 analyst views, Phase 10 |
| price-target (`/stock/price-target`) | OPT-OUT | explicitly out of scope — NEWS-08, Phase 10 |
| earnings-calendar (`/calendar/earnings`) | OPT-OUT | explicitly out of scope — NEWS-04/NEWS-06, Phase 10 |
| ipo-calendar (`/calendar/ipo`) | OPT-OUT | not needed — no requirement in v1 references IPOs |
| stock-splits (`/stock/split`) | OPT-OUT | not needed yet — INST-04 splits are Phase 5; `corporate_actions` ships empty in Phase 1 by D-01 |
| dividends (`/stock/dividend`) | OPT-OUT | not needed yet — LEDG-04 dividends are Phase 2, corporate-action application is Phase 5 |
| peers (`/stock/peers`) | OPT-OUT | not needed — no v1 requirement asks for peer comparison |
| insider-transactions (`/stock/insider-transactions`) | OPT-OUT | explicitly out of scope — ADV-04 notable-investor tracking is a v2 requirement |
| forex-rates (`/forex/rates`, `/forex/candle`) | OPT-OUT | explicitly out of scope — DATA-02 fixes TCMB EVDS as the authoritative FX source, not a third party's derived number |
| crypto endpoints | OPT-OUT | explicitly out of scope — crypto is on REQUIREMENTS.md's Out of Scope table |
| websocket trade stream | OPT-OUT | explicitly out of scope — real-time streaming is out of scope per PROJECT.md; the project is buy-and-hold EOD |

## OpenFIGI — REST v3

| capability | decision | reason |
|---|---|---|
| mapping (`POST /v3/mapping`) | INTEGRATE | |
| search (`POST /v3/search`) | OPT-OUT | not needed — Finnhub owns free-text discovery (D-13); OpenFIGI is used only to enrich an already-resolved identity |
| filter (`POST /v3/filter`) | OPT-OUT | not needed — same reason as search; no requirement asks for FIGI-side faceted browsing |

## Resend — Node SDK v6 / REST

| capability | decision | reason |
|---|---|---|
| `emails.send` | INTEGRATE | |
| `batch.send` | OPT-OUT | not needed — AUTH-02 sends one reset email at a time to one address |
| `emails.get` / `emails.update` / `emails.cancel` (scheduling) | OPT-OUT | not needed — reset emails are sent immediately and never rescheduled or cancelled |
| `domains.*` | OPT-OUT | not needed programmatically — the sending domain is verified once by hand in the Resend dashboard (see `user_setup` in `01-04-PLAN.md`) |
| `apiKeys.*` | OPT-OUT | not needed — the key is provisioned once by hand and stored as `RESEND_API_KEY` |
| `audiences.*` / `contacts.*` / `broadcasts.*` | OPT-OUT | explicitly out of scope — these are marketing-list features; this is a four-person private app with no mailing list |
| webhooks (delivery/bounce events) | OPT-OUT | not needed yet — a handful of reset emails per year does not justify a delivery-event pipeline; revisit if a reset email is ever reported missing |

## TCMB EVDS — REST v1 (`evds2.tcmb.gov.tr/service/evds/`)

| capability | decision | reason |
|---|---|---|
| series data (`/series=...&startDate=&endDate=&type=json`) | INTEGRATE | |
| categories (`/categories`) | OPT-OUT | not needed — the series code for daily USD buying rate (`TP.DK.USD.A`) is known and pinned in the script; runtime category discovery adds a call for no benefit |
| datagroups (`/datagroups`) | OPT-OUT | not needed — same reason as categories |
| series list (`/serieList`) | OPT-OUT | not needed — same reason as categories |
| gold series (Istanbul Gold Exchange) | OPT-OUT | explicitly out of scope for Phase 1 — D-05 scopes the one-shot script to USD/TRY only; gold rates are DATA-02, Phase 5 |
| scheduled/recurring sync | OPT-OUT | explicitly out of scope — D-05 ships a script, not a job; the recurring job is Phase 5 |
