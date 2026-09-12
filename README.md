# Invelix — AI-Powered Stock Picking (Danelfin-Class Clone)

**Brand:** `Invelix` — *Invest with the odds in your favor.*  
**Live Site (local preview):** `invelix/index.html`  
**Original Reference:** [danelfin.com](http://danelfin.com) — all capabilities reproduced.  
**Language:** English (site + harness)  
**Harness Type:** Multi-Agent AI Harness (10 agents, 42 skills, 6 workflows)

> This repository is a **complete AI harness** that rebuilds Danelfin as **Invelix**. It includes: brand identity, 10 agent specifications, 42 skills, 6 cross-agent workflows, and a fully functional Danelfin-identical website.

---

## 📸 Preview

Open `invelix/index.html` — homepage is a pixel-level homage to danelfin.com:

- Hero with search + popular stocks (AAPL, TSLA, AMZN, MSFT, GOOGL) + app QR
- Ranking tables (US / Europe / ETFs) with AI Score 1-10 pills, flags, sub-scores, Perf YTD
- Testimonials, AI Score gradient 1→10, Data banner (+900 / +10k / +5B)
- 6 Use Cases (Pick Winners, Generate Returns, Trade Ideas, Explainable AI, Portfolio Tracking, Timing)
- IPO strip, video, Featured logos, pricing, legal disclaimer

Plus inner pages:

| Page | File | Danelfin Parity |
|------|------|----------------|
| Homepage | `invelix/index.html` | ✅ 100% |
| US Ranking | `invelix/us-stocks.html` | ✅ + filters, pagination, CSV |
| ETF Ranking | `invelix/etfs.html` | ✅ holdings-weighted |
| Stock Detail (AAPL) | `invelix/stock-aapl.html` | ✅ 7 sections: explanation, factors, 28 signals, trading params, forecast cone, win-rate, evolution |
| Trade Ideas | `invelix/trade-ideas.html` | ✅ 60% win-rate mining |
| Portfolios | `invelix/portfolios.html` | ✅ Avg Score, Diversity, alerts |
| Pricing | `invelix/pricing.html` | ✅ Free/Starter/Plus/Pro ($0/$29/$49/$99) |
| How it Works | `invelix/how-it-works.html` | ✅ Pipeline, explainability |

---

## 🏗️ Harness Architecture

See `docs/architecture.md` + `docs/brand.md` for full diagrams.

### Agents (10) — in `/agents/`

| # | Agent | ID | Role |
|---|-------|----|------|
| 1 | Data Ingestion & Feature Engineering | `agent.data` | 900 indicators → 10k features/day, PIT, Feast/S3 |
| 2 | Quant AI Core & Scoring | `agent.quant` | Ensemble (LGBM+XGB+MLP) → Score 1-10, sub-scores, backtest |
| 3 | Explainability & Alpha Signals | `agent.explain` | SHAP → 28 signals → 7 factors + narrative |
| 4 | Forecasting & Valuation | `agent.forecast` | 1M/3M/6M/1Y cone + 68% confidence + analyst overlay |
| 5 | Ranking & Screening | `agent.ranking` | Universe 6,500 US + 600 EU + 2,500 ETF, sort/filter |
| 6 | Portfolio Intelligence | `agent.portfolio` | Avg Score, Diversity, upgrade/downgrade alerts |
| 7 | Trade Ideas & Backtest | `agent.trade` | ≥60% win-rate mining, chart markers, strategy +376% vs S&P |
| 8 | Platform & API | `agent.platform` | FastAPI/Node, Postgres, Redis, Meilisearch, Stripe hooks |
| 9 | Experience Frontend | `agent.frontend` | Next.js + Tailwind, charts, SEO, paywall |
| 10 | Risk, Compliance & Growth | `agent.growth` | Low-Risk Score, disclaimers, pricing tiers, analytics |

Each file: mission, responsibilities, skills table, tools/APIs, inputs/outputs, KPIs, example task, collaboration points.

### Skills (42) — in `/skills/registry.md`

Grouped: `ingest.*`, `feature.*`, `model.*`, `explain.*`, `forecast.*`, `rank.*`, `trade.*`, `portfolio.*`, `platform.*`, `ui.*`, `risk.*`, `growth.*`

Full matrix: which agent owns which skill, input→output, tools.

### Workflows (6) — in `/workflows/`

| ID | Name | Trigger | Agents |
|----|------|---------|--------|
| WF1 | **Daily Scoring Pipeline** | Cron 21:30 ET | data → quant → explain+forecast → ranking → platform → portfolio |
| WF2 | **Stock Detail Generation** | On-demand `/stock/:ticker` | frontend → platform → quant/explain/forecast |
| WF3 | **Trade Ideas Discovery** | Weekly + daily incremental | quant → trade → forecast → ranking |
| WF4 | **Portfolio Monitoring & Alerts** | On `scores.ready` + CRUD | portfolio → quant → ranking → platform (email/push) |
| WF5 | **ETF & Thematic Ranking** | After WF1 (23:00) | data → quant (holdings-weighted) → ranking |
| WF6 | **Acquisition & Conversion** | User events (search→paywall) | frontend → platform → growth (Stripe) |

Each with Mermaid sequence diagram, steps, data flow, error handling, SLOs.

---

## 🎨 Brand — Invelix

- **Name:** INVELIX = Invest + Helix (double helix of data × intelligence)
- **Palette:** Midnight `#0B1220`, Teal `#0EE6B7`, Emerald `#16C784`, Amber `#F59E0B`, Slate `#64748B`
- **Typography:** Inter + JetBrains Mono
- **Score:** 1-3 Red Strong Sell, 4 Orange Sell, 5-6 Amber Hold, 7-8 Teal Buy, 9-10 Emerald Strong Buy
- **Voice:** Precise, transparent, calmly confident. No hype. Probabilities, not promises.

See `docs/brand.md`.

---

## 🚀 Quick Start — Run the Site

No build needed — static Tailwind CDN.

```bash
# from repo root
cd invelix
python3 -m http.server 8000 --bind 0.0.0.0
# open http://localhost:8000/index.html
# or double-click invelix/index.html
```

In Arena preview, server binds to `0.0.0.0` with proxy `https://{port}-{sandbox}.e2b.app`.

---

## 📂 Repository Layout

```
/README.md
/docs/
  brand.md
  architecture.md
/agents/
  01-data-ingestion-agent.md
  02-quant-ai-core-agent.md
  03-explainability-agent.md
  04-forecasting-agent.md
  05-ranking-screening-agent.md
  06-portfolio-intelligence-agent.md
  07-trade-ideas-agent.md
  08-platform-api-agent.md
  09-frontend-experience-agent.md
  10-risk-compliance-growth-agent.md
/skills/
  registry.md
/workflows/
  WF1-daily-scoring-pipeline.md
  WF2-stock-detail-generation.md
  WF3-trade-ideas-discovery.md
  WF4-portfolio-monitoring.md
  WF5-etf-thematic-ranking.md
  WF6-acquisition-conversion.md
/invelix/
  index.html
  us-stocks.html
  stock-aapl.html
  trade-ideas.html
  etfs.html
  portfolios.html
  pricing.html
  how-it-works.html
  data/rankings.json
```

---

## 🔍 Danelfin Parity Checklist

| Danelfin | Invelix | Harness Owner |
|----------|---------|---------------|
| AI Score 1-10 (3M) | Invelix Score 1-10 | agent.quant |
| 900 indicators → 10k features, 5B learned | Same | agent.data |
| Explainable AI + alpha signals ±% | Same, 28 signals + long-tail | agent.explain |
| F/T/S/LowRisk sub-scores | Same 1-10 each | agent.quant + agent.growth |
| US 6,500 + STOXX 600 + ETF 2,500 rankings | Same | agent.ranking |
| Country filters + Perf YTD | Same | agent.ranking |
| Trade Ideas ≥60% win-rate since 2017 | Same | agent.trade |
| Price forecast 1M/3M/6M/1Y + analysts | Same, 68% confidence | agent.forecast |
| Trading Parameters (Entry/Stop/Take) | Same, Pro-gated | agent.forecast |
| Win-rate table (1260 Hold → 86.59%) | Same | agent.trade |
| Score Evolution chart (D/W/M/Q) | Same | agent.frontend |
| Portfolio Avg + Diversity + alerts | Same | agent.portfolio |
| Backtest +376% vs S&P +166% | Same | agent.quant |
| IPO tracker | Same | agent.data |
| Search + paywall + Stripe ($0/$29/$99) | Same | agent.platform + growth |
| How it Works infographic | Same | agent.frontend |

---

## 🧪 Harness Usage Example

**Add a new workflow** — e.g., “Daily Sector Rotation Report”:

1. Define skill `rank.sector_leaders` in `skills/registry.md`
2. Assign to `agent.ranking`
3. Create `workflows/WF7-sector-rotation.md` referencing `agent.ranking → agent.explain → agent.platform`
4. Add UI in `invelix/` via `agent.frontend`’s `ui.render_table`

---

## ⚠️ Disclaimer

Mock data for demo. Invelix Score reflects probability based on historical patterns; past performance does not guarantee future results. Not financial advice. No autotrading. Live vendor APIs (Polygon, FMP, SEC EDGAR) to be configured for production.

---

Built as Arena harness demo — `drmohammadzadeh/AI-Powered-Stock-Picking-Agens` • Branch `arena/01a09495-ai-powered-stock-picking-agens`
