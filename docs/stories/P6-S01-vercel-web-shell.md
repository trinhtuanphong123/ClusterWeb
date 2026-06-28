# P6-S01 Vercel Web Shell

## Risk
normal

## Goal
Create the Next.js dashboard shell deployable on Vercel that reads `NEXT_PUBLIC_API_BASE_URL`, with loading and error states and a mock fallback. No real data views yet.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/UI_SPEC.md`
- `docs/product/DESIGN_SYSTEM.md`
- `docs/product/API_CONTRACT.md`
- `docs/decisions/DEC-004-use-vercel-frontend.md`

## Prerequisites
- P1-S01 scaffold completed.
- `apps/web/` exists.


## Modify only
- `apps/web/` (app shell, layout, a single placeholder page)
- `apps/web/.env.example`

## Do not touch
- `pipelines/`
- `infra/aws/`
- backend source

## Acceptance criteria
- App builds and lints; page renders without runtime error.
- Loading and error states exist; mock fallback when API is unset.
- Only `NEXT_PUBLIC_*` config exposed to the client; no secrets.
- No buy/sell or prediction language anywhere in the shell.
- The shell must work with API unset by using mock fallback.
- The browser bundle must not include non-`NEXT_PUBLIC_*` environment variables.

## Verification
See `docs/test-matrix/frontend.md`:
```bash
cd apps/web && npm run lint && npm run build
```

## Decision record
Required? no, unless secret/env exposure changes (then add DEC). Reference DEC-004.

## Notes for agent
Shell only. No cluster/ticker views yet (those are P12). Keep client free of secrets.
