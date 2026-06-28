# DEC-001: Use FastAPI for the Backend API

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The dashboard needs a backend that serves **precomputed snapshots** (clusters, historical patterns, supporting outputs) to the Next.js frontend. The backend does no heavy computation at request time — that work lives in the worker. The team is Python-centric, since ingestion, features, clustering, and retrieval are all Python.

## Decision

Use **FastAPI** (Python) as the backend API, running on **AWS EC2** as a long-lived service, serving read-optimized snapshots from Supabase Postgres.

For MVP deployment, FastAPI may be exposed directly from EC2 using the selected service port and security-group rules. A reverse proxy such as Nginx may be added later for TLS termination, routing, and production-like hardening, but Nginx is not required for initial AWS deployment..

## Rationale

- **Same language as the data/ML stack** — shared models, types, and utilities with the worker; no second-language boundary.
- **Lightweight and fast** for a read-mostly API serving snapshots.
- **Good developer ergonomics** — typed request/response, automatic docs, async support.
- **Easy to run** as a long-lived systemd service on EC2, which fits the always-on serving plane.

## Consequences

- The API stays thin: it reads the latest valid snapshot and returns it; it must not run clustering/retrieval inline.
- Requires a Python runtime and process management (systemd) on EC2-API.
- CORS must allow the Vercel frontend origin.
- The API must be deployable on AWS EC2 without requiring Nginx.
- A reverse proxy is optional and belongs to deployment hardening, not core backend architecture.
- If HTTPS/TLS or port 80/443 routing is required, a reverse proxy such as Nginx can be introduced in the deployment story.

## Alternatives considered

- **Node/Express backend** — would split the stack into two languages and duplicate model definitions; rejected.
- **Serverless (Lambda + API Gateway)** — poor fit for always-on, stateful Python ML libraries and excluded by the MVP "no Lambda" constraint; rejected.
- **Direct DB access from the frontend** — leaks DB credentials and couples the UI to the schema; rejected.
- **Mandatory Nginx reverse proxy from the start** — useful for production-like TLS/routing, but too specific for the initial AWS MVP; kept optional rather than required.


## Related decisions

- DEC-003 — the Postgres database this API reads from.
- DEC-004 — the Vercel frontend that consumes this API.
- DEC-011 — API security and credential boundary this API must honor.