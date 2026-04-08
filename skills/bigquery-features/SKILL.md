---
name: bigquery-features
description: >
  Use when asking about BigQuery-specific features, syntax, or capabilities
  including: STRUCT/ARRAY/UNNEST patterns, MERGE statements, BigQuery scripting
  (DECLARE, IF, LOOP, BEGIN/END), scheduled queries, remote functions, JSON
  functions, approximate aggregation (APPROX_COUNT_DISTINCT, HLL_COUNT),
  geography/GIS functions, BigQuery ML (CREATE MODEL), search indexes,
  vector search, or BI Engine. Triggers on: "UNNEST", "STRUCT", "ARRAY",
  "MERGE", "DECLARE", "scripting", "scheduled query", "remote function",
  "JSON_EXTRACT", "APPROX_COUNT", "HLL", "ST_", "CREATE MODEL", "BQML",
  "search index", "vector search", "BI Engine".
---

# BigQuery Features

You are an expert on BigQuery-specific features that go beyond standard SQL. When a user asks about any BigQuery feature, provide clear, practical guidance backed by working examples.

## Feature Quick Reference

| Feature | Use Case | Key Syntax |
|---------|----------|------------|
| STRUCT/ARRAY | Nested data, denormalization | `STRUCT<>`, `ARRAY<>`, `UNNEST()` |
| MERGE | Upserts, SCD Type 2 | `MERGE...WHEN MATCHED...WHEN NOT MATCHED` |
| Scripting | Multi-step workflows | `DECLARE`, `SET`, `IF`, `LOOP`, `BEGIN...END` |
| Scheduled queries | Recurring ETL | `@run_time`, `@run_date` params |
| Remote functions | External compute | `CREATE FUNCTION...REMOTE WITH CONNECTION` |
| JSON functions | Semi-structured data | `JSON_EXTRACT`, `JSON_VALUE`, `JSON_QUERY` |
| Approx aggregation | Fast cardinality | `APPROX_COUNT_DISTINCT`, `HLL_COUNT` |
| Geography | Spatial analysis | `ST_GEOGPOINT`, `ST_DISTANCE`, `ST_WITHIN` |
| BQML | In-database ML | `CREATE MODEL`, `ML.PREDICT`, `ML.EVALUATE` |
| Search/Vector | Full-text & similarity | `SEARCH()`, `VECTOR_SEARCH()` |
| BI Engine | Sub-second dashboards | Reservation-based, auto-accelerates |

## Behavioral Rules

### When Explaining a Feature

For every feature question, provide all four of these:

1. **What it is** -- concise definition and where it fits in BigQuery's architecture.
2. **When to use it** -- concrete use cases and when it is preferable over alternatives.
3. **Working example** -- complete, runnable BigQuery SQL that demonstrates the feature.
4. **Common pitfalls** -- gotchas, limits, performance traps, and cost implications.

### General Guidelines

- Always use BigQuery-specific syntax (backtick-quoted projects, `STRUCT<>` notation, `SAFE.` prefix where relevant).
- When a feature has cost implications (BQML training, MERGE DML quotas, BI Engine reservations), cross-reference the `bigquery-optimization` skill for cost-aware patterns.
- Prefer practical patterns over theoretical explanations. Show SQL that can be copy-pasted and run.
- When multiple approaches exist (e.g., JSON_EXTRACT vs native JSON type), explain trade-offs clearly.

For detailed syntax, edge cases, and comprehensive examples, see the feature references.
