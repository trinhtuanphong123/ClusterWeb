# P9-S02 Correlation Graph + Leiden Clustering

## Risk
high-risk

## Execution status
future-draft (run after the P9-S01 baseline exists)

## Goal
Build a correlation/similarity graph over the window and apply Leiden community detection, writing edges to `cluster_edges` and communities to `cluster_runs`/`cluster_members`. Window-recomputed, nightly. This is a comparison/advanced method, not the first baseline.

## Prerequisites
- P9-S01 feature-based clustering completed.
- `cluster_edges` schema exists (P2-S01d).
- Graph method is approved as a comparison method or primary method.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/CLUSTERING_SPEC.md`
- `docs/product/METHODOLOGY.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/MODEL_RUNTIME_STRATEGY.md`
- `docs/decisions/DEC-009-main-task-is-current-behavior-clustering.md`

## Modify only
- `pipelines/clustering/graph_leiden.py`
- `pipelines/jobs/clustering_graph.py`
- `pipelines/common/graph.py`

## Do not touch
- ingestion/feature pipelines
- retrieval/outcome pipelines
- frontend files

## Acceptance criteria
- Builds graph from pairwise behavior similarity/correlation; prunes weak edges; writes `cluster_edges` per run.
- Leiden communities written to `cluster_runs`/`cluster_members`, versioned by run_id.
- Runs nightly after the daily update; calendar-gated.
- Not a predictor; no buy/sell output.

## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.clustering_graph --mock --once
```

## Decision record
Required? yes if this becomes the primary method or changes the result contract — reference DEC-009. Otherwise no.

## Notes for agent
Window-recomputed, nightly. Large matrices go to S3 via `model_artifacts`. Keep edges canonical to avoid duplicates. Do not start before the P9-S01 baseline exists.
