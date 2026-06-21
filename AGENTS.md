# AGENTS.md

## Mission

Build a **Vietnamese stock behavior clustering dashboard**: collect market data, cluster stocks by current movement behavior, explain clusters with historical pattern retrieval, and present results in a dashboard. This is **decision-support analytics, not a prediction or trading system**. No buy/sell recommendations, ever.

## Stack

- **Frontend:** Next.js on Vercel
- **Backend API:** FastAPI on AWS EC2
- **Worker:** Python scheduler on AWS EC2
- **Database:** Supabase Postgres
- **Optional storage:** AWS S3 (raw parquet, model artifacts, large matrices, vectors)

Avoid Lambda, ECS, EKS, Kafka, Airflow, Redshift, and other heavy enterprise infrastructure unless explicitly requested.

## Lean context rules

1. Do **not** scan the whole repository or the whole `docs/` directory.
2. Work on **one story packet at a time**.
3. Always read the assigned story in `docs/stories/` before editing.
4. Read **only** the docs listed in the story packet (use `docs/CONTEXT_INDEX.md` to confirm).
5. Modify **only** the files listed in the story packet's "Modify only" section.
6. Do not modify unrelated files; do not add dependencies unless the story requires it.
7. Never hard-code secrets or service-role keys.

Default context for any task: this file, `docs/CONTEXT_INDEX.md`, the assigned story packet, the product docs the story lists, and the implementation files the task touches. If more context seems necessary, ask which file to read next instead of scanning broadly.

## Work loop

1. **Plan before editing.** Briefly state the plan and the files you will change.
2. **Implement** the smallest change that satisfies the story.
3. **Summarize changed files** briefly after editing.
4. **Provide one verification command or check** (command output, test/lint/build result, migration check, API health check, or a manual step). If proof can't be run, give the exact command for the human.

## Decision records

Create or update a `docs/decisions/DEC-xxx-*.md` record when the work changes any of:

- architecture / deployment target
- database schema ownership or storage boundary
- scheduler strategy
- API contract
- model / clustering methodology
- security or secret-handling policy

Decision text in a trace is not a substitute for a durable decision record.

## Non-goals

No trading execution, no buy/sell recommendation engine, no brokerage integration, no complex enterprise cloud stack, no live 5-minute streaming in the MVP.
