# Cluster Outcome Evaluation

> Evaluates what historically followed each cluster's behavior — to **validate cluster meaning**, not to predict and **not** to create labels for the unsupervised clustering. Results are descriptive context.

## 1. Future return horizons

Evaluate outcomes over multiple forward horizons:

- **5 trading days**
- **20 trading days**
- **60 trading days**

Each horizon measures behavior in the window *after* a cluster's as-of date.

## 2. Hit rate

- The fraction of cases (cluster members or matched windows) whose forward outcome meets a defined condition (e.g., positive forward return, or positive excess return vs index) over the horizon.
- Reported per cluster and horizon as a descriptive statistic.

## 3. Excess return versus VNINDEX

- Forward return **net of the VNINDEX** over the same horizon, isolating cluster-specific behavior from market-wide moves.
- Summarized per cluster and horizon (mean/median and dispersion).

## 4. Max drawdown

- The largest peak-to-trough decline over the forward horizon for cluster members/matches.
- Captures downside risk associated with a cluster's behavior, not just average return.

## 5. Cluster-level outcome summary

- Aggregate the above into per-cluster, per-horizon statistics stored in `cluster_outcome_stats`: sample count, forward-return distribution (mean, median, p25/p75), excess return, and drawdown summaries.
- Presented as **historical context** characterizing what tended to follow similar behavior — explicitly not a forecast.

## 6. Walk-forward evaluation

- Evaluate outcomes in a **walk-forward** manner: form clusters/matches as of past dates using only data available then, then measure the subsequent (already-realized) outcomes.
- This rolls forward through history, mimicking how the system would have behaved in real time, and avoids using future information when forming clusters.

## 7. Random baseline comparison

- Compare cluster outcome statistics against a **random baseline** (e.g., random groupings of the same sizes, or random window matches).
- If clusters carry behavioral meaning, their outcome coherence should differ from random; if not, the clustering adds no descriptive signal.
- The baseline guards against over-interpreting noise.

## 8. Avoiding look-ahead bias

- Use **only information available at the as-of date** when forming clusters and selecting matches; never use forward data in the representation.
- Apply the **exclusion rule** (see `HISTORICAL_PATTERN_RETRIEVAL_SPEC.md`) so matches don't overlap the query period.
- Compute outcomes strictly from data **after** the as-of date.
- Keep feature, window, and outcome computations time-aligned via the trading calendar so no future bar leaks into a past representation.

## 9. Outcomes validate meaning, not labels

- Outcome analysis is a **validation layer**: it tells us whether clusters are behaviorally coherent and interpretable.
- It must **not** be fed back as labels to train or steer the **unsupervised** clustering. The clustering is unsupervised by design; using future outcomes to define clusters would introduce look-ahead bias and convert the system into an implicit predictor, which is out of scope.
- Outcomes inform method/parameter *comparison* (see `CLUSTERING_SPEC.md`) only as descriptive evidence, never as training targets.


- Outcome statistics must display sample size. If sample size is below a configured threshold, for example n < 30, the dashboard should mark the statistic as low-confidence or insufficient sample.