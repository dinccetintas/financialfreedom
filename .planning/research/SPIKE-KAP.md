# Spike: KAP as a BIST Fundamentals Source

**Date:** 2026-08-16
**Trigger:** Owner proposed `pykap` (https://github.com/cemsinano/pykap) and questioned whether KAP disclosures are needed at all, given that BIST financial results are obtainable directly.
**Status:** RESOLVED — hands-on tested, not desk research.
**Impact:** Materially downgrades the project's largest technical risk.

## Question

Research flagged "BIST fundamentals coverage" as the single largest technical risk, with no provider confirmed to supply them at hobbyist pricing, and a possible $59.99/mo EODHD Fundamentals upgrade as the mitigation. Can KAP serve as the fundamentals source instead?

## What was tested

`pykap` v0.2.0 (MIT licence, last release 2026-03-12) installed from PyPI and run live against KAP.

| Capability | Result |
|---|---|
| `bist_company_list()` | ✅ 759 tickers returned |
| `get_general_info(tick=...)` | ✅ Full company metadata (name, city, auditor, company_id) |
| `BISTCompany(...).get_financial_reports()` — **index** | ✅ Correct periods and disclosure IDs for THYAO, GARAN, BIMAS (e.g. THYAO 2026 H1 → disclosure 1643238) |
| `get_financial_reports()` — **parsed line items** | ❌ Empty (`results` = 0 keys) for every company and every period tested |
| Underlying KAP announcement document | ✅ Reachable, HTTP 200, 5 MB, contains full statement data |

## Root cause of the parser failure

`_get_announcement()` scrapes `https://www.kap.org.tr/tr/Bildirim/{id}` expecting the old GWT-era markup — it matches on CSS class `gwt-Label multi-language-content content-tr`.

**KAP has since rebuilt on Next.js.** The old GWT classes no longer exist, so the parser matches nothing and returns an empty dict. It fails silently rather than raising.

The data itself is unaffected. Direct inspection of announcement 1643238 (THYAO 2026 H1):

- Page contains the string `Finansal Rapor` ✅
- Page contains `taxonomy-context-value` ✅
- **4,600 nodes carrying the `taxonomy-context-value` class** — the full tagged financial statement, still server-rendered in the HTML

## Conclusion

**BIST fundamentals are freely and legally obtainable from the official source.** They are not behind a paywall, not missing, and not a licensing problem — they are public regulatory filings on the regulator's own platform, served as server-rendered HTML with taxonomy-tagged values.

What is needed is a parser written against current KAP markup. That is a bounded, testable engineering task, not an unresolved sourcing risk. The values carry a taxonomy class, which means extraction is structural rather than heuristic.

## Revised position vs. research SUMMARY.md

| SUMMARY.md said | Revised |
|---|---|
| BIST fundamentals unconfirmed at this budget; largest single technical risk | Available free from the official source; risk downgraded from *sourcing* to *parsing* |
| Possible $59.99/mo EODHD Fundamentals upgrade | Not required for BIST. Budget stays at ~$20–35/mo |
| KAP has no official API; scraping legally unreviewed | Still true, and still the correct framing for **high-frequency event monitoring**. But periodic financial statements are a low-volume, personal-use read of public regulatory filings — a materially weaker legal concern than continuous scraping |
| `borsapy` as fundamentals fallback | Demoted to secondary. KAP is the primary source, being the authoritative one |

## Two distinct uses of KAP — do not conflate

The owner correctly separated these:

1. **Periodic financial statements** (`FR` disclosure class) — quarterly and annual results. Low volume (roughly 4 fetches per company per year), high value: powers fundamentals, sector tagging, and the discovery/screening feature. **This is the valuable one and it is now solved.**
2. **Material event disclosures** (`ODA` class — özel durum açıklamaları) — the continuous event stream for thesis-invalidation monitoring. Higher volume, higher scraping-frequency concern, lower certainty of value. Correctly deferred behind price-based rule breaches.

## Implementation notes for the relevant phase

- **Do not depend on `pykap`'s `get_financial_reports()` line-item parsing.** It is broken against current KAP.
- **Do use its index functions** (`bist_company_list`, `get_general_info`, `get_historical_disclosure_list`) — these hit KAP's JSON endpoints and work correctly today.
- The stack is TypeScript; `pykap` is Python. Since the working parts are thin wrappers over KAP HTTP endpoints, **port the needed calls to TypeScript** rather than introducing a Python service — this is consistent with the research finding that no separate Python service is warranted.
- Parse against the `taxonomy-context-value` class, which is a stable structural hook rather than a positional guess.
- Cache aggressively. Statements change four times a year; there is no reason to re-fetch more often.
- Degrade gracefully: if a parse fails, fall back to agent web search rather than showing wrong numbers. A missing fundamental is recoverable; a wrong one is not.
- Consider upstreaming a parser fix to `pykap` — the maintainer is active and the licence is MIT.

## Residual risk

- KAP could restructure its markup again, breaking the parser. Mitigation: treat parse failure as an alert condition, never silently return partial data.
- Sector/industry taxonomy for exposure breakdown still needs a source; KAP company metadata may supply it, but this was not tested in this spike.
- The `ODA` event-stream volume question is untouched by this spike and remains open.
