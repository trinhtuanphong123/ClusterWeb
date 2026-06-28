# P2-S01e Schema Validation and Index Review

## Risk
high-risk

## Execution status
blocked (by P2-S01a–d)

## Goal
Validate the assembled schema end to end: primary keys, uniqueness constraints, latest-valid-snapshot lookup, and dashboard query indexes. No new tables; review and add only missing constraints/indexes.

## Prerequisites
- P2-S01a, P2-S01b, P2-S01c, P2-S01d completed.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/API_CONTRACT.md`
- `docs/decisions/DEC-003-use-supabase-postgres.md`
- `docs/decisions/DEC-011-api-security-and-credential-boundary.md`
- `docs/test-matrix/database.md`

## Modify only
- `supabase/migrations/` (only additive constraint/index fixes if review finds gaps)

## Do not touch
- table definitions beyond adding missing constraints/indexes
- frontend docs/files
- modeling pipelines

## Acceptance criteria
- All PKs from `DATABASE_SCHEMA.md` verified present.
- All uniqueness constraints verified (daily_bars, intraday_bars, features_daily, features_hourly, behavior_windows, outcome/match keys).
- Latest-valid `dashboard_snapshots` lookup verified fast and correct.
- Indexes that back documented dashboard/API queries verified present; missing ones added additively.
- Any change is additive; no destructive migration without a decision.

## Verification
See `docs/test-matrix/database.md`. Run constraint/index inspection on dev, e.g.:
```sql
select conname, contype from pg_constraint order by contype;
select indexname from pg_indexes where schemaname = 'public' order by indexname;
```

## Decision record
Required? no, unless the review changes the schema contract (then reference DEC-003 and update the schema decision).

## Notes for agent
Dev database only. This is a review/hardening pass — prefer additive fixes; if a destructive change seems required, stop and request a decision.
