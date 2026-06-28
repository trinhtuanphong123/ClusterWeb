# P8-S02 Behavior Window Generation

## Risk
normal

## Goal
Generate rolling behavior windows (5/20/60/120 trading days) per ticker into `behavior_windows`, storing shape vector and feature vector keyed by `(ticker,window_end_date,window_length,feature_schema_version)`.

## Prerequisites
- P8-S01 feature computation completed.
- `behavior_windows` schema exists.
- Fixture or dev data exists for at least one ticker with enough history.


## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/WINDOWING_SPEC.md`
- `docs/product/FEATURE_SPEC.md`
- `docs/product/DATABASE_SCHEMA.md`

## Modify only
- `pipelines/features/windows.py`
- `pipelines/jobs/behavior_windows.py`
- `pipelines/common/` (window helpers)

## Do not touch
- ingestion code
- clustering/retrieval pipelines
- frontend files

## Acceptance criteria
- Runs after daily feature computation; calendar-gated.
- Produces one representation per `(ticker,window_end_date,window_length,feature_schema_version)`.
- Stores shape vector and feature vector (or S3 `artifact_uri` for large vectors).
- Tickers with insufficient history are excluded for that window length.
- Mock verification must include both sufficient-history and insufficient-history tickers.
- Window generation must not create permanent cluster labels.


## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.behavior_windows --mock --once
```

## Decision record
Required? no, unless the stored window contract changes (then add DEC).

## Notes for agent
Multi-scale windows. A cluster is a current behavior state, not a permanent label — windows must be time-stamped by `window_end_date`.
