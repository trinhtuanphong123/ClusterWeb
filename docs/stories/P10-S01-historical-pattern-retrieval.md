# P10-S01 Historical Pattern Retrieval

## Risk
high-risk

## Goal
Retrieve top-K historically similar behavior windows for current query windows/clusters and write them to `historical_pattern_matches`. **Only match against windows available before the query date.**

## Prerequisites
- P8-S02 behavior windows completed.
- P9-S01 current clustering completed.
- `historical_pattern_matches` schema exists.


## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/HISTORICAL_PATTERN_RETRIEVAL_SPEC.md`
- `docs/product/MPDIST_SPEC.md`
- `docs/product/WINDOWING_SPEC.md`
- `docs/product/DATABASE_SCHEMA.md`
- `docs/decisions/ DEC-010-historical-pattern-retrieval-is-explanation-layer.md`

## Modify only
- `pipelines/retrieval/pattern_retrieval.py`
- `pipelines/jobs/retrieval.py`
- `pipelines/common/` (similarity helpers)

## Do not touch
- ingestion/feature/clustering pipelines
- frontend files

## Acceptance criteria
- Candidate set = historical windows of the same length/schema available **before** the query date; the exclusion rule prevents matching the same/overlapping current period.
- Top-K matches with similarity, rank, method, window length written to `historical_pattern_matches`, linked query→match windows.
- Runs after current clustering; calendar-gated.
- Framed as explanation/validation; **not prediction**; no buy/sell output.
- Unit/mock test must include a future-leakage prevention case.
- Candidate windows overlapping the query window or after the query date are excluded.

## Verification
See `docs/test-matrix/modeling.md`:
```bash
python -m pipelines.jobs.retrieval --mock --once
```

## Decision record
Required? no, unless retrieval method/output contract changes (then reference DEC-010).

## Notes for agent
Strictly no future leakage: candidates must predate the query window. Apply the exclusion rule and buffer around T.
