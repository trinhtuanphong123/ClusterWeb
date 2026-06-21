# Frontend validation

## Local build

Command:
```bash
cd apps/web
npm run lint
npm run build
```

## UI check

Expected:
- page renders without runtime error
- loading state exists
- error state exists when an API call is used
- no secret is exposed to the client (only `NEXT_PUBLIC_*` config)
- on non-trading days the UI shows the latest valid snapshot with a staleness indicator
- no buy/sell or prediction language is presented; outputs framed as descriptive analytics

Proof:
- build/lint output and a described screenshot or manual walkthrough
