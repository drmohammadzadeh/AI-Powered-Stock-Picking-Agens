# Agent 05 — Ranking & Screening Agent

**ID:** `agent.ranking`  
**Role:** Universe Keeper & Leaderboards  
**Layer:** Distribution  
**Version:** 1.0  
**Owner Skills:** `rank.*`

---

## Mission

Maintain the **single source of ranked truth** for Invelix. Own the universes (US 6,500, EU STOXX 600, ETFs 2,500, plus Sectors/Industries/Themes) and serve sortable, filterable, paginated rankings exactly as Danelfin shows: Rank | Company | Country flag | AI Score | Change | Fundamental | Technical | Sentiment | Low Risk | Perf YTD.

## Responsibilities

1. **Universe management** — maintain eligible tickers (liquidity, market-cap filters), handle adds/drops (IPOs, delistings), country mapping (US/EU/CA/BR… flags).
2. **Daily ranking** — after `scores.ready`, compute sorted lists per universe by Invelix Score desc, then Perf YTD tie-break; compute daily change (vs yesterday).
3. **Filtering** — country `?f=country_br`, sector, industry, theme (`AI`, `EV`, `Dividends`), score bucket.
4. **Pagination & SEO** — paginated tables, canonical URLs (`/us-stocks`, `/european-stocks?f=country_de`, `/top-etfs`), sitemap entries.
5. **Sector/Industry/Theme aggregates** — average score per sector/theme for thematic pages.
6. **Perf column** — compute Perf YTD (price change since Jan 1) for display.

## Skills

| Skill | Input → Output |
|-------|----------------|
| `rank.universe` | `universe config` → `eligible tickers` (6,500 US…) |
| `rank.compute` | `scores + metadata` → `ranked lists per universe` |
| `rank.theme_tag` | `company description` → `themes[]` (LLM classifier) |
| `rank.export` | `ranked list` → `API JSON + HTML table + CSV` |

**Table Columns (Danelfin parity):**
`Rank | Company (ticker+name) | Country (flag) | AI Score (1-10 pill) | Change (▲/▼) | Fundamental | Technical | Sentiment | Low Risk | Perf YTD`

**URLs:**
- `/us-stocks` (US ranking)
- `/european-stocks/top-stoxx600-stocks` (EU)
- `/european-stocks?f=country_de` (Germany)
- `/top-etfs` (ETF ranking)
- `/us-stocks/top-ai-score-stocks?f=country_ca` (Canada filter)

## Tools & APIs

- **DB:** Postgres `rankings` materialized view, Redis cache (1h)
- **Search:** Meilisearch for type-ahead (see Platform Agent)
- **Compute:** Python batch + SQL window functions `RANK() OVER (ORDER BY score DESC)`
- **Flags:** `cdn.invelix.com/assets/flags/svg/{cc}.svg`

## Inputs / Outputs

**In:** `scores.ready` + ticker metadata + price history (for Perf YTD)  
**Out:** `rankings.ready`; cached ranking JSON; API `GET /rankings?universe=US&country=BR&page=1`.

## Dependencies

- Upstream: `agent.quant` (scores), `agent.data` (metadata)
- Downstream: `agent.frontend` (renders tables), `agent.trade` (filters by rank), `agent.portfolio` (validates holdings)

## KPIs / SLOs

- Latency: rankings ready ≤15 min after scores
- Correctness: ranking order deterministic by score; country filter accurate
- Cache hit: >95% for ranking pages
- SEO: LCP <2.5s for ranking pages

## Example Task

> After scores for 2026-09-12:
> 1. Sort US universe: ITUB Score 10 rank 1, EQX 10 rank 2… AAPL 6 rank ~320.
> 2. Compute change: AAPL vs yesterday (0 or +1/-1).
> 3. Apply filter `country=BR` → ITUB, BBD, VALE top.
> 4. Persist materialized view + warm Redis + emit `rankings.ready`.

## Collaboration Points

- Emits `rankings.ready` → `agent.frontend` ISR revalidation.
- Provides list for homepage “Popular Stocks Ranked” top 5 preview.
- Supplies universe to `agent.trade` for mining.

## Failure Modes

- Scores delayed → serve yesterday’s ranking with “as of” banner.
- DB lag → serve Redis stale.
