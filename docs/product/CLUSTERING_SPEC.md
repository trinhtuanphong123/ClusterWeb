# Clustering Spec — Current Behavior Clustering

> Defines feature-based current clustering, its baselines and advanced methods, evaluation, stability, output tables, and a model-comparison plan. The main task is grouping stocks by **current behavior** (see `METHODOLOGY.md`).

## 1. Feature-based current clustering

- Operates on the **feature vectors** (and, for shape methods, **shape vectors**) from `behavior_windows` at a chosen window length.
- Input is the current universe `S` at as-of date `T`; output is cluster assignments versioned by `run_id`.
- Features are standardized (scaler) before distance-based methods.

## 2. Baseline methods

### 2.1 KMeans baseline
- Partitions feature vectors into `k` clusters by minimizing within-cluster variance.
- **Artifact-based:** scaler + fitted KMeans can be saved and reused for fast inference after feature computation.
- Requires choosing `k`; evaluated via the metrics in §9.

### 2.2 GMM baseline
- Gaussian Mixture Model; soft assignments with cluster probabilities/covariances.
- **Artifact-based:** fitted GMM reusable as an artifact.
- Captures elliptical clusters and membership uncertainty (useful for membership scores).

### 2.3 Hierarchical clustering baseline
- Agglomerative clustering producing a dendrogram; clusters cut at a chosen level.
- Useful for inspecting nested structure and for comparison; typically window-recomputed.

## 3. Correlation graph construction

- Build a graph where nodes are tickers and edge weights come from **pairwise return correlation** (or behavior similarity) over the window.
- Threshold/prune weak edges to form a meaningful similarity graph.
- Stored in `cluster_edges`; basis for community detection.

## 4. Leiden clustering

- Community detection on the correlation/similarity graph (Leiden algorithm) to find behavior communities.
- **Window-recomputed**, run nightly after the daily update.
- Produces graph-based clusters that don't require pre-specifying `k`.

## 5. k-Shape baseline

- Shape-based clustering on the **shape vectors** using a shape-based distance (cross-correlation alignment).
- Groups stocks with similar movement *trajectories* regardless of amplitude.
- Window-recomputed.

## 6. DTW clustering baseline

- Clustering using **Dynamic Time Warping** distance on shape sequences, tolerant of small time shifts/stretches.
- More expensive; window-recomputed nightly.

## 7. MPdist similarity graph (advanced)

- Build a similarity graph using **MPdist** (Matrix-Profile-based distance) over daily return sequences, then apply community detection.
- Advanced method; cost-controlled and nightly. See `MPDIST_SPEC.md`.

## 8. Cluster evaluation

- **Internal metrics:** silhouette, Davies–Bouldin, Calinski–Harabasz on the behavior representation.
- **Interpretability:** cluster profiles (typical volatility, drawdown, momentum, relative strength) should be human-readable and distinct.
- **Cross-method agreement:** agreement between methods (e.g., adjusted Rand index) as a sanity signal.
- Evaluation guides method/parameter choice but is **not** used to manufacture labels for the unsupervised task.

## 9. Cluster stability

- **Temporal stability:** how much assignments change run-to-run (expected some movement, since clusters are current states — see `WINDOWING_SPEC.md`).
- **Perturbation stability:** robustness to small changes in window length or feature subset.
- **Membership scores:** GMM probabilities / graph centrality quantify how strongly a stock belongs to its cluster.
- Stability is reported, not forced; excessive instability flags parameter problems.

## 10. Output tables

- **`cluster_runs`** — one row per run (`run_id`, as-of date, universe/feature versions, algo, params, validity).
- **`cluster_members`** — ticker→cluster assignments per run (membership score, representative flag).
- **`cluster_edges`** — pairwise similarity/graph edges per run (for correlation graph, Leiden, MPdist graph, neighbor queries).

## 11. Model comparison plan

- Run multiple methods (KMeans, GMM, hierarchical, Leiden, k-Shape, DTW, MPdist graph) over the same windows.
- Compare on internal metrics, interpretability, stability, and outcome coherence (via `CLUSTER_OUTCOME_EVALUATION.md`, used as **validation only**, not as labels).
- Select a primary method per window length, keep others for cross-checking; record method/params in `cluster_runs` so comparisons are reproducible.

- For official nightly clustering, methods may be recomputed on the latest current universe. Artifact-based KMeans/GMM inference is a fast runtime option, but the fitted artifact must record training window, feature schema, universe, and validity period.