# Lead-Lag Analysis Spec

> Detects whether behaviorally similar stocks move **together** or with a **delay**, and stores directed relationships in `lead_lag_edges`. Supports cluster interpretation.

## 1. Purpose

- Determine, among related stocks/clusters, whether they move **simultaneously** or whether one tends to **lead** and another to **follow** by some lag.
- Enrich cluster interpretation: a cluster may contain a leader and laggards, which is meaningful structure beyond co-movement.

## 2. Methods

- **Lagged correlation** — correlation between one series and a time-shifted other series; the lag with peak correlation indicates lead/lag direction and magnitude.
- **Cross-correlation** — full cross-correlation function across the lag range to find the dominant lead-lag relationship.
- **DTW alignment** — Dynamic Time Warping alignment to detect lead/lag where the relationship involves small time warping rather than a fixed shift.

## 3. Lag range

- Evaluate lags over a defined range, for example **−10 to +10 trading days**.
- The sign of the best lag indicates direction (leader vs follower); the magnitude indicates the delay.

## 4. Compute safeguard — candidate pair selection

Full pairwise lead-lag over the entire universe scales as the square of the number of tickers and becomes infeasible at universe scale. To keep the computation bounded:

- **Restrict candidate pairs.** Only compute lead-lag for:
  - pairs of stocks **within the same cluster** (from the current `cluster_runs` / `cluster_members`), and/or
  - pairs connected by **top-similarity edges** (the strongest entries in `cluster_edges`).
- **Do not run lead-lag across the entire pairwise universe** when that universe is large. The all-pairs computation is only acceptable for a small universe well within the nightly compute budget.
- Optionally cap the number of candidate pairs per cluster/edge set and prune low-similarity pairs before computing lags.

This keeps lead-lag aligned with its purpose — it explains structure **within and between already-related groups**, so restricting to same-cluster and top-similarity pairs loses little interpretive value while controlling cost. The computation runs nightly after clustering and is calendar-gated.

## 5. Outputs in `lead_lag_edges`

Directed edges per run:

- **`leader`** — the leading ticker (or cluster id).
- **`follower`** — the following ticker (or cluster id).
- **`lag`** — the lag in trading days at peak relationship.
- **`strength`** — the lagged correlation / alignment score at that lag.
- **`scope`** — `ticker` or `cluster`.
- tied to a `run_id` for versioning.

## 6. How lead-lag supports cluster interpretation

- Reveals **internal structure** of clusters: which members tend to move first and which lag, turning a flat "these move alike" group into a leader/follower picture.
- Helps explain **why** stocks cluster together (synchronized vs delayed co-movement).
- Surfaces **cross-cluster** relationships (one cluster leading another), adding context to the behavior map.
- Like all outputs here, lead-lag is **descriptive context**, not a trading signal or prediction.

## 7. No causality, statistical timing only

Lead-lag edges describe **statistical timing relationships only. They do not imply causality.** A leader→follower edge means that, over the analyzed window, one series' movements tended to precede another's at the measured lag — it does **not** mean the leader *causes* the follower's movement.

- **Do not infer causality** from a lead-lag edge. Apparent lead-lag can arise from shared exposure to a common factor, liquidity and information-diffusion effects, sector or market-wide moves, or chance over a finite window.
- The relationship is **not stable or guaranteed**: like all behavior here, timing relationships are non-stationary and can change across regimes.
- Edges must be presented and consumed as **descriptive timing context that supports cluster interpretation**, never as a causal mechanism, a forecast, or a trade signal.