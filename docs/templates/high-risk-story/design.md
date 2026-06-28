# Design

## Source of Truth

Which product docs and decisions control this design?

- `docs/product/...`
- `docs/decisions/DEC-xxx-*.md`

## Domain Model

Describe entities, value objects, and business rules.

## Application Flow

Describe commands, queries, and handlers.

## Interface Contract

Describe routes, messages, commands, request DTOs, response DTOs, and errors.

## Data Model

Describe tables, indexes, migrations, and retention concerns.

## Data Ownership

- Which service writes this data?
- Which service reads this data?

## Security / Credential Boundary

- What credentials/secrets are involved, and what privilege level each needs?
- How are secrets supplied (env / secret store)? Confirm none reach the client or logs.

## UI / Platform Impact

Describe browser, mobile, desktop, CLI, deployment, or platform-shell impact.

## Observability

Describe logs, audit records, metrics, or traces.

## Failure Modes

- What can go wrong, and how is each handled or surfaced?

## Alternatives Considered

1. Option.