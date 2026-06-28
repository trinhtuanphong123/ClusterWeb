# Validation

## Proof Strategy

Explain what must pass before the story is done.

## Required Proof Before Execution

- Pre-conditions/checks that must hold before any change is made.

## Required Proof After Execution

- Checks that must pass to consider the story done.

## Mock vs Real Validation

Mock validation:
- ...

Real/dev validation:
- ...

## Test Plan

| Layer | Cases |
| --- | --- |
| Unit | |
| Integration | |
| E2E | |
| Platform | |
| Performance | |
| Logs/Audit | |

## Non-Regression Checks

- What previously-working behavior must be confirmed still works.

## Safety Checks

- No secrets exposed (client bundle, logs, responses).
- No production DB writes unless explicitly allowed.
- No schema drift outside the approved migration.

## Fixtures

List deterministic users, accounts, records, provider responses, or other
fixtures needed for repeatable proof.

## Commands

Add commands after scripts exist.

```text
TBD
```

## Acceptance Evidence

Add results after verification.