# DEC-001: Use FastAPI for the Backend API

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The dashboard needs a backend that serves **precomputed snapshots** (clusters, historical patterns, supporting outputs) to the Next.js frontend. The backend does no heavy computation at request time — that work lives in the worker. The team is Python-centric, since ingestion, features, clustering, and retrieval are all Python.

## Decision

Use **FastAPI** (Python) as the backend API, running on AWS EC2 behind Nginx, serving read-optimized snapshots from Supabase Postgres.

## Rationale

- **Same language as the data/ML stack** — shared models, types, and utilities with the worker; no second-language boundary.
- **Lightweight and fast** for a read-mostly API serving snapshots.
- **Good developer ergonomics** — typed request/response, automatic docs, async support.
- **Easy to run** as a long-lived systemd service on EC2, which fits the always-on serving plane.

## Consequences

- The API stays thin: it reads the latest valid snapshot and returns it; it must not run clustering/retrieval inline.
- Requires a Python runtime and process management (systemd) on EC2-API.
- CORS must allow the Vercel frontend origin.

## Alternatives considered

- **Node/Express backend** — would split the stack into two languages and duplicate model definitions; rejected.
- **Serverless (Lambda + API Gateway)** — poor fit for always-on, stateful Python ML libraries and excluded by the MVP "no Lambda" constraint; rejected.
- **Direct DB access from the frontend** — leaks DB credentials and couples the UI to the schema; rejected.
