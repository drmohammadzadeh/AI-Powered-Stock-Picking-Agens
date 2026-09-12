# Agent 04 — Forecasting & Valuation Agent

**ID:** `agent.forecast`  
**Role:** Price Forecast Engine (1M / 3M / 6M / 1Y)  
**Layer:** Intelligence  
**Version:** 1.0  
**Owner Skills:** `forecast.*`

---

## Mission

For every scored ticker, produce **AI price forecasts** with a cone (low / avg / high) for horizons 1M, 3M, 6M, 1Y, with confidence (68%) and independent accuracy tracking. Show them vs. analyst targets to help users compare. This mirrors Danelfin’s “Invelix AI vs Analysts” chart and unlock page.

## Responsibilities

1. **Horizon forecasts** — train separate models (or multi-horizon) to predict price range; output: avg, high, low, upside %.
2. **Confidence intervals** — 68% interval derived from residual distribution; explain why dotted line is “most likely path”.
3. **Analyst overlay** — ingest consensus price target (FMP) to plot vs AI.
4. **Accuracy tracking** — weekly compute hit-rate, MAE vs analysts; publish `/ai-stock-forecast-accuracy` page.
5. **Chart data** — provide time series for cone chart + table (e.g., AAPL $332.17 avg +1.70% upside, high $372.53, low $272.88 at 3M).

## Skills

| Skill | Input → Output |
|-------|----------------|
| `forecast.horizon` | `features + scores` → `{1M:{avg 328.44 +0.56%}, 3M:{avg 332.17 +1.70%, high 372.53 low 272.88}, 6M:{337.92}, 1Y:{350.02}}` |
| `forecast.confidence` | `residuals` → `68% interval, dotted line` |
| `forecast.evaluate` | `past forecasts vs realized` → `accuracy page metrics` |

**AAPL Example (from Danelfin parity):**
- Last close: $326.61
- 3M avg: $332.17 (+1.70%), high $372.53, low $272.88, confidence 68%
- 1M avg: $328.44 (+0.56%), 6M $337.92 (+3.46%), 1Y $350.02 (+7.17%)
- Analysts avg $339.15 (+3.84%) shown alongside.

## Tools & APIs

- **Models:** Gradient boosting regressors + quantile regression for intervals
- **Data:** OHLCV history, scores, analyst targets (FMP)
- **Evaluation:** Backtest evaluator, MAE, hit-rate dashboards

## Inputs / Outputs

**In:** `scores.ready`, historical prices, analyst targets  
**Out:** `forecasts.ready`; Postgres `forecasts` table; API `GET /stock/:ticker/forecast`; chart JSON.

## Dependencies

- Upstream: `agent.quant`, `agent.data`
- Downstream: `agent.frontend` (cone chart), `agent.trade` (uses forecast for trade ideas filter)

## KPIs / SLOs

- Coverage: 100% of liquid tickers
- Latency: <20 min after scores
- Accuracy: publish weekly; MAE tracked; beaten-analyst % reported
- Freshness: forecasts recomputed daily

## Example Task

> For AAPL after scoring:
> 1. Pull 5Y price + features → predict 3M distribution.
> 2. Emit `{horizon: "3M", avg: 332.17, high: 372.53, low: 272.88, confidence: 0.68}`.
> 3. Evaluate last quarter’s forecasts → update accuracy page.

## Collaboration Points

- Feeds `agent.trade` expected return for filtering trade ideas.
- Provides data for `agent.frontend` to render cone vs analysts toggle.

## Failure Modes

- Insufficient history → no forecast, show “Insufficient data”.
- Analyst data missing → show AI only.
