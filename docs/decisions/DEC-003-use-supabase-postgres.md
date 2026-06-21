# DEC-003: Use Supabase Postgres as the Database

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The system needs a reliable relational store for the trading calendar, normalized OHLCV (daily + 5-minute), feature tables, rolling windows, clustering and historical-pattern outputs, supporting outputs, and dashboard snapshots. The team wants managed infrastructure with minimal ops overhead and standard SQL.

## Decision

Use **Supabase Postgres** as the primary database for all structured data and published snapshots. Large binary artifacts (raw parquet, model artifacts, distance matrices, vector artifacts) go to optional S3 instead.

## Rationale

- **Managed Postgres** — full SQL, relational integrity, and ecosystem with low operational burden.
- **Single source of truth** that both the worker (writer) and API (reader) connect to, cleanly decoupling the data and serving planes.
- **Good fit for snapshot serving** — the API reads read-optimized tables directly.
- **Avoids enterprise warehouses** (e.g., Redshift) that are excluded by the MVP constraints and unnecessary at this scale.

## Consequences

- Connection pooling must be configured for combined worker + API connections.
- Very large/binary artifacts should not be stored in Postgres; they belong in S3 (keeps the DB lean).
- Least-privilege credentials: API read-focused, worker read/write.

## Alternatives considered

- **Self-managed Postgres on EC2** — more ops burden (backups, patching, HA) for no MVP benefit; rejected.
- **Redshift / data warehouse** — excluded by constraints and oversized for this workload; rejected.
- **NoSQL store** — the data is relational and benefits from SQL joins and constraints; rejected.
