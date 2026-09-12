# Agent 09 — Experience Frontend Agent

**ID:** `agent.frontend`  
**Role:** Product Experience — Pixel-Perfect Invelix  
**Layer:** Platform  
**Version:** 1.0  
**Owner Skills:** `ui.*`

---

## Mission

Ship a **Danelfin-identical** experience under the **Invelix** brand: dark/light, responsive, blazing fast, SEO-perfect. Every page must feel institutional yet approachable — score pills, ranking tables, evolution charts, forecast cones, and the 6 use-case sections from Danelfin homepage. Own web performance, accessibility, and SEO.

## Responsibilities

1. **Design system** — Tailwind + Radix + custom ScorePill, Flag, Table, Chart components; midnight/teal palette (see brand.md).
2. **Pages (Danelfin parity):**
   - `/` homepage: hero search + app QR, Top 5 ranking preview, country pills, testimonials, AI Score scale 1-10, Data Analyzed banner (+900 / +10k / +5B), 6 use cases (Pick Winners, Generate Returns, Trade Ideas, Understand Features, Track Portfolio, Identify Moment), Upcoming IPOs strip, video, Featured logos, footer.
   - `/us-stocks`, `/european-stocks`, `/top-etfs` ranking tables
   - `/trade-ideas` + `/european-trade-ideas`
   - `/stock/:ticker` detail (7 sections: Score Explanation, Alpha Factors, Alpha Signals table, Trading Parameters, Forecast cone + analyst toggle, Win Rate table, Evolution chart, FAQs)
   - `/stock/:ticker/forecast`, `/stock/:ticker/advanced-chart`
   - `/user/my-portfolios` dashboard
   - `/best-stock-investment-strategy` backtest
   - `/how-it-works/infographic`, `/pricing`, `/ipo/:slug`
3. **Charts** — Recharts / TradingView Lightweight: score evolution (daily/weekly/monthly/quarterly), forecast cone (high/avg/low), price + signal markers.
4. **Search** — hero autocomplete (stocks/ETFs/themes) via `platform.search`, keyboard nav, recent.
5. **Paywall** — blur + “Upgrade to unlock” for signals/forecasts beyond free tier; Stripe Checkout modal.
6. **Perf & SEO** — ISR for rankings (revalidate 3600), dynamic OG, JSON-LD, sitemap, next/font, image optimization.

## Skills

| Skill | Input → Output |
|-------|----------------|
| `ui.render_score` | `score 6 Hold` → `amber pill 6 + gradient bar marker at 56%` |
| `ui.render_table` | `rankings` → `responsive table with flags, pills, change arrows` |
| `ui.chart_evolution` | `evolution[]` → `line chart with timeframe toggle` |
| `ui.chart_forecast` | `forecast + analyst` → `cone chart with confidence note` |
| `ui.search_bar` | `query` → `autocomplete dropdown` |
| `ui.seo` | `page` → `title, meta, OG, JSON-LD, sitemap` |

**Key UI Patterns (from Danelfin screenshots):**
- Score 1-10 pills colored red→amber→teal→emerald
- Ranking table sticky header, hover row, flag SVGs
- Alpha Signals table: Type badge, Signal name, Value, Decile dots (●○○), Impact green/red + progress bar
- Trading Parameters card: Entry $326.61, Horizon 3M, Stop Loss blurred for Pro
- Forecast toggle: 1M/3M/6M/1Y + confidence 68% + upside %

## Tools & APIs

- **Framework:** Next.js 14 (App Router), TypeScript, Tailwind CSS, Radix UI
- **Charts:** Recharts, TradingView Lightweight Charts
- **State:** TanStack Query, Zustand
- **Search:** Meilisearch instantsearch
- **Auth:** Clerk
- **Analytics:** PostHog, Sentry

## Inputs / Outputs

**In:** API JSON from `agent.platform`, design tokens  
**Out:** HTML/JS served via Vercel CDN; ISR revalidated on `rankings.ready` webhook.

## Dependencies

- Upstream: `agent.platform` (data), `agent.growth` (pricing flags)
- Downstream: End users

## KPIs / SLOs

- LCP <2.0s, CLS <0.05, Lighthouse 95+
- SEO: 100% pages indexed, sitemap daily
- Accessibility: WCAG 2.1 AA, keyboard nav for search/table
- Conversion: homepage→ranking CTR >45%, ranking→detail CTR >30%

## Example Task

> Build `/stock/AAPL`:
> 1. Fetch `GET /api/stocks/AAPL` → render Score 6 Hold, sub-scores 8/5/5/6.
> 2. Render Alpha Factors bars + Signals table (10 free, 18 blurred).
> 3. Render Trading Parameters card (blurred Pro fields).
> 4. Render Forecast cone toggle (default 3M) + Win Rate table (1260 Hold signals).
> 5. Render Evolution chart (daily default) + FAQ.

## Collaboration Points

- Listens to `rankings.ready` webhook to ISR revalidate `/us-stocks` etc.
- Calls `platform.search` on every keystroke (debounced).
- Shows upgrade modal via `agent.growth` entitlements.

## Failure Modes

- API timeout → skeleton + retry + stale banner.
- Empty evolution → show “Not enough history”.
- Mobile table → card layout fallback.
