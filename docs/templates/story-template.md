# <Story ID> <Story title>

> **Canonical story template.** Use this for all normal (and tiny) harness story packets. For high-risk work, use `docs/templates/high-risk-story/` instead.

## Status

draft | ready | blocked | done

## Lane

tiny | normal | high-risk

## Goal

One clear goal.

## Product Contract

Describe the behavior this story must make true.

## Relevant Product Docs

- `docs/product/...`

## Related Decisions

- `docs/decisions/DEC-xxx-*.md`

## Prerequisites

- List the conditions that must already be true (e.g., schema exists, fixture data available).

## Blocked by

- List the stories or artifacts that must complete first. Leave empty only if truly executable now.

## Read only

- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- ...

## Modify only

- List the exact files/paths this story may change. Avoid broad directories; name files where possible.

## Do not touch

- List sensitive files/areas explicitly out of scope.

## Acceptance Criteria

- [ ] ...
- [ ] ...
- [ ] ...

## Verification

Command or manual check:

```bash
...
```

If the check cannot be run (no DB/API/host yet), provide mock/local verification or the exact command for the human to run.

## Decision Record

Required? yes/no

If yes, create or update:

- `docs/decisions/DEC-xxx-*.md`

## Evidence

Add commands, reports, screenshots, logs, or links after verification exists.

## Notes for Agent

- Keep change minimal.
- Do not redesign unrelated systems.
- Stop and ask if file access outside scope is needed.