# Deployment validation

## EC2 service validation

Expected:
- FastAPI service can start through systemd
- worker service can start through systemd
- environment file is not committed with secrets

Commands:
```bash
sudo systemctl status stock-api
sudo systemctl status stock-worker
HTTP health check

Command:

curl http://<EC2_HOST>/health
Vercel validation

Expected:

NEXT_PUBLIC_API_BASE_URL points to backend
frontend build passes
dashboard loads API data or mock fallback