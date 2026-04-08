# Materialized Views and Caching

**Impact:** High

## What This Covers
Creating and leveraging materialized views (MVs) for cost savings, query cache behavior, and smart tuning (automatic query rewriting).

## Why It Matters
Materialized views pre-compute aggregations and store results. BigQuery can automatically rewrite queries to use MVs, reducing bytes scanned and slot consumption without changing application SQL.

## Best Practice

### Creating a Materialized View
```sql
CREATE MATERIALIZED VIEW `project.dataset.daily_revenue_mv`
PARTITION BY order_date
CLUSTER BY region
AS
SELECT
  DATE(order_timestamp) AS order_date,
  region,
  COUNT(*) AS order_count,
  SUM(amount) AS total_revenue
FROM `project.dataset.orders`
GROUP BY 1, 2
```

### Auto-Refresh and Smart Tuning
MVs refresh automatically. Control staleness with `OPTIONS (max_staleness = INTERVAL 30 MINUTE)`. BigQuery auto-rewrites queries to use MVs when the MV and base table are in the same dataset and the query is a subset of the MV aggregation (smart tuning, on by default).

### Query Cache
- Identical query text hitting the same table returns cached results (free on on-demand).
- Cache expires after 24 hours or when underlying data changes.
- Disabled by: DML on destination table, non-deterministic functions (`CURRENT_TIMESTAMP()`), streaming buffer active.

### When to Use MVs vs. Views vs. Tables
| Approach | Use When | Cost |
|----------|----------|------|
| Materialized view | Repeated aggregations, dashboard backing | Storage + auto-refresh compute |
| Logical view | Query abstraction, no compute savings | Full scan each execution |
| Pre-aggregated table | Complex transforms, MVs restricted | Storage + scheduled load |

## Edge Cases / Pitfalls
- Incremental MVs support INNER JOINs (left-side table only receives new data) and JOIN UNNEST. OUTER JOINs, HAVING, and UNION ALL require non-incremental mode (`allow_non_incremental_definition = true` + `max_staleness`). No JavaScript UDFs or non-deterministic functions.
- For smart tuning (auto-rewrite), the MV must be in the same dataset as the base table. Cross-dataset MVs are supported but do not benefit from automatic query rewriting.
- Auto-rewrite does not work if the query includes columns not in the MV.
- MV storage counts toward your storage billing.
- `max_staleness` trades freshness for cost; set appropriately per use case.
- Cached results still consume slots under editions pricing.
