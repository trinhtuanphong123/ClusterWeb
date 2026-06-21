# Project Brief — Vietnamese Stock Behavior Clustering & Historical Pattern Explanation Dashboard

## One-line summary

A decision-support analytics dashboard that groups Vietnamese-listed stocks by *how they are currently moving* (behavior similarity), then uses historical pattern retrieval to explain and contextualize each group. **This is not a price-prediction model and not a buy/sell recommendation system.**

## Goal

At any given time `T`, answer two questions:

1. **Which stocks are currently behaving alike?** Cluster the stock universe by recent movement behavior (returns, volatility, drawdown, volume, market- and sector-relative behavior) rather than by raw price.
2. **What has historically followed behavior like this?** Retrieve historically similar windows for each stock/cluster and summarize what tended to happen afterward — as *context*, not as a forecast.

## Non-goals

- **No price prediction.** The system never outputs a predicted price or target.
- **No trading signals.** No buy/sell/hold recommendations, no position sizing, no automated execution.
- **No financial advice.** Outputs are descriptive analytics for human interpretation.
- **No tick-level / real-time streaming** in the MVP. Intraday state is refreshed hourly at most.
- **No fundamental analysis** (earnings, valuation models) in the MVP — behavior is derived from price/volume only.

## Users

- **RRetail users who want descriptive market-structure analytics, not investment advice.** who want to understand current market structure ("what's grouping together right now").
- **Analysts and researchers** studying co-movement, rotation, and lead-lag relationships in the VN market.
- **Internal exploratory users** validating whether behavior clusters are stable and interpretable.

## Stack

- **Data source:** VNStock community/free API (daily OHLCV as the official source; 5-minute OHLCV only for hourly intraday dashboard state).
- **Storage:** Raw layer → normalized tables → feature tables → rolling behavior windows → cluster/pattern outputs → dashboard snapshots.
- **Compute:** Python (pandas / numpy / scikit-learn for clustering and similarity).
- **API:** FastAPI serving precomputed snapshots.
- **Frontend:** Dashboard deployed to Vercel, reading from the FastAPI/snapshot layer.
- **Scheduling:** Daily official pipeline + hourly intraday refresh.

## MVP scope

- Ticker universe sync from VNStock.
- Daily OHLCV historical bootstrap + daily incremental updates.
- Feature engineering: returns, normalized price paths, volatility, drawdown, volume behavior, market-relative return, sector-relative return.
- Rolling behavior windows per stock.
- **Current behavior clustering** (the main output).
- **Historical similarity retrieval** to explain clusters.
- Lead-lag and historical-outcome summaries as supporting layers.
- Current-pattern watchlist.
- Hourly intraday snapshot from 5-minute OHLCV for the "live-ish" dashboard state.
- Trading-day and non-trading-day handling.

## Data flow (high level)

```
VNStock API
  → Raw storage (as-fetched)
  → Normalized tables (clean OHLCV, deduped, typed, timezone-fixed)
  → Feature tables (returns, vol, drawdown, volume, mkt/sector-relative)
  → Rolling behavior windows
  → Clustering outputs  +  Historical pattern outputs
  → Dashboard snapshots
  → FastAPI
  → Vercel dashboard
```

Two separate tracks: a **daily official clustering flow** (authoritative, from daily OHLCV) and an **hourly intraday flow** (from 5-minute OHLCV, for current-state display only).

## Modeling approach

- **Represent behavior, not price.** Each stock's recent window is converted into behavior features (shape of the return path, dispersion, downside, participation, relative strength vs market and sector).
- **Cluster on similarity** of these representations so that "stocks moving the same way" land together — independent of absolute price level.
- **Retrieve historical analogs** for each current window via similarity search over past windows, then summarize the distribution of subsequent outcomes for explanation/validation only.
- No supervised price target is fit; the system is descriptive and retrieval-based.

## Dashboard outputs

- Current behavior clusters (membership, cluster profiles, representative stocks).
- Per-stock / per-cluster historical analogs with outcome distributions (framed as context).
- Lead-lag relationships between stocks/clusters.
- Current-pattern watchlist (stocks whose current window matches notable historical patterns).
- Hourly intraday snapshot of current state on trading days; latest valid snapshot on non-trading days.

## Deployment target

- **Frontend:** Vercel.
- **Backend/API:** FastAPI serving precomputed snapshots.
- **Pipelines:** scheduled daily (official) and hourly (intraday).

## Limitations

- Stock prices are stochastic, noisy, and non-stationary; clusters describe *recent* behavior and can shift.
- Historical analogs are **descriptive context, not predictions**; past outcome distributions do not guarantee future results.
- Data quality depends on the VNStock free/community API (coverage, rate limits, gaps); intraday 5-minute data is treated as provisional.
- The MVP refreshes intraday state hourly, not in real time.
- The system is a **decision-support analytics dashboard, not a trading system** — no signals, recommendations, or execution.
