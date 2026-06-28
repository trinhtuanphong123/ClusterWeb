# P10-S03 Lead-Lag Analysis

## Risk
normal

## Goal
Detect whether behaviorally similar stocks move together or with delay (lagged/cross-correlation, optional DTW alignment) over a lag range (e.g., -10..+10 trading days) and write directed edges to `lead_lag_edges`.

## Prerequisites
- P8-S02 behavior windows completed.
- P9-S01 clustering completed.
- `lead_lag_edges` schema exists.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/LEAD_LAG_ANALYSIS_SPEC.md`
- `docs/product/DATABASE_SCHEMA.md`

## Modify only
- `pipelines/leadlag/lead_lag.py`
- `pipelines/jobs/lead_lag.py`
- `pipelines/common/` (correlation helpers)

## Do not touch
- ingestion/feature/clustering/retrieval/outcome pipelines
- frontend files

## Acceptance criteria
- Computes lead/lag over the defined lag range; records `leader`, `follower`, `lag`, `strength`, `scope` in `lead_lag_edges`, versioned by run_id.
- Runs after clustering; calendar-gated.
- Presented as descriptive context supporting cluster interpretation; not a signal; no buy/sell output.
- Lag range must be configurable.
- Directed edges must be versioned by run_id.
- Output must be descriptive only, not a trading signal.


## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.lead_lag --mock --once
```

## Decision record
Required? no.

## Notes for agent
Descriptive only. Keep the lag range configurable. Store directed edges per run.
