# P3-S01 FastAPI Health Endpoint

## Risk

normal

## Goal
Stand up the FastAPI backend skeleton with a minimal `GET /health` endpoint. The endpoint confirms the API process is running and reports a database connectivity signal when database environment variables are configured.

This story is a backend smoke proof only. It must not implement business APIs, stock data APIs, model computation, ingestion, clustering, or database schema.

## Read only

* `AGENTS.md`
* `docs/CONTEXT_INDEX.md`
* `docs/HARNESS.md`
* `docs/FEATURE_INTAKE.md`
* `docs/product/API_CONTRACT.md`
* `docs/decisions/DEC-001-use-fastapi-backend.md`
* `docs/decisions/DEC-003-use-supabase-postgres.md`
* `docs/test-matrix/backend.md`
* `services/api/README.md`
* `docs/decisions/DEC-001-use-fastapi-backend.md`
* `docs/decisions/DEC-003-use-supabase-postgres.md`
* `docs/test-matrix/backend.md`
* `docs/decisions/DEC-011-api-security-and-credential-boundary.md`

## Modify only

* `services/api/app/main.py`
* `services/api/app/health.py`
* `services/api/app/db/`
* `services/api/requirements.txt`
* `.env.example`
- `services/api/requirements.txt`

## Do not touch

* `docs/product/`
* `docs/decisions/`
* `docs/HARNESS*`
* `scripts/bin/harness-cli.exe`
* `pipelines/`
* `apps/`
* `supabase/migrations/`
* `infra/`

## Acceptance criteria

* `GET /health` returns HTTP 200.
* Response JSON includes at least:

  * `status`
  * `service`
  * `database`
* `status` confirms the API service is up.
* `database` reports a connectivity signal such as `ok`, `unconfigured`, or `error`.
* Missing database environment variables must not crash the service.
* No secrets are hard-coded.
* Database configuration is read from environment variables only.
* Endpoint does not perform model computation.
* Endpoint does not implement stock, clustering, ingestion, retrieval, or dashboard data behavior.
* No database schema or migration is created.
* `GET /health` returns HTTP 200 when the API process is alive.
* Response JSON includes `status`, `service`, and `database`.
* `database` may be `ok`, `unconfigured`, or `error`.
* Missing DB environment variables must not crash the service.
* `/health` must not expose secrets, raw connection strings, stack traces, or infrastructure details.

## Verification
See `docs/test-matrix/backend.md` and `docs/test-matrix/deployment.md`.

Backend smoke check before AWS deployment:

cd services/api
uvicorn app.main:app --host 0.0.0.0 --port 8000
curl http://localhost:8000/health

AWS EC2 smoke check after deployment:

curl http://<EC2_HOST>:<API_PORT>/health

Optional reverse proxy check, only if reverse proxy is configured:

curl http://<EC2_HOST>/health

If AWS EC2 is not available yet, do not mark AWS deployment validation as passed. Mark only backend smoke validation as passed.

Expected result:

* API starts successfully.
* `/health` returns HTTP 200.
* Response includes service status and database signal.

## Decision record

Required? no.

Reason: this story creates a smoke endpoint for backend readiness. It does not change the public product API contract beyond a local operational health endpoint.

Reference:

* `docs/decisions/DEC-001-use-fastapi-backend.md`
* `docs/decisions/DEC-003-use-supabase-postgres.md`

## Notes for agent

Keep it minimal. Health is the only endpoint in this story.

Do not implement:

* stock data endpoints
* dashboard endpoints
* authentication
* Supabase schema
* migrations
* ingestion jobs
* clustering logic
* retrieval logic
* external data source calls

If a required dependency is missing, update only `services/api/requirements.txt`.
If implementing database connectivity requires changing files outside the allowed scope, stop and ask.
