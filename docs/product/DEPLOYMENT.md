# Deployment — Vietnamese Stock Behavior Clustering Dashboard

> Covers local dev, Supabase, EC2 API, EC2 worker, Vercel frontend, env vars, Nginx, systemd, health checks, rollback, cost control, and both one-EC2 and two-EC2 modes. **No application code here** — operational guidance only.

## 1. Deployment modes at a glance

| Mode | Hosts | Layout |
|---|---|---|
| **One-EC2 (staging)** | 1 × EC2 | FastAPI + worker as **separate systemd services** on one host; Nginx optional |
| **Two-EC2 (production-like)** | 2 × EC2 | **EC2-API:** Nginx + FastAPI. **EC2-WORKER:** scheduler + pipelines |

Both modes share the same Supabase Postgres, the same env-var contract, and optionally the same S3 bucket.

## 2. Local development

- Clone the repo; create a Python virtual environment for the backend/worker and install dependencies.
- Run the Next.js frontend locally against a local or staging API URL.
- Point local services at a **dev Supabase project** (never production).
- Use a `.env` file (git-ignored) following the env-var contract in §7.
- Optionally run the worker in a "manual" mode to execute single jobs on demand for debugging, rather than via the scheduler.

## 3. Supabase setup

- Create a Supabase project; record the Postgres connection string and service/anon keys.
- Apply the schema: trading calendar, normalized OHLCV (daily + 5-minute), feature tables, rolling windows, clustering outputs, historical pattern outputs, supporting outputs, dashboard snapshots.
- Seed the **trading_calendar** table (trading days, holidays) — this gates all market jobs.
- Configure connection pooling appropriate to the number of worker/API connections.
- Restrict network access / use least-privilege credentials for API vs worker where possible (API needs read; worker needs write).

## 4. EC2 API setup

- Provision an EC2 instance (Amazon Linux / Ubuntu).
- Install Python runtime and the backend dependencies.
- Deploy the FastAPI app; run it under a process manager via **systemd** (see §9).
- Put **Nginx** in front as a reverse proxy with TLS (see §8). Required in two-EC2 mode; optional but recommended in one-EC2 mode.
- Open only the necessary inbound ports (443/80 to the world; app port bound to localhost behind Nginx).

## 5. EC2 worker setup

- Provision EC2 (in two-EC2 mode this is **EC2-WORKER**; in one-EC2 mode it is the same host as the API).
- Install Python runtime and pipeline dependencies (data, ML, clustering, retrieval libraries).
- Deploy the worker/scheduler; run it as an **always-on** systemd service (see §9).
- Grant the worker write access to Postgres and (optional) S3 for parquet/artifacts/matrices/vectors.
- No inbound web ports are required for the worker; outbound access to VNStock API, Supabase, and S3 is sufficient.

## 6. Vercel frontend setup

- Connect the repo to Vercel; configure the Next.js project.
- Set the public API base URL env var to point at the EC2-API (via its domain behind Nginx).
- Configure preview deployments for branches and production deployment for the main branch.
- Ensure CORS on the API permits the Vercel domain(s).

## 7. Environment variables (contract)

> Names are indicative; keep secrets out of git and out of the frontend bundle.

**Backend API (EC2-API):**
- `DATABASE_URL` — Supabase Postgres connection string (read-focused).
- `SUPABASE_URL`, `SUPABASE_KEY` — if using Supabase client libs.
- `API_HOST`, `API_PORT` — bind address/port (bound to localhost behind Nginx).
- `ALLOWED_ORIGINS` — CORS origins (Vercel domains).
- `ENV` — `staging` / `production`.

**Worker (EC2-WORKER or shared host):**
- `DATABASE_URL` — Supabase Postgres connection string (read/write).
- `VNSTOCK_*` — any config needed for the VNStock source.
- `S3_BUCKET`, `AWS_REGION`, AWS credentials — if S3 is enabled.
- `TIMEZONE` — declared pipeline timezone.
- `ENV` — `staging` / `production`.

**Frontend (Vercel):**
- `NEXT_PUBLIC_API_BASE_URL` — public URL of the EC2-API.

## 8. Nginx reverse proxy

- Terminates TLS and proxies to the FastAPI app bound on localhost.
- Adds standard proxy headers; sets sane timeouts and body-size limits.
- Serves only the API; the frontend is on Vercel.
- In **two-EC2 mode** Nginx runs on EC2-API. In **one-EC2 mode** Nginx (optional) runs alongside both services on the single host.

## 9. systemd services

- **One-EC2 mode:** two services on one host:
  - `fastapi-api.service` — runs FastAPI.
  - `worker.service` — runs the scheduler/pipelines, **always on**.
- **Two-EC2 mode:**
  - On EC2-API: `fastapi-api.service` (+ Nginx as its own service).
  - On EC2-WORKER: `worker.service` (always on).
- All services: `Restart=always`, start on boot, log to journald, run as a non-root service user.

## 10. Health checks

- **API liveness/readiness:** a lightweight health endpoint that confirms the API is up and can reach Postgres; checked by Nginx/uptime monitoring and after each deploy.
- **Worker liveness:** systemd `active` state plus a heartbeat (e.g., last-successful-run timestamps per job written to Postgres) so stalled jobs are detectable.
- **Data freshness check:** verify the latest snapshot's as-of date matches the expected last trading day.
- **Post-deploy smoke check:** hit the health endpoint and one snapshot endpoint before considering a deploy successful.

## 11. Rollback plan

- **Code rollback:** deploy is tied to a versioned artifact / git ref; roll back by redeploying the previous ref and restarting the affected systemd service(s).
- **Frontend rollback:** use Vercel's instant rollback to the previous production deployment.
- **Data/snapshot rollback:** snapshots are versioned/timestamped; if a bad run publishes a faulty snapshot, repoint serving to the last known-good snapshot (the API always reads the latest **valid** snapshot).
- **Schema changes:** apply migrations forward-only where possible; keep a tested down-path or a restore-from-backup procedure for destructive changes.
- Always validate with the post-deploy smoke check (§10) before and after rollback.

## 12. Cost-control notes

- Prefer **one-EC2 staging mode** for non-production to halve EC2 cost.
- Right-size EC2 instances; stop/downscale staging when idle.
- Use S3 only for genuinely large artifacts (parquet, distance matrices, vectors); keep small structured outputs in Postgres.
- The worker is the cost driver (nightly heavy jobs) — schedule heavy jobs at night and avoid unnecessary recomputation via idempotent, incremental updates.
- The API is lightweight (reads snapshots), so it can run on a small instance.
- Avoided services (Lambda/ECS/EKS/Kafka/Airflow/Redshift) are intentionally excluded to keep fixed costs low.

## 13. Mode-specific summary

| Concern | One-EC2 | Two-EC2 |
|---|---|---|
| FastAPI | systemd on shared host | systemd on EC2-API |
| Worker/scheduler | systemd on shared host | systemd on EC2-WORKER |
| Nginx | optional, shared host | on EC2-API |
| DB | Supabase | Supabase |
| S3 | optional | optional |
| Use case | staging / low cost | production-like separation |
