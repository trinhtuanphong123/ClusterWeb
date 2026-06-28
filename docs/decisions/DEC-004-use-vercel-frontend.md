# DEC-004: Use Next.js on Vercel for the Frontend

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The dashboard presents current behavior clusters, historical pattern explanations, lead-lag relationships, outcome summaries, and a watchlist. The frontend reads precomputed snapshots from the FastAPI backend; it does no heavy computation itself. The team wants a fast, low-ops way to ship and iterate on the UI.

## Decision

Build the frontend in **Next.js** and deploy it on **Vercel**, reading data from the EC2-hosted FastAPI API via a public base URL.

## Rationale

- **Next.js + Vercel** is a first-class pairing: simple deploys, preview environments per branch, instant production rollback.
- **Low operational overhead** — no servers to manage for the UI.
- **Clean separation** — the UI is a pure consumer of the API's snapshots, matching the decoupled serving plane.
- **Fast iteration** on dashboard views without touching backend infrastructure.

## Consequences

- The API must expose CORS to the Vercel domain(s) and a stable public URL (via Nginx on EC2-API).
- Only public, non-secret config (`NEXT_PUBLIC_API_BASE_URL`) is exposed to the browser; no DB credentials in the frontend.
- Frontend and backend deploy independently.

## Alternatives considered

- **Static site hosted on the same EC2** — adds web-serving responsibility to EC2 and loses Vercel's preview/rollback ergonomics; rejected.
- **Server-rendered app co-located with FastAPI** — couples UI and API lifecycles and increases EC2 load; rejected.
- **Other SPA frameworks without managed hosting** — more setup for no clear benefit over Next.js/Vercel; rejected.


## Related decisions

- DEC-001 — the FastAPI API this frontend consumes.
- DEC-011 — browser-safe config and the no-DB-credentials rule for the frontend.