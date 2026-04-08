# Nested and Repeated Fields

**Impact:** High

## What This Covers
STRUCT (nested) and ARRAY of STRUCT (repeated) fields for hierarchical data. Includes UNNEST patterns.

## Why It Matters
Nested fields eliminate JOINs by co-locating related data. BigQuery's columnar format stores nested fields efficiently -- unused nested columns are not read.

## Best Practice

**STRUCT for 1:1 relationships:**
```sql
CREATE TABLE `project.dataset.customers`
(
  customer_id INT64,
  name STRING,
  address STRUCT<
    street STRING,
    city STRING,
    state STRING,
    zip STRING
  >
)
OPTIONS(description = "Customers with nested address");
```

**ARRAY of STRUCT for 1:N relationships:**
```sql
CREATE TABLE `project.dataset.orders`
(
  order_id INT64,
  customer_id INT64,
  order_date DATE,
  line_items ARRAY<STRUCT<
    product_id INT64,
    product_name STRING,
    quantity INT64,
    unit_price NUMERIC
  >>
)
PARTITION BY order_date
OPTIONS(description = "Orders with nested line items");
```

**Querying nested fields:**
```sql
-- Access STRUCT fields with dot notation
SELECT customer_id, address.city
FROM `project.dataset.customers`
WHERE address.state = 'CA';

-- UNNEST ARRAY fields to flatten 1:N
SELECT o.order_id, item.product_name, item.quantity
FROM `project.dataset.orders` o,
UNNEST(o.line_items) AS item
WHERE item.quantity > 5;
```

## Edge Cases / Pitfalls
- **15 levels max nesting.** Deeply nested schemas hit this limit; flatten intermediate levels if needed.
- ARRAY fields cannot be used directly in `WHERE` without `UNNEST` or an `EXISTS` subquery.
- DML on nested fields is verbose: `UPDATE` requires reconstructing the entire STRUCT or ARRAY.
- Avoid nesting when the child entity needs independent updates or is shared across multiple parents (M:N). Use a flat table with JOINs instead.
- Empty ARRAYs (`[]`) are valid and distinct from `NULL`. Filter accordingly.
