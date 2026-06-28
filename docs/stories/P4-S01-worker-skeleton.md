# P4-S01 Worker Skeleton

## Risk
normal

## Goal
Create the always-on Python scheduler/worker skeleton that registers jobs and logs to `pipeline_runs`, with no production scheduling frequency changes and no real ingestion yet.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATA_FLOW.md`
- `docs/product/DEPLOYMENT.md`
- `docs/product/SCHEDULER_STRATEGY.md`
- `docs/decisions/DEC-002-use-ec2-worker.md`
- `docs/decisions/DEC-008-trading-calendar-controls-market-jobs.md`

## Modify only
- `pipelines/scheduler.py`
- `pipelines/common/calendar.py`
- `pipelines/common/logging.py`
- `pipelines/jobs/`

## Do not touch
- `supabase/migrations/`
- frontend files
- real provider ingestion code

## Acceptance criteria
- Scheduler starts without import errors and logs job registration.
- Jobs are calendar-gated stubs: they check `trading_calendar` and skip on non-trading days.
- Mock mode runs without a real API key.
- Job status written to `pipeline_runs`; errors don't crash the whole worker.
- Mock mode runs without a real database and without API keys.
- If database environment variables are absent, worker logs to stdout and does not crash.
- If `pipeline_runs` is unavailable, DB logging is skipped or mocked; this story must not create migrations.

## Verification
See `docs/test-matrix/pipeline.md`:
```bash
python -m pipelines.scheduler
```

## Decision record
Required? no, unless production scheduler frequency/behavior changes (then add DEC). Reference DEC-008.

## Notes for agent
Skeleton + mock jobs only. Do not implement real ingestion or models. Preserve calendar gating as the standard pattern.

## Prerequisites
- P1-S01 scaffold completed.
- Either core schema with `pipeline_runs` exists, or this story must run in mock/no-DB mode.
