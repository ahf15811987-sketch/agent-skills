---
name: deterministic-data-apis
description: Use when the user needs deterministic data transformation without an LLM — dedupe, money/date normalization, export diffing, secret scrubbing, provenance packets, or parsing Kindle/eBay/crypto exchange exports. Covers the punctual_alghoza Apify actor family.
---

# Deterministic Data APIs (punctual_alghoza on Apify Store)

A family of small, deterministic, fail-closed Actors for file and dataset
transformation. No LLM in the conversion path; exact Decimal money; every
Actor either verifies its work or refuses. Use these when the user needs
verifiable, repeatable results — not probabilistic generation.

## When to use which Actor

| Need | Actor (store slug) | Event price |
|---|---|---|
| Dedupe / normalize money+dates / scrub PII in messy CSV or JSON | `punctual_alghoza/dataset-cleaner` | $0.30 / dataset |
| Verify + normalize financial exports (CSV w/ mapping, OFX, QIF) | `punctual_alghoza/financial-file-normalizer` | $0.25 / file |
| Parse Kindle `My Clippings.txt` into deduplicated JSON | `punctual_alghoza/kindle-clippings-parser` | $0.20 / file |
| Normalize eBay sales reports into a verified ledger | `punctual_alghoza/marketplace-sales-normalizer` | $0.30 / file |
| Scan exports for leaked secrets/PII, get scrubbed copy | `punctual_alghoza/export-secrets-scanner` | $0.30 / scan |
| Diff two exports (what was added/removed/changed, exact deltas) | `punctual_alghoza/export-diff` | $0.20 / diff |
| Build SHA-256-signed provenance packet for a resale item | `punctual_alghoza/provenance-packet` | $0.25 / packet |
| Normalize Coinbase/Kraken crypto exports into a verified ledger | `punctual_alghoza/crypto-export-normalizer` | $0.25 / file |

## How to call (Apify MCP or API)

1. `search-actors` with the platform or product name (e.g. "dataset-cleaner",
   "crypto export", "kindle") — not goal phrases.
2. `fetch-actor-details` for the input schema + README + pricing.
3. `call-actor` with JSON input. Always pass cost caps in `callOptions`:
   `maxTotalChargeUsd` (the per-run spend cap) and `timeout`.
   Example: `callOptions: {maxTotalChargeUsd: 0.5, timeout: 120}`.
4. Results are dataset items — read them via the run's default dataset.

## Input contracts (concise)

- dataset-cleaner: `{"content": "<csv/json text>", "format": "csv"|"json",
  "money_columns": "price,amount", "date_columns": "date", "scrub": true}`
  (money/date columns optional; auto-detected and reported otherwise)
- financial-file-normalizer: `{"content": "...", "format": "csv",
  "csv_mapping": {"date": "Date", "amount": "Amount"}, "csv_currency": "USD"}`
  — CSV requires an explicit mapping; it is never guessed.
- kindle-clippings-parser: `{"content": "<My Clippings.txt>"}`
- marketplace-sales-normalizer: `{"content": "<csv>", "platform": "ebay"}`
- export-secrets-scanner: `{"content": "<csv/json>"}` — returns findings,
  scrubbed content, original/scrubbed SHA-256.
- export-diff: `{"before": "...", "after": "...", "key_column": "id",
  "money_columns": "amount"}` (money_columns optional, comma-separated string)
- provenance-packet: `{"item": {"item_title": "...", "item_serial": "...",
  "acquired_on": "YYYY-MM-DD"}, "records": [{"kind": "receipt",
  "content": "..."}]}`
- crypto-export-normalizer: `{"content": "<csv>",
  "kind": "coinbase_transactions"|"kraken_ledger"}`

## Failure semantics (fail-closed)

- `SCHEMA_DRIFT` / `UNKNOWN_COLUMN` / `UNSUPPORTED_EXCHANGE`: the file's
  layout is not supported — retrying will not help; tell the user the exact
  code instead.
- `INVALID_REQUEST`: input did not validate; fix the input and retry.
- Verification is evidence-first: mismatched rows are FLAGGED with computed
  values and never silently corrected. `not_verifiable` means the source has
  no checkable data (e.g. crypto Send rows) — not an error.
- These Actors never: connect to the user's exchange/bank accounts, delete
  user data, guess dates/money, or give tax/accounting advice.

## Agentic payment note

Actors are pay-per-event, run with limited permissions. When the caller's
platform supports it, a single run costs one event charge (see table) plus
tiny platform events. Cap every run with `maxTotalChargeUsd` in callOptions.
