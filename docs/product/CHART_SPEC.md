# docs/product/CHART_SPEC.md

# Strata — Chart Specification

> All charts are **descriptive analytics of movement behavior**, never price predictions or signals. They are flat, hairline, mono-labelled, and use only design-system tokens. No gradients beyond the single subtle area fill defined here; no decorative animation (one optional draw-in on mount, respecting `prefers-reduced-motion`).
>
> Implement as lightweight inline SVG (the prototype does this with hand-built paths — no charting library is required for the MVP). Numbers/labels in `--font-mono`; signed percentages use a true minus `−`.

---

## Shared conventions

- **Color:** a chart tied to a cluster uses that cluster's color (`--cluster-x`); price/neutral series use `--ink-600`. Outcome direction uses muted `--pos-500` / `--neg-500`, always with a label.
- **Gridlines:** faint dotted hairlines `--border-subtle`, `stroke-dasharray: 2 3`. A zero baseline uses solid `--border-strong`.
- **Axes:** minimal. Y reference labels (top/mid/bottom) as small mono text in `--text-subtle`; x is usually implicit (window index / trading days).
- **Responsiveness:** charts render into a fixed `viewBox` and scale to container width (`width: 100%`). Heights are fixed per chart below.
- **Empty/short data:** if fewer than 2 points, render the empty frame (gridlines only) — never throw.

---

## 1. Sparkline  (inline, used in cards & table cells)

- **Meaning:** a tiny normalized **shape preview** of recent movement behavior — explicitly *not* a price chart.
- **Spec:** single path, `stroke-width` ~1.25, rounded joins; optional subtle area fill (linear gradient from `stroke @ 0.16` to `0`). Auto-scales to its own min/max.
- **Sizes:** ~84×22 in tables, ~260×44 in cluster cards.
- **Data shape:** `number[]` (normalized series; absolute scale irrelevant).

---

## 2. Normalized return paths  (Cluster Detail → Behavior profile)

- **Meaning:** how all members of a cluster have moved over the current window, rebased so each starts at 0%. Shows the cluster's coherence (members move alike).
- **Spec (viewBox ≈ 660×188):**
  - Each member series rebased to `v / v0 − 1` (fraction return). Plot as faint lines (cluster color, opacity ~0.18, `stroke-width` 1).
  - Overlay the **cluster mean** as a bold line (cluster color, `stroke-width` 2).
  - 3 horizontal gridlines (max / mid / min of the value range) + a solid zero line; right-aligned mono % labels at the gridlines.
- **Copy:** "All {size} members rebased to 0% at window start; bold line is the cluster mean. Shape of movement, not price level."
- **Data shape:** `members: { ticker: string; normalized_path: number[] }[]` (or derive from each member's `behavior_window`). MVP may synthesize the bundle from the cluster prototype when per-member paths aren't yet exposed by the API — mark as derived.

---

## 3. Prototype shape  (Cluster Detail → Behavior profile)

- **Meaning:** the cluster archetype — the centroid (median normalized path) with the cross-member dispersion band. This is the "what this cluster looks like" reference shape.
- **Spec (viewBox ≈ 660×188):**
  - Compute per-timestep **p25 / p50 / p75** across member normalized paths.
  - Render the p25–p75 area as a translucent band (cluster color, opacity ~0.13) with thin p25 & p75 edge lines (opacity ~0.4).
  - Render the **p50 centroid** as a bold line (`stroke-width` ~2.4).
  - Dotted zero baseline.
- **Copy:** "Median normalized path (centroid) with the p25–p75 dispersion band across members — the archetypal shape this cluster represents."
- **Data shape:** same member-path input as §2.

---

## 4. Price history  (Ticker Detail → Price history)

- **Meaning:** daily closes through the last trading day, for orientation. Behavior features are derived from price/volume — never clustered on raw price level (state this near the chart).
- **Spec (viewBox ≈ 980×220, `preserveAspectRatio: none`):**
  - Single close line (cluster color or `--ink-600`), `stroke-width` 1.5, `vector-effect: non-scaling-stroke`.
  - Subtle area fill below the line (linear gradient `color @ 0.14 → 0`).
  - 3 horizontal gridlines at 25/50/75%.
- **Data shape:** `bars: { date: string; open; high; low; close; volume }[]` from `/api/tickers/{t}/history`. MVP renders close only; OHLC candles are post-MVP.

---

## 5. Outcome distribution  (Cluster Detail → Outcomes)

- **Meaning:** the distribution of forward returns at a selected horizon (5d/20d/60d) across historical analog cases for the cluster. **Descriptive context only**, always shown with `n` and the disclaimer.
- **Spec (viewBox ≈ 660×150):**
  - Histogram of ~29 bins across a symmetric span centred on 0%. Bars colored muted `--pos-500` (x>0) / `--neg-500` (x<0) / `--ink-400` (≈0) at ~0.5 opacity. (MVP may shape the histogram from the quartiles `p25/median/p75` when raw samples aren't exposed.)
  - Shade the **p25–p75 interquartile band** behind the bars (`--blue-50`).
  - Solid zero axis (`--border-strong`); bold **median** tick (`--blue-600`).
  - Baseline axis + mono % labels at left / 0% / right.
- **Companion readout:** below the chart, `p25` / `median` / `p75` values (mono) + the note "Interquartile band shaded; sample sizes below 30 marked low-confidence."
- **Stat tiles above the chart:** Median fwd return, Excess vs VNINDEX, Median max drawdown (all `OutcomeValue`), Hit rate (fwd > 0) with a "low confidence" sub when `n < 30`.
- **Data shape (`/clusters/{id}/outcomes`):**
  ```json
  { "horizon": "20d", "n_samples": 240,
    "fwd_return": { "mean": 0.012, "median": 0.008, "p25": -0.01, "p75": 0.03 },
    "max_drawdown": { "mean": -0.04, "median": -0.03 },
    "excess_return_vs_index": { "mean": 0.004, "median": 0.002 } }
  ```

---

## 6. Lead-lag relationship view  (Cluster & Ticker Detail → Lead-lag)

- **Meaning:** behaviorally nearest tickers (graph neighbors / strongest edges) and which series has tended to move ahead. Describes historical co-movement **structure**, not causation or a signal. Uses the **teal** secondary color (structural relations).
- **MVP spec:** a 2-column list of neighbor cards rather than a force graph — `arrow-right-left` (teal), neighbor ticker (mono), `same cluster` `Badge` when applicable, "{leads|lags} by {n}d", and edge weight (mono). Row click → ticker detail.
- **Post-MVP option:** a small node-link or adjacency view; keep it hairline, teal-accented, with stable cluster colors.
- **Data shape (`/tickers/{t}/neighbors`):**
  ```json
  { "ticker": "AAA", "neighbors": [ { "ticker": "BBB", "weight": 0.88, "same_cluster": true } ] }
  ```
  Lead/lag direction + lag days are display fields; if the endpoint omits them in MVP, render weight + same-cluster only and hide the lead/lag phrase.

---

## Accessibility & export notes

- Every chart needs a text equivalent nearby (the stat tiles, p-value readouts, or table already provide this). Do not rely on color alone — outcome direction is always paired with a signed label.
- Respect `prefers-reduced-motion`: skip the draw-in.
- Charts are decorative-free; safe to rasterize for any PDF/report export.
