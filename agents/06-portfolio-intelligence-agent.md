# Agent 06 — Portfolio Intelligence Agent

**ID:** `agent.portfolio`  
**Role:** Portfolio Coach & Alert Engine  
**Layer:** Distribution  
**Version:** 1.0  
**Owner Skills:** `portfolio.*`

---

## Mission

Help users **track, improve, and act** on their portfolios. Compute portfolio-level intelligence (Average Invelix Score, Diversity Score), monitor daily score evolution per holding, and push upgrade/downgrade alerts so users can “make the necessary adjustments” — core Danelfin value prop on homepage.

## Responsibilities

1. **Portfolio CRUD** — create/edit/delete portfolios, add holdings (ticker + weight or shares), validate against universe.
2. **Metrics** — Average Invelix Score (weighted), Diversity Score (0-10, penalizes concentration by sector/geo/market-cap), per-holding contribution.
3. **Evolution tracking** — daily snapshot per holding; sparkline of score 1-10 over time.
4. **Alerts** — detect upgrade (e.g., 5→7) or downgrade (7→4) vs yesterday; also threshold cross (below 4). Push email + in-app + push.
5. **Rebalance suggestions** — “Replace X (Score 3) with Y (Score 9) in same sector to raise Avg Score from 5.8→7.2 and keep diversity”.
6. **Freemium gates** — free: 1 portfolio / 10 holdings; paid: unlimited.

## Skills

| Skill | Input → Output |
|-------|----------------|
| `portfolio.create` | `user_id, holdings` → `portfolio_id` (validated) |
| `portfolio.avg_score` | `holdings × scores` → `avg 6.4` (weighted) |
| `portfolio.diversity` | `holdings × sector/geo` → `diversity 7/10` + concentration breakdown |
| `portfolio.alert` | `today vs yesterday scores` → `alerts[]` (“AAPL downgraded 7→6”, “TSLA upgraded 5→8”) |
| `portfolio.evolution` | `holding × date range` → `score history + suggestion` |

**Portfolio Tools (as on Danelfin homepage):**
- Average AI Score
- Portfolio Diversity Score
- Daily alerts for upgrades/downgrades

**Alert Example:**
> “⚠️ AAPL downgraded from 7 to 6 (Hold). Probability of beating market fell 4.2pp. Consider review.”

## Tools & APIs

- **DB:** Postgres `portfolios`, `holdings`, `portfolio_snapshots`
- **Queue:** Redis + BullMQ for alert jobs
- **Notify:** SendGrid (email), Firebase Cloud Messaging (push), in-app bell
- **Compute:** Python/Node for metrics

## Inputs / Outputs

**In:** `rankings.ready` / `scores.ready` + user portfolios  
**Out:** Updated portfolio metrics; `portfolio.alerts` events; API `GET /user/my-portfolios`, `POST /portfolios`.

## Dependencies

- Upstream: `agent.quant`, `agent.ranking`
- Downstream: `agent.platform` (auth), `agent.frontend` (dashboard)

## KPIs / SLOs

- Alert latency: <60 min after scores
- Accuracy: Avg Score matches weighted calc; Diversity Score stable
- Engagement: alert open rate >35%
- Freemium enforcement correct

## Example Task

> User has portfolio: AAPL (6), ITUB (10), VALE (10), TSLA (5).
> 1. Compute Avg = (6+10+10+5)/4 = 7.75.
> 2. Diversity: 2 Brazil, 1 US, 1 auto → 6/10 (moderate concentration).
> 3. Detect: TSLA 4→5 upgrade → emit alert.
> 4. Suggest: replace TSLA (5) with NVDA (9) tech sector → Avg 8.25, diversity 7/10.

## Collaboration Points

- Triggered by `scores.ready` cron 22:00 ET + real-time on portfolio edit.
- Feeds alert counts to `agent.frontend` bell icon.
- Uses `agent.ranking` to find replacement candidates.

## Failure Modes

- Score missing for holding → use last known + stale flag.
- Email fail → retry queue, in-app still shows.
