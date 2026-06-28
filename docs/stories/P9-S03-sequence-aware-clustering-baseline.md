# P9-S03 Sequence-Aware Clustering Baseline

## Risk
high-risk

## Execution status
future-draft (run after the P9-S01 baseline and after cost control is accepted)

## Goal
Add sequence/shape-based clustering baselines (k-Shape, DTW, and an MPdist similarity graph on daily return sequences) for comparison, writing versioned outputs. Nightly, cost-controlled.

## Prerequisites
- P8-S02 behavior windows completed.
- P9-S01 baseline completed.
- MPdist cost-control strategy accepted.
- S3/artifact storage strategy available if large matrices are produced.

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
- `pipelines/common/distance.py`

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

## Optional split (to reduce risk)
- `P9-S03a-kshape-baseline.md`
- `P9-S03b-dtw-baseline.md`
- `P9-S03c-mpdist-graph-baseline.md`

## Notes for agent
Baselines for comparison. Daily sequences first. Respect MPdist cost controls and the out-of-scope 5-minute rule. Do not start before the P9-S01 baseline exists.
