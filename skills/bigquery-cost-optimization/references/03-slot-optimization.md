# Slot Optimization

**Impact:** High

## What This Covers
Understanding and optimizing BigQuery slot consumption for editions (capacity) pricing: what slots are, how to monitor them, and how to reduce usage.

## Why It Matters
Under editions pricing, cost is driven by slot-hours, not bytes scanned. A query that shuffles 10 TB across slots costs more than one that scans 10 TB but processes locally. Reducing slot consumption lowers cost and improves concurrency.

## Best Practice

### What Slots Are
A slot is a unit of computational capacity (CPU + memory + I/O). BigQuery automatically distributes work across available slots. More complex queries consume more slots for longer.

### Monitoring Slot Usage
```sql
SELECT
  job_id,
  total_slot_ms,
  ROUND(total_slot_ms / 1000 / 3600, 2) AS slot_hours,
  total_bytes_billed,
  query
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY total_slot_ms DESC
LIMIT 20
```

### Identifying Slot-Heavy Patterns
| Pattern | Why It's Expensive | Fix |
|---------|-------------------|-----|
| Large cross joins | Cartesian product explodes rows | Add join conditions or pre-filter |
| Unfiltered JOINs on large tables | Shuffles entire tables | Add WHERE before JOIN |
| Complex JavaScript UDFs | Single-threaded, blocks slots | Rewrite in native SQL |
| Repeated CTE evaluation | Same work done N times | Use `CREATE TEMP TABLE` |
| `ORDER BY` without `LIMIT` | Full sort across all data | Add `LIMIT` or remove sort |

### Reducing Slot Consumption
```sql
-- Bad: JS UDF blocks parallelism
CREATE TEMP FUNCTION parse_json(s STRING) RETURNS STRING LANGUAGE js AS 'return JSON.parse(s).key';
-- Good: native SQL
SELECT JSON_VALUE(raw_json, '$.key') FROM `project.dataset.events`

-- Bad: unfiltered JOIN shuffles full tables
-- Good: filter before joining to reduce shuffle volume
SELECT a.order_id, a.amount, b.name
FROM (SELECT order_id, amount, customer_id FROM `project.dataset.orders`
      WHERE DATE(order_date) = '2025-01-15') a
JOIN `project.dataset.customers` b ON a.customer_id = b.id
```

## Edge Cases / Pitfalls
- Slot consumption is not visible in dry runs -- dry runs only report bytes.
- Autoscaling in editions has a ramp-up delay; bursty workloads may queue.
- Approximate functions (`APPROX_COUNT_DISTINCT`) use far fewer slots than exact equivalents.
- BI Engine reservations are separate from query slot reservations.
