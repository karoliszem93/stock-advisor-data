# stock-advisor-data

Durable, agent-readable record of every analysis run, suggestion, and validation produced by [`stock-advisor-app`](https://github.com/karoliszem93/stock-advisor-app).

This repo is not the application — it's the data warehouse. The app commits to it once per scheduled run. Future AI agents can clone this repo and reason over the entire history without needing the application running.

> **For an AI agent reading this repo:** start with `SCHEMAS.md` for the exact JSON shapes, then read this file's "How to navigate" section. Every commit here is structured for programmatic consumption — see also `MANIFEST.json` files which pin the code SHA and model version that produced each day's output.

## How to navigate

```
stock-advisor-data/
├── README.md                  ← you are here
├── SCHEMAS.md                 ← canonical JSON schema definitions for every file
├── snapshots/                 ← per-day raw market & fundamentals snapshots
│   └── YYYY-MM-DD/
│       ├── prices.parquet     ← OHLCV for every analyzed ticker that day
│       ├── fundamentals.json  ← per-ticker fundamentals snapshot
│       ├── macro.json         ← VIX, rates, FX, sector indices
│       ├── news.json          ← news headlines + per-headline sentiment
│       ├── social.json        ← Reddit mention velocity / sentiment
│       └── universe.json      ← which tickers were analyzed and why
├── analysis/                  ← per-ticker, per-day full module-by-module analysis
│   └── YYYY-MM-DD/
│       └── <TICKER>.json
├── suggestions/               ← per-day suggestions, organized by risk × timeframe
│   ├── YYYY-MM-DD/
│   │   ├── conservative/
│   │   │   ├── 1w.json
│   │   │   ├── 2w.json
│   │   │   ├── 1m.json
│   │   │   ├── 3m.json
│   │   │   ├── 6m.json
│   │   │   ├── 1y.json
│   │   │   └── 3y.json
│   │   ├── balanced/...
│   │   ├── growth/...
│   │   └── aggressive/...
│   ├── index.json             ← searchable index of every suggestion ever made
│   └── MANIFEST-YYYY-MM-DD.json   ← pins code_sha + model_version for that day
├── validations/               ← outcome records — what happened on target_date
│   ├── YYYY-MM-DD/
│   │   └── results.json       ← every suggestion whose target_date hit today
│   └── aggregate_performance.json    ← rolling accuracy by (signal, risk, timeframe)
├── models/                    ← module weight history
│   └── weights_history.jsonl  ← append-only log; one line per recalibration event
└── reports/                   ← human-readable rollups
    └── weekly_YYYY-Www.md
```

### Standard query patterns

| Question | Where to look |
|---|---|
| "What did we suggest for AAPL on date D?" | Search `suggestions/index.json` for `ticker=AAPL, date=D`, follow `path` |
| "What was the full analysis for AAPL on date D?" | `analysis/D/AAPL.json` |
| "How accurate are 1-month conservative suggestions?" | `validations/aggregate_performance.json` → `by_cell.conservative.1m` |
| "When were the module weights last recalibrated?" | tail of `models/weights_history.jsonl` |
| "Was this suggestion produced under degraded data?" | the `data_quality` field in the suggestion JSON |

## Commit conventions

Every commit message follows this shape so it's easy to grep / diff:

- `daily: YYYY-MM-DD — N suggestions, V validations, accuracy A%` — the main daily run
- `validate: YYYY-MM-DD — V validations, accuracy A%` — validation-only run
- `recalibrate: weights updated for (risk × timeframe) cells` — module weight changes
- `report: weekly YYYY-Www` — generated weekly markdown report
- `chore: ...` — repo housekeeping

## Time, currency, and tax conventions

- All timestamps in UTC unless otherwise noted; `suggestion_date` is the local date in `Europe/Vilnius` when the run executed.
- All monetary values are in **EUR** by default. When a non-EUR price is shown, both native currency and EUR equivalent are present. FX rates used are stamped into the file (`fx_rates_used`).
- Tax annotations assume a Lithuanian-resident individual (default LT capital-gains rate **15%**, dividend tax **15%**, annual €500 capital-gains exemption). Configurable in the app; current values are stamped into `MANIFEST-<date>.json`.

## Survivorship-bias note

The universe is reconstructed from "today's" S&P 500 / curated ETF list etc. We do not currently use a point-in-time index membership history (paid feature). Backtests and historical accuracy numbers therefore have a small upward bias. Future agents should treat absolute accuracy numbers as approximate; relative accuracy across cells is more reliable.

## Data quality flags

Each suggestion carries `"data_quality": "full" | "degraded"`. When degraded, an `errors` block in the day's `MANIFEST-<date>.json` lists which providers failed and why. Suggestions generated under degraded data are usable but their confidence may be overstated.

## License

Private repository for personal use. No public license granted.
