# Model Runtime Strategy

> Separates **artifact-based models** (fit once, reuse) from **window-recomputed methods** (recomputed from scratch over windows). Scheduling is gated by the trading calendar; non-trading days skip scheduled model jobs unless a manual recompute is requested.

## 1. Two model runtime classes

### 1.1 Artifact-based models

- **Examples:** feature scaler/standardizer + **KMeans / GMM-like** clustering models.
- **Lifecycle:** fitted once (or periodically refit), serialized as artifacts, and reused for inference.
- **When they run:** **inference may run after feature computation** — once daily features exist, the saved scaler + model assigns current windows to clusters quickly, without refitting.
- **Storage:** the fitted objects live in `model_artifacts` (S3-backed), registered via `model_runs`.
- **Staleness caution:** because the official product is **current behavior clustering**, a long-lived fitted model can drift out of step with the market regime — repeatedly assigning today's stocks against centroids learned from an old period. Artifact reuse is therefore governed by explicit validity and refit rules (see Section 1.3).

### 1.2 Window-recomputed methods

- **Examples:** correlation graph construction, **Leiden** community detection, **k-Shape**, **DTW** clustering, **MPdist**, **Matrix Profile**, **historical pattern retrieval**, and **outcome analysis**.
- **Lifecycle:** recomputed from the current window data each run; they do not rely on a persistent fitted artifact in the same way.
- **When they run:** **nightly, after the daily update** (heavy/sequence-aware jobs), once daily bars/features/windows are in place.
- **Cost posture:** scheduled at night to control compute and avoid contention with daytime/intraday operations.

### 1.3 Artifact validity and refit governance

Artifact-based clustering models give fast inference but must not be trusted indefinitely. Every artifact-based model must record, in `model_artifacts` / `model_runs` metadata:

- **Training period** — the date range of data the model/scaler was fitted on.
- **Feature schema** — the `feature_schema_version` used at fit time.
- **Universe version** — the `stock_universe_members.universe_version` the model was fitted against.
- **Refit policy** — how often the model is refit (e.g., a fixed cadence and/or a trigger such as feature-schema change, universe drift, or a degradation metric).
- **Validity period** — the window during which the artifact may be reused for **official** inference (an explicit "valid until" or a max age in trading days).

Rules:

- An artifact may back an **official** clustering run only while it is **within its validity period** and its **feature schema and universe version match** the current run's. Otherwise it is **stale**.
- **If the artifact is stale, the pipeline must not publish its output as official** without either a **refit** or an explicit **manual validation**. A stale-backed run may still be computed for inspection, but it is marked invalid (`cluster_runs.is_valid = false`) and not served by the API.
- Refit/validation events are logged via `model_runs`; the resulting fresh artifact starts a new validity period and is the one referenced by subsequent official `cluster_runs`.

This prevents the failure mode of a KMeans/GMM model trained in a past regime being reused to assign clusters long after it has become unrepresentative. Window-recomputed methods (Section 1.2) are naturally immune, since they recompute from current windows each run.

## 2. Scheduling summary

| Class | Trigger | Timing |
|---|---|---|
| Artifact-based inference | After daily feature computation | Same evening, after features ready |
| Window-recomputed methods | Nightly after daily update | Night, after daily bars/features/windows |

- **Non-trading days:** scheduled model jobs (both classes) **skip internally** via the trading calendar. They run only if a **manual recompute** is explicitly requested (e.g., backfill or re-run).
- Ordering follows the dependency chain in `SCHEDULER_STRATEGY.md`: daily update → features → windows → clustering → nightly heavy jobs → historical retrieval → outcome analysis → snapshot.

## 3. Table responsibilities

- **`model_artifacts`** — registry of serialized artifacts (scalers, fitted KMeans/GMM, distance matrices, vector indexes) stored in S3; referenced by the run that created them. Records artifact validity/refit metadata (training period, feature schema, universe version, refit policy, validity period — see Section 1.3) so stale artifacts are not used for official runs.
- **`model_runs`** — lineage log for generic model/computation runs (run type, schema version, status, params, timing, errors). The generic `run_id` anchor for non-cluster outputs.
- **`model_outputs`** — generic per-run outputs without a dedicated table (payload or S3 pointer), keyed to `model_runs`.
- **`cluster_runs`** — one row per official clustering run; the **versioning anchor (`run_id`)** for all cluster outputs (as-of date, universe/feature versions, algo, validity).
- **`cluster_members`** — ticker→cluster assignments for a run (one assignment per ticker per run, with membership score and representative flag).
- **`cluster_edges`** — pairwise similarity/graph edges for a run; supports neighbor queries and graph methods (correlation graph, Leiden, MPdist graph).
- **`historical_pattern_matches`** — links current query windows to matched historical windows for a run (similarity, rank); the explanation layer's output.
- **`cluster_outcome_stats`** — future-return/drawdown summaries by cluster, horizon, and run; descriptive validation context only.
- **`dashboard_snapshots`** — precomputed, read-optimized snapshots assembled after all required outputs are ready, so the API/frontend read the latest valid state quickly.

## 4. Principles

- **Idempotent + versioned:** every model output is tied to a `run_id`; re-runs converge and multiple runs coexist.
- **Reader/writer split:** the worker writes all model outputs; the API only reads precomputed results (no model execution at request time).
- **Cost control:** heavy window-recomputed methods are nightly; artifact-based inference is cheap and runs right after features.