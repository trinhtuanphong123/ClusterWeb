# DEC-008: Trading Calendar Controls Market Jobs

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The worker is always on, but market data only changes on trading days and during trading hours. Running ingestion and model jobs on weekends/holidays would waste compute, risk spurious data, and produce misleading snapshots. The dashboard must keep working on non-trading days by serving the latest valid snapshot.

## Decision

Use a **`trading_calendar`** table (or equivalent calendar logic) as the single control for all market jobs. Each market job checks the calendar and **skips internally** on non-trading days; intraday refresh additionally checks **trading hours**. Manual backfill/recompute can override the skip.

## Rationale

- **Correctness** — no ingestion or model recomputation when there is no new market data.
- **Cost** — avoids unnecessary runs of heavy nightly jobs on non-trading days.
- **Single source of truth** — one calendar drives ingestion, features, windows, clustering, retrieval, outcome, and snapshot jobs consistently.
- **Resilience** — the always-on worker stays up; only the work is gated, so scheduling stays simple.

## Consequences

- The `trading_calendar` must be seeded and maintained (holidays, weekends, trading hours).
- Every market job must consult the calendar before doing work; this is a required pattern.
- A manual override path must exist for backfills/recomputes on non-trading days.
- The API is independent of the calendar and always serves the latest valid outputs.

## Alternatives considered

- **Cron with hardcoded weekday rules** — cannot express market holidays correctly; rejected.
- **External calendar service** — adds a dependency for data that is small and stable enough to store locally; rejected.
- **Always run, filter later** — wastes compute and risks publishing snapshots for non-trading days; rejected.


## Related decisions

- DEC-002 — the worker whose jobs this calendar gates.
- DEC-006 — the hourly refresh gated by trading days/hours.
- DEC-009 — official clustering runs only on valid trading-day data.