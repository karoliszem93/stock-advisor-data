# SCHEMAS — canonical JSON shapes for stock-advisor-data

This file defines the exact structure of every JSON document committed to this repo. Field-by-field, with types and examples. An AI agent can use this as the contract for reading any file in this repo.

> **Versioning:** the schema version of every file is in its `schema_version` field. Current version: **0.1**. When fields are added, the schema version bumps; older files retain their original version. Renames or removals will bump major.

## Types

- `iso_date` — `"YYYY-MM-DD"`
- `iso_datetime` — `"YYYY-MM-DDThh:mm:ssZ"` (UTC)
- `ticker` — bare ticker plus exchange suffix when needed (e.g. `AAPL`, `VWRL.L`, `EUNL.DE`)
- `confidence` — float in `[0, 1]`
- `direction` — one of `"buy" | "avoid" | "sell_short"`
- `timeframe` — one of `"1w" | "2w" | "1m" | "3m" | "6m" | "1y" | "3y"`
- `risk_profile` — one of `"conservative" | "balanced" | "growth" | "aggressive"`
- `asset_type` — one of `"equity" | "etf"`

---

## `suggestions/<date>/<risk>/<timeframe>.json`

A list (≥3) of suggestions for one (risk × timeframe) cell.

```json
{
  "schema_version": "0.1",
  "suggestion_date": "2026-04-27",
  "timeframe": "1m",
  "risk_profile": "growth",
  "suggestions": [
    {
      "id": "uuid-v4",
      "ticker": "NVDA",
      "asset_type": "equity",
      "direction": "buy",
      "confidence": 0.74,
      "confidence_calibrated": null,
      "target_date": "2026-05-27",
      "entry": { "price": 142.10, "currency": "USD", "price_eur": 132.45, "fx_used": 0.9321 },
      "stop_loss": { "price": 134.50, "price_eur": 125.40 },
      "target": { "price": 158.00, "price_eur": 147.30 },
      "suggested_risk_pct": 0.015,
      "headline": "NVDA — pullback to 50-EMA into supportive earnings setup.",
      "rationale": {
        "technical_case": "RSI 38 (oversold for trend), MACD bullish cross 4 days ago, price 2.3% above the 200-EMA, ADX 24 (trending), support cluster at 138-140.",
        "fundamental_case": "Forward P/E 28 vs sector 32, FCF yield 4.1%, Piotroski 7/9, Altman Z 6.2 (safe), 4 consecutive earnings beats with average surprise +8.4%.",
        "sentiment_case": "News sentiment +0.41 (FinBERT, last 7d, 31 articles). Reddit mention velocity +120% w/w but sentiment skew remains positive (+0.52). No insider sales last 90 days.",
        "macro_context": "VIX 14.2 (low), 10Y at 4.21% (down 12bp w/w), tech sector relative strength rank 3/11. Macro tailwind for high-multiple growth.",
        "why_this_timeframe": "1m horizon captures the post-pullback bounce + earnings on May 22 (in window). Shorter timeframe doesn't allow earnings catalyst; longer timeframe dilutes the technical edge.",
        "key_risks": ["Earnings miss reverses thesis sharply", "Broad tech rotation if VIX > 20"],
        "invalidation_triggers": ["Close below 134.50 on >1.5x avg volume invalidates technical setup", "Pre-announcement guidance cut", "VIX > 22 sustained 3 days"],
        "confidence_drivers": [
          { "factor": "earnings_in_window", "delta": -0.12, "reason": "Event uncertainty" },
          { "factor": "macro_tailwind", "delta": +0.06, "reason": "Falling rates support multiple expansion" },
          { "factor": "technical_alignment", "delta": +0.08, "reason": "RSI + MACD + ADX agree" }
        ],
        "tax_notes": "US-domiciled equity. LT capital gains 15% on profit at sale; €500 annual exemption applies. Dividends 15% withheld at source via US-LT treaty (no double taxation).",
        "data_quality": "full"
      },
      "module_scores": {
        "technical": 0.72,
        "momentum": 0.55,
        "volatility_regime": "normal",
        "mean_reversion": 0.61,
        "fundamental": 0.78,
        "quality": 0.81,
        "news_sentiment": 0.41,
        "social_sentiment": 0.52,
        "insider_institutional": 0.10,
        "macro": 0.34,
        "event_awareness": "earnings_2026-05-22",
        "risk_metrics": { "beta": 1.62, "vol_30d_pct": 38.4 }
      },
      "weights_used_path": "models/weights_history.jsonl#L142",
      "analysis_path": "analysis/2026-04-27/NVDA.json"
    }
  ]
}
```

## `suggestions/index.json`

Append-only index. One line per suggestion (JSONL would also work; we keep a JSON array for simpler consumption).

```json
{
  "schema_version": "0.1",
  "entries": [
    {
      "id": "uuid-v4",
      "suggestion_date": "2026-04-27",
      "ticker": "NVDA",
      "risk_profile": "growth",
      "timeframe": "1m",
      "direction": "buy",
      "confidence": 0.74,
      "target_date": "2026-05-27",
      "path": "suggestions/2026-04-27/growth/1m.json",
      "validated": false,
      "validation_path": null
    }
  ]
}
```

## `analysis/<date>/<TICKER>.json`

Full module-by-module analysis for one ticker on one day. Several KB per file. This is the audit trail.

```json
{
  "schema_version": "0.1",
  "ticker": "NVDA",
  "asset_type": "equity",
  "snapshot_date": "2026-04-27",
  "data_quality": "full",
  "data_sources": {
    "prices": "yfinance",
    "fundamentals": ["fmp", "edgar"],
    "news": ["finnhub", "newsapi"],
    "social": "reddit",
    "macro": "fred"
  },
  "modules": {
    "technical": { "rsi_14": 38.2, "macd": { "macd": 0.41, "signal": 0.12, "hist": 0.29 }, "ema_20": 140.3, "ema_50": 137.8, "ema_200": 121.2, "support_levels": [138.0, 134.5], "resistance_levels": [148.0, 158.0], "atr_14": 4.2, "adx_14": 24.1, "score": 0.72, "direction": "buy" },
    "momentum": { "ret_1m_pct": -3.2, "ret_3m_pct": 8.1, "ret_6m_pct": 22.4, "rs_vs_spy_3m": 1.05, "rs_vs_sector_3m": 1.12, "score": 0.55, "direction": "buy" },
    "fundamental": {  /* see schema below */  },
    "quality": { "piotroski": 7, "altman_z": 6.2, "beneish_m": -2.4, "score": 0.81 },
    "news_sentiment": { "headline_count_7d": 31, "mean_finbert": 0.41, "trend": "rising", "score": 0.41 },
    "social_sentiment": { "reddit_mentions_7d": 1240, "delta_wow_pct": 120.0, "mean_sentiment": 0.52, "score": 0.52 },
    "insider_institutional": { "insider_buys_90d": 2, "insider_sells_90d": 0, "13f_net_change_pct": 1.4, "score": 0.10 },
    "macro": { "vix": 14.2, "us10y": 4.21, "us10y_delta_wow_bp": -12, "sector_rs_rank": 3, "score": 0.34 },
    "event_awareness": { "earnings_in_window": "2026-05-22", "ex_div_in_window": null, "fomc_in_window": null, "confidence_adjustment": -0.12 },
    "risk_metrics": { "beta_3y": 1.62, "vol_30d_pct": 38.4, "max_dd_3y_pct": -34.2, "sharpe_3y": 1.08 }
  },
  "synthesis": {
    "weighted_score": { "1w": 0.42, "2w": 0.51, "1m": 0.66, "3m": 0.58, "6m": 0.49, "1y": 0.38, "3y": 0.27 },
    "weighted_score_per_risk": {
      "conservative": { "1m": 0.31 },
      "balanced":     { "1m": 0.55 },
      "growth":       { "1m": 0.66 },
      "aggressive":   { "1m": 0.71 }
    },
    "llm_thesis_path": "analysis/2026-04-27/NVDA-thesis.txt"
  }
}
```

## `validations/<date>/results.json`

```json
{
  "schema_version": "0.1",
  "validated_on": "2026-05-27",
  "results": [
    {
      "suggestion_id": "uuid-v4",
      "suggestion_path": "suggestions/2026-04-27/growth/1m.json",
      "ticker": "NVDA",
      "direction": "buy",
      "confidence": 0.74,
      "outcome": "correct",
      "outcome_score": 0.78,
      "actual": {
        "price_return_pct_native": 7.4,
        "dividend_return_pct_native": 0.0,
        "fx_effect_pct_eur": -1.1,
        "total_return_pct_eur": 6.3,
        "after_tax_return_pct_eur": 5.36,
        "max_favorable_excursion_pct": 9.8,
        "max_adverse_excursion_pct": -2.1,
        "target_hit": true,
        "stop_hit": false,
        "tax_breakdown": { "capital_gains_tax_pct": 0.94, "dividend_withholding_pct": 0.0 }
      }
    }
  ]
}
```

## `validations/aggregate_performance.json`

```json
{
  "schema_version": "0.1",
  "as_of": "2026-04-27",
  "total_validated": 142,
  "calibration_active": true,
  "calibration_method": "isotonic",
  "by_cell": {
    "conservative": {
      "1w": { "n": 18, "hit_rate": 0.61, "mean_outcome_score": 0.18 },
      "1m": { "n": 22, "hit_rate": 0.68, "mean_outcome_score": 0.31 }
    }
  },
  "by_module": {
    "technical": { "weight_share": 0.31, "predictive_correlation": 0.42 },
    "fundamental": { "weight_share": 0.22, "predictive_correlation": 0.55 }
  }
}
```

## `models/weights_history.jsonl`

One JSON object per line. Append-only.

```json
{"schema_version": "0.1", "ts": "2026-04-27T08:05:12Z", "trigger": "scheduled_recalibration", "weights_by_cell": { "growth.1m": { "technical": 0.30, "momentum": 0.20, "fundamental": 0.20, "quality": 0.10, "sentiment": 0.10, "macro": 0.10 } }, "based_on_n_validations": 142, "previous_weights_sha": "abc1234"}
```

## `MANIFEST-<date>.json`

Pinned context for a daily run. Lets agents reproduce/audit.

```json
{
  "schema_version": "0.1",
  "run_date": "2026-04-27",
  "timezone": "Europe/Vilnius",
  "code_sha": "deadbeefcafe1234",
  "ollama_model": "qwen2.5:14b-instruct",
  "config": {
    "base_currency": "EUR",
    "lt_capital_gains_rate": 0.15,
    "lt_dividend_tax_rate": 0.15,
    "lt_annual_cg_exemption_eur": 500.0
  },
  "providers_used": ["yfinance", "fmp", "finnhub", "newsapi", "fred", "reddit", "edgar"],
  "errors": {},
  "fx_rates_used": { "USDEUR": 0.9321, "GBPEUR": 1.1820 },
  "stats": { "tickers_analyzed": 73, "suggestions_generated": 84, "validations_processed": 12, "duration_seconds": 612 }
}
```
