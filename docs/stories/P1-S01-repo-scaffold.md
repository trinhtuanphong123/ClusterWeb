# P1-S01 Repository Scaffold

## Risk
normal

## Goal
Create the repository folder structure and placeholders for all subsystems (apps, services, pipelines, supabase, infra, scripts) with no business logic.

## Read only
- `AGENTS.md`
- `docs/CONTEXT_INDEX.md`
- `docs/product/PROJECT_BRIEF.md`
- `docs/product/ARCHITECTURE.md`

## Modify only
- root config placeholders (`README.md`, `.gitignore`, `.env.example`)
- `apps/web/` (placeholder)
- `services/api/` (placeholder)
- `pipelines/` (placeholder packages)
- `supabase/` (placeholder)
- `infra/` (placeholder)
- `scripts/` (placeholder)

## Do not touch
- `docs/decisions/`
- any product modeling/spec docs not listed above
- the harness CLI or `docs/HARNESS*`

## Acceptance criteria
- Folder structure exists for frontend, backend, worker/pipelines, supabase, infra, scripts.
- Placeholders only; no domain schema, no CRUD, no provider integration.
- `.env.example` lists variable names only (no secret values).
- `README.md` states the mission and stack at a high level.

## Verification
```bash
find apps services pipelines supabase infra scripts -maxdepth 2 -type d | sort
```

## Decision record
Required? no (scaffold only; no architecture change beyond the already-recorded stack).

## Notes for agent
Keep it placeholders-only. Do not install dependencies beyond what a scaffold needs. Do not add business logic.
