# P7-S02 Daily Incremental Update

## Risk
high-risk

## Goal
Implement the daily incremental ingestion that fetches and upserts daily OHLCV after market close on trading days (the official data track).

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATA_FLOW.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/INGESTION_STRATEGY.md`
- `docs/product/SCHEDULER_STRATEGY.md`

## Modify only
- `pipelines/ingestion/daily.py`
- `pipelines/jobs/daily_incremental.py`
- `pipelines/common/` (high-water-mark + upsert helpers)

## Do not touch
- intraday ingestion (P7-S01)
- modeling pipelines
- frontend files

## Acceptance criteria
- Runs after market close on trading days only; skips on non-trading days via `trading_calendar` (manual backfill override allowed).
- Determines per-ticker high-water mark and fetches only newer bars.
- Idempotent upsert on `(ticker,bar_date,source)`.
- Data-quality checks recorded in `data_quality_reports`; status in `pipeline_runs`.

## Verification
See `docs/test-matrix/pipeline.md`. Mock run:
```bash
python -m pipelines.jobs.daily_incremental --mock --once
```

## Decision record
Required? no, unless retention/backfill behavior changes (then add DEC). Reference DEC-008.

## Notes for agent
Daily official source. Idempotent upserts only. Respect calendar gating.
