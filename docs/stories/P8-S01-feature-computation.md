# P8-S01 Feature Computation

## Risk
normal

## Goal
Compute `features_daily` from daily OHLCV (returns, volatility, momentum, drawdown, volume anomaly, beta/correlation to index, market-relative and sector-relative returns, shape-support), keyed by `(ticker,feature_date,feature_schema_version)`.

## Prerequisites
- Daily OHLCV ingestion completed or fixture data available.
- `features_daily` and `data_quality_reports` schema exists.
- P7-S02 daily incremental update completed or mocked.


## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/FEATURE_SPEC.md`
- `docs/product/DATA_FLOW.md`
- `docs/product/DATABASE_SCHEMA.md`

## Modify only
- `pipelines/features/daily_features.py`
- `pipelines/jobs/feature_daily.py`
- `pipelines/common/features.py`
- `pipelines/common/quality.py`

## Do not touch
- ingestion code
- clustering/retrieval pipelines
- frontend files

## Acceptance criteria
- Runs after the daily data update; calendar-gated.
- Writes `features_daily` with the unique key and a `feature_schema_version`.
- Missing-data handling per `FEATURE_SPEC.md` (nulls, not misleading imputations).
- Feature validation checks recorded in `data_quality_reports`.
- Mock verification uses a small fixture and must not require production data.
- Feature formulas must match `FEATURE_SPEC.md`; if formula changes are needed, stop and request a decision/story update.

## Verification
See `docs/test-matrix/modeling.md`. Run on a small fixture:
```bash
python -m pipelines.jobs.feature_daily --mock --once
```

## Decision record
Required? yes if feature formulas/stored contract change — `docs/decisions/DEC-xxx-feature-contract.md`. Otherwise no.

## Notes for agent
Daily features only (hourly features are dashboard context, separate). Bump `feature_schema_version` on any formula change.
