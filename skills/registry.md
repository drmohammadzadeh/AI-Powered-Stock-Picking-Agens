# Invelix — Skills Registry

> Skills are atomic tool-capabilities. Agents *own* skills; workflows *compose* them. Naming: `domain.action` (e.g., `ingest.fundamentals`).

Total: **42 skills** across 8 domains. Each skill lists: input → output, tools, owner agent(s).

---

## 1. Ingest (Data Collection)

| Skill | ID | Owner | Input → Output | Tools / APIs |
|-------|----|-------|----------------|--------------|
| Fetch fundamentals | `ingest.fundamentals` | agent.data | ticker → income, balance, cashflow, ratios (150 indicators) | SEC EDGAR, FMP, Polygon fundamentals |
| Fetch technicals | `ingest.technicals` | agent.data | ticker → OHLCV, indicators (600) | Polygon, Tiingo, yfinance |
| Fetch sentiment | `ingest.sentiment` | agent.data | ticker → news, social, analyst revisions (150) | NewsAPI, Reddit/Twitter, Estimize, GICS |
| Fetch macro & sector | `ingest.macro` | agent.data | date → rates, sector momentum, breadth | FRED, sector ETFs |
| Normalize & validate | `ingest.normalize` | agent.data | raw → clean, point-in-time, survivorship-bias-free dataset | Great Expectations, dbt |
| Feature engineering | `feature.engineer` | agent.data | 900 indicators → 10,000 features | Python, TA-Lib, custom factors |
| Feature store write | `feature.store` | agent.data | features → Parquet/S3 + Feast | S3, DuckDB, Feast |
| IPO ingestion | `ingest.ipo` | agent.data | scraping → IPO list (announced/filed/rumored) | Manual + Crunchbase |

## 2. Model (Quant Core)

| Skill | Owner | Description |
|-------|-------|-------------|
| `model.train_ensemble` | agent.quant | Train ensemble (LGBM + XGB + MLP) on 5B point-in-time samples; cross-validate by time |
| `model.infer_score` | agent.quant | Score → `Invelix Score 1-10`, `probability`, `sub-scores` (fund/tech/sent/lowRisk) |
| `model.calibrate` | agent.quant | Platt scaling → probability beating S&P/STOXX in 3M |
| `model.backtest` | agent.quant | Walk-forward backtest 2017→present; compute alpha, win-rate per score decile |
| `model.retrain_monitor` | agent.quant | Drift detection, auto-retrain trigger, MLflow registry |

## 3. Explainability

| Skill | Owner | Description |
|-------|-------|-------------|
| `explain.shap_attribute` | agent.explain | SHAP per feature → impact on probability (±%) |
| `explain.group_factors` | agent.explain | Group into 7 Alpha Factors: Valuation, Momentum, Sentiment, Size & Liquidity, Financial Strength, Growth, Volatility |
| `explain.rank_signals` | agent.explain | Top 28 signals by absolute impact + long-tail aggregation |
| `explain.narrate` | agent.explain | LLM-generated narrative: “Strongest boost from long-tail fundamentals (+1.83%) … high P/B detracts …” |

## 4. Forecasting

| Skill | Owner | Description |
|-------|-------|-------------|
| `forecast.horizon` | agent.forecast | Predict price range for 1M/3M/6M/1Y (avg/high/low) |
| `forecast.confidence` | agent.forecast | 68% interval + accuracy history vs analysts |
| `forecast.evaluate` | agent.forecast | Track forecast accuracy page weekly |

## 5. Ranking & Screening

| Skill | Owner | Description |
|-------|-------|-------------|
| `rank.universe` | agent.ranking | Maintain 6,500 US + 600 EU + 2,500 ETF universe |
| `rank.compute` | agent.ranking | Sort by Invelix Score, perf YTD, sector; country filter |
| `rank.theme_tag` | agent.ranking | Tag themes (AI, EV, Dividends) |
| `rank.export` | agent.ranking | Paginate, CSV export, SEO tables |

## 6. Trade Ideas

| Skill | Owner | Description |
|-------|-------|-------------|
| `trade.mine` | agent.trade | Scan for ≥60% win-rate Buy / ≥60% loss-rate Sell since 2017 |
| `trade.winrate` | agent.trade | Compute per-ticker per-signal win-rate table (1/3/6/12M) |
| `trade.overlay` | agent.trade | Render buy/sell markers on price chart + forecast overlay |

## 7. Portfolio

| Skill | Owner | Description |
|-------|-------|-------------|
| `portfolio.create` | agent.portfolio | CRUD portfolio, validate holdings |
| `portfolio.avg_score` | agent.portfolio | Avg Invelix Score of holdings |
| `portfolio.diversity` | agent.portfolio | Diversity Score (sector/geo/market-cap concentration) |
| `portfolio.alert` | agent.portfolio | Detect upgrade/downgrade vs yesterday; push email/push |
| `portfolio.evolution` | agent.portfolio | Track score evolution + rebalance suggestion |

## 8. Platform

| Skill | Owner | Description |
|-------|-------|-------------|
| `platform.auth` | agent.platform | Clerk/NextAuth, freemium gates (10 stocks free) |
| `platform.crud` | agent.platform | Postgres + Prisma for stocks/portfolios/users |
| `platform.cache` | agent.platform | Redis cache (score 1h, ranking 1h) |
| `platform.search` | agent.platform | Meilisearch ticker/name/theme search |
| `platform.webhook` | agent.platform | Stripe webhooks → entitlements |
| `platform.rate_limit` | agent.platform | Tiered rate limiting |

## 9. UI

| Skill | Owner | Description |
|-------|-------|-------------|
| `ui.render_score` | agent.frontend | Score pill + gradient bar + sub-scores grid |
| `ui.render_table` | agent.frontend | Ranking table with change, perf, flags |
| `ui.chart_evolution` | agent.frontend | Score evolution (daily/weekly/monthly/quarterly) |
| `ui.chart_forecast` | agent.frontend | Forecast cone + analyst overlay |
| `ui.search_bar` | agent.frontend | Autocomplete search (stocks/ETFs/themes) |
| `ui.seo` | agent.frontend | Dynamic meta, sitemap, JSON-LD |

## 10. Risk & Growth

| Skill | Owner | Description |
|-------|-------|-------------|
| `risk.low_risk` | agent.growth | Low-Risk Score 1-10 (volatility/drawdown model) |
| `risk.disclaimer` | agent.growth | Inject disclaimers, compliance checks |
| `growth.pricing` | agent.growth | Define tiers Free/Starter/Plus/Pro, feature gates |
| `growth.stripe` | agent.growth | Checkout, portal, metered usage |
| `growth.analytics` | agent.growth | PostHog, conversion funnel |

---

### Skill → Agent Matrix (Owner)

```
agent.data      : ingest.*, feature.*
agent.quant     : model.*
agent.explain   : explain.*
agent.forecast  : forecast.*
agent.ranking   : rank.*
agent.trade     : trade.*
agent.portfolio : portfolio.*
agent.platform  : platform.*
agent.frontend  : ui.*
agent.growth    : risk.*, growth.*
```

Workflows compose skills in DAGs — see `/workflows/*.md`.
