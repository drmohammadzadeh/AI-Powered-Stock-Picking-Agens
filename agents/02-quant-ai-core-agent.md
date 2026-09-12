# Agent 02 — Quant AI Core & Scoring Agent

**ID:** `agent.quant`  
**Role:** Alpha Engine — The Brain That Scores  
**Layer:** Intelligence  
**Version:** 1.0  
**Owner Skills:** `model.*`

---

## Mission

Transform 10k features/stock/day into a single trustworthy number: **Invelix Score 1-10**, the probability of beating the market (S&P 500 / STOXX 600) in the next 3 months. Also produce the 4 sub-scores (Fundamental, Technical, Sentiment, Low Risk) that power rankings and portfolio analytics. The ensemble must be walk-forward validated since 2017 and outperform random by measurable alpha.

## Responsibilities

1. **Model inference** — daily, for entire universe (9k+ tickers), load ensemble (LightGBM + XGBoost + MLP) from MLflow registry, infer probability.
2. **Calibration** — Platt scaling → probability; bucket into 1-10 (decile of probability advantage vs. universe mean 50.73%).
3. **Sub-scores** — train 4 specialist models (fund/tech/sent/risk) or slice ensemble to produce 1-10 sub-scores.
4. **Training & retraining** — monthly retrain on 5B+ point-in-time samples; time-series CV; no leakage; MLflow versioning.
5. **Backtesting** — daily walk-forward from 2017-01-03, compute per-score alpha (Score 10 +21.05% annualized, Score 1 -33.28%), Strategy returns (+376% vs S&P +166%).
6. **Monitoring** — drift (PSI), accuracy, Sharpe of top decile; auto-retrain if degradation > threshold.

## Skills

| Skill | Description | Input → Output |
|-------|-------------|----------------|
| `model.train_ensemble` | Train ensemble on 5B samples | `feature_store` → `model artifact` (MLflow) |
| `model.infer_score` | Daily scoring | `features (10k) per ticker` → `{score 6, probability 56.00%, advantage +5.27%, subScores {F:8 T:5 S:5 LR:6}}` |
| `model.calibrate` | Probability calibration | `raw logit` → `calibrated prob` |
| `model.backtest` | Walk-forward validation | `historical features` → `alpha table, strategy equity curve` |
| `model.retrain_monitor` | Drift & retrain | `live predictions vs realized` → `retrain signal` |

**Scoring Example (AAPL):**
- Universe mean prob: 50.73%
- AAPL prob: 56.00% → advantage +5.27% → Score 6 (Hold)
- Sub-scores: Fundamental 8, Technical 5, Sentiment 5, Low Risk 6

## Tools & APIs

- **ML:** LightGBM, XGBoost, PyTorch (MLP), scikit-learn, SHAP (for downstream)
- **Tracking:** MLflow, Weights & Biases, Optuna (hyperparam)
- **Data:** Feast, DuckDB, Parquet on S3
- **Compute:** GPU (training), CPU batch (inference)
- **Orchestration:** Triggered by `feature_store.ready`

## Inputs / Outputs

**In:** `feature_store.ready` event + feature partitions  
**Out:** `scores.ready` event with:
```json
{
  "ticker": "AAPL",
  "date": "2026-09-12",
  "invelixScore": 6,
  "label": "Hold",
  "probability": 0.56,
  "advantage": 0.0527,
  "subScores": {"fundamental": 8, "technical": 5, "sentiment": 5, "lowRisk": 6}
}
```
Published to Postgres (`scores` table) + Redis cache.

## Dependencies

- Upstream: `agent.data` (features)
- Downstream: `agent.explain`, `agent.forecast`, `agent.ranking`, `agent.portfolio`

## KPIs / SLOs

- Inference latency: <15 min for full universe
- Coverage: 100% of ingested tickers scored
- Backtest: Score 10 alpha > +20% annualized (2017→present); monotonic alpha by score
- Calibration: Brier score <0.245; score bucket empirical win-rate matches predicted
- Freshness: `scores.ready` ≤ 23:00 ET

## Example Task

> 1. Consume `2026-09-12` features for 9,100 tickers.
> 2. Inference: AAPL logit → calibrated 56% → Score 6.
> 3. Persist to `scores` + emit `scores.ready`.
> 4. If drift detected (top decile hit-rate drop >3pp), flag `model.retrain_monitor`.

## Collaboration Points

- Emits `scores.ready` → `agent.explain` and `agent.forecast` can start.
- Feeds backtest numbers to `agent.trade` for win-rate mining.
- Provides sub-scores to `agent.ranking` for table columns.

## Failure Modes

- Model load fail → fallback to previous version + alert.
- Partial inference → mark unscored tickers, retry; ranking shows stale badge.
