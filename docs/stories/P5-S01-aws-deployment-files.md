# P5-S01 AWS Deployment Files

## Risk
high-risk

## Goal
Add EC2 deployment files (systemd units, Nginx reverse proxy config, env templates) supporting one-EC2 and two-EC2 modes, without committing secrets.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DEPLOYMENT.md`
- `docs/product/ARCHITECTURE.md`

## Modify only
- `infra/aws/systemd/` (`stock-api.service`, `stock-worker.service`)
- `infra/aws/nginx/` (reverse proxy config)
- `scripts/` (deploy/start helpers)
- `.env.example`

## Do not touch
- application source in `services/api/app/` and `pipelines/` business logic
- frontend files unless Vercel config is affected
- migrations

## Acceptance criteria
- systemd units start FastAPI and worker as separate services.
- Nginx terminates TLS and proxies to the API; serves no secrets.
- One-EC2 and two-EC2 layouts documented and parameterized.
- No secret values committed; env via templates only.

## Verification
See `docs/test-matrix/deployment.md`:
```bash
sudo systemctl status stock-api
sudo systemctl status stock-worker
curl http://<EC2_HOST>/health
```
If EC2 is unavailable, provide the exact commands for the human.

## Decision record
Required? yes — `docs/decisions/DEC-xxx-aws-deployment.md` (deployment target/scripts). Reference DEC-002, DEC-007.

## Notes for agent
Never commit secrets. Keep both deployment modes working from the same files. No app logic changes.
