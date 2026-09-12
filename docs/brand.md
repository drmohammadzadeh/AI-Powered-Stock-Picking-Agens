# Invelix — Brand & Product Identity

**Brand Name:** Invelix  
**Tagline:** *Invest with the odds in your favor.*  
**Domain Concept:** invelix.com  
**Language:** English (primary)  
**Positioning:** AI-Powered Stock Picking — The Danelfin-class alternative, built as an open harness.

> Inspired by `danelfin.com` — Invelix reproduces 100% of Danelfin capability while exposing the underlying **AI Harness** (agents + skills + workflows) so the product is buildable, auditable, and extensible.

---

## 1. Brand Story

**Invelix = Invest + Helix** — the double helix of data & intelligence spiraling into better decisions. Where Danelfin says “AI Score 1-10”, Invelix says **Invelix Score 1-10**: transparent, explainable, daily.

Mission: Democratize institutional-grade quantitative research for 150k+ retail investors through explainable AI.

Personality: **Precise. Transparent. Calmly confident.** Not hype. No black boxes. Numbers with narrative.

## 2. Visual Identity

**Logo Concept:** Interlocked “I” + “X” forming an upward helix / candlestick. Wordmark `INVELIX` with “VELI” in weight 700, “IN” + “X” in 400. Icon: abstract helix in gradient.

**Palette:**
- Primary Midnight: `#0B1220` (backgrounds, nav)
- Invelix Teal: `#0EE6B7` (primary CTA, score 7-10)
- Signal Green: `#16C784` (positive alpha)
- Signal Red: `#EA3943` (negative alpha)
- Amber: `#F5A623` (Hold / 5-6)
- Surface: `#F8F9FB` / Card: `#FFFFFF`
- Slate: `#64748B` (secondary text)
- Border: `#E2E8F0`

**Typography:**
- Display: Inter / Sora 700-800 (headlines)
- Body: Inter 400-500
- Mono: JetBrains Mono (tickers, prices, scores)

**Score Visualization:**
- 1-3: Red pill — Strong Sell
- 4: Orange — Sell
- 5-6: Amber — Hold
- 7-8: Teal — Buy
- 9-10: Emerald — Strong Buy
- Gradient bar low→high with marker

## 3. Tone of Voice

English, global. Short sentences. Evidence over promises.
- “Probability of beating the market in 3 months” not “will beat the market”
- “Our AI analyzes 10,000+ features per stock per day” not “magic AI”
- Always show: source, horizon, confidence, disclaimer.

## 4. Product Principles (Danelfin Parity)

1. **Daily AI Score 1-10** for every US stock + STOXX 600 + US ETFs
2. **Explainable AI** — every score broken into Fundamental / Technical / Sentiment alpha signals with impact %
3. **Sub-scores:** Fundamental, Technical, Sentiment, Low Risk (1-10 each)
4. **Rankings:** US, Europe, ETFs, Sectors, Industries, Themes + country filters + pagination
5. **Stock Detail Page:** Score explanation, Alpha Factors, Alpha Signals table, Trading Parameters, Forecasts (1M/3M/6M/1Y), Win Rate table, Score Evolution chart, News
6. **Trade Ideas:** ≥60% win-rate Buy and ≥60% loss-rate Sell signals since 2017
7. **Portfolio Tools:** Average AI Score, Diversity Score, daily upgrade/downgrade alerts, evolution tracking
8. **Strategies:** Backtested Best Stocks vs S&P 500 track record
9. **Search + Watchlist + Alerts**
10. **Freemium:** Free (10 stocks), Starter ($29), Plus ($49), Pro ($99)

## 5. Competitive Parity Checklist vs Danelfin.com

| Danelfin Capability | Invelix Equivalent | Status in Harness |
|---|---|---|
| AI Score 1-10 (3M horizon) | Invelix Score 1-10 | Quant Agent |
| 900+ indicators → 10k features | Same pipeline | Data Agent |
| Explainable AI + alpha signals | Identical UX | Explainability Agent |
| Fundamental/Technical/Sentiment/Low Risk | Same 4 sub-scores | Quant Agent |
| US + STOXX 600 + ETFs | Same universe | Ranking Agent |
| Trade Ideas (60% win) | Invelix Trade Ideas | Trade Ideas Agent |
| Portfolio Avg Score + Diversity | Same | Portfolio Agent |
| Forecast 1/3/6/12M + confidence | Same | Forecasting Agent |
| Score Evolution chart | Same | Frontend Agent |
| Alerts (upgrade/downgrade) | Same | Portfolio Agent |
| Pricing/ Paywall | Same tiers | Growth Agent |
| IPO tracker | Same (stretch) | Data Agent |

## 6. Domain Map (Sitemap)

- `/` — Homepage (hero search, top rankings preview, use cases, data banner, testimonials)
- `/us-stocks` — US Ranking
- `/european-stocks` — EU STOXX 600 Ranking
- `/top-etfs` — ETF Ranking
- `/trade-ideas` — Trade Ideas US + EU
- `/stock/:ticker` — Stock Detail (+ #ai-analysis, #forecast, #score-evolution-chart)
- `/stock/:ticker/forecast` — Full forecast
- `/stock/:ticker/advanced-chart` — Advanced chart
- `/user/my-portfolios` — Portfolios
- `/best-stock-investment-strategy` — Strategy backtest
- `/how-it-works/infographic` — Explain methodology
- `/pricing` — Plans
- `/ipo/:slug` — IPO pages
- `/api/*` — Backend API (see Platform Agent)

## 7. Legal

Disclaimer required on every scored page: “Nobody can predict the future. Invelix Score reflects probability based on historical patterns; past performance does not guarantee future results.”
