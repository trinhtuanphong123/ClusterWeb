# Data Source Capabilities — VNStock and DNSE Probe Template

> **Purpose:** This is a **template for recording real probe results** against the VNStock community/free API. Do **not** fill in values from assumption or memory. Every cell must be backed by an actual probe. **Do not claim exact API limits unless verified by probing.**
>
> Fill the `Result` / `Observed` columns after running probes. Leave unknowns as `UNVERIFIED`.

---

## 0. Probe metadata

| Field | Value |
|---|---|
| Probe date (local) | _____ |
| VNStock version | _____ |
| Source/provider used | _____ |
| Operator | _____ |
| Notes | _____ |

---

## 1. Ticker universe

| Check | How probed | Result |
|---|---|---|
| Can list full ticker universe? | _____ | UNVERIFIED |
| Number of tickers returned | _____ | UNVERIFIED |
| Exchanges covered (HOSE/HNX/UPCOM) | _____ | UNVERIFIED |
| Symbol format / casing | _____ | UNVERIFIED |
| Sector/industry mapping available? | _____ | UNVERIFIED |
| Listing status / delisted handling | _____ | UNVERIFIED |
| Columns returned | _____ | UNVERIFIED |
| Refresh cadence of universe list | _____ | UNVERIFIED |

---

## 2. Daily OHLCV (official source)

| Check | How probed | Result |
|---|---|---|
| Available for full universe? | _____ | UNVERIFIED |
| **Lookback** — earliest date available | probe earliest fetchable date | UNVERIFIED |
| **Lookback** — max history per request | _____ | UNVERIFIED |
| **Source check** — which provider serves it | _____ | UNVERIFIED |
| **Columns** returned (e.g. time, open, high, low, close, volume) | inspect response | UNVERIFIED |
| Adjusted vs unadjusted prices? | _____ | UNVERIFIED |
| **Timezone** of timestamps | inspect raw timestamps | UNVERIFIED |
| **Duplicates** — repeated bars per day? | check for dup keys | UNVERIFIED |
| **Missing bars** — gaps on trading days? | compare vs trading calendar | UNVERIFIED |
| Handling of non-trading days in response | _____ | UNVERIFIED |
| **Rate-limit observations** (qualitative) | record throttling/errors seen | UNVERIFIED |

---

## 3. 5-minute OHLCV (hourly dashboard state only)

| Check | How probed | Result |
|---|---|---|
| Available for full universe? | _____ | UNVERIFIED |
| **Lookback** — how many days back | probe earliest fetchable | UNVERIFIED |
| **Source check** — provider | _____ | UNVERIFIED |
| **Columns** returned | inspect response | UNVERIFIED |
| Bar alignment (e.g. 5-min boundaries) | inspect timestamps | UNVERIFIED |
| **Timezone** of timestamps | inspect raw timestamps | UNVERIFIED |
| **Duplicates** — repeated intraday bars? | check dup keys | UNVERIFIED |
| **Missing bars** — intraday gaps? | compare vs session hours | UNVERIFIED |
| Latency vs real time (delay observed) | _____ | UNVERIFIED |
| **Rate-limit observations** | record throttling/errors | UNVERIFIED |

---

## 4. Hourly OHLCV (if available)

| Check | How probed | Result |
|---|---|---|
| Endpoint/interval exists? | _____ | UNVERIFIED |
| **Lookback** range | _____ | UNVERIFIED |
| **Source check** | _____ | UNVERIFIED |
| **Columns** returned | _____ | UNVERIFIED |
| **Timezone** | _____ | UNVERIFIED |
| **Duplicates** | _____ | UNVERIFIED |
| **Missing bars** | _____ | UNVERIFIED |
| **Rate-limit observations** | _____ | UNVERIFIED |
| Decision: derive hourly from 5-min instead? | _____ | UNVERIFIED |

---

## 5. Price board (if available)

| Check | How probed | Result |
|---|---|---|
| Price board endpoint exists? | _____ | UNVERIFIED |
| Fields returned (bid/ask/last/volume/etc.) | inspect response | UNVERIFIED |
| Universe coverage per call | _____ | UNVERIFIED |
| **Source check** | _____ | UNVERIFIED |
| **Timezone** of snapshot time | _____ | UNVERIFIED |
| Snapshot freshness / delay | _____ | UNVERIFIED |
| **Duplicates** across calls | _____ | UNVERIFIED |
| **Rate-limit observations** | _____ | UNVERIFIED |
| Intended use (intraday context only) | _____ | UNVERIFIED |

---

## 6. Raw intraday trades / tick (if available)

| Check | How probed | Result |
|---|---|---|
| Tick/trade-level endpoint exists? | _____ | UNVERIFIED |
| **Lookback** available | _____ | UNVERIFIED |
| **Source check** | _____ | UNVERIFIED |
| **Columns** (time, price, volume, side) | inspect response | UNVERIFIED |
| **Timezone** | _____ | UNVERIFIED |
| Volume / size of payloads | _____ | UNVERIFIED |
| **Duplicates** | _____ | UNVERIFIED |
| **Missing data** | _____ | UNVERIFIED |
| **Rate-limit observations** | _____ | UNVERIFIED |
| Decision: in/out of MVP scope | _____ | UNVERIFIED |

---

## 7. Cross-cutting observations

| Topic | Observation |
|---|---|
| Authentication required? | UNVERIFIED |
| Stability / error rates seen | UNVERIFIED |
| Pagination behavior | UNVERIFIED |
| Bulk vs per-ticker fetch efficiency | UNVERIFIED |
| Recommended polite request pacing (empirical) | UNVERIFIED |
| Known caveats / surprises | UNVERIFIED |

> **Reminder:** Any rate limit, lookback bound, or capability above must be marked `UNVERIFIED` until directly observed via probing. Replace placeholders only with probe-backed facts.

# MVP source decision after probing

Daily OHLCV: usable / not usable.
5-minute OHLCV: usable / not usable.
Ticker universe: usable / not usable.
Sector mapping: usable / not usable.
Official clustering source: confirmed / blocked.