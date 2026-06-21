# DEC-007: Support Both One-EC2 and Two-EC2 Deployment Modes

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The system has two roles — a thin always-on API and a heavier always-on worker. In staging, cost matters most; in production-like environments, isolating the worker's heavy nightly jobs from the API's serving path matters more. The same codebase should serve both without changes.

## Decision

Support two deployment modes sharing one codebase and database:

- **One-EC2 (staging):** API and worker run on a **single EC2** as **separate systemd services** (`fastapi-api.service`, `worker.service`); Nginx optional.
- **Two-EC2 (production-like):** **EC2-API** runs **FastAPI + Nginx**; **EC2-WORKER** runs the scheduler, ingestion, features, clustering, historical pattern retrieval, and snapshot generation.

## Rationale

- **Cost flexibility** — one host for cheap staging; two hosts when isolation is worth the cost.
- **Resource isolation in production** — heavy nightly jobs on the worker don't degrade API responsiveness.
- **No code divergence** — both modes use the same services, env-var contract, DB, and optional S3; mode is an operational choice.
- **Smooth scaling path** — start on one EC2, split to two without rework.

## Consequences

- Deployment tooling/config must parameterize which services run on which host.
- The DB and S3 contracts are identical across modes, so the worker and API behave the same regardless of co-location.
- Health checks and systemd units exist for both layouts.

## Alternatives considered

- **Single mode only (always two EC2)** — wastes cost in staging; rejected.
- **Single mode only (always one EC2)** — risks API/worker contention under heavy production jobs; rejected.
- **Container orchestration to manage placement** — excluded by the lean-harness constraint (no ECS/EKS) and unnecessary; rejected.
