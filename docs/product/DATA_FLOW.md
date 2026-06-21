# Data Flow — VNStock API → Vercel Dashboard

> Two distinct tracks share the same storage foundation:
> - **Daily official clustering flow** — authoritative, from **daily OHLCV**.
> - **Hourly intraday flow** — current-state display only, from **5-minute OHLCV**.
>
> The intraday flow **never** produces official clusters; it only updates the "current state" snapshot.

## 1. Full flow (end to end)

```
                         VNStock community/free API
                                    │
                ┌───────────────────┴───────────────────┐
                │                                        │
        (daily OHLCV)                            (5-minute OHLCV)
                │                                        │
                ▼                                        ▼
        ┌───────────────────────── RAW STORAGE ─────────────────────────┐
        │  as-fetched responses, provenance, append-with-dedup          │
        │  (S3 parquet — not a Postgres table; see Section 2.1)         │
        └───────────────────────────────────────────────────────────────┘
                │                                        │
                ▼                                        ▼
        ┌──────────────────── NORMALIZED TABLES ────────────────────────┐
        │  cleaned OHLCV, typed, deduped, single timezone,              │
        │  daily track + 5-minute track kept separate                   │
        └───────────────────────────────────────────────────────────────┘
                │                                        │
   ┌────────────┴───────────── DAILY OFFICIAL ───────────┐    │ (intraday)
   ▼                                                      │    ▼
FEATURE TABLES                                            │  INTRADAY
(returns, normalized paths, volatility, drawdown,         │  CURRENT-STATE
 volume behavior, market-relative, sector-relative)       │  COMPUTATION
   │                                                      │  (light features
   ▼                                                      │   for display)
ROLLING BEHAVIOR WINDOWS                                  │    │
(per stock, window ending at T)                           │    ▼
   │                                                      │  INTRADAY
   ├───────────────┬──────────────────────┐              │  SNAPSHOT
   ▼               ▼                       ▼              │    │
CLUSTERING     HISTORICAL PATTERN     (lead-lag,          │    │
OUTPUTS        OUTPUTS                outcome,            │    │
(current       (analog retrieval,     watchlist)         │    │
 clusters)      outcome summaries)                        │    │
   │               │                       │              │    │
   └───────────────┴───────────┬───────────┘              │    │
                               ▼                          │    │
                    DASHBOARD SNAPSHOTS  ◄────────────────┘────┘
                    (daily official snapshot
                     + latest intraday snapshot)
                               │
                               ▼
                            FastAPI
                    (serves precomputed snapshots)
                               │
                               ▼
                        Vercel dashboard
```

## 2. Stage descriptions

1. **VNStock API** — single external source; two intervals fetched (daily, 5-minute).
2. **Raw storage** — exact as-fetched payloads with provenance; append-with-dedup; never edited. **Raw storage is not a Postgres table** — see Section 2.1 for where it lives.
3. **Normalized tables** — cleaned, typed, deduplicated OHLCV in one declared timezone; daily and 5-minute kept as separate tracks.
4. **Feature tables** *(daily official only)* — returns, normalized paths, volatility, drawdown, volume behavior, market-relative return, sector-relative return.
5. **Rolling behavior windows** *(daily official only)* — per-stock behavior representation over a window ending at `T`.
6. **Clustering outputs** — current behavior clusters: membership, profiles, representatives.
7. **Historical pattern outputs** — analog windows retrieved for stocks/clusters plus outcome-distribution summaries (descriptive context only).
8. **Supporting outputs** — lead-lag relationships, historical outcome analysis, current-pattern watchlist.
9. **Dashboard snapshots** — precomputed, read-optimized snapshots: the daily official snapshot plus the latest intraday snapshot.
10. **FastAPI** — serves the precomputed snapshots; does no heavy computation at request time.
11. **Vercel dashboard** — frontend reading from FastAPI.

## 2.1 Where raw storage lives

The schema (`DATABASE_SCHEMA.md`) does **not** define a dedicated raw table, and that is intentional. Raw storage is **object storage (S3), not Postgres**:

- **System of record for raw payloads:** as-fetched VNStock responses are written to the **optional S3 bucket as raw parquet** (the same bucket the schema references for large/binary artifacts by URI). This is where "as-fetched responses, provenance, append-with-dedup" physically lives. Files are partitioned by source/interval/date and never edited in place.
- **Provenance in Postgres:** the normalized OHLCV tables (`daily_bars`, `intraday_bars`) already carry provenance columns — `source` (part of their uniqueness keys) and `ingested_at`/`updated_at` — so each normalized bar records which source it came from and when it was ingested. This preserves lineage without duplicating full raw payloads in Postgres.
- **If S3 is not enabled:** the raw layer is optional. When the optional S3 bucket is not provisioned, the pipeline may skip persisting raw parquet and treat the **normalized tables (plus their `source`/`ingested_at` provenance) as the durable record**, fetching directly into normalization. In that mode, "RAW STORAGE" in the diagram is a transient in-pipeline stage rather than a persisted layer.

In short: raw storage = **S3 parquet (optional)**; Postgres holds the **normalized** tables onward. No new Postgres raw table is required.

## 3. Daily official clustering flow (authoritative)

- Trigger: after market close on **trading days**.
- Path: daily OHLCV → raw (S3 parquet, optional) → normalized → **feature tables → rolling behavior windows → clustering + historical pattern outputs + supporting outputs** → daily official snapshot → FastAPI → dashboard.
- This snapshot is the **source of truth** for clusters and historical pattern explanations.

## 4. Hourly intraday flow (current-state only)

- Trigger: **hourly during trading sessions** (not every 5 minutes).
- Path: 5-minute OHLCV → raw (S3 parquet, optional) → normalized (5-min track) → **light current-state computation** → intraday snapshot → FastAPI → dashboard.
- Purpose: show an up-to-date "how things are moving right now" view.
- **Does not** run clustering or alter the official daily outputs.

## 5. What the dashboard reads on non-trading days

- **No new computation occurs** (no daily official run, no hourly intraday refresh).
- FastAPI continues serving the **latest valid snapshots** already in storage:
  - The **most recent daily official snapshot** (clusters, historical patterns, lead-lag, outcomes, watchlist) from the last trading day.
  - The **last intraday snapshot** captured during the previous session, shown as the latest available current-state.
- The dashboard indicates the snapshot is the latest valid one (e.g., as-of the last trading day) rather than live.
- Behavior is identical whether the user visits on a weekend, a holiday, or before the first refresh of a trading day: **serve the latest valid snapshot**.