# P10-S02 Cluster Outcome Analysis

## Risk
high-risk

## Goal
Summarize future returns, drawdowns, and excess return vs VNINDEX by cluster, horizon (5/20/60d), and run into `cluster_outcome_stats`, using walk-forward evaluation. **Avoid look-ahead bias.**

## Prerequisites
- P9-S01 clustering completed.
- P10-S01 retrieval completed if outcomes are tied to retrieved matches.
- Outcome schema exists.
- Sufficient historical daily bars exist.


## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/CLUSTER_OUTCOME_EVALUATION.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/product/METHODOLOGY.md`

## Modify only
- `pipelines/outcomes/outcome_analysis.py`
- `pipelines/jobs/outcomes.py`
- `pipelines/common/` (evaluation helpers)

## Do not touch
- ingestion/feature/clustering/retrieval pipelines
- frontend files

## Acceptance criteria
- Forms clusters/matches using only data available at the as-of date; outcomes computed strictly from data **after** that date (no look-ahead).
- Writes `cluster_outcome_stats` per `(run_id, cluster_id, horizon)`: hit rate, forward-return distribution, excess return vs VNINDEX, max drawdown.
- Walk-forward evaluation and a random baseline comparison available.
- Outcomes are NOT fed back as labels into the unsupervised clustering; descriptive context only.
- Runs after retrieval; calendar-gated.
- Mock verification must include a no-look-ahead test.
- Outcomes must be displayed/stored as descriptive distributions, not labels or predictions.



## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.outcomes --mock --once
```

## Decision record
Required? no, unless evaluation methodology changes (then add DEC referencing the methodology).

## Notes for agent
Walk-forward only. Never use forward data to form clusters/matches. Outcomes validate meaning; they are not training labels and not predictions.
