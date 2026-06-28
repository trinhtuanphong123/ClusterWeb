# Story Backlog

This backlog tracks story readiness for harness-based implementation.

Story packets may exist as drafts, but only stories marked `ready` should be executed.

## Status meaning

- `done`: implemented and verified
- `ready`: can be executed next
- `needs-hardening`: story needs scope/prerequisite/verification cleanup
- `blocked`: depends on earlier stories
- `split-required`: too broad and must be split before execution
- `future-draft`: kept as roadmap only

## Story readiness

| Story | Title | Status | Notes |
| --- | --- | --- | --- |
| P1-S01 | Repository Scaffold | done | Scaffold completed. |
| P3-S01 | FastAPI Health Endpoint | needs-hardening | Add dependency scope and safer DB health behavior. |
| P6-S01 | Vercel Web Shell | ready | Ready after apps/web scaffold is confirmed. |
| P4-S01 | Worker Skeleton | needs-hardening | DB logging must be optional or blocked by core schema. |
| P2-S00 | Data Source Probe | needs-hardening | Clarify scratch path, dependencies, and no-DB guardrails. |
| P2-S01 | Supabase Schema | split-required | Too broad for one high-risk story. |
| P5-S01 | AWS Deployment Files | blocked | Depends on API and worker entrypoints. |
| P7-S01 | Hourly 5m Refresh | blocked | Depends on probe, schema, worker. |
| P7-S02 | Daily Incremental Update | blocked | Depends on probe, schema, worker. |
| P8-S01 | Feature Computation | blocked | Depends on daily ingestion and schema. |
| P8-S02 | Behavior Window Generation | blocked | Depends on feature computation and schema. |
| P9-S01 | Feature-Based Current Clustering | blocked | Depends on behavior windows and method decision. |
| P9-S02 | Correlation Graph Leiden Clustering | future-draft | Later comparison method. |
| P9-S03 | Sequence-Aware Clustering Baseline | future-draft | Later comparison method. |
| P10-S01 | Historical Pattern Retrieval | blocked | Depends on windows and clustering. |
| P10-S02 | Cluster Outcome Analysis | blocked | Depends on clustering/retrieval. |
| P10-S03 | Lead-Lag Analysis | blocked | Depends on clustering and windows. |
| P11-S01 | Dashboard API Integration | blocked | Depends on API health, schema, snapshots. |
| P12-S01 | Dashboard Cluster Views | blocked | Depends on web shell and API endpoints. |
| P12-S02 | Dashboard Ticker Explanation | blocked | Depends on web shell and API endpoints. |