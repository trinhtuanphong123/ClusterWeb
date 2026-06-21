# P2-S01 Supabase Schema

## Risk
high-risk

## Goal
Create the initial Supabase Postgres schema and migrations for all tables defined in the schema doc, with correct primary keys, uniqueness constraints, indexes, and run_id versioning.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/DATA_FLOW.md`

## Modify only
- `supabase/migrations/`
- `pipelines/common/db.py` (connection helper only, no secrets)

## Do not touch
- frontend docs/files
- design docs
- modeling pipelines

## Acceptance criteria
- All tables from `DATABASE_SCHEMA.md` created with stated PKs.
- Uniqueness: `daily_bars (ticker,bar_date,source)`, `intraday_bars (ticker,bar_timestamp,interval,source)`, `features_daily (ticker,feature_date,feature_schema_version)`, `features_hourly (ticker,feature_time,feature_schema_version)`, `behavior_windows (ticker,window_end_date,window_length,feature_schema_version)`.
- Cluster outputs versioned by `run_id`; `dashboard_snapshots` supports fast latest-valid lookup.
- `trading_calendar` distinguishes trading/weekend/holiday/manual_closure.
- Indexes for dashboard queries present.

## Verification
See `docs/test-matrix/database.md`. Apply migrations to a dev Supabase project, then:
```sql
select * from stocks limit 1;
select * from pipeline_runs order by started_at desc limit 5;
```

## Decision record
Required? yes — `docs/decisions/DEC-xxx-supabase-schema.md` (data model ownership). Reference DEC-003.

## Notes for agent
Use a dev database, never production data. Keep migrations forward-only where possible. No secrets in migrations or committed env files.
