# API Contract — FastAPI Backend

> The backend is a **reader of precomputed outputs**. It never runs models, clustering, or retrieval at request time. On non-trading days it returns the **latest valid snapshot** and market status. Secrets and service-role keys are never exposed. Response shapes below are indicative JSON; all timestamps are ISO-8601 UTC.

## Cross-cutting behavior

- **No model execution at request time.** Endpoints read `dashboard_snapshots`, `cluster_*`, `ticker_current_state`, etc.
- **Freshness fields** appear where relevant: `last_data_update`, `last_feature_update`, `last_model_update`, `latest_cluster_run`, `latest_snapshot`, and `is_stale`.
- **Non-trading days:** read endpoints return the latest valid snapshot (as-of the last trading day) and indicate staleness via `market_status` and `is_stale`. No recomputation is triggered.
- **`run_id` selection:** cluster endpoints default to the latest **valid** `cluster_run`; an explicit `run_id` query param can request a specific run.
- **Caching:** snapshot-backed responses are cacheable with short TTLs and revalidate against `latest_snapshot`. `/health` is never cached.
- **Standard errors:** `400` invalid params, `404` unknown resource, `503` no valid snapshot/data yet, `500` unexpected. Error body: `{ "error": { "code": str, "message": str } }`.
- **Standard horizons:** all forward-looking statistics use the standard horizons **5d, 20d, 60d** (5, 20, and 60 trading days), system-wide. No other horizons are used.
- **Descriptive-context disclaimer:** every endpoint that returns outcome statistics **or** historical pattern matches (cluster outcomes, cluster/ticker historical-patterns) must include a `disclaimer` field stating the data is descriptive historical context, not a prediction or recommendation. Historical matches are easy to misread as forecasts, so the disclaimer is required on retrieval endpoints too, not only `/outcomes`.

---

## 1. `GET /health`

- **Purpose:** Liveness/readiness for monitoring, Nginx, and post-deploy smoke checks.
- **Query parameters:** none.
- **Response JSON:**
  ```json
  { "status": "ok", "db": "ok", "time": "2026-06-22T09:00:00Z" }
  ```
- **Source tables:** none (a lightweight DB connectivity ping).
- **Error cases:** `503` if the DB is unreachable (`{ "status": "degraded", "db": "down" }`).
- **Caching/staleness:** never cached.
- **Non-trading days:** unaffected; always reflects live service health.

---

## 2. `GET /api/market/status`

- **Purpose:** Report whether today is a trading day, session state, and data freshness.
- **Query parameters:** none.
- **Response JSON:**
  ```json
  {
    "as_of_date": "2026-06-22",
    "is_trading_day": false,
    "day_type": "weekend",
    "session": { "open": null, "close": null, "is_open_now": false },
    "last_data_update": "2026-06-20T08:30:00Z",
    "last_feature_update": "2026-06-20T09:10:00Z",
    "last_model_update": "2026-06-20T15:40:00Z",
    "latest_cluster_run": 1287,
    "latest_snapshot": { "id": 5562, "as_of_date": "2026-06-20" },
    "is_stale": true
  }
  ```
- **Source tables:** `trading_calendar`, `dashboard_snapshots`, `cluster_runs`, `pipeline_runs`.
- **Error cases:** `503` if no calendar/snapshot exists yet.
- **Caching/staleness:** short TTL; `is_stale` true when the latest snapshot is older than the most recent trading day's expected update.
- **Non-trading days:** returns `is_trading_day: false`, `day_type` (`weekend`/`holiday`/`manual_closure`), and the latest valid snapshot reference.

---

## 3. `GET /api/overview`

- **Purpose:** Top-level dashboard payload: cluster summary, market state, and freshness — the landing view.
- **Query parameters:** `run_id` (optional).
- **Response JSON:**
  ```json
  {
    "as_of_date": "2026-06-20",
    "market_status": { "is_trading_day": false, "day_type": "weekend", "is_stale": true },
    "cluster_summary": {
      "run_id": 1287,
      "n_clusters": 14,
      "n_tickers": 412,
      "clusters": [
        { "cluster_id": "C1", "size": 38, "profile": { "volatility": "high", "trend": "up" } }
      ]
    },
    "freshness": {
      "last_data_update": "2026-06-20T08:30:00Z",
      "last_feature_update": "2026-06-20T09:10:00Z",
      "last_model_update": "2026-06-20T15:40:00Z",
      "latest_snapshot": { "id": 5562, "as_of_date": "2026-06-20" }
    }
  }
  ```
- **Source tables:** `dashboard_snapshots` (primary, `snapshot_type='overview'`), `cluster_runs`, `trading_calendar`.
- **Error cases:** `503` if no valid overview snapshot; `404` if a specified `run_id` doesn't exist.
- **Caching/staleness:** served from the latest valid snapshot; cacheable with short TTL.
- **Non-trading days:** returns the latest valid overview snapshot with `is_stale: true`.

---

## 4. `GET /api/clusters/latest`

- **Purpose:** Full list of current behavior clusters for the latest valid run.
- **Query parameters:** `run_id` (optional), `include` (optional: `members`, `profiles`).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "as_of_date": "2026-06-20",
    "clusters": [
      {
        "cluster_id": "C1",
        "size": 38,
        "profile": { "volatility": "high", "drawdown": "moderate", "mkt_rel": "positive" },
        "representatives": ["AAA", "BBB"],
        "members": ["AAA", "BBB", "CCC"]
      }
    ],
    "is_stale": false
  }
  ```
- **Source tables:** `cluster_runs`, `cluster_members`, (snapshot-backed where available).
- **Error cases:** `503` if no valid cluster run; `404` for unknown `run_id`.
- **Caching/staleness:** cacheable; revalidates against `latest_cluster_run`.
- **Non-trading days:** returns the latest valid run (last trading day) with `is_stale: true`.

---

## 5. `GET /api/clusters/{cluster_id}`

- **Purpose:** Detail for a single cluster: members, profile, representative tickers, edges summary.
- **Query parameters:** `run_id` (optional).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "cluster_id": "C1",
    "size": 38,
    "profile": { "volatility": "high", "drawdown": "moderate", "volume": "elevated" },
    "members": [
      { "ticker": "AAA", "membership_score": 0.91, "is_representative": true }
    ],
    "is_stale": false
  }
  ```
- **Source tables:** `cluster_members`, `cluster_runs`, `cluster_edges` (summary).
- **Error cases:** `404` if `cluster_id` not in the run; `503` if no valid run.
- **Caching/staleness:** cacheable; tied to run validity.
- **Non-trading days:** latest valid run, `is_stale: true`.

---

## 6. `GET /api/clusters/{cluster_id}/outcomes`

- **Purpose:** Historical outcome statistics (future returns/drawdowns) for the cluster, by horizon — descriptive context only.
- **Query parameters:** `run_id` (optional), `horizon` (optional filter, one of `5d`,`20d`,`60d`).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "cluster_id": "C1",
    "outcomes": [
      {
        "horizon": "20d",
        "n_samples": 240,
        "fwd_return": { "mean": 0.012, "median": 0.008, "p25": -0.01, "p75": 0.03 },
        "max_drawdown": { "mean": -0.04, "median": -0.03 },
        "excess_return_vs_index": { "mean": 0.004, "median": 0.002 }
      }
    ],
    "disclaimer": "Descriptive historical context, not a prediction or recommendation.",
    "is_stale": false
  }
  ```
- **Source tables:** `cluster_outcome_stats`, `cluster_runs`.
- **Error cases:** `404` unknown cluster; `503` if outcomes not computed for the run.
- **Caching/staleness:** cacheable; tied to run.
- **Non-trading days:** latest valid run, `is_stale: true`.

---

## 7. `GET /api/clusters/{cluster_id}/historical-patterns`

- **Purpose:** Historical analog windows that explain/validate the cluster's current behavior.
- **Query parameters:** `run_id` (optional), `limit` (optional, default e.g. 20).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "cluster_id": "C1",
    "matches": [
      {
        "query_ticker": "AAA",
        "query_window_end": "2026-06-20",
        "match_ticker": "AAA",
        "match_window_start": "2023-06-30",
        "match_window_end": "2023-09-14",
        "method": "dtw",
        "window_length": 60,
        "similarity": 0.94,
        "rank": 1,
        "future_return_5d": 0.006,
        "future_return_20d": 0.018,
        "future_return_60d": 0.041,
        "max_drawdown_20d": -0.03,
        "excess_return_20d": 0.009
      }
    ],
    "disclaimer": "Historical analogs are descriptive context, not a prediction or recommendation. Similar past behavior does not imply similar future behavior.",
    "is_stale": false
  }
  ```
- **Source tables:** `historical_pattern_matches`, `behavior_windows`, `cluster_runs`.
- **Error cases:** `404` unknown cluster; `503` if retrieval not computed for the run.
- **Caching/staleness:** cacheable; tied to run.
- **Non-trading days:** latest valid run, `is_stale: true`.

---

## 8. `GET /api/tickers`

- **Purpose:** List the active ticker universe with light metadata and current cluster assignment.
- **Query parameters:** `sector` (optional), `exchange` (optional), `q` (optional search), `run_id` (optional), `limit`/`offset` (pagination).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "count": 412,
    "tickers": [
      { "ticker": "AAA", "company_name": "AAA Corp", "sector": "Industrials", "cluster_id": "C1" }
    ]
  }
  ```
- **Source tables:** `stocks`, `stock_universe_members`, `cluster_members`.
- **Error cases:** `400` invalid pagination; `503` if universe not available.
- **Caching/staleness:** cacheable; cluster field tied to latest valid run.
- **Non-trading days:** returns the universe with the latest valid cluster assignments.

---

## 9. `GET /api/tickers/{ticker}`

- **Purpose:** Per-ticker detail: metadata, current cluster, latest behavior summary.
- **Query parameters:** `run_id` (optional).
- **Response JSON:**
  ```json
  {
    "ticker": "AAA",
    "company_name": "AAA Corp",
    "sector": "Industrials",
    "exchange": "HOSE",
    "is_active": true,
    "current_cluster": { "run_id": 1287, "cluster_id": "C1", "membership_score": 0.91 },
    "behavior_summary": { "volatility": "high", "drawdown": "moderate", "mkt_rel": "positive" },
    "is_stale": false
  }
  ```
- **Source tables:** `stocks`, `cluster_members`, `features_daily`/`behavior_windows` (summary).
- **Error cases:** `404` unknown ticker; `503` if no valid run for cluster fields.
- **Caching/staleness:** cacheable; cluster fields tied to run.
- **Non-trading days:** latest valid assignment, `is_stale: true`.

---

## 10. `GET /api/tickers/{ticker}/history`

- **Purpose:** Daily OHLCV history for charting.
- **Query parameters:** `from` (date, optional), `to` (date, optional), `limit` (optional).
- **Response JSON:**
  ```json
  {
    "ticker": "AAA",
    "interval": "1d",
    "bars": [
      { "date": "2026-06-20", "open": 25.1, "high": 25.8, "low": 24.9, "close": 25.6, "volume": 1200000 }
    ]
  }
  ```
- **Source tables:** `daily_bars`.
- **Error cases:** `404` unknown ticker; `400` invalid date range.
- **Caching/staleness:** cacheable; daily data changes at most once per trading day.
- **Non-trading days:** returns history through the last trading day (no change vs a normal read).

---

## 11. `GET /api/tickers/{ticker}/current-state`

- **Purpose:** Latest intraday/current-state for the ticker (hourly-refreshed view).
- **Query parameters:** none.
- **Response JSON:**
  ```json
  {
    "ticker": "AAA",
    "as_of": "2026-06-20T07:00:00Z",
    "last_price": 25.6,
    "intraday_return": 0.004,
    "intraday_volatility": 0.011,
    "volume": 540000,
    "source_interval": "5m",
    "is_trading_session": false,
    "is_stale": true
  }
  ```
- **Source tables:** `ticker_current_state`.
- **Error cases:** `404` unknown ticker; `503` if no current-state row yet.
- **Caching/staleness:** very short TTL on trading days; `is_stale` true outside sessions.
- **Non-trading days:** returns the last captured state from the previous session with `is_stale: true`, `is_trading_session: false`.

---

## 12. `GET /api/tickers/{ticker}/historical-patterns`

- **Purpose:** Historical analog windows for this specific ticker's current window.
- **Query parameters:** `run_id` (optional), `limit` (optional).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "ticker": "AAA",
    "query_window_end": "2026-06-20",
    "matches": [
      {
        "match_ticker": "AAA",
        "match_window_start": "2022-08-12",
        "match_window_end": "2022-11-03",
        "method": "mpdist",
        "window_length": 60,
        "similarity": 0.93,
        "rank": 1,
        "future_return_5d": 0.004,
        "future_return_20d": 0.015,
        "future_return_60d": 0.033,
        "max_drawdown_20d": -0.028,
        "excess_return_20d": 0.007
      }
    ],
    "disclaimer": "Historical analogs are descriptive context, not a prediction or recommendation. Similar past behavior does not imply similar future behavior.",
    "is_stale": false
  }
  ```
- **Source tables:** `historical_pattern_matches`, `behavior_windows`.
- **Error cases:** `404` unknown ticker; `503` if retrieval not computed.
- **Caching/staleness:** cacheable; tied to run.
- **Non-trading days:** latest valid run, `is_stale: true`.

---

## 13. `GET /api/tickers/{ticker}/neighbors`

- **Purpose:** Behaviorally nearest tickers (graph neighbors / strongest edges) for the current run.
- **Query parameters:** `run_id` (optional), `limit` (optional, default e.g. 10).
- **Response JSON:**
  ```json
  {
    "run_id": 1287,
    "ticker": "AAA",
    "neighbors": [
      { "ticker": "BBB", "weight": 0.88, "same_cluster": true }
    ],
    "is_stale": false
  }
  ```
- **Source tables:** `cluster_edges`, `cluster_members`.
- **Error cases:** `404` unknown ticker; `503` if no valid run.
- **Caching/staleness:** cacheable; tied to run.
- **Non-trading days:** latest valid run, `is_stale: true`.

---

## 14. `GET /api/pipeline/status`

- **Purpose:** Operational status of recent pipeline jobs and overall freshness for an ops/status view.
- **Query parameters:** `limit` (optional), `job_name` (optional filter).
- **Response JSON:**
  ```json
  {
    "freshness": {
      "last_data_update": "2026-06-20T08:30:00Z",
      "last_feature_update": "2026-06-20T09:10:00Z",
      "last_model_update": "2026-06-20T15:40:00Z",
      "latest_cluster_run": 1287,
      "latest_snapshot": { "id": 5562, "as_of_date": "2026-06-20" },
      "is_stale": true
    },
    "recent_jobs": [
      {
        "job_name": "clustering",
        "status": "succeeded",
        "trigger": "scheduled",
        "started_at": "2026-06-20T15:20:00Z",
        "ended_at": "2026-06-20T15:40:00Z",
        "error_message": null
      }
    ]
  }
  ```
- **Source tables:** `pipeline_runs`, `cluster_runs`, `dashboard_snapshots`, (optionally `data_quality_reports`).
- **Error cases:** `503` if logging unavailable.
- **Caching/staleness:** short TTL; reflects operational state.
- **Non-trading days:** shows skipped jobs and the latest valid freshness; `is_stale: true`.

---

## 15. `GET /api/methodology/summary`

- **Purpose:** Human-readable summary of the methodology and framing (clustering is primary; retrieval is explanation; not prediction/advice). Powers an "About/Methodology" page.
- **Query parameters:** none.
- **Response JSON:**
  ```json
  {
    "main_task": "Current behavior clustering of Vietnamese stocks.",
    "explanation_layer": "Historical pattern retrieval explains and validates clusters.",
    "not": ["price prediction", "buy/sell recommendations", "financial advice"],
    "features": ["returns","normalized_paths","volatility","drawdown","volume","market_relative","sector_relative"],
    "data_sources": { "official": "daily OHLCV", "intraday": "5-minute OHLCV (hourly state only)" },
    "update_frequency": { "official": "daily after close", "intraday": "hourly during trading hours" },
    "disclaimer": "Outputs are descriptive analytics, not predictions or recommendations."
  }
  ```
- **Source tables:** mostly static config; may reference `data_source_capabilities` for verified capability notes.
- **Error cases:** `500` only on unexpected failure.
- **Caching/staleness:** long TTL (rarely changes).
- **Non-trading days:** unaffected (static content).

---

## Security note

The API exposes only read endpoints over precomputed outputs. It uses read-focused database credentials, never service-role/secret keys, and never returns secrets, connection strings, or internal credentials in any response. CORS is restricted to the Vercel frontend origins.