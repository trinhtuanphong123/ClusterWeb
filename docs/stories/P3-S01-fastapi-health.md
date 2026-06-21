# P3-S01 FastAPI Health Endpoint

## Risk
normal

## Goal
Stand up the FastAPI backend skeleton with a `GET /health` endpoint that confirms the service is up and can reach Postgres.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/API_CONTRACT.md`

## Modify only
- `services/api/app/main.py`
- `services/api/app/health.py`
- `services/api/app/db/` (read-only connection helper)
- `.env.example`

## Do not touch
- `pipelines/`
- frontend files
- migrations

## Acceptance criteria
- `GET /health` returns HTTP 200 with JSON including `status` and a DB connectivity signal.
- No secrets hard-coded; DB config via env vars.
- Endpoint does no model computation.

## Verification
See `docs/test-matrix/backend.md`:
```bash
cd services/api && uvicorn app.main:app --reload
curl http://localhost:8000/health
```

## Decision record
Required? no (health endpoint is smoke proof, not a public contract change). Reference DEC-001.

## Notes for agent
Keep it minimal. Health is the only endpoint in this story. Do not implement data endpoints yet.
