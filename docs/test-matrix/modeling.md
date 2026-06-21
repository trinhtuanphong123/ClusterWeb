# Modeling validation (clustering / retrieval / outcomes)

## Current clustering

Expected:
- clustering runs on behavior features/windows, not raw price, and not as a price/return predictor
- output written to `cluster_runs`, `cluster_members`, `cluster_edges`, versioned by `run_id`
- cluster assignments are reproducible given the same run inputs and `feature_schema_version`

## Historical pattern retrieval

Expected:
- query windows match only against historical windows available **before** the query date (exclusion rule applied)
- top-K matches with similarity, rank, method written to `historical_pattern_matches`
- framed as explanation/validation, never prediction

## Outcome analysis

Expected:
- forward-return/drawdown/excess-return stats by cluster, horizon, run in `cluster_outcome_stats`
- walk-forward evaluation; no look-ahead bias (only data available at as-of date used to form clusters/matches)
- random baseline comparison available
- outcomes are not fed back as labels into the unsupervised clustering

## Lead-lag

Expected:
- directed edges in `lead_lag_edges` over a defined lag range (e.g., -10..+10 trading days)
- presented as descriptive context, not a signal

Proof:
- script run output on a small fixture window, or a documented manual check
