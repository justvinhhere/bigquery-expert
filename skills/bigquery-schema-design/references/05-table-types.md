# Table Types

**Impact:** Medium

## What This Covers
Native tables, external tables, views, materialized views, snapshots, and clones.

## Why It Matters
Each type has different performance, cost, and freshness trade-offs. Choosing wrong leads to slow queries or unnecessary storage costs.

## Best Practice

**Native table** (best performance): standard `CREATE TABLE` with partitioning/clustering.

**External table** (no data loading):
```sql
CREATE EXTERNAL TABLE `project.dataset.csv_import`
(id INT64, name STRING, value FLOAT64)
OPTIONS(format = 'CSV', uris = ['gs://bucket/data/*.csv'], skip_leading_rows = 1);
```

**View** (logical, no storage): `CREATE VIEW ... AS SELECT ...` -- re-executed every query.

**Materialized view** (cached aggregation):
```sql
CREATE MATERIALIZED VIEW `project.dataset.daily_signups`
OPTIONS(enable_refresh = true, refresh_interval_minutes = 60)
AS
SELECT DATE(created_at) AS signup_date, COUNT(*) AS signups
FROM `project.dataset.users`
GROUP BY signup_date;
```

**Snapshot** (point-in-time backup):
```sql
CREATE SNAPSHOT TABLE `project.dataset.users_snap`
CLONE `project.dataset.users`
FOR SYSTEM_TIME AS OF '2025-01-01 00:00:00 UTC';
```

**Clone** (zero-copy, writable): `CREATE TABLE ... CLONE ...` -- divergence increases storage.

| Type | Storage | Speed | Freshness | Writable |
|------|---------|-------|-----------|----------|
| Native | Full | Fastest | Real-time | Yes |
| External | None in BQ | Slowest | Real-time | No |
| View | None | Base query | Real-time | No |
| Mat. View | Delta | Fast | Near real-time | No |
| Snapshot | Delta | Native | Point-in-time | No |
| Clone | Delta | Native | Point-in-time | Yes |

## Edge Cases / Pitfalls
- External tables lack clustering, partitioning (except Hive), and caching. Use for infrequent access.
- Views re-execute every query. For expensive transforms, prefer materialized views.
- Materialized views: incremental MVs support INNER JOINs and JOIN UNNEST. OUTER JOINs require non-incremental mode. No JavaScript UDFs. Best for aggregations.
- Snapshots/clones use copy-on-write. Initial cost is near zero; divergence increases storage.
