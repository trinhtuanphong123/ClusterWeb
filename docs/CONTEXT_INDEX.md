# CONTEXT_INDEX.md 

This file tells coding agents which documents to read for each work type. Do not read all docs by default. Start from the assigned story packet. 

## Default context 

Always read: 
- `AGENTS.md` 
- assigned story packet in `docs/stories/` 
Usually read: 
- `docs/product/PROJECT_BRIEF.md` 
Only read detailed docs when the story packet lists them. 

## Work type: Repository scaffold 
Read: 
- `docs/product/PROJECT_BRIEF.md` 
- `docs/product/ARCHITECTURE.md` 
Relevant paths: 
- root 
- `apps/` 
- `services/` 
- `pipelines/` 
- `supabase/` 
- `infra/` 
- `scripts/` 
Do not read: 
- `docs/product/METHODOLOGY.md` 
- `docs/product/DESIGN_SYSTEM.md` 

## Work type: Backend API 
Read: 
- `docs/product/API_CONTRACT.md` 
- `docs/product/DEPLOYMENT.md` only if service startup/deploy is affected Relevant paths: 
- `services/api/` 
- `.env.example` 
Do not read: 
- `docs/product/METHODOLOGY.md` 
- `docs/product/DESIGN_SYSTEM.md` 
- `pipelines/` unless API uses pipeline output directly 

## Work type: Database / Supabase
Read: 
-`docs/product/DATABASE_SCHEMA.md` 
- `docs/product/DATA_FLOW.md` 
Relevant paths: 
- `supabase/migrations/` 
- `services/api/app/db/` 
- `pipelines/common/db.py` 
Do not read: 
- frontend docs 
- design docs 

## Work type: Data ingestion 
Read: 
- `docs/product/DATA_FLOW.md` 
- `docs/product/DATABASE_SCHEMA.md` 
- `docs/product/PROJECT_BRIEF.md` 
Relevant paths: 
- `pipelines/ingestion/` 
- `pipelines/common/` 
Do not read: 
- `docs/product/DESIGN_SYSTEM.md` 
- `apps/web/` 

## Work type: Feature engineering 
Read: 
- `docs/product/METHODOLOGY.md` 
- `docs/product/DATA_FLOW.md` 
- `docs/product/DATABASE_SCHEMA.md` 
Relevant paths: 
- `pipelines/features/` 
- `pipelines/common/` 
Do not read: 
- frontend implementation 
- deployment scripts unless required 

## Work type: Clustering / Modeling 
Read: 
- `docs/product/METHODOLOGY.md` 
- `docs/product/DATABASE_SCHEMA.md` 
- `docs/product/DATA_FLOW.md` 
Relevant paths: 
- `pipelines/clustering/` 
- `pipelines/common/` 
Do not read: 
- frontend docs unless output shape affects dashboard  
## Work type: Frontend dashboard 
Read: 
- `docs/product/UI_SPEC.md` 
- `docs/product/DESIGN_SYSTEM.md` 
- `docs/product/CHART_SPEC.md` 
- `docs/product/API_CONTRACT.md` 
Relevant paths: 
- `apps/web/` 
Do not read: 
- `pipelines/` 
- `infra/aws/` 
- long methodology docs unless the Methodology page is being implemented 

## Work type: Deployment 
Read: 
- `docs/product/DEPLOYMENT.md` 
- `docs/product/ARCHITECTURE.md` 
Relevant paths: 
- `infra/aws/` 
- `scripts/` 
- `.env.example` 
- `services/api/` 
- `pipelines/` 
Do not read: 
- frontend component files unless Vercel config is affected 

## Work type: Harness maintenance 
Read: 
- `docs/HARNESS.md` 
- `docs/FEATURE_INTAKE.md` 
- `docs/TEST_MATRIX.md` 
- `docs/templates/` 
Relevant paths: 
- `AGENTS.md` 
- `docs/` 
- `scripts/bin/harness-cli*`

 Only use this work type when explicitly asked to change the harness process.