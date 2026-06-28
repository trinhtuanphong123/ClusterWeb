# P2-S01a Core Reference Schema

## Risk
high-risk

## Execution status
ready (after P2-S01 umbrella plan)

## Goal
Create migrations for the core reference tables: `stocks`, `trading_calendar`, and `pipeline_runs`. These are foundational and have no upstream table dependencies.

## Prerequisites
- P1-S01 scaffold completed.
- P2-S01 umbrella plan reviewed.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/DATA_FLOW.md`
- `docs/decisions/DEC-003-use-supabase-postgres.md`
- `docs/decisions/DEC-008-trading-calendar-controls-market-jobs.md`
- `docs/decisions/DEC-011-api-security-and-credential-boundary.md`
- `docs/test-matrix/database.md`

## Modify only
- `supabase/migrations/` (only the migration files for this slice)
- `pipelines/common/db.py` (connection helper only, no secrets) — create if absent

## Do not touch
- market bars, feature/window, clustering/snapshot tables (later slices)
- frontend docs/files
- modeling pipelines

## Acceptance criteria
- `stocks`, `trading_calendar`, `pipeline_runs` created with the PKs stated in `DATABASE_SCHEMA.md`.
- `trading_calendar` distinguishes trading/weekend/holiday/manual_closure.
- `pipeline_runs` includes status, started_at, ended_at, error_message, metadata.
- No secrets in migrations or committed env files; connection helper reads credentials from env only.

## Verification
See `docs/test-matrix/database.md`. Apply migrations to a dev Supabase project, then:
```sql
select * from stocks limit 1;
select * from trading_calendar limit 5;
select * from pipeline_runs order by started_at desc limit 5;
```

## Decision record
Required? yes — contributes to `docs/decisions/DEC-xxx-supabase-schema.md` (data model ownership). Reference DEC-003.

## Notes for agent
Dev database only. Forward-only migrations where possible. This slice unblocks the worker skeleton's DB logging and all calendar-gated jobs.
