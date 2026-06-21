# Backend API validation

## Health endpoint

Expected:
- `GET /health` returns HTTP 200
- JSON includes `status`
- a DB connectivity signal is present

Command:
```bash
cd services/api
uvicorn app.main:app --reload
curl http://localhost:8000/health
```

## API contract

Expected:
- each endpoint response matches `docs/product/API_CONTRACT.md`
- errors use the stable JSON shape `{ "error": { "code": ..., "message": ... } }`
- read endpoints serve precomputed outputs only (no model execution at request time)
- freshness fields (`last_data_update`, `last_feature_update`, `last_model_update`, `latest_cluster_run`, `latest_snapshot`, `is_stale`) appear where the contract specifies
- on non-trading days, endpoints return the latest valid snapshot and correct market status

Proof:
- curl output or test output per endpoint
