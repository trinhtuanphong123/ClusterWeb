# docs/product/UI_SPEC.md

# Strata — UI Specification (pages, structure, states)

> Page-by-page spec for the MVP Next.js frontend. Each page lists: purpose, layout structure, components used, API endpoints, and the loading / empty / error / stale / non-trading-day behavior. Charts are specified in `CHART_SPEC.md`. Tokens and components are in `DESIGN_SYSTEM.md`.
>
> Framing is binding: **current behavior clustering is the product; historical retrieval explains/validates; outcome stats are descriptive context only; no buy/sell/hold, no prediction, no recommendation.**

---

## App shell (every page)

- **Grid:** `248px | 1fr`, full viewport height, no body scroll; the main column scrolls.
- **Sidebar** (`--ink-950`): Strata mark + wordmark; `SidebarNav` with two sections —
  - Primary: Overview (`layout-grid`), Clusters (`boxes`), Tickers (`list`)
  - Analysis: Watchlist (`eye`), Pipeline status (`activity`), Methodology (`book-open`)
  - Footer: "DATA FRESHNESS" eyebrow + `FreshnessPill(stale, asOf)` + `run {id} · {day_type}` in mono.
- **Top bar** (white, 56px): back breadcrumb (detail pages only) + page title (sentence case) on the left; search field, `run {id}` accent badge, and a refresh `IconButton` on the right.
- **Main:** centered max-1440 column, 24px padding, 16–18px vertical rhythm between panels.
- **Active nav** maps detail routes to their parent (cluster-detail → Clusters, ticker-detail → Tickers).

---

## 1. Overview  ·  route `/`  ·  `GET /api/overview` (+ `GET /api/market/status`)

**Purpose:** landing view — market/session status, data freshness, and the cluster summary.

**Structure (top to bottom):**
1. **Stale-data banner** (amber, `alert-triangle`) — shown when `market_status.is_stale`. Copy: "Markets are closed ({day_type}). Showing the latest valid snapshot, as-of {as_of_date}. No new official computation runs on non-trading days."
2. **Status card row (3 cards):** Market status (`clock`, value = Closed/Open, sub = session + exchanges, pill = Stale/Live), Latest data update (`refresh-cw`, value = HH:MM, sub = date · daily OHLCV, pill = Valid), Latest model update (`boxes`, value = HH:MM, sub = date · clustering run #id, pill = Valid).
3. **Stat tile row (4 `StatTile`):** Behavior clusters (`n_clusters`), Tickers in universe (`n_tickers`), As-of date (accent), Retrieval methods (4 · DTW · MPdist · corr · MP).
4. **`PanelHeader`** "Cluster summary" (eyebrow CURRENT MARKET STRUCTURE) + secondary "All clusters" button → `/clusters`.
5. **Cluster summary cards** — first 6 clusters as `ClusterCard` (see Clusters page), 3-up grid.
6. **Two-up footer:** "Pipeline freshness" `Card` (last data/feature/model update + latest run rows) and "What this dashboard is" `Card` (serif lead + `DisclaimerNote`).

**Components:** status cards, `StatTile`, `PanelHeader`, `Button`, `ClusterCard`, `Card`, `FreshnessPill`, `DisclaimerNote`, `ClusterDot`, `Sparkline`, `ProfileChip`.

---

## 2. Clusters  ·  route `/clusters`  ·  `GET /api/clusters/latest?include=profiles`

**Purpose:** full list of current behavior clusters for the latest valid run.

**Structure:**
1. **`PanelHeader`** (eyebrow LATEST VALID RUN) title "{n} current behavior clusters" + description (grouping uses behavior representations, never raw price level). Actions: sector `Select` + `SegmentedControl` (Grid / Table).
2. **Grid view:** responsive `ClusterCard` grid (min 280px). Each card: `ClusterDot` + size, `Sparkline` (cluster color, filled), 4 `ProfileChip` (Vol / Shape / Drawdown / Mkt-rel), representatives row. Whole card links to `/clusters/{id}`.
3. **Table view:** `Card(padded=false)` table — Cluster, Size (right, mono), Volatility, Path shape, Drawdown, Mkt-rel, Representatives (mono), chevron. Row click → detail.

**Components:** `PanelHeader`, `Select`, `SegmentedControl`, `ClusterCard`, `Card`, table cells, `ClusterDot`, `Sparkline`, `Badge`.

---

## 3. Cluster Detail  ·  route `/clusters/{cluster_id}`  ·  `GET /api/clusters/{id}` + `/outcomes` + `/historical-patterns` + `/tickers/{rep}/neighbors`

**Purpose:** members, behavior profile, descriptive outcome distributions, historical analogs, lead-lag.

**Structure:**
1. **Header `Card`:** `ClusterDot` + "Cluster {id}" (h2) + size `Badge`; `ProfileChip` row (Volatility / Path shape / Drawdown / Volume / Mkt-relative); cluster `Sparkline` (right).
2. **`Tabs`:** Members (count) · Behavior profile · Outcomes · Historical patterns (count) · Lead-lag (count).
3. **Members tab** → member table: Ticker (+ `rep` badge), Company, Sector, Membership (right, mono), Behavior `Sparkline`, Mkt-rel 20d (`OutcomeValue`), chevron → `/tickers/{ticker}`.
4. **Behavior profile tab** → profile stat grid (6 descriptive cells) + **Normalized return paths** chart + **Prototype shape** chart (see `CHART_SPEC.md`).
5. **Outcomes tab** → horizon `SegmentedControl` (5d/20d/60d) + `DisclaimerNote` + 4 `StatTile` (median fwd return, excess vs VNINDEX, median max drawdown, hit rate with low-confidence flag if `n<30`) + **outcome distribution** chart with p25/median/p75 readout. All values `OutcomeValue` / mono; always show `n = {n_samples}`.
6. **Historical patterns tab** → `PanelHeader` (eyebrow EXPLANATION & VALIDATION) + `DisclaimerNote` (analogs, not prediction) + matches table (rank, match ticker → detail, window range, method `Badge`, similarity, fwd 5d/20d/60d, drawdown 20d).
7. **Lead-lag tab** → neighbor cards (`arrow-right-left` teal, ticker, same-cluster `Badge`, "leads/lags by Nd", weight).

**Components:** `Card`, `ClusterDot`, `Badge`, `ProfileChip`, `Sparkline`, `Tabs`, `StatTile`, `SegmentedControl`, `DisclaimerNote`, `OutcomeValue`, tables, charts.

---

## 4. Tickers  ·  route `/tickers`  ·  `GET /api/tickers?q=&sector=&limit=&offset=`

**Purpose:** active universe with current cluster assignment and behavior summary.

**Structure:** `PanelHeader` (eyebrow UNIVERSE) title "{count} tickers" + actions: search `Input` (debounced → `q`) + sector `Select`. Table (`Card padded=false`): Ticker (mono), Company, Sector, Exch (mono), Cluster (`ClusterDot`), Behavior `Sparkline`, Mkt-rel 20d (`OutcomeValue`), chevron. Row click → `/tickers/{ticker}`. Paginate via `limit`/`offset` (MVP: simple "load more" or fixed page size).

**Components:** `PanelHeader`, `Input`, `Select`, `Card`, table cells, `ClusterDot`, `Sparkline`, `OutcomeValue`.

---

## 5. Ticker Detail  ·  route `/tickers/{ticker}`  ·  `GET /api/tickers/{t}` + `/history` + `/current-state` + `/historical-patterns` + `/neighbors`

**Purpose:** per-ticker metadata, current cluster, OHLCV history, analogs, lead-lag.

**Structure:**
1. **Header `Card`:** ticker (mono h2) + exchange `Badge`; "{company} · {sector}"; cluster button (`ClusterDot` + "membership {score}") → cluster detail. Right: two `StatTile` — Last price (sub "VND '000 · last close") and Intraday (current state) `OutcomeValue` (sub "5m source · stale" when not in session).
2. **Chip row:** Volatility 20d, Mkt-rel 20d, Cluster `ProfileChip` + `FreshnessPill(stale, asOf, label="Current state stale")` when `current_state.is_stale`.
3. **`Tabs`:** Price history · Historical patterns (count) · Lead-lag (count).
4. **Price history tab** → `Card` + `PanelHeader` (eyebrow DAILY OHLCV) + **price area chart** (daily closes). Note: behavior features derive from price/volume — never clustered on raw price level.
5. **Historical patterns tab** → `DisclaimerNote` + matches table (same shape as cluster patterns).
6. **Lead-lag tab** → neighbor cards.

**Components:** `Card`, `Badge`, `StatTile`, `OutcomeValue`, `ClusterDot`, `ProfileChip`, `FreshnessPill`, `Tabs`, price chart, tables.

---

## 6. Watchlist  ·  route `/watchlist`  ·  derived (MVP: from clusters/patterns; future dedicated endpoint)

**Purpose:** analytical watchlist — tickers whose current window strongly matches notable historical patterns or cluster archetypes, flagged for analyst attention by descriptive criteria. **Not a trading signal.**

**Structure:** `PanelHeader` (eyebrow ANALYTICAL WATCHLIST) + `DisclaimerNote` + table: Ticker (mono), Cluster (`ClusterDot`), Why flagged (descriptive reason, e.g. "High similarity + large sample"), Top similarity (right, mono), Sample (`n = ...`), chevron → ticker detail.

**MVP data source:** compose from `/api/clusters/latest` + `/historical-patterns` (highest-similarity, largest-sample matches). Mark as derived in code comments; swap to a dedicated endpoint when available.

**Components:** `PanelHeader`, `DisclaimerNote`, `Card`, table cells, `ClusterDot`.

---

## 7. Pipeline Status  ·  route `/pipeline`  ·  `GET /api/pipeline/status`

**Purpose:** operational freshness and recent job runs. No model execution at request time — the dashboard reads precomputed snapshots.

**Structure:**
1. **`PanelHeader`** (eyebrow OPERATIONS) + `FreshnessPill(stale, asOf)`.
2. **Stat tile row (4):** Last data update, Last feature update, Last model update, Latest cluster run (accent).
3. **Run timeline `Card`:** vertical timeline of `recent_jobs` ordered by `started_at` — status dot (fresh/running/skipped/failed color), job name (mono), status `Badge`, trigger + ended/in-progress, error message in `--neg-600` when present.
4. **Recent jobs table:** Job, Status `Badge(dot)`, Trigger, Started, Ended, Note (error or —).

**Status tones:** succeeded → positive, running → accent, skipped → neutral, failed → negative.

**Components:** `PanelHeader`, `FreshnessPill`, `StatTile`, `Card`, `Badge`, table.

---

## 8. Methodology  ·  route `/methodology`  ·  `GET /api/methodology/summary`

**Purpose:** the framing/"About" page — clustering is primary, retrieval explains, nothing predicts.

**Structure:** `PanelHeader` (eyebrow ABOUT) "Methodology"; serif lead paragraph; definition rows (Main task, Explanation layer, Features, Data sources, Update frequency, Horizons); two `Card`s ("How clusters form", "How retrieval validates"); a dark `Card` ("This dashboard is not": a price forecaster · a recommendation engine · financial advice · a real-time trading terminal). Max width ~820px.

**Components:** `PanelHeader`, `Card`, definition list, pill chips.

---

## States (apply to every page)

### Loading
- **Server-rendered pages:** stream the shell (sidebar + top bar) immediately; show a centered muted "Loading latest valid snapshot…" in the content area until data resolves. Use skeleton rows for tables (hairline placeholder rows) and a flat placeholder box at chart dimensions.
- Never block the whole viewport — the shell and freshness footer render first.

### Empty
- Use `EmptyState` with neutral copy. Examples: "No valid cluster run yet." / "Retrieval not computed for this run." / "No members in this cluster." / "No jobs recorded." Provide a single plain action where useful ("Select a cluster to view its members").

### Error
- API error body is `{ "error": { "code", "message" } }`. Map:
  - `400` invalid params → inline "Invalid request" empty-state; reset filters/pagination.
  - `404` unknown resource → "Not found" `EmptyState` + back link to the parent list.
  - `503` no valid snapshot/data yet → "No valid snapshot is available yet." `EmptyState` with `alert-triangle`; keep the freshness footer visible.
  - `500` → generic "Something went wrong." with a retry affordance (re-fetch).
- Errors never fabricate values; show the empty/error surface instead of zeros.

### Stale data (`is_stale: true`)
- Sidebar `FreshnessPill` renders amber `Stale · as-of {date}`.
- Overview shows the amber stale banner.
- Outcome / retrieval / pipeline surfaces show `as-of` timestamps.
- Ticker current-state shows the "Current state stale" pill.
- Use the API `is_stale` flag as the source of truth; never compute staleness ad hoc on the client beyond display.

### Non-trading day (`is_trading_day: false`)
- Read endpoints return the **latest valid snapshot** (as-of the last trading day); render it normally with stale treatment above.
- `day_type` (`weekend` / `holiday` / `manual_closure`) drives the banner copy.
- Pipeline shows `intraday_snapshot` as **skipped** with note "Non-trading day ({day_type})".
- No recomputation is triggered or implied anywhere in the UI; copy states "No new official computation runs on non-trading days."
