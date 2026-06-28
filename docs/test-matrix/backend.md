# Backend API Validation

This matrix defines backend verification for FastAPI stories.

The backend must remain a thin read/API layer. It must not run clustering,
retrieval, feature computation, ingestion, or model jobs at request time.

## Validation modes

| Mode | Use when | Requirement |
| --- | --- | --- |
| local-smoke | API skeleton or health endpoint exists | Must run without production secrets |
| dev-db | A dev Supabase/Postgres database is configured | May check DB connectivity and read queries |
| mock | DB/API data is not available yet | Must use fixtures or explicit mock data |
| manual | External infrastructure is required | Must provide exact human-run steps |

Never claim a check passed if the command was not run.

## Health endpoint

Applies to:
- `P3-S01 FastAPI Health Endpoint`
- Deployment smoke checks
- API availability checks

Expected:
- `GET /health` returns HTTP 200 when the API process is alive.
- JSON includes:
  - `status`
  - `service`
  - `database`
- `database` reports one of:
  - `ok`
  - `unconfigured`
  - `error`
- Missing DB environment variables must not crash the service.
- `/health` must not expose:
  - secrets
  - raw connection strings
  - stack traces
  - service-role keys
  - detailed infrastructure internals

Local command:

```bash
cd services/api
uvicorn app.main:app --reload
curl http://localhost:8000/health