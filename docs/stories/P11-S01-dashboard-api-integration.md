# P11-S01 Dashboard API Integration

## Risk
high-risk

## Goal
Implement the read-only FastAPI endpoints from the API contract that serve precomputed snapshots (overview, clusters, tickers, pipeline status, market status, methodology), including freshness fields and non-trading-day behavior. **No model execution at request time.**

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/API_CONTRACT.md`
- `docs/product/DATABASE_SCHEMA.md` (read access only; do not modify schema)

## Modify only
- `services/api/app/routes/`
- `services/api/app/db/` (read queries)
- `services/api/app/main.py` (router registration)

## Do not touch
- `supabase/migrations/` (schema)
- `pipelines/`
- frontend files

## Acceptance criteria
- Endpoints match `API_CONTRACT.md` shapes and the stable error envelope.
- Backend reads precomputed outputs only (snapshots/cluster tables); runs no models.
- Freshness fields present where specified (`last_data_update`, `last_feature_update`, `last_model_update`, `latest_cluster_run`, `latest_snapshot`, `is_stale`).
- On non-trading days, endpoints return the latest valid snapshot + correct market status.
- No secrets/service-role keys in responses; read-focused DB credentials.

## Verification
See `docs/test-matrix/backend.md`:
```bash
cd services/api && uvicorn app.main:app --reload
curl http://localhost:8000/api/market/status
curl http://localhost:8000/api/clusters/latest
```

## Decision record
Required? yes if any response shape diverges from `API_CONTRACT.md` — `docs/decisions/DEC-xxx-api-contract.md`. Reference DEC-001.

## Notes for agent
Reader only. Never compute clusters/retrieval in the API. Serve latest valid snapshot on non-trading days. No buy/sell content.
