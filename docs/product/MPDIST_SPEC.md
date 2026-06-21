# MPdist Spec

> MPdist (Matrix-Profile-based distance) measures similarity between two sequences based on how many of their subsequences are close. Used as an advanced similarity method for clustering and historical retrieval. **Compute-cost controlled and nightly.**

## 1. Purpose

- Provide a **subsequence-aware distance** between behavior sequences that is robust to noise and small misalignments, capturing similarity in movement *patterns* rather than point-by-point values.
- Feed a **similarity graph** for community detection and/or a **historical match list** for the explanation layer.

## 2. Use daily return sequences first

- MPdist operates on **daily return sequences** in the MVP (derived from `daily_bars`/`features_daily`).
- Daily sequences are the official, lower-volume data and keep compute manageable.

## 3. Why 5-minute MPdist is not MVP

- 5-minute sequences are far longer and far more numerous, making pairwise MPdist **computationally expensive** at universe scale.
- 5-minute data is **provisional** (hourly current-state only) and not the official clustering source.
- Therefore **5-minute MPdist is out of scope for the MVP**; daily MPdist is sufficient for behavior similarity.

## 4. Outputs

MPdist results can be materialized as any of:

- a **distance matrix** (pairwise MPdist over the universe), stored as an artifact in `model_artifacts` (S3) when large,
- a **similarity graph** (thresholded edges) in `cluster_edges`,
- a **cluster run** (via community detection on the graph) in `cluster_runs` / `cluster_members`, or
- a **historical match list** in `historical_pattern_matches`.

## 5. Compute-cost control

- Run **nightly after the daily update**, never intraday.
- **Subsample / restrict** the candidate set (e.g., within sector or within an initial coarse cluster) before full pairwise MPdist.
- **Cap sequence length** and universe size per run; prune low-similarity pairs early.
- Persist large matrices to **S3** (referenced by `model_artifacts`) rather than Postgres.
- Reuse results across downstream consumers (graph + retrieval) from one computation.

## 6. When not to use MPdist

- When a cheaper method (feature-vector nearest neighbor, normalized correlation) already gives adequate, interpretable similarity.
- For intraday/5-minute data (out of MVP scope).
- When the universe/sequence size would make pairwise computation infeasible within the nightly budget.

## 7. Fallback methods if MPdist is too expensive

- **Feature-vector nearest neighbor** on window feature vectors (cheapest).
- **Normalized correlation** on shape vectors.
- **DTW** on a reduced candidate set (still expensive, but targeted).
- Coarse pre-clustering (e.g., KMeans) to shrink the candidate set before any sequence-distance method.
- MPdist requires a subsequence length m. In MVP, m should be tied to the selected behavior window length or set as a configurable fraction of it, for example m = 10 for 20-day windows, m = 20 for 60-day windows, and m = 30 for 120-day windows. The selected m must be recorded in cluster_runs metadata or historical_pattern_matches metadata.
