# Problem Definition — Vietnamese Stock Behavior Clustering

## 1. Main problem (formal)

**Given:**

- A point in time `T`.
- A stock universe `S = {s_1, s_2, ..., s_n}` of Vietnamese-listed tickers.
- For each stock `s_i`, a history of daily OHLCV up to `T`.
- A market reference series and a sector mapping for each stock.

**Compute:**

1. For each stock `s_i`, a **current behavior representation** `b_i = f(W_i)`, where `W_i` is a rolling window of recent observations ending at `T` and `f(·)` maps the window to a behavior feature vector (returns, normalized path shape, volatility, drawdown, volume behavior, market-relative return, sector-relative return).
2. A **clustering** `C = cluster({b_1, ..., b_n})` that partitions (or groups) the universe so that stocks with similar current behavior are placed together.

**Output:** cluster assignments, cluster profiles, and representative members at time `T`.

**Key constraint:** clustering operates on **behavior representations**, never on raw price levels.

This is the core of the system: *which stocks are currently behaving alike?*

## 2. Supporting problems

These exist to **explain, validate, and enrich** the main clustering output.

### 2.1 Historical similarity retrieval

For a current window `W_i` (or a cluster-level behavior profile), find the set of **past windows** `{W_i^(t1), W_i^(t2), ...}` (across the same stock and/or the universe) whose behavior representations are most similar to the current one. Used to answer: *"When has behavior like this happened before?"*

### 2.2 Lead-lag analysis

Estimate, among stocks and clusters, which series have **tended to move ahead of** others (e.g., via lagged cross-correlation of returns). Used to surface structural relationships within and across clusters.

### 2.3 Historical outcome analysis

For the analog windows retrieved in 2.1, summarize the **distribution of subsequent behavior** (e.g., forward return shape, drawdown, volatility over the window after the analog). Presented strictly as **descriptive context** that characterizes what historically followed similar behavior — **not a forecast**.

### 2.4 Current pattern watchlist

Flag stocks whose current window strongly matches historical windows or cluster archetypes that have been marked as notable by predefined descriptive criteria, such as high similarity, sufficient historical sample size, distinct post-window outcome distribution, or strong cluster stability. This is a watchlist for analyst attention, not a trading signal.

## 3. Why prices are stochastic, noisy, and non-stationary

- **Stochastic:** price moves are driven by a continuous flow of orders reflecting countless unobservable factors; the realized path is one draw from a random process.
- **Noisy:** at short horizons, microstructure effects, liquidity, and one-off order flow inject fluctuations unrelated to any persistent behavior signal.
- **Non-stationary:** the statistical properties (mean drift, volatility, correlations) **change over time** with regimes, liquidity conditions, and market structure. A relationship that holds in one period may not hold in another.

Consequence: any representation must focus on **relative, scale-free, recent behavior**, and any historical comparison must be framed as context rather than a stable predictive law.

## 4. Why raw price should not be clustered directly

- **Scale incomparability:** a stock trading at 10,000 and one at 150,000 are not comparable by price level; raw-price clustering would group by magnitude, not by behavior.
- **Non-stationary level:** absolute price drifts and trends dominate the signal, swamping the *shape* of movement we care about.
- **Noise sensitivity:** raw prices carry level-dependent noise that distorts distance measures.
- **Wrong notion of similarity:** two stocks can have identical movement *shape* at very different price levels — clustering on raw price would miss this similarity entirely.

Therefore the universe is clustered on **behavior features derived from price/volume**, not on price itself.

## 5. Why these specific features

The behavior representation is built from features chosen to be **scale-free, comparable across stocks, and behaviorally meaningful**:

- **Returns** — convert level to change; comparable across price scales; the basic unit of movement.
- **Normalized paths** — the *shape* of recent movement rebased so two stocks can be compared regardless of starting price.
- **Volatility** — magnitude/dispersion of movement; distinguishes calm vs turbulent behavior.
- **Drawdown** — downside behavior and peak-to-trough stress, capturing asymmetry that volatility alone misses.
- **Volume** — participation and conviction behind the movement; separates quiet drift from high-participation moves.
- **Market-relative return** — behavior **net of the broad market**, isolating stock-specific movement from market-wide moves.
- **Sector-relative return** — behavior **net of the sector**, isolating idiosyncratic behavior from sector rotation.

Together these describe *how* a stock is moving (shape, magnitude, downside, participation, and strength relative to market and sector), which is exactly the basis on which behavior clusters should form.
