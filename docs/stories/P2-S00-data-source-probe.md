# P2-S00 Data Source Probe

## Risk
high-risk

## Goal
Probe the VNStock community/free API to record real capabilities (ticker universe, daily OHLCV, 5-minute OHLCV, optional price board / intraday, lookback, timezone, duplicates, missing bars, rate-limit observations) into the capabilities template. **Must not write to the production database.**

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATA_SOURCE_CAPABILITIES.md`
- `docs/product/DATA_FLOW.md`

## Modify only
- `pipelines/probe/` (read-only probing scripts)
- `docs/product/DATA_SOURCE_CAPABILITIES.md` (fill probe-backed results)
- local probe output files under a gitignored scratch path

## Do not touch
- `supabase/migrations/`
- any production database connection
- ingestion pipelines that write to storage

## Acceptance criteria
- Probe reads only; writes nothing to Supabase/production tables.
- Capabilities template filled only with directly observed values; unverified items remain `UNVERIFIED`.
- No exact API limits are claimed unless observed by probing.
- Secrets/keys are not hard-coded.

## Verification
```bash
python -m pipelines.probe.run --dry-run   # prints probe results to stdout, writes no DB rows
```
If it cannot run without credentials, provide the exact command and required env var names for the human.

## Decision record
Required? no, unless probing reveals a capability that changes ingestion strategy or schema (then add `docs/decisions/DEC-xxx-*.md`).

## Notes for agent
This is a read-only investigation. Never insert into production tables. Treat all observed limits as provisional and mark anything unconfirmed as `UNVERIFIED`.
