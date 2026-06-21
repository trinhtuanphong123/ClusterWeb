# Scheduler Strategy — Worker Job Orchestration

> The worker is **always on**. Jobs **skip themselves internally on non-trading days** using the trading calendar. The dashboard API is independent and always serves the latest valid outputs.

## 1. Principles

- **Always-on worker:** the scheduler service runs continuously (systemd, `Restart=always`). It does not start/stop per job.
- **Calendar-gated jobs:** each market job first checks the **`trading_calendar`** table (or equivalent calendar logic). On non-trading days, market ingestion and model jobs **skip internally** — the scheduler stays up, the jobs simply no-op.
- **Dependency ordering:** jobs run in a fixed dependency order so each stage consumes the previous stage's outputs.
- **Idempotent + resumable:** re-running a job converges to the same state; a missed or retried run self-heals.
- **API decoupling:** the FastAPI service is unaffected by the scheduler and always reads the **latest valid** snapshot.

## 2. Trading calendar control

- A **`trading_calendar`** table marks each date as trading or non-trading (holidays, weekends).
- Every market job consults it before doing work:
  - **Trading day** → proceed.
  - **Non-trading day** → skip (no ingestion, no model jobs) **unless** a manual backfill/recompute is explicitly requested.
- The calendar also defines **trading hours**, which gate the intraday hourly refresh.

## 3. Intraday schedule (trading days only)

- **Hourly 5-minute refresh** runs **only on trading days and during trading hours**.
  - Pulls recent 5-minute bars, upserts them, and refreshes the **current-state intraday snapshot**.
  - Hourly cadence — **not** every 5 minutes (see DEC-006).
  - Does **not** trigger official clustering.
- Outside trading hours and on non-trading days, this job skips.

## 4. Daily official chain (trading days, after close)

Runs in this dependency order after market close:

1. **Universe sync** — reconcile the active ticker universe and sector mapping (add new tickers, mark delisted inactive, update metadata) **before** ingestion, so the day's update covers the correct set `S`. (Matches `INGESTION_STRATEGY.md`, where universe sync precedes daily bootstrap/incremental.)
2. **Daily incremental update** — fetch and upsert the day's daily OHLCV (after market close) for the synced universe.
3. **Data-quality checks (ingestion gate)** — run quality checks on the freshly ingested daily bars **before** feature computation; results recorded in `data_quality_reports`. Checks include schema/columns, single timezone, no duplicate `(ticker,bar_date,source)` keys, no missing bars vs the trading calendar, sanity bounds (`high ≥ low`, OHLC within `[low,high]`, volume ≥ 0), and coverage (fraction of active tickers updated). Failing bars/tickers are quarantined for backfill and excluded from feature computation rather than silently flowing downstream; if coverage falls below threshold, the chain pauses/alerts instead of producing a misleading official run.
4. **Daily feature computation** — runs **after** the daily data update **and only on data that passed the quality gate**; computes returns, normalized paths, volatility, drawdown, volume behavior, market-relative and sector-relative returns. Feature-level validation is also recorded in `data_quality_reports`.
5. **Rolling behavior window generation** — runs **after** daily feature computation; builds per-stock behavior windows ending at `T`.
6. **Official current behavior clustering** — runs **after** daily features and windows are ready; produces the authoritative current behavior clusters (the primary output).

## 5. Nightly heavy jobs (after daily update)

Computationally heavy, sequence/graph-aware jobs run **at night after the daily update**:

- **Graph construction**, **Leiden community detection**, **MPdist**, **Matrix Profile**, and other **sequence-aware** jobs.
- These run after the daily data is in place and feed/refine clustering and pattern layers.
- Scheduling them at night keeps daytime/intraday operations light and controls cost.

## 6. Explanation and analysis chain

After clustering and the nightly heavy jobs:

1. **Historical pattern retrieval** — runs **after** current clustering; retrieves analog historical windows to explain/validate current clusters.
2. **Outcome analysis** — runs **after** historical pattern retrieval; summarizes what historically followed the analogs (descriptive context only).

## 7. Snapshot generation

- **Dashboard snapshot generation** runs **after all required outputs are ready** (clustering, historical patterns, outcome analysis, lead-lag, watchlist).
- It assembles read-optimized snapshots and publishes them to Postgres for the API to serve.

## 8. Full daily dependency order (summary)

```
[trading day, after close]
  universe sync
        ▼
  daily incremental update
        ▼
  data-quality checks (ingestion gate)  ──fail──▶ quarantine + backfill / pause-alert
        ▼ (pass)
  daily feature computation
        ▼
  rolling behavior window generation
        ▼
  official current behavior clustering
        ▼
  (nightly heavy: graph / Leiden / MPdist / Matrix Profile / sequence-aware)
        ▼
  historical pattern retrieval
        ▼
  outcome analysis
        ▼
  dashboard snapshot generation  →  published for API
```

Intraday (parallel, trading hours only):
```
hourly 5-minute refresh  →  intraday current-state snapshot
```

## 9. Non-trading-day behavior

- Market ingestion and all model jobs **skip** (no daily update, no features/windows/clustering/retrieval/outcome/snapshot recompute).
- The intraday hourly refresh does not run.
- **Exception:** a **manual backfill or recompute** can be explicitly requested to run jobs on a non-trading day.
- The **dashboard API remains always on** and serves the **latest valid outputs** (the most recent trading day's snapshots).

## 10. Reliability behavior

- Each job records last-run and last-success timestamps (heartbeat) for health checks.
- Failed jobs quarantine bad data and are retried/backfilled without corrupting downstream tables.
- Because jobs are idempotent and dependency-ordered, the pipeline can be safely re-run from any stage.