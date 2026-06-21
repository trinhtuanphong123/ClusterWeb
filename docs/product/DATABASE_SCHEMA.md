# Database Schema — Supabase Postgres

> Single source of truth for the data and serving planes. The **worker** is the sole writer of analytical outputs; the **API** is a reader. Large binary artifacts (parquet, distance matrices, vectors) live in optional S3 and are referenced here by URI. Column types are indicative Postgres types; `id` columns are surrogate keys unless a natural key is stated.

## Conventions

- **Timestamps** are stored in UTC (`timestamptz`); a declared pipeline timezone governs trading-day logic.
- **`*_schema_version`** columns let features/outputs evolve without breaking old rows.
- **`run_id`** versions all model/cluster outputs so multiple runs coexist and the API serves a chosen run.
- **Write owner** = the process expected to write; **Read owner** = the main consumers.

---

## 1. `trading_calendar`

- **Purpose:** Authoritative calendar that gates all market jobs; distinguishes trading days, weekends, holidays, and manual closures.
- **Columns:**
  - `cal_date date` (natural key)
  - `day_type text` — enum-like: `trading`, `weekend`, `holiday`, `manual_closure`
  - `is_trading_day boolean`
  - `session_open time` (nullable; null on non-trading days)
  - `session_close time` (nullable)
  - `note text` (nullable; e.g., holiday name, reason for manual closure)
  - `created_at timestamptz`, `updated_at timestamptz`
- **Primary key:** `cal_date`
- **Unique constraints:** `cal_date` (PK)
- **Indexes:** `(is_trading_day)`, `(day_type)`
- **Relationships:** referenced logically by all market jobs and by `market/status` reads.
- **Write owner:** calendar seeding/maintenance job (worker).
- **Read owner:** scheduler, all market jobs, API (`/api/market/status`).
- **Retention:** permanent (small, durable reference).

---

## 2. `stocks`

- **Purpose:** Master record of each ticker and its metadata.
- **Columns:**
  - `ticker text` (natural key)
  - `exchange text` — e.g., HOSE/HNX/UPCOM
  - `company_name text`
  - `sector text` (nullable)
  - `industry text` (nullable)
  - `is_active boolean`
  - `first_seen_date date`, `last_seen_date date`
  - `created_at timestamptz`, `updated_at timestamptz`
- **Primary key:** `ticker`
- **Unique constraints:** `ticker` (PK)
- **Indexes:** `(is_active)`, `(sector)`, `(exchange)`
- **Relationships:** parent of `daily_bars`, `intraday_bars`, `features_*`, `behavior_windows`, `ticker_current_state`, `cluster_members`, etc. (FK on `ticker`).
- **Write owner:** universe sync job (worker).
- **Read owner:** all pipeline stages, API.
- **Retention:** permanent; inactive tickers are flagged, not deleted.

---

## 3. `stock_universe_members`

- **Purpose:** Versioned membership of the active investable universe over time (the set `S` used at each point).
- **Columns:**
  - `id bigint`
  - `universe_version text` — version/label of the universe snapshot
  - `ticker text` → `stocks.ticker`
  - `effective_from date`, `effective_to date` (nullable; null = current)
  - `is_current boolean`
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(universe_version, ticker)`
- **Indexes:** `(ticker)`, `(is_current)`, `(universe_version)`
- **Relationships:** FK `ticker` → `stocks`; consumed by feature/cluster jobs to define `S`.
- **Write owner:** universe sync job (worker).
- **Read owner:** feature/window/cluster jobs, API.
- **Retention:** permanent (audit of universe changes).

---

## 4. `daily_bars`

- **Purpose:** Normalized daily OHLCV — the **official** data source for clustering.
- **Columns:**
  - `id bigint`
  - `ticker text` → `stocks.ticker`
  - `bar_date date`
  - `source text` — provider/source identifier
  - `open numeric`, `high numeric`, `low numeric`, `close numeric`
  - `volume numeric`
  - `is_adjusted boolean`
  - `ingested_at timestamptz`, `updated_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(ticker, bar_date, source)`** — idempotent upsert key.
- **Indexes:** `(ticker, bar_date)`, `(bar_date)`
- **Relationships:** FK `ticker` → `stocks`; feeds `features_daily`.
- **Write owner:** daily bootstrap + daily incremental jobs (worker).
- **Read owner:** feature computation, API (`/api/tickers/{ticker}/history`).
- **Retention:** permanent (full history needed for behavior windows and historical retrieval).

---

## 5. `intraday_bars`

- **Purpose:** Normalized intraday OHLCV (primarily 5-minute) for the hourly current-state view. **Provisional**, never the official clustering source.
- **Columns:**
  - `id bigint`
  - `ticker text` → `stocks.ticker`
  - `bar_timestamp timestamptz`
  - `interval text` — e.g., `5m` (also allows `1h` if derived)
  - `source text`
  - `open numeric`, `high numeric`, `low numeric`, `close numeric`
  - `volume numeric`
  - `ingested_at timestamptz`, `updated_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(ticker, bar_timestamp, interval, source)`** — idempotent upsert key.
- **Indexes:** `(ticker, interval, bar_timestamp)`, `(bar_timestamp)`
- **Relationships:** FK `ticker` → `stocks`; feeds `features_hourly` and `ticker_current_state`.
- **Write owner:** hourly 5-minute refresh + optional intraday bootstrap (worker).
- **Read owner:** hourly feature/current-state jobs, API.
- **Retention:** rolling (keep a recent window, e.g., enough for hourly state and short context); older intraday bars may be pruned or archived to S3.

---

## 6. `features_daily`

- **Purpose:** Per-ticker daily behavior features derived from `daily_bars`.
- **Columns:**
  - `id bigint`
  - `ticker text` → `stocks.ticker`
  - `feature_date date`
  - `feature_schema_version text`
  - **Explicit feature columns** (typed, indexed-friendly, for fast dashboard/query without JSONB extraction):
    - `ret_5d numeric`, `ret_20d numeric`, `ret_60d numeric` — returns over 5/20/60 trading days
    - `vol_20d numeric`, `vol_60d numeric` — volatility over 20/60 trading days
    - `max_drawdown_60d numeric` — max drawdown over 60 trading days
    - `volume_ratio_20d numeric` — volume vs 20-day baseline (volume anomaly)
    - `mkt_rel_ret_20d numeric` — market-relative return over 20 trading days
    - `sector_rel_ret_20d numeric` — sector-relative return over 20 trading days
  - `norm_path jsonb` (nullable; normalized path segment/metadata for shape support)
  - `extra jsonb` (nullable; additional/experimental features not yet promoted to columns)
  - `computed_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(ticker, feature_date, feature_schema_version)`**
- **Indexes:** `(ticker, feature_date)`, `(feature_date)`, `(feature_schema_version)`; optionally on hot filter/sort columns such as `(feature_date, ret_20d)` and `(feature_date, vol_20d)` for dashboard queries.
- **Relationships:** FK `ticker` → `stocks`; feeds `behavior_windows`.
- **Write owner:** daily feature computation job (worker).
- **Read owner:** window generation, clustering, API.
- **Retention:** permanent for current schema versions; superseded schema versions may be archived.
- **Note:** explicit columns are the canonical, queryable features; `extra` JSONB is acceptable for the MVP only for additional features not yet stabilized. Promote any feature the dashboard filters/sorts on into a typed column. Adding or changing the meaning of a column requires bumping `feature_schema_version` (see `FEATURE_SPEC.md`).

---

## 7. `features_hourly`

- **Purpose:** Per-ticker hourly/intraday features derived from `intraday_bars` for current-state display.
- **Columns:**
  - `id bigint`
  - `ticker text` → `stocks.ticker`
  - `feature_time timestamptz`
  - `feature_schema_version text`
  - `ret numeric`, `volatility numeric`, `drawdown numeric`, `volume_behavior numeric`
  - `mkt_rel_return numeric`, `sector_rel_return numeric`
  - `extra jsonb` (nullable)
  - `computed_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(ticker, feature_time, feature_schema_version)`**
- **Indexes:** `(ticker, feature_time)`, `(feature_time)`
- **Relationships:** FK `ticker` → `stocks`; feeds `ticker_current_state`.
- **Write owner:** hourly feature job (worker).
- **Read owner:** current-state generation, API.
- **Retention:** rolling (recent window only).

---

## 8. `behavior_windows`

- **Purpose:** One rolling-window behavior representation per ticker and window end, used as input to clustering and historical retrieval.
- **Columns:**
  - `id bigint`
  - `ticker text` → `stocks.ticker`
  - `window_end_date date` — last trading date in the window (as-of date `T`)
  - `window_start_date date` — first trading date in the window; trading-calendar-aligned, `= window_end_date` offset back by `window_length - 1` trading days (see `WINDOWING_SPEC.md`). Stored explicitly so retrieval/exclusion can compare windows as concrete `[window_start_date, window_end_date]` intervals without recomputation.
  - `window_length int` — number of trading observations in the window
  - `feature_schema_version text`
  - `representation jsonb` — the behavior vector/summary (or `artifact_uri text` to S3 for large vectors)
  - `artifact_uri text` (nullable; S3 pointer for large representations)
  - `computed_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(ticker, window_end_date, window_length, feature_schema_version)`**
- **Indexes:** `(ticker, window_end_date)`, `(window_end_date)`, `(window_start_date)`, `(feature_schema_version)`
- **Relationships:** FK `ticker` → `stocks`; consumed by `cluster_runs` and referenced by `historical_pattern_matches` (both query and matched windows).
- **Write owner:** rolling behavior window generation job (worker).
- **Read owner:** clustering, historical retrieval, API.
- **Retention:** permanent (historical windows are the search space for retrieval).

---

## 9. `model_artifacts`

- **Purpose:** Registry of serialized model/algorithm artifacts (e.g., fitted transforms, distance matrices, vector indexes) stored in S3.
- **Columns:**
  - `id bigint`
  - `artifact_type text` — e.g., `distance_matrix`, `vector_index`, `scaler`
  - `artifact_uri text` — S3 URI
  - `schema_version text`
  - `size_bytes bigint` (nullable)
  - `checksum text` (nullable)
  - `created_by_run_id bigint` (nullable) → `model_runs.id`
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(artifact_uri)`
- **Indexes:** `(artifact_type)`, `(created_by_run_id)`
- **Relationships:** referenced by `model_runs`/`cluster_runs`; FK `created_by_run_id` → `model_runs`.
- **Write owner:** model/cluster jobs (worker).
- **Read owner:** model/cluster jobs; rarely the API.
- **Retention:** keep current + recent versions; prune stale artifacts from S3 with their rows.

---

## 10. `model_runs`

- **Purpose:** Log of generic model/computation runs (features, retrieval, heavy nightly jobs) for lineage.
- **Columns:**
  - `id bigint` (this is the generic `run_id`)
  - `run_type text` — e.g., `feature_daily`, `pattern_retrieval`, `matrix_profile`
  - `schema_version text`
  - `status text` — `pending`, `running`, `succeeded`, `failed`
  - `params jsonb`
  - `started_at timestamptz`, `ended_at timestamptz` (nullable)
  - `error_message text` (nullable)
- **Primary key:** `id`
- **Unique constraints:** none beyond PK (optionally `(run_type, started_at)`).
- **Indexes:** `(run_type, status)`, `(started_at)`
- **Relationships:** parent of `model_outputs`, `model_artifacts`.
- **Write owner:** model jobs (worker).
- **Read owner:** orchestration, API (`/api/pipeline/status` may join here).
- **Retention:** keep history for audit; old rows may be summarized/archived.

---

## 11. `model_outputs`

- **Purpose:** Generic outputs of `model_runs` that don't have a dedicated table.
- **Columns:**
  - `id bigint`
  - `run_id bigint` → `model_runs.id`
  - `output_type text`
  - `ticker text` (nullable) → `stocks.ticker`
  - `payload jsonb` (or `artifact_uri text` for large outputs)
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(run_id, output_type, ticker)` (where ticker applies)
- **Indexes:** `(run_id)`, `(output_type)`, `(ticker)`
- **Relationships:** FK `run_id` → `model_runs`; FK `ticker` → `stocks`.
- **Write owner:** model jobs (worker).
- **Read owner:** downstream jobs, API.
- **Retention:** tied to parent run retention.

---

## 12. `cluster_runs`

- **Purpose:** One row per official current-behavior clustering run — the **versioning anchor** (`run_id`) for all cluster outputs.
- **Columns:**
  - `id bigint` (the cluster `run_id`)
  - `as_of_date date` — trading day the run represents
  - `universe_version text` → `stock_universe_members.universe_version`
  - `feature_schema_version text`
  - `algo text` — clustering algorithm (e.g., graph/Leiden-based)
  - `params jsonb`
  - `status text` — `pending`, `running`, `succeeded`, `failed`
  - `is_valid boolean` — only valid runs are served
  - `started_at timestamptz`, `ended_at timestamptz` (nullable)
  - `error_message text` (nullable)
- **Primary key:** `id`
- **Unique constraints:** `(as_of_date, feature_schema_version, algo)` for the canonical run (optional; multiple runs allowed otherwise).
- **Indexes:** `(as_of_date)`, `(status, is_valid)`
- **Relationships:** parent of `cluster_members`, `cluster_edges`, `historical_pattern_matches`, `cluster_outcome_stats`, `lead_lag_edges`.
- **Write owner:** official clustering job (worker).
- **Read owner:** retrieval/outcome jobs, snapshot job, API (`/api/clusters/latest`).
- **Retention:** keep recent valid runs; older runs archivable.

---

## 13. `cluster_members`

- **Purpose:** Ticker-to-cluster assignment for a given run.
- **Columns:**
  - `id bigint`
  - `run_id bigint` → `cluster_runs.id`
  - `cluster_id text` — cluster label within the run
  - `ticker text` → `stocks.ticker`
  - `membership_score numeric` (nullable; strength/centrality)
  - `is_representative boolean` — representative member of the cluster
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(run_id, ticker)`** — a ticker has one assignment per run.
- **Indexes:** `(run_id, cluster_id)`, `(ticker)`
- **Relationships:** FK `run_id` → `cluster_runs`; FK `ticker` → `stocks`.
- **Write owner:** clustering job (worker).
- **Read owner:** API (`/api/clusters/{cluster_id}`, `/api/tickers/{ticker}`), snapshot job.
- **Retention:** tied to parent run.

---

## 14. `cluster_edges`

- **Purpose:** Pairwise relationships/similarities used to form clusters (graph edges) for a run; supports neighbor queries and visualization.
- **Columns:**
  - `id bigint`
  - `run_id bigint` → `cluster_runs.id`
  - `ticker_a text` → `stocks.ticker`
  - `ticker_b text` → `stocks.ticker`
  - `weight numeric` — similarity/edge weight
  - `edge_type text` (nullable)
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(run_id, ticker_a, ticker_b)` (store canonical ordering to avoid duplicates).
- **Indexes:** `(run_id, ticker_a)`, `(run_id, ticker_b)`, `(run_id, weight)`
- **Relationships:** FK `run_id` → `cluster_runs`; FKs `ticker_a`, `ticker_b` → `stocks`.
- **Write owner:** clustering/graph job (worker).
- **Read owner:** API (`/api/tickers/{ticker}/neighbors`), snapshot job.
- **Retention:** tied to parent run; can be large — consider pruning low-weight edges or archiving matrices to S3 (`model_artifacts`).

---

## 15. `historical_pattern_matches`

- **Purpose:** Links a **current query window** to **matched historical windows** to explain/validate clusters.
- **Columns:**
  - `id bigint`
  - `run_id bigint` → `cluster_runs.id`
  - `query_ticker text` → `stocks.ticker`
  - `query_window_id bigint` → `behavior_windows.id`
  - `match_ticker text` → `stocks.ticker`
  - `match_window_id bigint` → `behavior_windows.id`
  - `match_window_start_date date` — first trading date of the matched window (for date-range display/exclusion)
  - `match_window_end_date date` — last trading date of the matched window
  - `window_length int` — window length used for this match (query and match share the same length)
  - `method text` — retrieval method that produced the match, e.g., `feature_nn`, `normalized_correlation`, `dtw`, `matrix_profile`, `mpdist` (required; methods are mandatory to record per `HISTORICAL_PATTERN_RETRIEVAL_SPEC.md`)
  - `distance_or_similarity_type text` — how `similarity` should be interpreted, e.g., `cosine`, `euclidean`, `correlation`, `dtw_distance`, `mpdist` (distinguishes "higher is closer" similarity from "lower is closer" distance)
  - `similarity numeric` — the similarity/distance score under `method` / `distance_or_similarity_type`
  - `rank int` — rank of this match for the query (under the given method/window_length)
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(run_id, query_window_id, match_window_id, method)` — a query/match pair may be recorded once per method
- **Indexes:** `(run_id, query_ticker)`, `(run_id, query_window_id, method, rank)`, `(match_ticker)`, `(run_id, method)`, `(window_length)`
- **Relationships:** FK `run_id` → `cluster_runs`; FKs `query_window_id`, `match_window_id` → `behavior_windows`; FKs tickers → `stocks`.
- **Write owner:** historical pattern retrieval job (worker).
- **Read owner:** outcome analysis job, API (`/api/.../historical-patterns`), snapshot job.
- **Retention:** tied to parent run.

---

## 16. `cluster_outcome_stats`

- **Purpose:** Summarizes **future returns and drawdowns by cluster, horizon, and run** — descriptive historical context only (not predictions).
- **Columns:**
  - `id bigint`
  - `run_id bigint` → `cluster_runs.id`
  - `cluster_id text`
  - `horizon text` — one of `5d`, `20d`, `60d` (the standard forward horizons of 5, 20, and 60 trading days; see `CLUSTER_OUTCOME_EVALUATION.md` and `METHODOLOGY.md`)
  - `n_samples int`
  - `fwd_return_mean numeric`, `fwd_return_median numeric`
  - `fwd_return_p25 numeric`, `fwd_return_p75 numeric`
  - `max_drawdown_mean numeric`, `max_drawdown_median numeric`
  - `extra jsonb` (nullable; additional distribution stats)
  - `computed_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** **`(run_id, cluster_id, horizon)`**
- **Indexes:** `(run_id, cluster_id)`, `(run_id, horizon)`
- **Relationships:** FK `run_id` → `cluster_runs`.
- **Write owner:** outcome analysis job (worker).
- **Read owner:** API (`/api/clusters/{cluster_id}/outcomes`), snapshot job.
- **Retention:** tied to parent run.

---

## 17. `lead_lag_edges`

- **Purpose:** Directed lead-lag relationships between tickers/clusters for a run.
- **Columns:**
  - `id bigint`
  - `run_id bigint` → `cluster_runs.id`
  - `leader text` — leading ticker (or cluster id)
  - `follower text` — following ticker (or cluster id)
  - `lag int` — lag in periods
  - `strength numeric` — lagged correlation/score
  - `scope text` — `ticker` or `cluster`
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(run_id, leader, follower, lag, scope)`
- **Indexes:** `(run_id, leader)`, `(run_id, follower)`, `(run_id, scope)`
- **Relationships:** FK `run_id` → `cluster_runs`; leader/follower reference `stocks.ticker` when `scope = ticker`.
- **Write owner:** lead-lag analysis job (worker).
- **Read owner:** API, snapshot job.
- **Retention:** tied to parent run.

---

## 18. `ticker_current_state`

- **Purpose:** Latest intraday/current-state per ticker for the hourly dashboard view; one current row per ticker.
- **Columns:**
  - `ticker text` → `stocks.ticker` (natural key)
  - `as_of timestamptz` — timestamp of the state
  - `last_price numeric`
  - `intraday_return numeric`, `intraday_volatility numeric`
  - `volume numeric`
  - `state_summary jsonb` (nullable; compact display payload)
  - `source_interval text` — e.g., `5m`
  - `is_trading_session boolean`
  - `updated_at timestamptz`
- **Primary key:** `ticker`
- **Unique constraints:** `ticker` (PK)
- **Indexes:** `(as_of)`, `(is_trading_session)`
- **Relationships:** FK `ticker` → `stocks`; derived from `features_hourly`/`intraday_bars`.
- **Write owner:** hourly current-state job (worker).
- **Read owner:** API (`/api/tickers/{ticker}/current-state`), snapshot job.
- **Retention:** current value only (overwritten each refresh); history is implicitly in `intraday_bars`/`features_hourly`.

---

## 19. `dashboard_snapshots`

- **Purpose:** Precomputed, read-optimized snapshots so the frontend can fetch the **latest valid state quickly** without joins or model computation.
- **Columns:**
  - `id bigint`
  - `snapshot_type text` — e.g., `overview`, `clusters`, `full`
  - `as_of_date date` — trading day represented
  - `run_id bigint` (nullable) → `cluster_runs.id`
  - `payload jsonb` — fully assembled snapshot for the frontend
  - `is_valid boolean` — only valid snapshots are served
  - `is_latest boolean` — fast flag for "serve this"
  - `last_data_update timestamptz` (nullable)
  - `last_feature_update timestamptz` (nullable)
  - `last_model_update timestamptz` (nullable)
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** partial unique on `(snapshot_type)` where `is_latest = true` (one latest per type).
- **Indexes:** `(snapshot_type, is_latest)`, `(snapshot_type, as_of_date)`, `(is_valid)`
- **Relationships:** optional FK `run_id` → `cluster_runs`.
- **Write owner:** dashboard snapshot generation job (worker).
- **Read owner:** API (most read endpoints), frontend (indirectly).
- **Retention:** keep recent snapshots for rollback; prune old non-latest snapshots.

---

## 20. `pipeline_runs`

- **Purpose:** Operational log of every scheduled job for monitoring, health checks, and `/api/pipeline/status`.
- **Columns:**
  - `id bigint`
  - `job_name text` — e.g., `daily_incremental`, `feature_daily`, `clustering`, `snapshot`
  - `status text` — `started`, `succeeded`, `failed`, `skipped`
  - `trigger text` — `scheduled` or `manual`
  - `started_at timestamptz`, `ended_at timestamptz` (nullable)
  - `error_message text` (nullable)
  - `metadata jsonb` (nullable; counts, ranges, run_ids touched)
- **Primary key:** `id`
- **Unique constraints:** none beyond PK.
- **Indexes:** `(job_name, started_at)`, `(status)`, `(started_at)`
- **Relationships:** logically references runs in `model_runs`/`cluster_runs` via `metadata`.
- **Write owner:** scheduler/all jobs (worker).
- **Read owner:** API (`/api/pipeline/status`), monitoring.
- **Retention:** keep a rolling window (e.g., recent N days/weeks); archive or prune older logs.

---

## 21. `data_quality_reports`

- **Purpose:** Records data-quality check results per job/run for observability and quarantine decisions.
- **Columns:**
  - `id bigint`
  - `report_scope text` — e.g., `daily_bars`, `intraday_bars`, `features_daily`
  - `ref_date date` (nullable)
  - `pipeline_run_id bigint` (nullable) → `pipeline_runs.id`
  - `check_name text` — e.g., `duplicates`, `missing_bars`, `sanity_bounds`, `coverage`
  - `passed boolean`
  - `severity text` — `info`, `warning`, `error`
  - `details jsonb` — counts, offending keys, thresholds
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(report_scope, ref_date, check_name, pipeline_run_id)`
- **Indexes:** `(report_scope, ref_date)`, `(passed)`, `(severity)`
- **Relationships:** FK `pipeline_run_id` → `pipeline_runs`.
- **Write owner:** ingestion/feature jobs (worker).
- **Read owner:** monitoring, API (optionally surfaced in pipeline status).
- **Retention:** rolling window; keep failures longer than passes if pruning.

---

## 22. `data_source_capabilities`

- **Purpose:** Persisted, **probe-backed** record of VNStock API capabilities (the structured counterpart to `DATA_SOURCE_CAPABILITIES.md`). Never store assumed limits.
- **Columns:**
  - `id bigint`
  - `probe_date date`
  - `vnstock_version text` (nullable)
  - `capability text` — e.g., `daily_ohlcv`, `intraday_5m`, `price_board`
  - `subkey text` (nullable) — e.g., `lookback`, `timezone`, `rate_limit`
  - `observed_value text` — recorded observation (or `UNVERIFIED`)
  - `verified boolean` — true only if directly probed
  - `notes text` (nullable)
  - `created_at timestamptz`
- **Primary key:** `id`
- **Unique constraints:** `(probe_date, capability, subkey)`
- **Indexes:** `(capability)`, `(verified)`, `(probe_date)`
- **Relationships:** standalone reference; consulted by ingestion tuning.
- **Write owner:** probing/ingestion tooling (worker, manual).
- **Read owner:** ingestion jobs, API (`/api/methodology/summary` may reference), operators.
- **Retention:** permanent (history of capability observations over time).

---

## Relationship summary

```
stocks ──< daily_bars
       ──< intraday_bars
       ──< features_daily ──< behavior_windows ──< cluster_runs ──< cluster_members
       ──< features_hourly ──> ticker_current_state                 ──< cluster_edges
       ──< stock_universe_members                                   ──< historical_pattern_matches >── behavior_windows
                                                                     ──< cluster_outcome_stats
                                                                     ──< lead_lag_edges
cluster_runs ──> dashboard_snapshots
model_runs ──< model_outputs, model_artifacts
pipeline_runs ──< data_quality_reports
trading_calendar (gates all market jobs)
data_source_capabilities (reference)
```

## Access-control note

- The **worker** uses read/write credentials; the **API** uses read-focused credentials. Service-role/secret keys are never exposed to the frontend. Row-level security and least-privilege roles should restrict the API to read-only on serving tables (`dashboard_snapshots`, `cluster_*`, `ticker_current_state`, etc.).