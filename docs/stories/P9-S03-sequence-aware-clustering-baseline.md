# P9-S03 Sequence-Aware Clustering Baseline

## Risk
high-risk

## Goal
Add sequence/shape-based clustering baselines (k-Shape, DTW, and an MPdist similarity graph on daily return sequences) for comparison, writing versioned outputs. Nightly, cost-controlled.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/CLUSTERING_SPEC.md`
- `docs/product/MPDIST_SPEC.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/MODEL_RUNTIME_STRATEGY.md`

## Modify only
- `pipelines/clustering/sequence_clustering.py`
- `pipelines/jobs/clustering_sequence.py`
- `pipelines/common/` (distance helpers)

## Do not touch
- ingestion/feature pipelines
- retrieval/outcome pipelines
- frontend files

## Acceptance criteria
- k-Shape and DTW operate on shape vectors; MPdist uses **daily return sequences** (5-minute MPdist is out of MVP scope).
- Outputs versioned by run_id in `cluster_runs`/`cluster_members`/`cluster_edges`.
- Compute-cost controls applied (candidate restriction, length caps, S3 for large matrices); nightly + calendar-gated.
- Fallback to cheaper methods documented when MPdist is too expensive.
- Not a predictor; no buy/sell output.

## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.clustering_sequence --mock --once --method kshape
```

## Decision record
Required? no, unless a sequence method replaces the primary method (then reference DEC-009).

## Notes for agent
Baselines for comparison. Daily sequences first. Respect MPdist cost controls and out-of-scope 5-minute rule.
