# Pipeline validation

## Worker startup

Expected:
- scheduler starts without import errors
- logs job registration
- does not require a real API key in mock mode

Command:
```bash
python -m pipelines.scheduler
```

## Trading-calendar gating

Expected:
- market jobs check `trading_calendar` and skip internally on non-trading days
- hourly 5-minute refresh runs only on trading days during trading hours (hourly, not every 5 minutes)
- daily incremental runs after market close on trading days
- manual backfill/recompute can override the skip

## Pipeline logging

Expected:
- job status is written to `pipeline_runs` (status, started_at, ended_at, error_message, metadata)
- errors are captured without crashing the whole worker
- data-quality results recorded in `data_quality_reports`
