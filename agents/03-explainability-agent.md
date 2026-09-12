# Agent 03 — Explainability & Alpha Signals Agent

**ID:** `agent.explain`  
**Role:** Explainable AI — No Black Boxes  
**Layer:** Intelligence  
**Version:** 1.0  
**Owner Skills:** `explain.*`

---

## Mission

Make every Invelix Score **inspectable and trustworthy**. For each ticker, decompose the probability advantage (+5.27% for AAPL) into ranked **alpha signals** with precise impact (±%) and grouped **Alpha Factors** (Valuation +2.60%, Momentum +1.30%, …). Generate human-readable narratives so a retail investor understands *why* a score is 6, not just *what* it is. This is Danelfin’s key differentiator — Invelix reproduces it fully.

## Responsibilities

1. **SHAP attribution** — per-stock, per-date SHAP values for 10k features → map to interpretable signals (e.g., “Net PP&E $51,431M top 10% +0.85%”).
2. **Signal ranking** — select top 28 signals by |impact| + aggregate long-tail (Fundamentals Impact +1.83%, Sentiment Impact +0.56%).
3. **Factor grouping** — roll up signals into 7 factors: Valuation, Momentum, Sentiment, Size & Liquidity, Financial Strength, Growth, Earnings Quality, Volatility (with net impact per factor).
4. **Narrative generation** — LLM template that writes the “AI Score Explanation” paragraph (see AAPL example).
5. **Decile & value annotation** — for each signal, show current value + decile dot (top 10%/20%) + impact.
6. **Paywall handling** — top 10 signals free, rest behind `Upgrade to unlock` gate (teaser +0.33% etc.).

## Skills

| Skill | Input → Output | Details |
|-------|----------------|---------|
| `explain.shap_attribute` | `features + model` → `per-feature impact %` | TreeSHAP for LGBM/XGB, Gradient SHAP for MLP; sum to advantage |
| `explain.group_factors` | `signals` → `7 factors with net impact` | Valuation +2.60%, Momentum +1.30%, Sentiment +0.83%, Size +0.29%, Financial Strength +0.20%, Growth +0.15%, Earnings Quality +0.05%, Volatility -0.15% |
| `explain.rank_signals` | `all signals` → `top 28 + long-tail + rest` | Sorted by |impact|; include decile position, value, impact |
| `explain.narrate` | `signals + factors` → `paragraph` | Template: “Strongest boost from long tail fundamentals (+1.83%), followed by Chart Pattern (180d) (+1.08%)… Chart Pattern (60d) top detractor (-0.87%)… Net PP&E top 10%… high P/B detracts (-0.32%)…” |

**AAPL Example Output (from Danelfin parity):**

| Type | Signal | Value | Decile | Impact |
|------|--------|-------|--------|--------|
| Fundamental | Fundamentals Impact (Long Tail) | — | N/A | +1.83% |
| Technical | Chart Pattern (180d) | figure_145 | — | +1.08% |
| Technical | Chart Pattern (60d) | figure_168 | — | -0.87% |
| Fundamental | Net PP&E | 51.43K | top 10% | +0.85% |
| Technical | Chart Pattern (504d) | figure_40 | — | +0.71% |
| … | … | … | … | … |
| Fundamental | Price/Book (MRQ) | 44.16 | top 10% | -0.32% |
| Rest | — | — | — | +0.03% |
| **Total** | | | | **+5.27%** |

## Tools & APIs

- **Explain:** SHAP, Captum, custom factor taxonomy JSON
- **LLM:** OpenAI / Anthropic for narrative (templated, not hallucinated)
- **Store:** Postgres `alpha_signals` table + Redis

## Inputs / Outputs

**In:** `scores.ready` event + features + model artifacts  
**Out:** `explanations.ready` event; persisted explanations; API endpoint `GET /stock/:ticker/explanation`.

## Dependencies

- Upstream: `agent.quant` (scores), `agent.data` (raw values for display)
- Downstream: `agent.frontend` (renders explanation), `agent.portfolio` (uses factor tilts)

## KPIs / SLOs

- Coverage: 100% of scored tickers have ≥20 signals
- Latency: <30 min after `scores.ready` for full universe
- Consistency: sum(signals) = advantage ±0.01%
- Readability: narrative ≤180 words, 8th-grade level

## Example Task

> After AAPL scored 6:
> 1. Compute SHAP for 10k features → 28 top signals.
> 2. Group into factors: Valuation +2.60% etc.
> 3. Generate paragraph: “According to Invelix’s proprietary AI model, Apple today receives 6/10 (Hold)… strongest boost long tail +1.83%… Net PP&E $51,431M… high P/B 44.16 detracts…”
> 4. Persist + emit `explanations.ready`.

## Collaboration Points

- Waits for `scores.ready`, then fans out in parallel per ticker (async).
- Provides factor tilts to `agent.portfolio` for diversity analysis.
- Paywall metadata to `agent.growth` for gating.

## Failure Modes

- SHAP timeout → fallback to last day’s explanation + stale flag.
- LLM fail → use deterministic template without narrative.
