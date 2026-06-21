# Database validation

## Migration check

Expected:
- SQL is Supabase/Postgres compatible
- primary keys exist
- unique constraints exist for time-series tables:
  - `daily_bars (ticker, bar_date, source)`
  - `intraday_bars (ticker, bar_timestamp, interval, source)`
  - `features_daily (ticker, feature_date, feature_schema_version)`
  - `features_hourly (ticker, feature_time, feature_schema_version)`
  - `behavior_windows (ticker, window_end_date, window_length, feature_schema_version)`
- cluster outputs are versioned by `run_id`
- useful indexes exist for dashboard queries
- `dashboard_snapshots` supports a fast "latest valid" lookup

Suggested checks:
```sql
select * from stocks limit 1;
select * from pipeline_runs order by started_at desc limit 5;
```
