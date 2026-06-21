# Methodology — Current Stock Behavior Clustering with Historical Pattern Explanation

> This project is a **decision-support analytics dashboard**, not a prediction or trading system. It does **not** forecast prices and does **not** emit buy/sell recommendations.

## 1. What the project is

The project **clusters current stocks by behavior**: at time `T`, over a universe `S` of Vietnamese-listed tickers, it represents each stock's recent movement as a behavior vector and groups stocks that are currently moving in similar ways.

**The main task is current behavior clustering** — answering "which stocks are behaving alike right now."

## 2. The role of historical pattern retrieval

Historical pattern retrieval is **not the main task**. It is an **explanation and validation layer** layered on top of clustering. After clusters are formed, the system retrieves historically similar behavior windows to:

- **Explain** a cluster ("this current behavior resembles past windows X, Y, Z"), and
- **Validate** that clusters are coherent and interpretable.

Retrieval outputs are **descriptive context, never forecasts**.

## 3. Why stock price is stochastic, noisy, and non-stationary

- **Stochastic:** the realized price path is one draw from a random process driven by countless unobservable factors and order flow.
- **Noisy:** at short horizons, microstructure effects, liquidity, and one-off trades inject fluctuations unrelated to any persistent signal.
- **Non-stationary:** the statistical properties (drift, volatility, correlations) change over time with regimes and market structure; a relationship valid in one period may not hold in another.

Consequence: behavior representations must be **relative, scale-free, and recent**, and historical comparisons must be framed as context, not stable predictive laws.

## 4. Why raw price should not be clustered directly

- **Scale incomparability:** a stock at 10,000 and one at 150,000 are not comparable by price level; raw-price clustering groups by magnitude, not behavior.
- **Non-stationary level:** absolute price drift dominates and swamps the *shape* of movement.
- **Noise sensitivity:** level-dependent noise distorts distance measures.
- **Wrong similarity:** two stocks with identical movement shape at different price levels would never group together under raw-price clustering.

Therefore clustering operates on **behavior features derived from price/volume**, not on price itself.

## 5. Full methodology (seven steps)

1. **Collect data.** Ingest daily OHLCV (the official source) and 5-minute OHLCV (hourly current-state only) from the VNStock community/free API into raw → normalized storage.
2. **Compute rolling features.** Derive daily behavior features (returns, volatility, momentum, drawdown, volume anomaly, beta/correlation to index, market-relative and sector-relative returns, shape-supporting features) and hourly features for dashboard context.
3. **Build rolling behavior windows.** Assemble per-ticker rolling windows (5/20/60/120 trading days) ending at `T` into shape and feature vectors.
4. **Perform current stock behavior clustering.** Cluster the universe on behavior representations (baselines: KMeans/GMM/hierarchical; advanced: correlation graph + Leiden, k-Shape, DTW, MPdist similarity graph).
5. **Retrieve historical similar windows.** For each current window/cluster, find the most similar past windows, excluding the current period itself.
6. **Analyze future outcomes after similar historical patterns.** Summarize what historically followed the matched windows (forward returns, drawdowns, excess return vs index) by cluster and horizon — descriptive context only.
7. **Visualize clusters, explanations, and pipeline health.** Serve precomputed snapshots to the dashboard: current clusters, historical-pattern explanations, lead-lag relationships, outcome summaries, and pipeline/freshness status.

## 6. Framing and limitations

- Clusters describe **recent** behavior and can shift as regimes change.
- Historical analogs and outcome statistics are **descriptive, not predictive**.
- The system gives **no price targets and no trade signals**; it is decision-support analytics for human interpretation.
- Cross-references: see `FEATURE_SPEC.md`, `WINDOWING_SPEC.md`, `CLUSTERING_SPEC.md`, `HISTORICAL_PATTERN_RETRIEVAL_SPEC.md`, and `CLUSTER_OUTCOME_EVALUATION.md` for stage detail, and `MODEL_RUNTIME_STRATEGY.md` for execution.
- Future outcome statistics are computed only after clustering and retrieval are completed. They must not be used as input features for unsupervised clustering.
