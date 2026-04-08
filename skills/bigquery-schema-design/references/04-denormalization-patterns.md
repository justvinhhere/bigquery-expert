# Denormalization Patterns

**Impact:** High

## What This Covers
When to denormalize vs. keeping normalized schemas. Materialized views as a middle ground.

## Why It Matters
BigQuery is optimized for sequential reads. JOINs shuffle data across slots. Denormalization pre-joins at write time, trading cheap storage for expensive query time.

## Best Practice

**Fully denormalized (best for read-heavy analytics):**
```sql
CREATE TABLE `project.dataset.sales_denormalized`
(
  sale_id INT64,
  sale_date DATE,
  amount NUMERIC,
  -- Customer fields embedded
  customer_id INT64,
  customer_name STRING,
  customer_region STRING,
  -- Product fields embedded
  product_id INT64,
  product_name STRING,
  product_category STRING
)
PARTITION BY sale_date
CLUSTER BY customer_region, product_category
OPTIONS(description = "Denormalized sales fact with dimension attributes");
```

**Materialized view as middle ground (keeps source normalized):**
```sql
CREATE MATERIALIZED VIEW `project.dataset.daily_sales_by_region`
OPTIONS(enable_refresh = true, refresh_interval_minutes = 30)
AS
SELECT
  DATE(sale_ts) AS sale_date,
  region,
  COUNT(*) AS sale_count,
  SUM(amount) AS total_amount
FROM `project.dataset.sales`
GROUP BY sale_date, region;
```

**When to keep normalized:**
- Data changes frequently (e.g., customer addresses update daily).
- Multiple fact tables reference the same dimension (avoid duplication drift).
- Storage cost matters more than query speed.

**When to denormalize:**
- Read-to-write ratio is 100:1 or higher.
- Dashboard queries repeatedly JOIN the same tables.
- Query latency SLAs require sub-second response.

## Edge Cases / Pitfalls
- Denormalized tables require rebuilding when dimension attributes change. Use scheduled queries or `MERGE` statements to keep them fresh.
- Incremental MVs support INNER JOINs and JOIN UNNEST. OUTER JOINs, HAVING, and UNION ALL require non-incremental mode. Best for simple aggregations; complex transforms need pre-aggregated tables.
- BigQuery's smart tuning automatically rewrites queries to use materialized views when beneficial (enabled by default, no configuration needed). No query changes needed.
- Over-denormalization creates extremely wide tables (200+ columns) that are hard to maintain. Group related attributes into STRUCTs to stay organized.
