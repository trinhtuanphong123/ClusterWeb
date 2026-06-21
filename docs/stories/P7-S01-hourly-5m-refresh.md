# P7-S01 Hourly 5-Minute Refresh

## Risk
high-risk

## Goal
Implement the hourly intraday refresh that pulls recent 5-minute OHLCV and updates the current-state snapshot. **Hourly refresh only — NOT live 5-minute ingestion every 5 minutes.**

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATA_FLOW.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/INGESTION_STRATEGY.md`
- `docs/product/SCHEDULER_STRATEGY.md`

## Modify only
- `pipelines/ingestion/intraday_5m.py`
- `pipelines/jobs/hourly_refresh.py`
- `pipelines/common/` (upsert helpers as needed)

## Do not touch
- daily ingestion code (P7-S02)
- clustering/retrieval pipelines
- frontend files

## Acceptance criteria
- Runs **hourly** during trading hours only; skips on non-trading days and outside session via `trading_calendar`.
- Upserts to `intraday_bars` idempotently on `(ticker,bar_timestamp,interval,source)`.
- Refreshes `ticker_current_state`; does NOT trigger official clustering.
- 5-minute data treated as provisional; never feeds daily official clustering.
- No per-5-minute polling loop.

## Verification
See `docs/test-matrix/pipeline.md`. Run in mock mode:
```bash
python -m pipelines.jobs.hourly_refresh --mock --once
```

## Decision record
Required? no (matches DEC-006). Add DEC only if refresh cadence/behavior changes.

## Notes for agent
Hourly, not every 5 minutes. Enforce calendar + trading-hours gating. Provisional data only.
