# P9-S01 Feature-Based Current Clustering

## Risk
high-risk

## Goal
Implement current behavior clustering from feature vectors (KMeans/GMM baseline, artifact-based scaler+model), writing versioned output to `cluster_runs`, `cluster_members`. **Current behavior clustering from features — NOT trading/price prediction.**

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/CLUSTERING_SPEC.md`
- `docs/product/METHODOLOGY.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/MODEL_RUNTIME_STRATEGY.md`

## Modify only
- `pipelines/clustering/feature_clustering.py`
- `pipelines/jobs/clustering.py`
- `pipelines/common/` (model artifact helpers)

## Do not touch
- ingestion/feature pipelines
- retrieval/outcome pipelines
- frontend files

## Acceptance criteria
- Clusters the current universe from behavior feature vectors at a chosen window length; never raw price; never a price/return predictor.
- Writes `cluster_runs` (run_id, as-of date, universe/feature versions, algo, validity) and `cluster_members` (one assignment per ticker per run).
- Artifact-based scaler+model registered in `model_artifacts`; inference may run after feature computation.
- Reproducible given the same inputs and `feature_schema_version`.

## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.clustering --mock --once --method kmeans
```

## Decision record
Required? yes — `docs/decisions/DEC-xxx-clustering-method.md` (model methodology). Reference DEC-009.

## Notes for agent
This produces behavior clusters, not forecasts. No buy/sell outputs. Keep outputs versioned by run_id.
