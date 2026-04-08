# BigQuery Optimizer

The essential BigQuery plugin for Claude Code. Comprehensive skill suite covering query optimization, SQL generation, schema design, cost optimization, and BigQuery-specific features.

Based on anti-pattern knowledge from [GoogleCloudPlatform/bigquery-antipattern-recognition](https://github.com/GoogleCloudPlatform/bigquery-antipattern-recognition).

## What It Does

This plugin makes Claude a BigQuery expert. It covers five skill areas that activate automatically based on what you're doing:

| Skill | What It Does | Triggers When You... |
|-------|-------------|---------------------|
| **Query Optimization** | Detects 11 SQL anti-patterns, provides before/after fixes | Review or optimize existing SQL |
| **Query Generation** | Generates optimized SQL from natural language, converts between dialects | Ask Claude to write a query |
| **Schema Design** | Recommends partitioning, clustering, nested fields, data types | Design tables or ask about schema |
| **Cost Optimization** | Estimates costs, reduces bytes billed, optimizes slot usage | Ask about costs or expensive queries |
| **BigQuery Features** | Expert guidance on STRUCT/ARRAY, MERGE, scripting, BQML, and more | Ask about specific BQ features |

## Installation

### Step 1: Add the Marketplace

```bash
/plugin marketplace add justvinhhere/bigquery-optimizer
```

This registers the plugin catalog with Claude Code.

### Step 2: Install the Plugin

**Option A -- From the UI:**

```bash
/plugin
```

Go to the **Discover** tab, select **bigquery-optimizer**, and choose your installation scope (User, Project, or Local).

**Option B -- From the command line:**

```bash
/plugin install bigquery-optimizer@justvinhhere-bigquery-optimizer
```

### Step 3: Activate

```bash
/reload-plugins
```

This loads the plugin without restarting Claude Code.

### Verify Installation

Run `/plugin` and go to the **Installed** tab to confirm `bigquery-optimizer` is listed and enabled.

## Commands

| Command | Description | Example |
|---------|-------------|---------|
| `bq-review` | Review SQL for anti-patterns | `/bigquery-optimizer:bq-review path/to/query.sql` |
| `bq-optimize` | Fix all detected anti-patterns | `/bigquery-optimizer:bq-optimize path/to/query.sql` |
| `bq-generate` | Generate SQL from a description | `/bigquery-optimizer:bq-generate "daily active users by country"` |
| `bq-design-table` | Design a table schema | `/bigquery-optimizer:bq-design-table "user events with timestamps and metadata"` |
| `bq-estimate-cost` | Estimate query or table cost | `/bigquery-optimizer:bq-estimate-cost path/to/query.sql` |
| `bq-explain` | Explain a BigQuery feature | `/bigquery-optimizer:bq-explain "MERGE for upserts"` |

All commands also accept inline SQL or work with the most recent SQL in the conversation.

## Agents

Ask Claude naturally and the right agent will activate:

| Agent | Trigger Phrases |
|-------|----------------|
| **bq-reviewer** | "Review all SQL files for anti-patterns", "Scan this project for BigQuery issues" |
| **bq-schema-advisor** | "Audit my table schemas", "Recommend partitioning strategies for this project" |
| **bq-cost-analyzer** | "Which queries are most expensive?", "Find cost optimization opportunities" |

## Skills Reference

### Query Optimization (11 Anti-Patterns)

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

### Schema Design

Covers partitioning strategies (time-unit, integer-range, ingestion-time), clustering (up to 4 columns), nested/repeated fields (STRUCT/ARRAY), denormalization patterns, table types (native, external, views, materialized views, clones), and data type best practices.

### Cost Optimization

Covers pricing models (on-demand vs editions), bytes-billed reduction, slot optimization, materialized views and query caching, storage optimization, and cost estimation with `--dry_run`.

### Query Generation

Generates optimized BigQuery SQL from natural language descriptions. Handles schema context, proactively avoids all 11 anti-patterns, and supports dialect conversion from PostgreSQL, MySQL, Snowflake, Redshift, and SQL Server.

### BigQuery Features

Expert guidance on STRUCT/ARRAY/UNNEST, MERGE DML, scripting (DECLARE, IF, LOOP), JSON functions, approximate aggregation, geography/GIS functions, BigQuery ML, search indexes, and vector search.

## Examples

### Generate a Query

```
/bigquery-optimizer:bq-generate "top 10 customers by total order value in the last 30 days"
```

### Review Existing SQL

```
/bigquery-optimizer:bq-review "SELECT * FROM `project.dataset.orders`"
```

### Design a Table

```
/bigquery-optimizer:bq-design-table "user click events with timestamp, page URL, session ID, and user agent"
```

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

## Uninstall

```bash
/plugin uninstall bigquery-optimizer@justvinhhere-bigquery-optimizer
```

To also remove the marketplace:

```bash
/plugin marketplace remove justvinhhere-bigquery-optimizer
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
