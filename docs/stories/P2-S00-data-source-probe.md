# P2-S00 Data Source Probe

## Risk
high-risk

## Execution status
needs-hardening (ready to execute after the edits below)

## Goal
Probe the VNStock community/free API to record real capabilities (ticker universe, daily OHLCV, 5-minute OHLCV, optional price board / intraday, lookback, timezone, duplicates, missing bars, rate-limit observations) into the capabilities template. **Read-only investigation. Must not write to the production database.**

## Prerequisites
- P1-S01 scaffold completed.
- Probe output scratch location `.scratch/probes/` is defined and gitignored.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/DATA_SOURCE_CAPABILITIES.md`
- `docs/product/DATA_FLOW.md`

## Modify only
- `pipelines/probe/` (read-only probing scripts)
- `docs/product/DATA_SOURCE_CAPABILITIES.md` (fill probe-backed results)
- `.scratch/probes/` (local probe output only)
- probe dependency file only if required by the current repo layout
- Do not modify `.gitignore`; if `.scratch/probes/` is not already ignored, stop and ask.

## Do not touch
- `supabase/migrations/`
- any production database connection
- Supabase write helpers
- production database connection helpers
- ingestion jobs that write to database or storage

## Acceptance criteria
- Probe reads only; writes nothing to Supabase/production tables.
- Probe must not require Supabase credentials.
- Probe must not import or call production DB write helpers.
- Capabilities template filled only with directly observed values; unverified items remain `UNVERIFIED`.
- If the external API fails or rate-limits, mark the relevant capability as `UNVERIFIED`; do not fabricate values.
- No exact API limits are claimed unless observed by probing.
- Probe output may be saved only under `.scratch/probes/`.
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
