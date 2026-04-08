# Approximate Aggregation

**Impact:** Medium

## What This Covers

BigQuery's approximate aggregate functions for fast cardinality estimation, quantile computation, and top-N counts, including HyperLogLog++ sketches for mergeable aggregation.

## Why It Matters

Exact `COUNT(DISTINCT)` on high-cardinality columns over large datasets is expensive (requires shuffling all unique values). Approximate functions provide results within ~1% error at a fraction of the cost and time.

## Example

```sql
-- APPROX_COUNT_DISTINCT: fast cardinality (~1% error)
SELECT
  event_date,
  APPROX_COUNT_DISTINCT(user_id) AS approx_unique_users
FROM `project.dataset.events`
GROUP BY event_date;

-- Compare cost: exact vs approximate
SELECT COUNT(DISTINCT user_id) AS exact_count          -- slower, full shuffle
FROM `project.dataset.events`;
SELECT APPROX_COUNT_DISTINCT(user_id) AS approx_count  -- faster, ~1% error
FROM `project.dataset.events`;

-- APPROX_QUANTILES: approximate percentiles
SELECT
  APPROX_QUANTILES(response_time_ms, 100)[OFFSET(50)] AS p50,
  APPROX_QUANTILES(response_time_ms, 100)[OFFSET(95)] AS p95,
  APPROX_QUANTILES(response_time_ms, 100)[OFFSET(99)] AS p99
FROM `project.dataset.requests`;

-- APPROX_TOP_COUNT: most frequent values
SELECT APPROX_TOP_COUNT(page_path, 10) AS top_pages
FROM `project.dataset.pageviews`;

-- HLL_COUNT: mergeable sketches for pre-aggregation
-- Step 1: Create daily sketches
CREATE OR REPLACE TABLE `project.dataset.daily_sketches` AS
SELECT
  event_date,
  HLL_COUNT.INIT(user_id) AS user_sketch
FROM `project.dataset.events`
GROUP BY event_date;

-- Step 2: Merge sketches for arbitrary date ranges
SELECT
  HLL_COUNT.EXTRACT(HLL_COUNT.MERGE(user_sketch)) AS unique_users_this_week
FROM `project.dataset.daily_sketches`
WHERE event_date BETWEEN '2025-01-01' AND '2025-01-07';
```

## Edge Cases / Pitfalls

- **Error bounds:** `APPROX_COUNT_DISTINCT` guarantees ~1% relative error for typical cardinalities. For very low cardinality (<1000), exact `COUNT(DISTINCT)` is fast enough and more precise.
- **NULL handling:** All approximate functions ignore NULLs, same as their exact counterparts.
- **HLL_COUNT sketch size:** Each sketch is ~32 KB. For tables with many groups, the sketch column can add significant storage.
- **APPROX_TOP_COUNT output:** Returns an ARRAY of STRUCT<value, count>. Access results with UNNEST or array indexing.
- **Not deterministic:** Approximate results may vary slightly between runs. Do not use for financial reporting or exact counts in billing.
- **When NOT to use:** Queries with low cardinality, small datasets, or where exact results are required (compliance, billing, SLAs).
