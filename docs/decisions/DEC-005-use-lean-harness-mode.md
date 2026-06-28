# DEC-005: Use Lean Harness Mode for the MVP

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The MVP must be cheap, debuggable, and operable by a small team. There is a temptation to adopt heavy enterprise infrastructure (Lambda, ECS, EKS, Kafka, Airflow, Redshift) for scheduling, streaming, and orchestration. The actual workload is a handful of long-lived services plus scheduled batch jobs gated by a trading calendar.

## Decision

Adopt a **lean harness mode**: a minimal, intentionally small operational footprint — a few long-lived services on EC2 (FastAPI, worker/scheduler), Supabase Postgres, optional S3, and a Vercel frontend. Explicitly avoid Lambda, ECS, EKS, Kafka, Airflow, Redshift, and similar heavy infrastructure in the MVP.

## Rationale

- **Operability** — fewer moving parts means fewer failure modes and easier debugging for a small team.
- **Cost control** — avoids fixed costs and complexity of orchestration/streaming/warehouse platforms.
- **Right-sized** — the workload (daily batch + hourly intraday refresh) does not need streaming or a distributed scheduler.
- **Reversible** — the lean shape can be scaled later (e.g., two-EC2 mode) without rework; nothing here blocks future growth.

## Consequences

- Some capabilities (autoscaling, managed orchestration, streaming) are deliberately deferred.
- Scheduling is handled by a simple always-on worker with calendar gating rather than a platform like Airflow.
- If scale demands change, a future decision can revisit specific components — but the MVP stays lean.

## Alternatives considered

- **Full enterprise stack (Airflow + ECS/EKS + Kafka + Redshift)** — high complexity and cost for an MVP with modest, batchy workloads; rejected.
- **Serverless-first** — poor fit for always-on scheduling and heavy stateful ML jobs; rejected.


## Related decisions

- DEC-001, DEC-002, DEC-003, DEC-004 — the minimal component set this lean mode comprises.
- DEC-007 — the one-EC2 / two-EC2 topology that keeps the lean footprint scalable.