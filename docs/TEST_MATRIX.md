# TEST_MATRIX.md

This is the validation **index only**. Do not read every validation file by default. Read only the validation file related to the assigned story.

## Validation files

| Area | File |
|---|---|
| Backend API | `docs/test-matrix/backend.md` |
| Frontend dashboard | `docs/test-matrix/frontend.md` |
| Data pipeline | `docs/test-matrix/pipeline.md` |
| Database / Supabase | `docs/test-matrix/database.md` |
| Modeling (clustering / retrieval / outcomes) | `docs/test-matrix/modeling.md` |
| Deployment | `docs/test-matrix/deployment.md` |

## Default proof rule

Every story must provide at least one of: command run, test/lint/build output, migration check, health check, or a manual verification step. If proof cannot be run, state why and provide the exact command for the human to run.
