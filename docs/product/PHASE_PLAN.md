# Phase Plan

Build order for the Vietnamese stock behavior clustering dashboard. Phases are sequenced so each depends only on earlier ones. Each story is a bounded packet under `docs/stories/`. The system is **decision-support analytics, not prediction or trading** — no story implements buy/sell recommendations.

## Phases

| Phase | Theme | Stories |
|---|---|---|
| P1 | Repository scaffold | `P1-S01-repo-scaffold` |
| P2 | Data foundation | `P2-S00-data-source-probe`, `P2-S01-supabase-schema` |
| P3 | Backend skeleton | `P3-S01-fastapi-health` |
| P4 | Worker skeleton | `P4-S01-worker-skeleton` |
| P5 | Deployment files | `P5-S01-aws-deployment-files` |
| P6 | Frontend shell | `P6-S01-vercel-web-shell` |
| P7 | Ingestion | `P7-S01-hourly-5m-refresh`, `P7-S02-daily-incremental-update` |
| P8 | Features & windows | `P8-S01-feature-computation`, `P8-S02-behavior-window-generation` |
| P9 | Clustering | `P9-S01-feature-based-current-clustering`, `P9-S02-correlation-graph-leiden-clustering`, `P9-S03-sequence-aware-clustering-baseline` |
| P10 | Explanation & analysis | `P10-S01-historical-pattern-retrieval`, `P10-S02-cluster-outcome-analysis`, `P10-S03-lead-lag-analysis` |
| P11 | Dashboard API integration | `P11-S01-dashboard-api-integration` |
| P12 | Dashboard views | `P12-S01-dashboard-cluster-views`, `P12-S02-dashboard-ticker-explanation` |

## Sequencing rules

- P2 schema precedes all ingestion/feature/model work.
- P7 ingestion precedes P8 features/windows; P8 precedes P9 clustering; P9 precedes P10 retrieval/outcomes/lead-lag.
- P11 wires the dashboard to precomputed outputs; P12 builds views on top.
- Scheduler-driven stories (P7+) must skip jobs on non-trading days via `trading_calendar`; the dashboard reads the latest valid snapshot on non-trading days.

## Cross-cutting constraints

- Daily OHLCV is the official source; 5-minute OHLCV is hourly current-state only (no live 5-minute streaming).
- Clustering operates on behavior features/windows, never raw price, and is not a predictor.
- Retrieval and outcome analysis are explanation/validation layers; avoid look-ahead bias.
- No secrets or service-role keys in code or the frontend bundle.
