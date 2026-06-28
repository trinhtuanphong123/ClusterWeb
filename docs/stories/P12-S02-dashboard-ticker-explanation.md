# P12-S02 Dashboard Ticker Explanation

## Risk
normal

## Goal
Build the per-ticker view showing cluster membership, neighbors, historical pattern matches, outcome context, and current-state, reading from the API. Explanation-focused, descriptive only.

## Prerequisites
- P6-S01 web shell completed.
- P11-S01 dashboard API integration completed.
- Historical pattern and outcome endpoints exist, or this story uses explicitly labeled mock data.


## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/UI_SPEC.md`
- `docs/product/DESIGN_SYSTEM.md`
- `docs/product/CHART_SPEC.md`
- `docs/product/API_CONTRACT.md`

## Modify only
- `apps/web/` (ticker detail page/components)

## Do not touch
- `pipelines/`
- `infra/aws/`
- backend source

## Acceptance criteria
- Ticker view shows cluster membership, neighbors, historical matches, outcome context, and current-state from the API.
- Historical matches and outcomes labeled as descriptive context, explicitly **not predictions**; no buy/sell language.
- Loading/error/staleness states present; latest valid snapshot on non-trading days with `is_stale` shown.
- Intraday current-state labeled as a periodic (hourly) snapshot, not live.
- No secrets exposed to the client.
- Historical matches and outcomes must be labeled as descriptive context.
- Intraday state must be labeled as hourly/periodic, not live.
- No forecast, recommendation, buy/sell, or signal language.

## Verification
See `docs/test-matrix/frontend.md`:
```bash
cd apps/web && npm run lint && npm run build
```

## Decision record
Required? no.

## Notes for agent
Explanation layer in the UI. Keep all wording descriptive; never imply forecasts or recommendations.
