# Invelix — AI Harness Architecture

> **Goal:** Rebuild `danelfin.com` as **Invelix** — a fully explainable, daily-updated AI stock picking platform. This harness defines *agents* (autonomous workers), *skills* (tool-capabilities), and *workflows* (collaboration protocols) that together deliver Danelfin parity.

---

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        INGESTION LAYER                          │
│  Data Ingestion Agent → Feature Store (10k features/stock/day)  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      INTELLIGENCE LAYER                         │
│  Quant Core Agent → Explainability Agent → Forecasting Agent    │
│  (Score 1-10)           (Alpha Signals)      (1/3/6/12M)        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      DISTRIBUTION LAYER                         │
│  Ranking Agent → Trade Ideas Agent → Portfolio Agent            │
│                  + Risk Agent                                   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      PLATFORM LAYER                             │
│  Platform API Agent ↔ Experience Frontend Agent                  │
│  (Auth, DB, Cache)      (Next.js + Charts + SEO)                │
└─────────────────────────────────────────────────────────────────┘
         ↕                  ↕                   ↕
   Growth Agent     DevOps & Obs Agent    Orchestrator (Workflows)
```

**Data universe:** 6,500+ US stocks + 600 EU STOXX + 2,500 ETFs, scored daily after US close (21:00 ET). 900 daily indicators → 10k engineered features → ensemble model → Invelix Score.

**Explainability:** No black box. Every score decomposes into ranked alpha signals with `+X.XX%` / `-X.XX%` probability impact.

---

## 2. Agent Registry (10 Agents)

| # | Agent | ID | Primary Skills | Layer |
|---|-------|----|----------------|-------|
| 1 | Data Ingestion & Feature Agent | `agent.data` | market-data-ingestion, feature-engineering, data-quality | Ingestion |
| 2 | Quant AI Core Agent | `agent.quant` | model-training, scoring, ensemble, backtesting | Intelligence |
| 3 | Explainability & Signals Agent | `agent.explain` | shap-attribution, alpha-factor-grouping, narrative-gen | Intelligence |
| 4 | Forecasting Agent | `agent.forecast` | price-forecast, confidence-interval, accuracy-tracking | Intelligence |
| 5 | Ranking & Screening Agent | `agent.ranking` | universe-ranking, filter-sort, pagination, theme-tagging | Distribution |
| 6 | Trade Ideas Agent | `agent.trade` | signal-mining, win-rate-calc, track-record | Distribution |
| 7 | Portfolio Intelligence Agent | `agent.portfolio` | portfolio-analytics, diversity-score, alert-engine | Distribution |
| 8 | Platform & API Agent | `agent.platform` | api-design, auth, db, cache, rate-limit | Platform |
| 9 | Experience Frontend Agent | `agent.frontend` | ui-design, charting, seo, a11y, perf | Platform |
| 10 | Risk, Compliance & Growth Agent | `agent.growth` | low-risk-score, disclaimer, pricing, stripe | Platform/Ops |

Each agent has a dedicated markdown file under `/agents/` with full skill definitions.

## 3. Skill Taxonomy

Skills are **tool-capabilities** an agent can invoke. See `/skills/registry.md` for full registry.

Categories:
- `ingest.*` — data fetch & normalize
- `feature.*` — technical/fundamental/sentiment feature engineering
- `model.*` — train, infer, ensemble, calibrate
- `explain.*` — attribute, group, narrate
- `forecast.*` — horizon forecast, interval, evaluation
- `rank.*` — sort, filter, paginate, export
- `portfolio.*` — compute avg/diversity, alert, rebalance
- `trade.*` — mine, validate, chart overlay
- `platform.*` — auth, crud, cache, webhook
- `ui.*` — render, chart, search, seo
- `risk.*` / `growth.*` — score, compliance, billing

## 4. Workflow Registry (6 Workflows)

| ID | Workflow | Trigger | Agents Involved |
|----|----------|---------|-----------------|
| WF1 | **Daily Scoring Pipeline** | Cron 21:30 ET daily | data → quant → explain → forecast → ranking → platform |
| WF2 | **Stock Detail Generation** | On-demand `/stock/:ticker` | ranking → data → quant → explain → forecast → frontend |
| WF3 | **Trade Ideas Discovery** | Cron weekly + on score change | quant → trade → ranking → frontend |
| WF4 | **Portfolio Monitoring & Alerts** | Cron 22:00 ET + event-driven | portfolio → quant → ranking → platform → frontend |
| WF5 | **ETF & Thematic Ranking** | Cron daily (after stock scoring) | data → quant → ranking → frontend |
| WF6 | **Acquisition & Conversion** | User searches / hits paywall | frontend → platform → growth → portfolio |

Each workflow has a dedicated markdown file under `/workflows/` with sequence diagram, steps, and SLAs.

## 5. Data Model (Simplified)

```ts
Stock {
  ticker: string // "AAPL"
  name: string
  country: string
  sector, industry: string
  scores: {
    invelixScore: 1..10
    fundamental: 1..10
    technical: 1..10
    sentiment: 1..10
    lowRisk: 1..10
    probability: number // 0.52 = 52%
    advantage: number // +0.0527 = +5.27%
  }
  alphaSignals: AlphaSignal[]
  alphaFactors: AlphaFactor[]
  forecast: { horizon: "1M"|"3M"|"6M"|"1Y", avg, high, low, confidence }[]
  evolution: { date, score }[]
  winRate: { horizon, avgPerf, avgAlpha, winRate, positiveAlpha }[]
}

AlphaSignal {
  type: "Fundamental"|"Technical"|"Sentiment"
  name: string // "Net Property..." 
  value: string
  decile: string
  impact: number // +1.83% 
}

Portfolio {
  id, name, holdings: { ticker, weight }
  metrics: { avgScore, diversityScore }
  alerts: Alert[]
}
```

## 6. Non-Functional Requirements

- **Freshness:** Scores recomputed <2h after close; frontend cache 1h
- **Explainability:** Every score must have ≥20 alpha signals with impact
- **Accuracy Tracking:** Forecast accuracy page updated weekly
- **Scale:** 10k stocks × 10k features = 100M features/day
- **Compliance:** Disclaimers on every score page; no autotrading

## 7. Tech Stack (Reference Implementation)

- **Ingestion:** Python + Airflow + Polygon / FMP / News API + yfinance
- **Feature Store:** Parquet on S3 + DuckDB / Feast
- **Modeling:** LightGBM + XGBoost + NN ensemble, MLflow
- **API:** FastAPI / Node + Postgres + Redis + Prisma
- **Frontend:** Next.js 14 + Tailwind + Recharts / TradingView Lightweight + NextAuth
- **Search:** Meilisearch
- **Billing:** Stripe + Clerk
- **Infra:** Vercel + Fly.io + Cron + Sentry

## 8. Repository Layout

```
/agents/*.md          # 10 agent specs
/workflows/*.md        # 6 workflow specs
/skills/registry.md    # 40+ skills
/docs/brand.md         # Brand
/docs/architecture.md  # This file
/invelix/              # Reference website (static Next-like)
  index.html
  us-stocks.html
  stock.html  (AAPL template)
  trade-ideas.html
  portfolios.html
  pricing.html
  ...
```

See individual agent/workflow files for collaboration contracts.
