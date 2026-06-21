# Ingestion Strategy — VNStock → Storage

> Source: VNStock community/free API. Daily OHLCV is the **official** track; 5-minute OHLCV serves **hourly dashboard state only**. No claims about exact API limits here are assumed — pacing/limits are tuned from probe results (see `DATA_SOURCE_CAPABILITIES.md`).

## 1. Ingestion jobs overview

| Job | Track | Cadence | Purpose |
|---|---|---|---|
| Universe sync | Reference | Daily (pre-pipeline) | Keep ticker list + sector mapping current |
| Daily historical bootstrap | Official | One-time / backfill | Load full daily OHLCV history |
| Daily incremental update | Official | Daily after close | Append latest daily bars |
| 5-minute historical bootstrap (if needed) | Intraday | One-time / as needed | Seed recent 5-min history for hourly state |
| Hourly 5-minute refresh | Intraday | Hourly on trading days | Update current intraday snapshot |

> **Explicitly removed from MVP:** live 5-minute ingestion every 5 minutes. Intraday refresh is **hourly**, not per-bar.

## 2. Ticker universe sync

- Fetch the current ticker universe and sector/industry mapping from VNStock.
- Reconcile against the stored universe:
  - **New tickers** → add.
  - **Missing/delisted tickers** → mark inactive (do not hard-delete history).
  - **Changed metadata** (sector, name) → update.
- Output: an authoritative active-universe list that downstream daily and intraday jobs iterate over.
- Run **before** the daily bootstrap/incremental so new tickers are picked up.

## 3. Daily historical bootstrap (official)

- For each active ticker, fetch the **maximum verified daily history** available.
- Land raw responses in the **raw layer**, then transform into **normalized daily OHLCV**.
- Idempotent: re-running does not duplicate; existing bars are matched and upserted, not appended blindly.
- Tracks per-ticker bootstrap status so the job is resumable after interruption.
- Runs once initially and on demand for backfills or newly added tickers.

## 4. Daily incremental update (official)

- Runs **after market close on trading days**.
- For each active ticker, determine the **last stored daily bar date** (the high-water mark) and fetch only bars after it.
- Upsert new bars into normalized daily OHLCV.
- This is the authoritative input to the daily clustering / historical-pattern pipeline.

## 5. 5-minute historical bootstrap (if needed)

- Only if hourly intraday state requires recent 5-minute context that isn't already stored.
- Fetch the verified available 5-minute lookback for active tickers; land raw, then normalize.
- Treated as **provisional** data — never promoted to the official daily track.
- Idempotent upsert, same as the daily bootstrap.

## 6. Hourly 5-minute refresh (intraday)

- Runs **hourly during trading sessions only**.
- Fetches recent 5-minute bars since the last intraday high-water mark and upserts them.
- Recomputes/refreshes the **current-state intraday snapshot** used by the dashboard.
- Does **not** trigger official clustering; it only updates the "current state" view.
- Off-hours and non-trading days: this job does not run.

## 7. Bootstrap vs incremental logic

For each ticker and each track, choose mode by stored state:

- **No data stored** → run **bootstrap** (full available history).
- **Data exists** → run **incremental** (fetch only after the high-water mark).
- **Detected gap** below the high-water mark (missing bars on known trading days) → trigger a **targeted backfill** for the gap range.
- High-water mark is tracked per ticker per track (daily, 5-minute) so the two tracks advance independently.

## 8. Idempotent upsert behavior

- Daily bars natural key: (ticker, bar_date, source)
- Intraday bars natural key: (ticker, bar_timestamp, interval, source)
- Insert if the key is new; **update in place** if it already exists (last-write-wins on corrected values).
- No blind appends — re-running any job over an overlapping range produces the same final state.
- Raw layer is append-with-dedup (keep provenance); normalized layer is upsert-by-key.

## 9. Data quality checks

Run on every ingestion job before data is marked usable:

- **Schema/columns:** expected columns present and correctly typed.
- **Timezone:** timestamps normalized to a single declared timezone.
- **Duplicates:** no duplicate `(ticker, interval, timestamp)` keys after upsert.
- **Missing bars:** compare against the trading calendar / session hours; flag gaps.
- **Sanity bounds:** `high ≥ low`, `open/close` within `[low, high]`, volume ≥ 0, no absurd jumps.
- **Coverage:** fraction of active tickers successfully updated; alert if below threshold.
- Failures quarantine the affected bars/tickers and are logged for backfill; they do not silently corrupt normalized tables.

## 10. Trading-day vs non-trading-day behavior

- **Trading days:**
  - Universe sync → daily incremental (after close) → daily official pipeline.
  - Hourly 5-minute refresh runs during the session.
- **Non-trading days (weekends/holidays):**
  - No daily incremental, no official recomputation.
  - No hourly intraday refresh.
  - Storage retains the **latest valid snapshot**, which the dashboard serves unchanged.
- A trading-calendar service determines day type and drives scheduler gating.

## 11. Scheduler behavior (high level)

- A scheduler gates jobs by **day type** (from the trading calendar) and **time of day**.
- **Daily official chain** (trading days, post-close): universe sync → incremental update → quality checks → official clustering/pattern pipeline → snapshot publish.
- **Hourly intraday chain** (trading days, during session): 5-minute refresh → quality checks → intraday snapshot publish.
- Jobs are **idempotent and resumable**, so a missed or retried run converges to the correct state.
- On non-trading days the scheduler **skips** compute jobs; the API continues serving the last published snapshots.
