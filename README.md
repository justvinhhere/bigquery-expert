# BigQuery Optimizer

A Claude Code plugin that detects 11 BigQuery SQL performance anti-patterns and helps you write optimized queries.

Based on the anti-pattern knowledge from [GoogleCloudPlatform/bigquery-antipattern-recognition](https://github.com/GoogleCloudPlatform/bigquery-antipattern-recognition).

## What It Does

This plugin teaches Claude to recognize common BigQuery SQL performance pitfalls and apply best practices. When you work with BigQuery SQL, Claude will:

- **Write optimized SQL from the start** -- avoiding known anti-patterns proactively
- **Review existing queries** for performance issues with actionable before/after fixes
- **Rewrite suboptimal queries** preserving original semantics
- **Scan entire projects** for BigQuery SQL anti-patterns across `.sql` and code files

## Anti-Patterns Detected

| # | Anti-Pattern | Description | Severity |
|---|-------------|-------------|----------|
| 1 | SimpleSelectStar | `SELECT *` when only specific columns are needed | High |
| 2 | SemiJoinWithoutAgg | `IN`/`NOT IN` subquery without `DISTINCT` | Medium |
| 3 | CTEsEvalMultipleTimes | CTE referenced multiple times (re-executed each time) | High |
| 4 | OrderByWithoutLimit | `ORDER BY` without `LIMIT` on outermost query | Medium |
| 5 | StringComparison | `REGEXP_CONTAINS` where `LIKE` suffices | Low |
| 6 | LatestRecordWithAnalyticFun | `ROW_NUMBER() + WHERE rn=1` instead of `ARRAY_AGG` | High |
| 7 | DynamicPredicate | Subquery inside WHERE clause | Medium |
| 8 | WhereOrder | WHERE predicates not ordered by selectivity | Medium |
| 9 | JoinOrder | Smaller table placed before larger in JOIN | High |
| 10 | MissingDropStatement | TEMP table created without DROP statement | Low |
| 11 | ConvertTableToTemp | Persistent table created and dropped in same script | Low |

## Installation

```bash
claude plugin add justvinhhere/bigquery-optimizer
```

## Usage

### Automatic Activation

The plugin activates automatically when you work with BigQuery SQL. Claude will apply anti-pattern awareness when writing or discussing BigQuery queries.

### Commands

**Review a query for anti-patterns:**
```
/bq-review path/to/query.sql
```

**Optimize a query by fixing all detected anti-patterns:**
```
/bq-optimize path/to/query.sql
```

Both commands also work without arguments -- they'll review the most recent SQL in the conversation.

### Agent: Project-Wide Scan

Ask Claude to scan your entire project:

> "Review all SQL files in this project for BigQuery anti-patterns"

The `bq-reviewer` agent will autonomously scan all `.sql` files and code files with embedded BigQuery SQL, then produce a consolidated report.

## Examples

### Before: ROW_NUMBER for Latest Record (High Severity)

```sql
SELECT taxi_id, trip_seconds, fare
FROM (
  SELECT taxi_id, trip_seconds, fare,
    ROW_NUMBER() OVER (PARTITION BY taxi_id ORDER BY fare DESC) rn
  FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
)
WHERE rn = 1
```

### After: ARRAY_AGG (Optimized)

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

---

### Before: CTE Referenced Multiple Times (High Severity)

```sql
WITH expensive_cte AS (
  SELECT col1, col2 FROM `dataset.large_table` WHERE col1 = 'abc'
),
result_a AS (SELECT col2 FROM expensive_cte),
result_b AS (SELECT col1 FROM expensive_cte)
SELECT * FROM result_a, result_b
```

### After: TEMP Table (Optimized)

```sql
CREATE TEMP TABLE expensive_cte AS
SELECT col1, col2 FROM `dataset.large_table` WHERE col1 = 'abc';

WITH
result_a AS (SELECT col2 FROM expensive_cte),
result_b AS (SELECT col1 FROM expensive_cte)
SELECT * FROM result_a, result_b;

DROP TABLE expensive_cte;
```

## Attribution

The anti-pattern detection knowledge in this plugin is derived from [BigQuery Anti-Pattern Recognition](https://github.com/GoogleCloudPlatform/bigquery-antipattern-recognition) by Google Cloud Platform, licensed under the Apache License 2.0.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

For bug reports and feature requests, please [open an issue](https://github.com/justvinhhere/bigquery-optimizer/issues).

## License

Apache License 2.0 -- see [LICENSE](LICENSE) for details.
