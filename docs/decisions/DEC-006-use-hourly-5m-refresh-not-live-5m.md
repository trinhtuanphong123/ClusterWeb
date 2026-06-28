# DEC-006: Use Hourly 5-Minute Refresh, Not Live 5-Minute Ingestion

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The dashboard shows a "current state" intraday view derived from 5-minute OHLCV. One option is to ingest 5-minute bars **live every 5 minutes**; another is to refresh the intraday snapshot **hourly**. The 5-minute data is provisional and used only for display — it is never the official clustering source (daily OHLCV is). The project is decision-support analytics, not a real-time trading terminal.

## Decision

Refresh the intraday 5-minute state **hourly** during trading hours. **Remove live 5-minute-every-5-minutes ingestion from the MVP.**

## Rationale

- **Sufficient for the use case** — hourly current-state is adequate for behavior analytics; sub-hourly freshness adds little analytical value.
- **Cost and rate-limit friendly** — far fewer calls to the VNStock free/community API, reducing throttling risk and load.
- **Operational simplicity** — one hourly job instead of a high-frequency polling loop.
- **Consistent framing** — reinforces that this is not a real-time trading system.

## Consequences

- The intraday view can lag real time by up to ~an hour; the UI labels it as a periodic snapshot, not live.
- Intraday 5-minute data remains **provisional** and never feeds official clustering.
- The hourly job runs only on trading days during trading hours (calendar-gated).

## Alternatives considered

- **Live 5-minute ingestion every 5 minutes** — higher cost, more rate-limit pressure, more operational complexity, and unnecessary for analytics; rejected for MVP.
- **No intraday view at all** — loses useful current-state context that hourly refresh cheaply provides; rejected.


## Related decisions

- DEC-008 — trading-calendar/trading-hours gating for the hourly refresh.
- DEC-009 — clustering uses daily data; intraday 5-minute is current-state only.