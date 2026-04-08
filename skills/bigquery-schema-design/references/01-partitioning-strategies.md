# Partitioning Strategies

**Impact:** High

## What This Covers
Time-unit (DAY, HOUR, MONTH, YEAR), integer-range, and ingestion-time partitioning strategies.

## Why It Matters
Partition pruning lets BigQuery skip entire segments of data, reducing bytes scanned by 90%+ and cutting cost proportionally.

## Best Practice

**Time-unit partitioning** (most common):
```sql
CREATE TABLE `project.dataset.events`
(
  event_id INT64,
  event_ts TIMESTAMP,
  user_id INT64,
  event_type STRING
)
PARTITION BY DATE(event_ts)
OPTIONS(
  require_partition_filter = true,
  partition_expiration_days = 365,
  description = "User events partitioned by day"
);
```

**Integer-range partitioning:**
```sql
CREATE TABLE `project.dataset.customers`
(
  customer_id INT64,
  name STRING,
  region STRING
)
PARTITION BY RANGE_BUCKET(customer_id, GENERATE_ARRAY(0, 1000000, 1000))
OPTIONS(description = "Customers partitioned by ID range");
```

**Ingestion-time partitioning:**
```sql
CREATE TABLE `project.dataset.raw_logs`
(
  payload STRING,
  source STRING
)
PARTITION BY _PARTITIONTIME
OPTIONS(description = "Raw logs partitioned by load date");
```

Queries must filter on the partition column to benefit:
```sql
SELECT * FROM `project.dataset.events`
WHERE DATE(event_ts) = '2025-01-15';  -- prunes to 1 partition
```

## Edge Cases / Pitfalls
- **10,000 partition limit** per table (4,000 per single DML/load operation). Daily partitions cover ~27 years. Use MONTH or YEAR for very long retention or low-volume tables.
- **Do not partition tables under 1 GB.** The metadata overhead outweighs pruning gains.
- **HOUR partitioning** is only worthwhile for very high-volume streaming tables (100+ GB/day).
- **`require_partition_filter = true`** prevents accidental full-table scans but blocks queries that intentionally need all data. Set it on cost-sensitive tables.
- Partition columns must be `DATE`, `TIMESTAMP`, `DATETIME`, or `INT64`. No STRING partitioning.
