# CONTEXT_INDEX.md

Tells coding agents which docs and paths to read for each work type. Do **not** read all docs by default. Start from the assigned story packet; this index only confirms what that story already lists.

## Default context

Always read: `AGENTS.md`, this file, and the assigned story packet in `docs/stories/`.
Usually read: `docs/product/PROJECT_BRIEF.md`.
Only read detailed docs when the story packet lists them.

Never read by default (only if the story explicitly asks): `docs/HARNESS.md`, everything in `docs/templates/`, everything in `docs/decisions/`, all of `docs/product/`, and the full `docs/test-matrix/` set.

---

## Repo scaffold
Read: `docs/product/PROJECT_BRIEF.md`, `docs/product/ARCHITECTURE.md`.
Paths: root, `apps/`, `services/`, `pipelines/`, `supabase/`, `infra/`, `scripts/`.
Do not read: `docs/product/METHODOLOGY.md`, `docs/product/DESIGN_SYSTEM.md`, modeling specs, frontend specs.

## Backend API
Read: `docs/product/API_CONTRACT.md`; `docs/product/DEPLOYMENT.md` only if startup/deploy is affected.
Paths: `services/api/`, `.env.example`.
Do not read: `docs/product/METHODOLOGY.md`, design docs, `pipelines/` unless the API reads pipeline output directly.

## Database
Read: `docs/product/DATABASE_SCHEMA.md`, `docs/product/DATA_FLOW.md`.
Paths: `supabase/migrations/`, `services/api/app/db/`, `pipelines/common/db.py`.
Do not read: frontend docs, design docs, methodology/modeling specs.

## Ingestion
Read: `docs/product/DATA_FLOW.md`, `docs/product/DATABASE_SCHEMA.md`, `docs/product/INGESTION_STRATEGY.md`, `docs/product/DATA_SOURCE_CAPABILITIES.md`.
Paths: `pipelines/ingestion/`, `pipelines/common/`.
Do not read: `docs/product/DESIGN_SYSTEM.md`, `apps/web/`, clustering/retrieval specs.

## Feature engineering
Read: `docs/product/FEATURE_SPEC.md`, `docs/product/DATA_FLOW.md`, `docs/product/DATABASE_SCHEMA.md`.
Paths: `pipelines/features/`, `pipelines/common/`.
Do not read: frontend implementation, deployment scripts unless required, clustering/retrieval specs.

## Behavior windows
Read: `docs/product/WINDOWING_SPEC.md`, `docs/product/FEATURE_SPEC.md`, `docs/product/DATABASE_SCHEMA.md`.
Paths: `pipelines/features/`, `pipelines/common/`.
Do not read: frontend docs, deployment scripts, retrieval/outcome specs.

## Current clustering
Read: `docs/product/CLUSTERING_SPEC.md`, `docs/product/METHODOLOGY.md`, `docs/product/DATABASE_SCHEMA.md`, `docs/product/MODEL_RUNTIME_STRATEGY.md`.
Paths: `pipelines/clustering/`, `pipelines/common/`.
Do not read: frontend docs unless output shape affects the dashboard; deployment scripts.

## Historical pattern retrieval
Read: `docs/product/HISTORICAL_PATTERN_RETRIEVAL_SPEC.md`, `docs/product/MPDIST_SPEC.md`, `docs/product/WINDOWING_SPEC.md`, `docs/product/DATABASE_SCHEMA.md`.
Paths: `pipelines/retrieval/`, `pipelines/common/`.
Do not read: frontend docs, deployment scripts, design docs.

## Lead-lag analysis
Read: `docs/product/LEAD_LAG_ANALYSIS_SPEC.md`, `docs/product/DATABASE_SCHEMA.md`.
Paths: `pipelines/leadlag/`, `pipelines/common/`.
Do not read: frontend docs, deployment scripts, design docs.

## Outcome evaluation
Read: `docs/product/CLUSTER_OUTCOME_EVALUATION.md`, `docs/product/DATABASE_SCHEMA.md`, `docs/product/METHODOLOGY.md`.
Paths: `pipelines/outcomes/`, `pipelines/common/`.
Do not read: frontend docs, deployment scripts, design docs.

## Frontend dashboard
Read: `docs/product/UI_SPEC.md`, `docs/product/DESIGN_SYSTEM.md`, `docs/product/CHART_SPEC.md`, `docs/product/API_CONTRACT.md`.
Paths: `apps/web/`.
Do not read: `pipelines/`, `infra/aws/`, long methodology/modeling docs unless the Methodology page is being implemented.

## Deployment
Read: `docs/product/DEPLOYMENT.md`, `docs/product/ARCHITECTURE.md`, `docs/product/SCHEDULER_STRATEGY.md`.
Paths: `infra/aws/`, `scripts/`, `.env.example`, `services/api/`, `pipelines/`.
Do not read: frontend component files unless Vercel config is affected; methodology/modeling specs.

## Harness maintenance
Read: `docs/HARNESS.md`, `docs/FEATURE_INTAKE.md`, `docs/TEST_MATRIX.md`, `docs/templates/`.
Paths: `AGENTS.md`, `docs/`, `scripts/bin/harness-cli*`.
Do not read: product/modeling/frontend implementation docs unless explicitly required. Only use this work type when explicitly asked to change the harness process.
