---
description: Design a BigQuery table schema with optimal partitioning, clustering, and data types. Describe your data or paste a sample.
argument-hint: "<data description or sample>"
---

# BigQuery Table Design

Design an optimal BigQuery table schema using the `bigquery-schema-design` skill.

## Instructions

1. **Determine the input:**
   - If an argument is provided, treat it as the data description or sample data.
   - If a file path is provided, read that file for sample data or an existing schema.
   - If no argument is provided, ask the user to describe their data, its volume, and primary query patterns.

2. **Gather context** by asking (if not already provided):
   - What queries will run against this table? (filter columns, aggregations, joins)
   - Approximate data volume and growth rate (rows/day, total size).
   - Write pattern: batch loads, streaming inserts, or frequent updates?
   - Data retention requirements (how long to keep data).

3. **Design the schema** using the `bigquery-schema-design` skill:
   - Choose partitioning strategy based on primary filter column and data volume.
   - Choose clustering columns based on secondary filters and join keys.
   - Decide nested vs. flat based on entity relationships and update patterns.
   - Select optimal data types for each column.
   - Consider table type (native, materialized view, external) based on access patterns.

4. **Output the recommendation** in the standard schema design format:
   - Design decisions summary (partitioning, clustering, nesting, data types).
   - Complete `CREATE TABLE` DDL with all options.
   - Rationale explaining trade-offs and why this design fits the workload.
   - If applicable, suggest materialized views or complementary tables.
