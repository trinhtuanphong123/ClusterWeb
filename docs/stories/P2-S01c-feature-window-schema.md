# P2-S01c Feature and Window Schema

## Risk
high-risk

## Execution status
blocked (by P2-S01b)

## Goal
Create migrations for `features_daily`, `features_hourly`, and `behavior_windows`, including explicit feature columns, `feature_schema_version`, and `window_start_date`/`window_end_date`.

## Prerequisites
- P2-S01b market bars schema completed (`daily_bars`/`intraday_bars` exist as feature inputs).

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/FEATURE_SPEC.md`
- `docs/product/WINDOWING_SPEC.md`
- `docs/decisions/DEC-003-use-supabase-postgres.md`
- `docs/decisions/DEC-011-api-security-and-credential-boundary.md`
- `docs/test-matrix/database.md`

## Modify only
- `supabase/migrations/` (only the migration files for this slice)

## Do not touch
- core reference and market bars tables (earlier slices)
- clustering/snapshot tables (later slice)
- frontend docs/files
- modeling pipelines

## Acceptance criteria
- `features_daily` unique on `(ticker, feature_date, feature_schema_version)` with the explicit typed feature columns from `DATABASE_SCHEMA.md`.
- `features_hourly` unique on `(ticker, feature_time, feature_schema_version)`.
- `behavior_windows` unique on `(ticker, window_end_date, window_length, feature_schema_version)`, includes `window_start_date`, and is indexed on `window_start_date`/`window_end_date`.
- FKs to `stocks(ticker)` present.
- No secrets in migrations.

## Verification
See `docs/test-matrix/database.md`:
```sql
select * from features_daily limit 1;
select * from behavior_windows limit 1;
```

## Decision record
Required? yes — contributes to `docs/decisions/DEC-xxx-supabase-schema.md`. Reference DEC-003. If feature columns/contract change, also reference the feature contract decision.

## Notes for agent
Dev database only. Keep explicit feature columns as canonical (not JSONB-only). `window_start_date` is trading-calendar-aligned per `WINDOWING_SPEC.md`.
