# Project Spec — Vietnamese Stock Behavior Clustering & Historical Pattern Explanation Dashboard

## 1. Problem statement

Investors and analysts in the Vietnamese market lack a clear, structural view of *which stocks are currently moving in similar ways* and *what that behavior has historically meant*. Raw price charts are hard to compare across stocks (different price levels, volatility, and noise), and ad-hoc visual inspection does not scale across the full universe.

This project provides a **decision-support analytics dashboard** that:

- Clusters stocks by **current movement behavior**, and
- Retrieves **historical patterns** that resemble each stock's/cluster's current behavior, summarizing what tended to follow — as explanation and validation, not prediction.

**It is explicitly not a price-prediction model and not a buy/sell recommendation system.**

## 2. Target users

- **Retail / semi-professional investors** seeking to understand current market structure and rotation.
- **Market analysts / researchers** studying co-movement, sector dynamics, and lead-lag effects.
- **Internal users** validating cluster stability and interpretability before broader use.

## 3. MVP scope

In scope:

- Ticker universe synced from VNStock.
- Daily OHLCV bootstrap + daily incremental updates (the **official** data track).
- Feature engineering: returns, normalized paths, volatility, drawdown, volume behavior, market-relative return, sector-relative return.
- Rolling behavior windows per stock.
- **Current behavior clustering** (primary output).
- **Historical similarity retrieval** to explain/validate clusters.
- Supporting layers: lead-lag analysis, historical outcome analysis, current-pattern watchlist.
- Hourly intraday snapshot from 5-minute OHLCV for current-state display.
- Trading-day and non-trading-day handling (non-trading days show the latest valid snapshot).
- FastAPI serving snapshots; Vercel dashboard frontend.

Out of scope for MVP: real-time/tick streaming, fundamentals, portfolio optimization, alerts/notifications, multi-market coverage.

## 4. Non-goals

- **Not** a price forecaster (no predicted prices or targets).
- **Not** a recommendation engine (no buy/sell/hold, no sizing).
- **Not** financial advice.
- **Not** a real-time trading terminal (intraday refresh is hourly).
- **Not** a fundamentals platform.

## 5. Main outputs

1. **Current behavior clusters** — the central output. Each cluster contains stocks that are currently moving in similar ways, with a cluster profile (typical return shape, volatility, drawdown, volume, relative strength) and representative members.
2. **Historical pattern retrieval** — for each stock/cluster, the most similar historical windows, used to **explain and validate** the current cluster (e.g., "this behavior resembles past windows X, Y, Z").
3. **Historical outcome analysis** — distribution of what followed those analog windows, presented strictly as descriptive context.
4. **Lead-lag relationships** — which stocks/clusters have tended to move ahead of others.
5. **Current-pattern watchlist** — stocks whose current window strongly matches notable historical patterns.

The relationship between the two layers is deliberate: **clusters answer "who is behaving alike now," and historical retrieval answers "what has behavior like this looked like before,"** so retrieval serves the clustering layer rather than standing alone.

## 6. Update frequency

- **Daily official pipeline:** runs after market close on trading days; produces the authoritative clustering and historical-pattern outputs from daily OHLCV.
- **Hourly intraday refresh:** on trading days, refreshes the current-state snapshot from 5-minute OHLCV (hourly, not every 5 minutes).
- **Non-trading days:** no new official computation; dashboard serves the latest valid snapshot.

## 7. Success criteria

- **Coverage:** universe sync and daily bootstrap/incremental complete reliably for the supported tickers.
- **Cluster interpretability:** clusters are human-readable (members share visibly similar behavior) and reasonably stable day-to-day.
- **Explanatory usefulness:** retrieved historical analogs are plausibly similar to current windows and help users understand each cluster.
- **Freshness correctness:** trading-day snapshots reflect the latest available data; non-trading days correctly show the last valid snapshot.
- **Operational reliability:** daily and hourly pipelines run on schedule with idempotent updates and pass data-quality checks.
- **Clarity of framing:** the UI consistently communicates that outputs are descriptive context, not predictions or recommendations.
- **Leakage control:** clustering and historical retrieval use only data available up to the evaluation date; future outcomes are computed only after cluster formation for descriptive evaluation.

## 8. Limitations

- Prices are **stochastic, noisy, and non-stationary**; clusters describe recent behavior and can change as regimes shift.
- Historical analogs are **descriptive, not predictive**; outcome distributions are context only.
- Dependent on **VNStock free/community API** quality (coverage, gaps, rate limits). Exact limits are not assumed; they are recorded only after probing.
- **Intraday 5-minute data** is provisional and used solely for hourly current-state display, never as the official clustering source.
- MVP is **hourly, not real-time**.
- Behavior is derived from **price/volume only**; no fundamentals.
