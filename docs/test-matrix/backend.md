# Backend API validation

## Health endpoint

Expected:
- `GET /health` returns HTTP 200
- JSON includes `status`

Command:
```bash
cd services/api
uvicorn app.main:app --reload
curl http://localhost:8000/health
API contract

Expected:

endpoint response matches docs/product/API_CONTRACT.md
errors use stable JSON shape

Proof:

curl output or test output