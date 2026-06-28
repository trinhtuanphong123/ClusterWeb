# DEC-002: Use an AWS EC2 Worker for Pipelines

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The system runs scheduled, sometimes heavy pipelines: ingestion, feature computation, rolling window generation, clustering, nightly sequence/graph jobs (Graph, Leiden, MPdist, Matrix Profile), historical pattern retrieval, outcome analysis, and snapshot generation. These need an **always-on scheduler**, stateful Python ML libraries, and predictable long-running execution.

## Decision

Run the worker (Python scheduler + all pipelines) on an **AWS EC2** instance as an always-on systemd service. In one-EC2 mode it shares a host with the API; in two-EC2 mode it is a dedicated **EC2-WORKER**.

## Rationale

- **Always-on scheduling** fits a long-lived process better than per-invocation compute.
- **Heavy ML/sequence jobs** (Matrix Profile, MPdist, Leiden) need real CPU/memory and long runtimes that don't fit short-lived serverless execution.
- **Stateful Python libraries and large in-memory artifacts** are natural on a persistent host.
- **Simple and cheap** — one VM with systemd, no orchestration platform.

## Consequences

- The team manages an EC2 host (patching, sizing, monitoring).
- Heavy jobs are scheduled at night to control cost and avoid contention.
- Worker needs write access to Postgres and optional S3 for large artifacts.

## Alternatives considered

- **AWS Lambda** — execution time/memory limits and cold starts make heavy ML jobs impractical; excluded by MVP constraint; rejected.
- **ECS/EKS** — container orchestration is overkill for a single worker process in the MVP; excluded by constraint; rejected.
- **Airflow** — a full orchestration platform is heavier than needed; a simple in-process scheduler with calendar gating suffices; rejected.


## Related decisions

- DEC-003 — the database the worker writes to.
- DEC-007 — one-EC2 / two-EC2 modes that place this worker.
- DEC-008 — trading-calendar gating that controls when worker jobs run.