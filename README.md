# Vietnamese Stock Behavior Clustering & Historical Pattern Explanation Dashboard

A decision-support analytics dashboard that groups Vietnamese-listed stocks by *how they are currently moving* (behavior similarity), then uses historical pattern retrieval to explain and contextualize each group.

**This is not a price-prediction model and not a buy/sell recommendation system.**

## Stack

| Component | Technology | Role |
|---|---|---|
| Frontend | Next.js on Vercel | Dashboard UI; reads precomputed snapshots via the API |
| Backend API | FastAPI on AWS EC2 | Serves precomputed snapshots; no heavy compute at request time |
| Worker | Python scheduler on AWS EC2 | Ingestion, features, windows, clustering, retrieval, snapshot generation |
| Database | Supabase Postgres | Normalized tables, feature tables, cluster/pattern outputs, snapshots |
| Object storage | AWS S3 (optional) | Raw parquet, model artifacts, large distance matrices |

## Repository structure

```
apps/web/              # Next.js frontend
services/api/          # FastAPI backend API
pipelines/             # Python data pipelines (worker)
  common/              #   shared utilities
  ingestion/           #   data ingestion from VNStock
  features/            #   feature engineering
  clustering/          #   behavior clustering
  retrieval/           #   historical pattern retrieval
supabase/              # Supabase config and migrations
  migrations/          #   SQL migration files
infra/                 # Infrastructure and deployment
  aws/                 #   AWS EC2 / deployment configs
scripts/               # Harness automation and tooling
docs/                  # Product docs, stories, decisions
```

## Agent workflow

This repo uses a lean repository harness. Agents should start from:

1. `AGENTS.md`
2. `docs/CONTEXT_INDEX.md`
3. The assigned story in `docs/stories/`

Do not read the whole repo by default. Only read docs listed in the story packet.

## Key docs

- Product brief: `docs/product/PROJECT_BRIEF.md`
- Architecture: `docs/product/ARCHITECTURE.md`
- API contract: `docs/product/API_CONTRACT.md`
- Database schema: `docs/product/DATABASE_SCHEMA.md`
- Deployment: `docs/product/DEPLOYMENT.md`
- UI spec: `docs/product/UI_SPEC.md`
- Methodology: `docs/product/METHODOLOGY.md`