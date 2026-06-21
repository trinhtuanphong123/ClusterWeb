# Deployment validation

## EC2 service validation

Expected:
- FastAPI service can start through systemd
- worker service can start through systemd
- environment file is not committed with secrets
- supports one-EC2 (API + worker on one host) and two-EC2 (EC2-API + EC2-WORKER) modes

Commands:
```bash
sudo systemctl status stock-api
sudo systemctl status stock-worker
```

## HTTP health check

Command:
```bash
curl http://<EC2_HOST>/health
```

## Vercel validation

Expected:
- `NEXT_PUBLIC_API_BASE_URL` points to the backend
- frontend build passes
- dashboard loads API data or mock fallback
- no secrets/service-role keys exposed to the client
