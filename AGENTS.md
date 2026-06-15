## Project 
This repository is a Vietnamese stock co-movement clustering dashboard.


The system collects stock market data from VNStock community/free API, stores it in Supabase Postgres, computes hourly features, runs clustering models, exposes results through a FastAPI backend, and displays them in a Next.js dashboard deployed on Vercel. 

Deployment target: 
- Frontend: Next.js on Vercel 
- Backend: FastAPI on AWS EC2 
- Worker: Python scheduler on AWS EC2 
- Database: Supabase Postgres 
- Optional storage: AWS S3 
## Agent operating rules
 1. Do not scan the whole repository unless explicitly asked. 
 2. Work on one story packet at a time. 
 3. Always read the assigned story in `docs/stories/` before editing. 
 4. Read only the product docs listed in the story packet. 
 5. Modify only the files listed in the story packet. 
 6. Do not modify unrelated files. 
 7. Do not add dependencies unless the story requires it. 
 8. Do not hard-code secrets. 
 9. Before editing, explain the plan briefly. 
 10. After editing, summarize changed files briefly. 
 11. Provide one verification command or check. 
 12. If the requested change affects architecture, deployment, database schema, or security, update or propose a decision record. 
 ## Context budget rule 
 For normal implementation tasks, read only: 
 1. `AGENTS.md` 
 2. `docs/CONTEXT_INDEX.md` 
 3. The assigned story packet in `docs/stories/` 
 4. The product docs explicitly listed in the story 
 5. The implementation files directly affected by the task 

 Do not read these files during normal coding tasks unless the story explicitly asks:
  - `docs/HARNESS.md` 
  - all files in `docs/templates/` 
  - all files in `docs/decisions/` 
  - all files in `docs/product/` 
  - the entire `docs/TEST_MATRIX.md` 
  ## Work type routing 
  Use `docs/CONTEXT_INDEX.md` to decide which docs are relevant. 
  Examples: 
  - Backend API work: read `API_CONTRACT.md`, relevant backend files. 
  - Database work: read `DATABASE_SCHEMA.md`, relevant migrations. 
  - Pipeline work: read `DATA_FLOW.md`, `METHODOLOGY.md` if modeling is involved. 
  - Frontend work: read `UI_SPEC.md`, `DESIGN_SYSTEM.md`, `API_CONTRACT.md`. 
  - Deployment work: read `DEPLOYMENT.md`, `ARCHITECTURE.md`, relevant infra files. 
  ## Non-goals 
  Do not implement: 
  - trading execution 
  - buy/sell recommendation engine 
  - user brokerage integration 
  - complex enterprise cloud stack 
  - Lambda/ECS/EKS migration unless explicitly requested 
  ## Required proof 
  Every completed story must include at least one proof: 
  - command output 
  - test result 
  - lint/build result 
  - SQL migration check 
  - API health check 
  - screenshot description 
  - manual verification step 
  
  If proof cannot be run, explain why and provide the exact command the human should run.