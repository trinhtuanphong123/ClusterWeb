# P9-S01 Feature-Based Current Clustering

## Risk
high-risk

## Execution status
blocked (high-risk design gate — lock method before implementation)

## Goal
Implement the **first** feature-based current clustering baseline from behavior feature vectors, starting with **one primary method** (KMeans) before adding alternatives. Behavior clustering from features — **NOT trading/price prediction**.

## Prerequisites
- P8-S02 behavior windows completed.
- Clustering output schema exists (P2-S01d: `cluster_runs`, `cluster_members`).
- Feature schema version is available.
- Clustering-method decision exists, **or** this story includes a decision step (record `docs/decisions/DEC-xxx-clustering-method.md`) before implementation.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/CLUSTERING_SPEC.md`
- `docs/product/METHODOLOGY.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/MODEL_RUNTIME_STRATEGY.md`
- `docs/decisions/DEC-009-main-task-is-current-behavior-clustering.md`

## Modify only
- `pipelines/clustering/feature_clustering.py`
- `pipelines/jobs/clustering.py`
- `pipelines/common/model_artifacts.py`

## Do not touch
- ingestion/feature pipelines
- retrieval/outcome pipelines
- frontend files

## Acceptance criteria
- Clusters the current universe from behavior feature vectors at a chosen window length; never raw price; never a price/return predictor.
- Writes `cluster_runs` (run_id, as-of date, universe/feature versions, algo, validity) and `cluster_members` (one assignment per ticker per run).
- Artifact-based scaler+model registered in `model_artifacts` with validity/refit metadata per `MODEL_RUNTIME_STRATEGY.md`; inference may run after feature computation.
- Method choice and parameters are recorded.
- Reproducible given the same inputs and `feature_schema_version`.
- No buy/sell, price prediction, or ranking language in code, output, or comments.
- Mock verification uses fixture features and checks deterministic output.

## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.clustering --mock --once --method kmeans
```

## Decision record
Required? yes — `docs/decisions/DEC-xxx-clustering-method.md` (model methodology). Reference DEC-009.

## Optional split (to reduce scope)
- `P9-S01a-kmeans-baseline.md`
- `P9-S01b-gmm-comparison.md`

## Notes for agent
This produces behavior clusters, not forecasts. Start with one primary method; add GMM/alternatives only after the baseline is in place. Keep outputs versioned by run_id.
