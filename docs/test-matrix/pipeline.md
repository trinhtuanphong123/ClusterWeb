# Pipeline validation

## Worker startup

Expected:
- scheduler starts without import errors
- logs job registration
- does not require real API key in mock mode

Command:
```bash
python -m pipelines.scheduler
Pipeline logging

Expected:

job status is written to pipeline_runs
errors are captured without crashing the whole worker