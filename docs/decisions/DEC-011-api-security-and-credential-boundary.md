# DEC-011: API Security and Credential Boundary

- **Status:** Accepted
- **Date:** 2026-06-27
- **Deciders:** Project team

## Context

The system spans a Vercel frontend, a FastAPI API on EC2, a Python worker on EC2, and Supabase Postgres (DEC-001–DEC-004, DEC-007). These components handle credentials with different privilege needs. Without an explicit boundary, a future coding agent could leak database credentials into the browser bundle, give the API write access it should not have, commit secrets, or expose sensitive details through the health endpoint. This decision fixes the security and credential boundary. It does **not** design authentication.

## Decision

Define and enforce the following boundaries:

- The **frontend never receives database credentials**. It uses only browser-safe public configuration (e.g., `NEXT_PUBLIC_API_BASE_URL`).
- **FastAPI uses read-only (read-privilege) database credentials.** It serves precomputed outputs and never needs write access.
- The **worker uses read/write database credentials**, as the sole writer of ingested data and model outputs.
- **Secrets are provided only via environment variables or the deployment environment's secret store** — never hard-coded and never embedded in client code.
- The **`.env` file is local-development only** and must not be included in any handover, public, or committed artifact.
- **CORS follows an allowlist of approved frontend origins.**
- The **`/health` endpoint exposes no secrets, raw connection strings, stack traces, or sensitive infrastructure details** — only minimal liveness/readiness status.

## Rationale

- Least privilege: a read-only API limits blast radius if the API host is compromised; only the worker can mutate data.
- Keeping credentials out of the browser and out of committed files removes the most common leak paths.
- An origins allowlist and a minimal health endpoint reduce exposure without adding infrastructure.
- Scoping this to a boundary (not authentication) keeps the MVP lean (DEC-005) while making credential handling safe for future coding agents.

## Consequences

- Supabase needs at least two credential profiles: read-only (API) and read/write (worker).
- Deployment config must inject secrets from env/secret store; `.env` stays out of handover.
- The frontend build may expose only `NEXT_PUBLIC_*` values.

## Validation implications

- Confirm no `NEXT_PUBLIC_*` value (or any client-exposed config) carries a database credential or secret.
- Confirm `/health` output contains no connection strings, secrets, or stack traces.
- Confirm CORS rejects origins outside the approved allowlist.
- Confirm the API connects with read-only credentials and the worker with read/write credentials.

## Alternatives considered

1. Single shared DB credential for API and worker — rejected; violates least privilege and lets the API write.
2. Passing config to the browser without a public/secret split — rejected; risks leaking secrets into the bundle.
3. Adding authentication now — out of scope; this decision defines boundaries only, not authn/authz.

## Related decisions

- DEC-001 — the FastAPI API bound by the read-only credential rule.
- DEC-003 — the Postgres database whose credential profiles this splits.
- DEC-004 — the frontend bound by the browser-safe-config rule.
- DEC-005, DEC-007 — the lean topology this secures without new infrastructure.