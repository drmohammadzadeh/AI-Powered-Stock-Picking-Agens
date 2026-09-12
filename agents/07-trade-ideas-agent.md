# Agent 07 — Trade Ideas & Strategy Backtest Agent

**ID:** `agent.trade`  
**Role:** Track-Record Miner — “Stocks With the Best Track Record”  
**Layer:** Distribution  
**Version:** 1.0  
**Owner Skills:** `trade.*`

---

## Mission

Surface **Trade Ideas** where Invelix AI has been historically sharp: Buy/Strong Buy signals with ≥60% win rate (positive performance) and Sell/Strong Sell with ≥60% loss rate since 2017. Render historical buy/sell markers on price chart + forecast for 1/3/6M, as on Danelfin’s Trade Ideas page. Also own strategy backtests (Best Stocks Strategy +376% vs S&P +166%).

## Responsibilities

1. **Mining** — per ticker, per signal type (Buy 7-8, Strong Buy 9-10, Sell 3-4, Strong Sell 1-2), compute win-rate and avg performance over horizons 1M/3M/6M/1Y since 2017.
2. **Filtering** — only surface ideas meeting ≥60% threshold; rank by win-rate × sample size × recency.
3. **Chart overlay** — provide buy/sell markers (green/red triangles) + forecast cone + historical signal track record.
4. **Backtest pages** — maintain `/best-stock-investment-strategy` equity curve, annual returns, Sharpe, max drawdown vs S&P 500.
5. **EU vs US** — separate Trade Ideas for US and Europe (`/trade-ideas`, `/european-trade-ideas`).

## Skills

| Skill | Input → Output |
|-------|----------------|
| `trade.mine` | `historical scores + prices (2017→present)` → `candidates where winRate >=0.60` |
| `trade.winrate` | `ticker, signalType, horizon` → `{avgPerf +30.35%, avgAlpha +17.89%, winRate 86.59%, positiveAlpha 70.80%, count 1260}` (per AAPL Hold example) |
| `trade.overlay` | `price series + signals` → `chart markers + forecast bands` |

**Danelfin Parity Table (AAPL Hold signals example):**

| 1260 Past Hold Signals | 1M Later | 3M Later | 6M Later | 1Y Later |
|------------------------|----------|----------|----------|----------|
| Avg Performance | … | … | … | +30.35% |
| Avg Alpha | … | … | … | +17.89% |
| Win Rate | … | … | … | 86.59% |
| % Positive Alpha | … | … | … | 70.80% |

Only ideas with ≥60% win (Long) or ≥60% loss (Short) are shown; forecast row adds AI projection.

## Tools & APIs

- **Compute:** Python backtester (vectorized), Postgres `signal_performance` table
- **Chart:** TradingView Lightweight overlay data
- **Schedule:** Weekly full mine + daily incremental after scores

## Inputs / Outputs

**In:** `scores.ready` + historical price + historical scores  
**Out:** `trade-ideas.ready`; API `GET /trade-ideas?universe=US`, `GET /stock/:ticker/winrate`.

## Dependencies

- Upstream: `agent.quant`, `agent.ranking`, `agent.forecast`
- Downstream: `agent.frontend` (Trade Ideas pages, stock chart)

## KPIs / SLOs

- Threshold integrity: no idea shown below 60%
- Sample size: ≥50 signals per idea (avoid small-sample luck)
- Recency: mine weekly; incremental daily
- Coverage: ≥200 ideas US + ≥80 EU at any time

## Example Task

> 1. For each ticker, compute 3M forward return after every Buy signal (Score ≥7) since 2017.
> 2. ITUB: 420 Buys → 68% win, avg +18% → qualify.
> 3. Emit Trade Idea: ITUB Buy, win-rate 68%, forecast +5.2% 3M.
> 4. Push to `/trade-ideas` ranked list.

## Collaboration Points

- Consumes backtest numbers from `agent.quant`.
- Provides markers to `agent.frontend` chart.
- Filtered by `agent.ranking` top scores for relevance.

## Failure Modes

- Low liquidity ticker → exclude (spread filter).
- Backtest compute heavy → chunked, cached.
