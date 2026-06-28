# P5-S01 AWS Deployment Files

## Risk
high-risk

## Goal
Add AWS EC2 deployment files for running the FastAPI API and worker as systemd services, with env templates and support for one-EC2 and two-EC2 modes. Reverse proxy configuration such as Nginx may be included only as an optional deployment hardening component, not as a required dependency.

## Prerequisites
- P3-S01 FastAPI health endpoint completed.
- P4-S01 worker skeleton completed.
- API and worker entry commands are known.
- DEC-007 one-EC2/two-EC2 deployment mode is accepted.

## Read only
## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DEPLOYMENT.md`
- `docs/product/ARCHITECTURE.md`
- `docs/decisions/DEC-001-use-fastapi-backend.md`
- `docs/decisions/DEC-002-use-ec2-worker.md`
- `docs/decisions/DEC-007-support-one-ec2-and-two-ec2-deployment.md`
- `docs/decisions/DEC-011-api-security-and-credential-boundary.md` 

## Modify only
## Modify only
- `infra/aws/systemd/` (`stock-api.service`, `stock-worker.service`)
- `infra/aws/env/` or env template files if the repo uses another deployment-template location
- `infra/aws/nginx/` only if adding optional reverse proxy config
- `scripts/` deployment/start helper scripts only
- `.env.example`

## Do not touch
- application source in `services/api/app/` and `pipelines/` business logic
- frontend files unless Vercel config is affected
- migrations

## Acceptance criteria
- systemd units start FastAPI and worker as separate services on AWS EC2.
- One-EC2 and two-EC2 layouts are documented and parameterized.
- FastAPI can be exposed directly for initial AWS validation.
- Reverse proxy config, if included, is optional and must not be required for the API service to start.
- No secret values are committed; env values are represented as templates or variable names only.
- Deployment files do not change application business logic.

## Verification
See `docs/test-matrix/deployment.md`.

AWS EC2 service checks:
```bash
sudo systemctl status stock-api
sudo systemctl status stock-worker

Direct FastAPI health check:

curl http://<EC2_HOST>:<API_PORT>/health

Optional reverse proxy health check, only if reverse proxy is configured:

curl http://<EC2_HOST>/health

If AWS EC2 is unavailable, provide the exact human-run AWS commands and mark deployment validation as not run, not passed.

## Decision record
Required? yes — `docs/decisions/DEC-xxx-aws-deployment.md` (deployment target/scripts). Reference DEC-002, DEC-007.


## Notes for agent
Never commit secrets. Keep both deployment modes working from the same files. Do not change application logic. Nginx or any reverse proxy is optional; do not make API deployment depend on it unless the story is explicitly changed to require reverse proxy hardening.
