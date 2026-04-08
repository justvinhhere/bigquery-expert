---
description: Generate optimized BigQuery SQL from a natural language description. Pass a description of the data you need.
argument-hint: "<description>"
---

# BigQuery Query Generation

Generate optimized BigQuery SQL using the `bigquery-query-generation` skill.

## Instructions

1. **Determine what SQL to generate** from the argument or current conversation context. If neither provides a clear description, ask the user what data they need.

2. **Gather schema context** if available. Check whether the user has mentioned specific table names, column names, or project/dataset references. If not, use descriptive placeholders and state assumptions.

3. **Generate the SQL** using the `bigquery-query-generation` skill. Proactively avoid all 11 anti-patterns from the `bigquery-optimization` skill while generating.

4. **Output using this format:**

   ```
   ## Generated Query

   (fenced SQL code block)

   ## Assumptions
   - Schema, data type, and business logic assumptions.
   - Placeholder table/column names that need replacing.

   ## Anti-Patterns Proactively Avoided
   - List only the anti-patterns that were relevant to this query and actively avoided.

   ## Customization Notes
   - Suggestions for adapting the query: filters, additional columns, partitioning, etc.
   ```

5. **Prefer generating with stated assumptions** over asking too many clarifying questions. Generate a working query first, then offer to refine.
