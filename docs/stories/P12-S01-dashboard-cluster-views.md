# P12-S01 Dashboard Cluster Views

## Risk
normal

## Goal
Build the dashboard views for current behavior clusters (overview + cluster list/detail) reading from the API, with loading/error/staleness states. Reads the latest valid snapshot on non-trading days.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/UI_SPEC.md`
- `docs/product/DESIGN_SYSTEM.md`
- `docs/product/CHART_SPEC.md`
- `docs/product/API_CONTRACT.md`

## Modify only
- `apps/web/` (cluster overview and detail pages/components)

## Do not touch
- `pipelines/`
- `infra/aws/`
- backend source

## Acceptance criteria
- Overview and cluster views render current clusters from the API.
- Loading, error, and staleness indicators present; on non-trading days shows the latest valid snapshot labeled as such.
- Outputs framed as descriptive behavior clusters; no prediction or buy/sell language.
- No secrets exposed to the client.

## Verification
See `docs/test-matrix/frontend.md`:
```bash
cd apps/web && npm run lint && npm run build
```

## Decision record
Required? no.

## Notes for agent
Read the API only. Respect the not-prediction framing in all copy. Show staleness clearly on non-trading days.
