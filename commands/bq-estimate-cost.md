---
description: Estimate the cost of a BigQuery query or table. Pass a query, file path, or table description.
argument-hint: "[query, file_path, or table description]"
---

# BigQuery Cost Estimator

Estimate the cost of a BigQuery query or table using the `bigquery-cost-optimization` skill.

## Instructions

1. **Determine the input type:**
   - If a `.sql` file path is provided, read that file to get the query.
   - If raw SQL is provided, use it directly.
   - If a table name or description is provided, estimate storage cost.
   - If no input is found, ask the user to provide a query, file path, or table description.

2. **For queries -- estimate compute cost:**
   - Identify all tables referenced and note partitioning/clustering.
   - Estimate bytes billed based on columns selected, partition filters present, and clustering alignment.
   - Calculate on-demand cost: `bytes_billed_TB * $6.25`.
   - Flag if no partition filter is present on a partitioned table.
   - Remind the user to validate with `--dry_run`:
     ```
     bq query --dry_run --use_legacy_sql=false 'YOUR_QUERY_HERE'
     ```

3. **For tables -- estimate storage cost:**
   - Estimate active vs. long-term storage split.
   - Calculate monthly cost: `active_GB * $0.02 + longterm_GB * $0.01`.
   - Suggest expiration policies if applicable.

4. **Suggest cost reduction strategies** by referencing the bigquery-cost-optimization skill:
   - Column pruning opportunities.
   - Missing partition filters.
   - Materialized view candidates for repeated queries.
   - Pricing model recommendation if usage pattern is known.

5. **Output format:**
   ```
   ## Cost Estimate

   **Query/Table:** [summary]
   **Estimated bytes billed:** X GB (X TB)
   **Estimated cost (on-demand):** $X.XX
   **Pricing note:** [editions equivalent if applicable]

   ### Cost Reduction Opportunities
   1. [Opportunity with estimated savings]
   2. [Opportunity with estimated savings]

   ### Validate
   Run --dry_run to confirm: bq query --dry_run --use_legacy_sql=false '...'
   ```
