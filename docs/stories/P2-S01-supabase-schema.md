# P2-S01 Supabase Schema Plan

## Risk
high-risk

## Execution status
split-required (umbrella — do not execute as one story)

## Goal
Plan and split the Supabase schema implementation into smaller high-risk stories. Do not create migrations in this story. This umbrella exists so the full schema is never built in a single high-risk pass.

## Prerequisites
- P1-S01 scaffold completed.
- `docs/product/DATABASE_SCHEMA.md` is current.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/DATA_FLOW.md`
- `docs/decisions/DEC-003-use-supabase-postgres.md`
- `docs/decisions/DEC-008-trading-calendar-controls-market-jobs.md`
- `docs/decisions/DEC-011-api-security-and-credential-boundary.md`
- `docs/test-matrix/database.md`

## Split into
- `P2-S01a-core-reference-schema.md`
- `P2-S01b-market-bars-schema.md`
- `P2-S01c-feature-window-schema.md`
- `P2-S01d-clustering-snapshot-schema.md`
- `P2-S01e-schema-validation-index-review.md`

## Do not implement
- No SQL migrations in this umbrella story.
- No production database connection.
- No schema changes outside the split stories.

## Acceptance criteria
- The five child stories exist and each owns a bounded slice of `DATABASE_SCHEMA.md`.
- Each child story lists its own prerequisites, read-only docs, modify-only paths, and verification.
- No migration files are created by this umbrella story.

## Verification
Manual review: confirm the five child story files exist and collectively cover every table in `DATABASE_SCHEMA.md` with no overlap. No command is run for the umbrella.

## Decision record
Required? yes — `docs/decisions/DEC-xxx-supabase-schema.md` (data model ownership) is recorded by the child stories that create tables. Reference DEC-003.

## Notes for agent
This is a planning/umbrella story. Execute the child stories one at a time, in order (a → e). Use a dev database only, never production data.
