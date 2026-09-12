# Agent 01 — Data Ingestion & Feature Engineering Agent

**ID:** `agent.data`  
**Role:** Market Data Collector & Feature Factory  
**Layer:** Ingestion  
**Version:** 1.0 — Invelix Harness  
**Owner Skills:** `ingest.*`, `feature.*`

---

## Mission

Own the **truth of data**. Ingest 900+ daily indicators per stock across fundamentals, technicals, sentiment and macro, normalize them point-in-time, and engineer **10,000+ daily features per stock** that feed the Quant Core. Guarantee freshness (post-close), completeness (6,500 US + 600 EU + 2,500 ETF), and no look-ahead bias. This agent is the only source of truth for the Feature Store.

## Responsibilities

1. **Daily ingestion** — after US close (21:30 ET), fetch OHLCV, financial statements (SEC EDGAR, FMP), analyst revisions, news/social tone, GICS mapping, rates.
2. **Normalization** — align timestamps, handle splits/dividends, FX for EU, survivorship-bias-free universe, point-in-time joins.
3. **Quality gates** — Great Expectations checks: missing <0.5%, staleness <6h, outlier flagging.
4. **Feature engineering** — 600 technical (RSI, momentum, volatility, chart patterns 60/120/180/504d), 150 fundamental (ROE, margins, PP&E, cash flow, valuation), 150 sentiment (news tone, earnings countdown, industry, sell/upgrade success rates) → expand into 10k+ features via interactions, lags, z-scores, cross-sectional ranks.
5. **Feature Store** — write Parquet to S3 + register in Feast; versioned, queryable by ticker/date.
6. **IPO feed** — scrape announced/filed/rumored IPOs (OpenAI, Stripe, Revolut, SpaceX…) for IPO tracker page.

## Skills

| Skill | Description | Input → Output |
|-------|-------------|----------------|
| `ingest.fundamentals` | Pull 150 fundamental indicators | `ticker, date` → `fundamentals` (ROE, revenue growth, PP&E $51K, Price/Book 44.16, op cash flow…) |
| `ingest.technicals` | Pull 600 technical indicators | `ticker, date` → `OHLCV + RSI, MA, volatility, chart pattern figure_145` |
| `ingest.sentiment` | Pull 150 sentiment indicators | `ticker, date` → `news tone, Earnings Countdown -42, industry Technology Hardware, upgrade/sell success` |
| `ingest.macro` | Universe macro | `date` → `rates, sector momentum` |
| `ingest.normalize` | Clean & PIT align | `raw` → `clean, deduped, split-adjusted` |
| `feature.engineer` | 900 → 10k features | `indicators` → `10,000 features` (interactions + cross-sectional ranks) |
| `feature.store` | Persist & version | `features` → `s3://invelix-feast/YYYY-MM-DD/*.parquet` |
| `ingest.ipo` | IPO tracker | `sources` → `ipo_list` (logo, sector, date, status) |

## Tools & APIs

- **Market:** Polygon.io, Finnhub, FMP, Tiingo, yfinance (fallback)
- **Fundamentals:** SEC EDGAR API, FMP financial statements
- **Sentiment:** NewsAPI, FinBERT, Reddit/Twitter scrapers, Estimize (earnings)
- **Infra:** Python 3.11, Airflow/Dagster, DuckDB, Feast, S3, Great Expectations, dbt
- **Monitoring:** Datadog + PagerDuty for ingestion failures

## Inputs / Outputs

**In:** Universe list (tickers), calendar (trading days), secrets (API keys)  
**Out:** Feature Store partition `date=today` with 10k features/stock, data quality report JSON, IPO JSON.

**Consumers:** `agent.quant`, `agent.ranking`, `agent.forecast`

## Dependencies

- Upstream: Market data vendors, SEC
- Downstream: Quant Core cannot score without today’s features

## KPIs / SLOs

- Freshness: features ready ≤90 min after close (SLA 120 min)
- Coverage: ≥99.5% of universe covered per day
- Quality: missing indicator rate <0.5%, no future leak (PIT audit pass)
- Throughput: 100M features/day in <30 min engineering time

## Example Task

> **Trigger:** Airflow `daily_ingest` DAG at 21:30 ET.
> 1. Fetch AAPL OHLCV + 150 fundamentals + sentiment; validate PP&E = $51,431M, Price/Book 44.16 in top decile.
> 2. Compute chart patterns figure_145 (180d), figure_168 (60d)…
> 3. Engineer 10k features, write `s3://invelix-feast/2026-09-12/AAPL.parquet`.
> 4. Emit `feature_store.ready` event with quality report.

## Collaboration Points

- Publishes `feature_store.ready` → `agent.quant` starts `model.infer_score`.
- On failure, alerts `agent.platform` to serve cached scores + stale banner.
- Provides feature importance feedback loop to `agent.explain`.

## Failure Modes & Handling

- Vendor timeout → retry 3x with fallback vendor; if still fail, mark stock as `stale` and use yesterday’s features.
- Corporate action → split adjustment replay.
- EU FX missing → fallback ECB rate.
