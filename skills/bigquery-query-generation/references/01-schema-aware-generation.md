# Schema-Aware Generation

**Impact:** High

## What This Covers
How to handle schema context when generating BigQuery SQL -- from fully specified table references to purely conceptual data descriptions.

## Why It Matters
Generating SQL without correct schema context produces queries that fail on execution or return wrong results. Consistent placeholder conventions make generated queries easy to adapt to real schemas.

## Best Practice

### When the user provides table names
Use them exactly as given, with backtick quoting:
```sql
SELECT user_id, event_name, event_timestamp
FROM `my-project.analytics.events`
WHERE event_date = '2024-01-15'
```

### When the user describes data conceptually
Use `project.dataset.table` format with descriptive names. Always note placeholders in output:
```sql
SELECT order_id, customer_id, total_amount
FROM `project.dataset.orders`
WHERE order_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
```

### Discovering schema with INFORMATION_SCHEMA
When working with a real project, generate a discovery query first:
```sql
SELECT column_name, data_type, is_nullable
FROM `my-project.my_dataset.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'target_table'
ORDER BY ordinal_position
```

### Placeholder naming conventions
| Scenario | Placeholder Format | Example |
|----------|-------------------|---------|
| Unknown project | `project` | `project.dataset.users` |
| Unknown dataset | `dataset` | `my-project.dataset.orders` |
| Unknown table | Descriptive noun | `project.dataset.user_events` |
| Unknown column | Descriptive name | `user_id`, `created_at`, `amount` |

## Edge Cases / Pitfalls
- Always use three-part table references (`project.dataset.table`). Two-part references depend on the default project setting and break in cross-project queries.
- Dataset-scoped `INFORMATION_SCHEMA` (`dataset.INFORMATION_SCHEMA.COLUMNS`) covers one dataset. Use the region-scoped syntax (`region-us.INFORMATION_SCHEMA.COLUMNS`) to query across all datasets in a region. These queries are billed normally (10 MB minimum).
- Partitioned tables may have `_PARTITIONTIME` or `_PARTITIONDATE` pseudo-columns not visible in `INFORMATION_SCHEMA`. Mention these when generating queries against date-partitioned tables.
- Views and table functions appear in `INFORMATION_SCHEMA.TABLES` but their column metadata may differ from the underlying tables.
