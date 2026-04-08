# LatestRecordWithAnalyticFun

**Severity:** High

## What It Detects
The pattern of using `ROW_NUMBER()` or `RANK()` window function with `PARTITION BY ... ORDER BY ...` and then filtering `WHERE rn = 1` (or `rnk = 1`) to get the latest/first record per group.

## Why It Matters
`ROW_NUMBER()` computes a row number for every row in every partition, then discards all rows except `rn = 1`. `ARRAY_AGG` with `LIMIT 1` is significantly more efficient in BigQuery because it only keeps the top record per group without numbering all rows.

## Before
```sql
SELECT taxi_id, trip_seconds, fare
FROM (
  SELECT
    taxi_id, trip_seconds, fare,
    ROW_NUMBER() OVER (PARTITION BY taxi_id ORDER BY fare DESC) rn
  FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
)
WHERE rn = 1
```

## After
```sql
SELECT event.*
FROM (
  SELECT ARRAY_AGG(
    t ORDER BY t.fare DESC LIMIT 1
  )[OFFSET(0)] event
  FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips` t
  GROUP BY t.taxi_id
)
```

## Edge Cases
- Only applies when filtering to `rn = 1` or `rnk = 1`. If filtering to `rn <= 5` (top N), `ROW_NUMBER()` is the correct approach.
- Does not flag `DENSE_RANK()` or other window functions.
- If you need multiple columns from the latest record, `ARRAY_AGG` of a struct is still more efficient.
