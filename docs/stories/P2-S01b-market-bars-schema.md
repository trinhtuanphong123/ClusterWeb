# P2-S01b Market Bars Schema

## Risk
high-risk

## Execution status
blocked (by P2-S01a)

## Goal
Create migrations for the normalized market data tables: `daily_bars` and `intraday_bars`, with their uniqueness constraints and market-data indexes.

## Prerequisites
- P2-S01a core reference schema completed (`stocks` exists for FK references).

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

## Do not touch
- core reference tables (already created in P2-S01a)
- feature/window, clustering/snapshot tables (later slices)
- frontend docs/files
- modeling pipelines

## Acceptance criteria
- `daily_bars` unique on `(ticker, bar_date, source)`.
- `intraday_bars` unique on `(ticker, bar_timestamp, interval, source)`.
- FKs to `stocks(ticker)` present; provenance columns (`source`, `ingested_at`) present.
- Market-data indexes present (e.g., `(ticker, bar_date)`, `(ticker, interval, bar_timestamp)`).
- No secrets in migrations.

## Verification
See `docs/test-matrix/database.md`. Apply migrations to dev, then attempt a duplicate insert to confirm the uniqueness constraints reject it:
```sql
select count(*) from daily_bars;
select count(*) from intraday_bars;
```

## Decision record
Required? yes — contributes to `docs/decisions/DEC-xxx-supabase-schema.md`. Reference DEC-003.

## Notes for agent
Dev database only. The uniqueness keys are the idempotent-upsert keys used by ingestion — do not change them without a decision update.
