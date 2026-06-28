# P2-S01d Clustering and Snapshot Schema

## Risk
high-risk

## Execution status
blocked (by P2-S01c)

## Goal
Create migrations for the model/output tables: `cluster_runs`, `cluster_members`, `cluster_edges`, `historical_pattern_matches`, `cluster_outcome_stats`, `lead_lag_edges`, `dashboard_snapshots`, and `model_artifacts` (plus `model_runs`/`model_outputs` if defined), all versioned by `run_id`.

## Prerequisites
- P2-S01c feature/window schema completed (`behavior_windows` exists for FK references).

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/MODEL_RUNTIME_STRATEGY.md`
- `docs/decisions/DEC-003-use-supabase-postgres.md`
- `docs/decisions/DEC-009-main-task-is-current-behavior-clustering.md`
- `docs/decisions/DEC-010-historical-pattern-retrieval-is-explanation-layer.md`
- `docs/decisions/DEC-011-api-security-and-credential-boundary.md`
- `docs/test-matrix/database.md`

## Modify only
- `supabase/migrations/` (only the migration files for this slice)

## Do not touch
- earlier schema slices (reference/market/feature/window)
- frontend docs/files
- modeling pipelines (this is schema only)

## Acceptance criteria
- All listed output tables created and versioned by `run_id` via `cluster_runs`.
- `historical_pattern_matches` links query→match windows and stores `method`, `window_length`, `distance_or_similarity_type`.
- `cluster_outcome_stats` keyed by `(run_id, cluster_id, horizon)` with horizons `5d/20d/60d`.
- `dashboard_snapshots` supports a fast latest-valid lookup (e.g., partial-unique `is_latest`).
- `model_artifacts` carries validity/refit metadata fields per `MODEL_RUNTIME_STRATEGY.md`.
- No secrets in migrations.

## Verification
See `docs/test-matrix/database.md`:
```sql
select * from cluster_runs order by id desc limit 5;
select * from dashboard_snapshots where is_latest = true limit 5;
```

## Decision record
Required? yes — contributes to `docs/decisions/DEC-xxx-supabase-schema.md`. Reference DEC-003, DEC-009, DEC-010.

## Notes for agent
Dev database only. Cluster output contract is consumed by the API and modeling pipelines — do not alter shapes without a decision update.
