# Database validation 
## Migration check 
Expected: 
- SQL is Supabase/Postgres compatible 
- primary keys exist 
- unique constraints exist for time-series tables 
- useful indexes exist for dashboard queries 
Suggested checks: 
sql select * from stocks limit 1; 
select * from pipeline_runs order by started_at desc limit 5;