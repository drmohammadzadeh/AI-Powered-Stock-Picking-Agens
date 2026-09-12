# Agent 08 — Platform & API Agent

**ID:** `agent.platform`  
**Role:** Backend Platform — Auth, DB, Cache, Search  
**Layer:** Platform  
**Version:** 1.0  
**Owner Skills:** `platform.*`

---

## Mission

Be the **reliable platform** that serves 150k investors. Own REST/GraphQL API, auth & entitlements, Postgres, Redis cache, Meilisearch, rate limiting, and Stripe webhooks. Guarantee <200ms p95 for ranking pages, correct freemium gates (free = 10 stock views), and idempotent daily score ingestion.

## Responsibilities

1. **API design** — endpoints: `GET /stocks`, `GET /stock/:ticker`, `GET /rankings`, `GET /trade-ideas`, `GET /forecast`, `POST /portfolios`, `GET /search`, `POST /webhook/stripe`.
2. **Auth & tiers** — Clerk/NextAuth + JWT; tiers Free/Starter/Plus/Pro with feature flags; enforce 10-stock limit for free (paywall modal).
3. **Database** — Prisma + Postgres: tables `stocks`, `scores`, `alpha_signals`, `forecasts`, `rankings_mv`, `portfolios`, `users`, `entitlements`.
4. **Cache** — Redis: `score:{ticker}:today` TTL 1h, `ranking:{universe}:{country}:page` TTL 1h, stale-while-revalidate.
5. **Search** — Meilisearch index `stocks` (ticker, name, theme) for hero autocomplete; debounced 150ms.
6. **Ingestion API** — internal `POST /internal/scores` idempotent upsert from `agent.quant`.
7. **Webhooks** — Stripe checkout → entitlements update → unlock signals/forecasts.

## Skills

| Skill | Input → Output |
|-------|----------------|
| `platform.auth` | `request` → `user + tier` (free sees 10 signals, Pro sees all) |
| `platform.crud` | `DB ops` → `persist` |
| `platform.cache` | `key` → `cached value` or miss |
| `platform.search` | `query "AAP"` → `[{AAPL Apple}, {AAP Adv Auto}]` |
| `platform.webhook` | `stripe event` → `entitlement` |
| `platform.rate_limit` | `IP/user` → `429 if exceeded` |

**API Examples:**
```
GET /api/stocks/AAPL → {ticker, name, scores, alphaSignals, forecast}
GET /api/rankings?universe=US&page=1 → {items, total, page}
GET /api/search?q=tesla → [{ticker:TSLA, name:Tesla}]
POST /api/portfolios → {id}
GET /api/trade-ideas?universe=US → [idea...]
```

## Tools & APIs

- **Backend:** Node 20 + FastAPI (for ML internal), Prisma, Postgres 16, Redis 7, Meilisearch
- **Auth:** Clerk / NextAuth, Stripe
- **Infra:** Vercel Functions / Fly.io, Sentry, Datadog APM
- **Ops:** Rate limit via Upstash, idempotency keys

## Inputs / Outputs

**In:** Events from other agents (`scores.ready`, `stripe webhook`), frontend requests  
**Out:** HTTP responses, cache warms, DB writes

## Dependencies

- Upstream: All intelligence agents publish via this agent’s internal ingest.
- Downstream: `agent.frontend` is primary consumer.

## KPIs / SLOs

- Availability: 99.9%
- Latency p95: <200ms for cached, <600ms for miss
- Correctness: freemium gate never leaks Pro signals to free
- Idempotency: duplicate `scores.ready` does not duplicate rows

## Example Task

> Frontend calls `GET /api/stocks/AAPL`.
> 1. Check Redis `score:AAPL:2026-09-12` → hit → return + tier filter (free sees 10 signals, rest “Upgrade to unlock”).
> 2. On miss, query Postgres + join alpha_signals.
> 3. Log to analytics via `agent.growth`.

## Collaboration Points

- Receives `POST /internal/scores` from `agent.quant` → writes DB + warms Redis → emits to frontend ISR.
- Notifies `agent.portfolio` on portfolio CRUD.
- Enforces `agent.growth` pricing flags.

## Failure Modes

- DB down → serve Redis stale + banner.
- Meilisearch down → fallback Postgres ILIKE.
- Stripe webhook fail → retry queue.
