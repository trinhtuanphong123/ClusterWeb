# Historical Pattern Retrieval Spec

> The **explanation and validation layer** for clustering — not the main task and **not direct prediction**. It retrieves historically similar behavior windows to explain current stocks and clusters.

## 1. Purpose

Explain current stocks and clusters by retrieving **similar historical windows**: "this stock's/cluster's current behavior resembles these past windows." This makes clusters interpretable and credible. Outputs are **descriptive context, never forecasts or recommendations**.

## 2. Query window

- The **query window** is a current rolling behavior window (from `behavior_windows`) for a ticker or a cluster-representative behavior profile, ending at the current as-of date `T`.
- Defined by `(ticker, window_end_date, window_length, feature_schema_version)`.

## 3. Candidate historical window set

- The **candidate set** is the collection of past behavior windows of the **same window length and feature schema** across the universe (and/or the same ticker's history), drawn from `behavior_windows`.
- This is the search space the query is compared against.

## 4. Exclusion rule

- To avoid trivially matching the **same current period**, exclude candidate windows that overlap the query window's date range (and a buffer around `T`).
- This prevents the retrieval from "finding itself" and ensures matches are genuinely historical.

- Default exclusion buffer: at least max(window_length, evaluation_horizon) trading days around the query end date, configurable by method.

## 5. Top-K nearest matches

- For each query window, return the **top K** most similar historical windows, ranked by similarity.
- K is configurable; results are stored with their rank.

## 6. Methods

- **Feature-vector nearest neighbor** — distance over window feature vectors (fast baseline).
- **Normalized correlation** — correlation between normalized shape vectors.
- **DTW** — Dynamic Time Warping distance on shape sequences (tolerant of small time shifts).
- **Matrix Profile** — subsequence-similarity-based matching.
- **MPdist** — Matrix-Profile-based distance over daily return sequences (see `MPDIST_SPEC.md`).

Methods can be combined or compared; the method used is recorded per match.

## 7. Outputs

Stored in `historical_pattern_matches` (linking query window → matched window), with supporting outcome fields:

- **matched ticker** — the ticker of the historical match.
- **date range** — the matched window's start/end (`match_window_end_date`, length).
- **method** — which similarity method produced the match.
- **similarity score** — distance/similarity value.
- **rank** — position among the top K for the query.
- **window length** — the length used.
- **future returns** — return following the matched window (context).
- **drawdown** — max drawdown following the matched window (context).
- **excess return** — return vs the index following the matched window (context).

## 8. Not direct prediction

Because prices are non-stationary, similar past behavior does **not** guarantee similar future behavior. Retrieval and its outcome fields are presented strictly as **historical context that explains and validates current clusters** — never as a forecast, target, or trade signal.
