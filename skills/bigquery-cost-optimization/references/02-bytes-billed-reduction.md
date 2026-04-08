# Bytes Billed Reduction

**Impact:** High

## What This Covers
Techniques to reduce bytes billed on on-demand pricing: column pruning, partition pruning, clustering, sampling, and dry-run validation.

## Why It Matters
On-demand cost is directly proportional to bytes billed. Reducing bytes billed by 50% cuts your compute bill in half. Partition pruning alone can reduce scans from TB to GB.

## Best Practice

### Column Pruning
Select only needed columns. BigQuery is columnar -- unused columns are never read.
```sql
-- Bad: SELECT * FROM `project.dataset.events`
-- Good:
SELECT user_id, event_type, created_at FROM `project.dataset.events`
```

### Partition Pruning
Always filter on the partition column with literals or `DECLARE` variables.
```sql
SELECT user_id, event_type FROM `project.dataset.events`
WHERE DATE(created_at) = '2025-01-15'
-- For ingestion-time: WHERE _PARTITIONTIME >= '2025-01-01' AND _PARTITIONTIME < '2025-02-01'
```

### Clustering and Sampling
Cluster columns used in WHERE/ORDER BY. Use `TABLESAMPLE SYSTEM (10 PERCENT)` for approximate analysis. Note: TABLESAMPLE operates on data blocks (~1 GB each). For small tables, actual sample may be 0% or 100%. Results are never cached.

### Dry Run and Bytes Check
```bash
bq query --dry_run --use_legacy_sql=false 'SELECT user_id FROM `project.dataset.events`'
```
```sql
-- Check actual bytes billed for recent queries
SELECT job_id, total_bytes_billed, ROUND(total_bytes_billed / POW(1024, 4) * 6.25, 2) AS cost_usd
FROM `region-us`.INFORMATION_SCHEMA.JOBS
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY total_bytes_billed DESC LIMIT 10
```

## Edge Cases / Pitfalls
- Minimum billing is 10 MB per query, even if the query reads less.
- Functions on partition columns (`YEAR(created_at) = 2025`) may defeat pruning -- use range filters.
- `_PARTITIONTIME` only works with ingestion-time partitioned tables.
- Clustering benefits degrade over time; BigQuery auto-reclusters but not instantly.
