# Design Context Pack — Vietnamese Stock Behavior Clustering Dashboard

> This document is the **sole context** for Claude Design when designing the dashboard. It synthesizes the product, data, model, and API documentation into what a designer needs. Read it fully before designing. Do not infer features beyond what is described here.

---

## 1. Project name

**Vietnamese Stock Behavior Clustering and Historical Pattern Explanation Dashboard.**

---

## 2. Product goal

A decision-support analytics dashboard that, at any given time, answers two questions:

1. **Which Vietnamese-listed stocks are currently behaving alike?** — by clustering stocks on their recent movement behavior.
2. **What has behavior like this looked like before?** — by retrieving historically similar windows and summarizing what tended to follow, purely as context.

The product helps users *understand current market structure*, not act on a trade signal.

---

## 3. Main problem

Investors and analysts have no clear, structural view of which stocks are moving in similar ways right now, or what that behavior has historically meant. Raw price charts are hard to compare across stocks (different price levels, volatility, and noise), and manual inspection does not scale across the whole universe.

The dashboard solves this by grouping stocks into **current behavior clusters** and explaining each cluster with **historical analogs**.

---

## 4. Non-goals

The UI must never present or imply any of the following:

- **No price prediction** — no predicted prices, targets, or forecasts.
- **No buy/sell/hold recommendations** — no trade signals, no position sizing.
- **No financial advice.**
- **No brokerage / trading execution.**
- **No real-time trading terminal** — intraday state refreshes hourly, not live.
- **No gamification** — no scores-as-rewards, streaks, leaderboards, or celebratory framing.
- **No fundamentals** (earnings, valuation) — behavior is derived from price/volume only.

---

## 5. Target users

- **Retail and semi-professional investors** who want to understand current market structure and rotation.
- **Market analysts and researchers** studying co-movement, sector dynamics, and lead-lag relationships.
- **Internal exploratory users** validating whether clusters are stable and interpretable.

Assume an analytically literate audience that values clarity, honesty about uncertainty, and explainability over flashy visuals.

---

## 6. Core workflow

The user's mental journey through the product:

1. **Land on an overview** — see the current market split into behavior clusters, plus market status and data freshness.
2. **Explore a cluster** — open a cluster to see its members, its behavior profile (volatility, drawdown, momentum, relative strength), and representative stocks.
3. **Understand why it groups** — view historical analogs that resemble the cluster's current behavior, and the descriptive outcome statistics that followed those analogs.
4. **Drill into a stock** — open a ticker to see its cluster membership, behavioral neighbors, historical pattern matches, outcome context, and current intraday state.
5. **Check trust signals** — at any point, see how fresh the data is, whether it is stale, and whether today is a trading day.

Throughout, the framing is **explanation and context**, never instruction.

---

## 7. Main dashboard pages

1. **Overview / Home** — current clusters at a glance, market status, freshness banner, cluster summary cards.
2. **Cluster detail** — one cluster's members, behavior profile, representatives, neighbors, and its historical-pattern explanation + outcome context.
3. **Ticker detail** — one stock's cluster membership, behavioral neighbors, historical matches, outcome context, daily history, and hourly current-state.
4. **Pipeline / status** — operational view: recent job runs, freshness fields, and any failures (can be a lightweight admin/ops page).
5. **Methodology / About** — plain-language explanation of what the product does and does not do (clustering is primary; retrieval/outcomes are explanation; not prediction or advice).

A ticker list / universe browse view (searchable, filterable by sector/exchange) supports navigation into ticker detail.

---

## 8. Data and model outputs the UI must display

- **Current behavior clusters** — cluster membership, size, behavior profile (volatility, drawdown, momentum, volume, market-relative and sector-relative strength), and representative members. Versioned by a run with an as-of date.
- **Cluster edges / neighbors** — pairwise behavioral similarity, used for neighbor lists and optional graph views.
- **Historical pattern matches** — for a stock or cluster: matched ticker, matched date range, method, similarity score, rank, and the window length.
- **Cluster outcome statistics** — by cluster and horizon (5/20/60 trading days): hit rate, forward-return distribution (mean/median/quartiles), excess return vs VNINDEX, and max drawdown. **Descriptive context only.**
- **Lead-lag edges** — directed leader→follower relationships with a lag (in trading days) and strength.
- **Ticker current-state** — last price, intraday return, intraday volatility, volume, and an "as-of" time; refreshed hourly during trading sessions.
- **Daily price history** — OHLCV for charting.
- **Freshness / market status** — `last_data_update`, `last_feature_update`, `last_model_update`, `latest_cluster_run`, `latest_snapshot`, `is_stale`, plus whether today is a trading day and the day type.

The backend serves **precomputed snapshots**; the UI reads and displays them. The UI never triggers model computation.

---

## 9. API endpoints relevant to the frontend

All are read-only and return precomputed outputs. Indicative shapes are described in words; the UI should treat responses as already-assembled snapshots.

- `GET /api/market/status` — trading-day flag, day type, session state, freshness fields, `is_stale`, latest snapshot reference.
- `GET /api/overview` — landing payload: cluster summary, market status, freshness.
- `GET /api/clusters/latest` — list of current clusters for the latest valid run.
- `GET /api/clusters/{cluster_id}` — one cluster: profile, members, representatives.
- `GET /api/clusters/{cluster_id}/outcomes` — outcome statistics by horizon (descriptive).
- `GET /api/clusters/{cluster_id}/historical-patterns` — historical analogs explaining the cluster.
- `GET /api/tickers` — searchable/filterable universe list with current cluster assignment.
- `GET /api/tickers/{ticker}` — ticker detail: metadata, current cluster, behavior summary.
- `GET /api/tickers/{ticker}/history` — daily OHLCV for charts.
- `GET /api/tickers/{ticker}/current-state` — hourly intraday state.
- `GET /api/tickers/{ticker}/historical-patterns` — historical analogs for the ticker.
- `GET /api/tickers/{ticker}/neighbors` — behaviorally nearest tickers.
- `GET /api/pipeline/status` — recent job runs and freshness for the status page.
- `GET /api/methodology/summary` — content for the methodology/about page.
- `GET /health` — service health (not a user-facing view).

Every data-bearing response may carry an `is_stale` flag and freshness timestamps; surface these in the UI.

---

## 10. Key concepts the UI must explain

The UI is responsible for making these understandable to a non-expert. Each should have a clear, plain-language explanation available (tooltip, info panel, or methodology page).

- **Current stock behavior clustering** — stocks are grouped by *how they are moving now* (return shape, volatility, drawdown, volume, strength relative to market and sector), not by price level. A cluster is a **current behavior state, not a permanent label** — a stock can move between clusters over time as its behavior changes.
- **Historical pattern retrieval** — for a stock or cluster, the system finds *past windows that resemble the current behavior*. This **explains and validates** the current grouping ("this looks like these past periods"). It is **not a prediction**.
- **Cluster outcome analysis** — for the historical analogs, the system summarizes what *tended to follow* (forward returns, excess return vs index, drawdowns) by horizon. This is **descriptive historical context**, explicitly **not a forecast** and not a recommendation.
- **Lead-lag analysis** — within and across clusters, some stocks tend to move *before* others. The UI shows directed leader→follower relationships with a lag, as context for interpreting a cluster's internal structure.
- **Stale data and non-trading-day behavior** — on weekends, holidays, and before the day's refresh, no new computation happens. The dashboard shows the **latest valid snapshot** (as of the last trading day) and must clearly indicate that the view is not live (stale badge, as-of date, "markets closed" state).

---

## 11. Visual tone

**Serious, analytical, explainable.** The aesthetic should read like a research/analytics instrument:

- Calm, professional, information-dense but legible.
- Restrained color used for meaning (e.g., cluster identity, relative strength), not for excitement.
- Explanation-first: every nontrivial number or grouping should be explainable on demand.
- **Not gamified.** No reward mechanics, no hype, no celebratory motion.
- **Not a trading recommendation tool.** Nothing should look like a signal to act.

Avoid red/green being read as "sell/buy." If using directional color for returns, pair it with neutral labeling and never with action verbs.

---

## 12. Strict UI language rules

These are hard rules for all copy, labels, tooltips, empty states, and chart annotations:

- **Do not show buy / sell / hold** — anywhere, in any form, including icons or color coding implying action.
- **Do not claim prediction** — no "will", "expected to", "forecast", "target", "signal", or implied future certainty about a specific stock.
- **Use decision-support language only** — frame everything as description and context: "currently behaves like", "historically, similar behavior was followed by", "this cluster's recent profile", "as of the last trading day".
- Outcome statistics must be labeled as **historical context, not predictions**.
- Where a user might over-read a result, add a brief clarifying note rather than removing the information.

Preferred verbs/phrases: *behaves like, resembles, is grouped with, historically followed, tended to, descriptive, as-of.*
Banned verbs/phrases: *buy, sell, hold, recommend, signal, forecast, predict, will, target price, guaranteed.*

---

## 13. Recommended charts

Choose charts that aid understanding, not spectacle:

- **Cluster map / scatter** — a 2D layout of the universe colored by cluster (e.g., projected behavior space) to show structure at a glance.
- **Cluster cards / small multiples** — compact profile per cluster (volatility, drawdown, momentum, relative strength) as mini bar/radar summaries.
- **Normalized behavior path overlay** — rebased (scale-free) line overlays comparing a stock to its cluster or to a historical analog, so *shape* is comparable across price levels.
- **Historical analog comparison** — side-by-side current window vs matched historical window (normalized), with similarity and date range.
- **Outcome distribution** — for a cluster/horizon, a distribution or box-style summary of forward returns / excess returns / drawdowns, clearly labeled as historical.
- **Lead-lag diagram** — a small directed graph or arrow list showing leader→follower with lag and strength.
- **Neighbor list** — ranked behavioral neighbors with similarity weights.
- **Daily price + volume chart** — standard OHLCV for the ticker detail (context, not a trading chart with signals).
- **Freshness / status indicators** — a compact freshness banner and a pipeline status view.

Keep axes, units, and "as-of" dates explicit on every chart.

---

## 14. Edge cases

Design explicit states for each. None should look broken or alarming; all should be honest.

- **No data** — a ticker or universe has no records yet. Show a clear empty state explaining data isn't available, not an error.
- **Stale data** — the latest snapshot is older than expected. Show a stale badge, the as-of date, and a short explanation; still render the latest valid content.
- **Non-trading day** — weekend/holiday/manual closure. Show a "markets closed / showing latest valid snapshot (as of [date])" state; the dashboard remains fully usable on last-trading-day data.
- **Failed pipeline** — a recent job failed. On the status page, surface the failure plainly; on user pages, fall back to the last valid snapshot and indicate staleness rather than showing a broken view.
- **Missing model output** — clusters, historical matches, or outcomes not yet computed for a run. Show a graceful "not available for this run yet" state for the affected section while keeping the rest of the page functional.

Across all edge cases: prefer the **latest valid snapshot + an honest freshness indicator** over an error screen.

---

## 15. Instruction block for Claude Design

> **Design brief:**
> Design a serious, analytical, explanation-first dashboard for clustering Vietnamese stocks by current behavior and explaining those clusters with historical analogs.
>
> **Always:**
> - Treat outputs as descriptive analytics and historical context.
> - Make every concept (clustering, retrieval, outcomes, lead-lag, staleness) explainable in plain language on demand.
> - Show data freshness and trading-day status prominently; design the non-trading-day and stale states as first-class views.
> - Use the five page types: Overview, Cluster detail, Ticker detail, Pipeline/status, Methodology.
> - Pull only from the listed data outputs and read-only API endpoints; the UI never runs models.
>
> **Never:**
> - Show buy/sell/hold, signals, or any call to action on a trade.
> - Claim or imply prediction, forecasts, or price targets.
> - Gamify, hype, or use color/iconography that reads as a recommendation.
>
> **Tone:** calm, professional, research-instrument quality; restrained color used for meaning; legible and information-rich.
>
> If a design choice risks implying a recommendation or a forecast, choose the more neutral, descriptive alternative.
