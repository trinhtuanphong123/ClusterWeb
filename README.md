# Vietnamese Stock Co-Movement Clustering Dashboard 
## What this project does 
This project collects Vietnamese stock data, computes features, clusters stocks by co-movement behavior, and displays the results in a dashboard. 
## Stack 
- Frontend: Next.js on Vercel 
- Backend: FastAPI on AWS EC2 
- Worker: Python scheduler on AWS EC2 
- Database: Supabase Postgres 
- Optional storage: AWS S3 
## Agent workflow 
This repo uses a lean repository harness. Agents should start from: 
1. `AGENTS.md` 
2. `docs/CONTEXT_INDEX.md` 
3. the assigned story in `docs/stories/` 
Do not ask agents to read the whole repo by default. 
## Important docs 
- Product brief: `docs/product/PROJECT_BRIEF.md` 
- API contract: `docs/product/API_CONTRACT.md` 
- Database schema: `docs/product/DATABASE_SCHEMA.md` 
- Deployment: `docs/product/DEPLOYMENT.md` 
- UI spec: `docs/product/UI_SPEC.md` 
- Methodology: `docs/product/METHODOLOGY.md`