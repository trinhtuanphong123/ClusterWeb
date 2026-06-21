# Feature Spec — Daily and Hourly Features

> Defines `features_daily` (used for current clustering) and `features_hourly` (dashboard context only). All features are scale-free or relative so they are comparable across stocks.

## 1. `features_daily`

Per-ticker daily behavior features derived from `daily_bars`. Keyed by `(ticker, feature_date, feature_schema_version)`.

| Feature | Description | Role |
|---|---|---|
| `ret` (returns) | Daily/period returns over the window | Clustering |
| `volatility` | Dispersion of returns (e.g., rolling std) | Clustering |
| `momentum` | Trend/drift over recent periods | Clustering |
| `drawdown` | Peak-to-trough downside within the window | Clustering |
| `volume_anomaly` | Volume vs its own recent baseline (participation anomaly) | Clustering |
| `beta_to_index` / `corr_to_index` | Sensitivity/correlation to the market index | Clustering |
| `mkt_rel_return` | Return net of the broad market | Clustering |
| `sector_rel_return` | Return net of the sector | Clustering |
| `shape_support_*` | Shape-supporting features (normalized path summary, slope/curvature descriptors) | Clustering (shape) |

These features describe **how** a stock is moving — shape, magnitude, downside, participation, and strength relative to market and sector — and form the input to **current behavior clustering**.

## 2. `features_hourly`

Per-ticker intraday features derived from `intraday_bars` (5-minute). Keyed by `(ticker, feature_time, feature_schema_version)`. **Dashboard context only — not used for official clustering.**

| Feature | Description |
|---|---|
| `last_1h_return` | Return over the last hour |
| `last_1h_volatility` | Volatility over the last hour |
| `intraday_return_since_open` | Return from the session open to now |
| `intraday_volume_ratio` | Intraday volume vs a typical baseline |
| `volume_acceleration` | Rate of change of volume |
| `price_position_in_day_range` | Where the current price sits within the day's high-low range |

## 3. Which features are used where

- **Current clustering:** `features_daily` only. Daily OHLCV is the official source; the behavior representation and all cluster runs use daily features.
- **Dashboard context only:** `features_hourly` powers the hourly current-state view. It is **provisional** and never feeds clustering or historical retrieval.

## 4. Missing data handling

- **Gaps on trading days:** flagged against the trading calendar; short gaps may be forward-filled within documented limits, longer gaps leave the feature null for that date.
- **Insufficient history:** if a ticker lacks enough history for a window, window-dependent features are null (the ticker is excluded from clustering for that window length until enough data exists).
- **Null propagation:** features computed from missing inputs are set null rather than imputed with misleading values; downstream jobs skip nulls explicitly.
- **Intraday gaps:** missing 5-minute bars within a session leave the affected hourly feature null; current-state shows the last valid value with a staleness flag.

## 5. Feature validation checks

- **Schema/columns:** expected feature columns present and typed.
- **Range/sanity bounds:** volatility ≥ 0, drawdown ≤ 0 (or in expected sign convention), returns within plausible bounds; flag absurd values.
- **Coverage:** fraction of active tickers with non-null features per date; alert below threshold.
- **Consistency:** market-/sector-relative returns reconcile with the underlying return and reference series.
- **No duplicates:** unique on the keying tuple.
- Failures are recorded in `data_quality_reports` and quarantine the affected rows rather than corrupting feature tables.

## 6. Feature schema versioning

- Every feature row carries a **`feature_schema_version`**, part of its unique key.
- Changing feature definitions or formulas → bump the version; old and new versions coexist so historical windows and clusters remain reproducible.
- `behavior_windows`, `cluster_runs`, and retrieval all record the `feature_schema_version` they used, so outputs are traceable to a feature definition.




6. recommended concrete features

ret_5d, ret_20d, ret_60d
vol_20d, vol_60d
max_drawdown_60d
volume_ratio_20d
mkt_rel_ret_20d
sector_rel_ret_20d
price_to_ma20
price_to_ma60