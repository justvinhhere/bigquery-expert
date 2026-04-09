# STRUCT, ARRAY, and UNNEST

**Impact:** High

## What This Covers

BigQuery's native support for nested and repeated data via STRUCT (record) and ARRAY (repeated) types, and flattening arrays with UNNEST.

## Why It Matters

Denormalized schemas with STRUCTs and ARRAYs avoid expensive JOINs, reduce slot usage, and align with BigQuery's columnar storage. Proper use of UNNEST is essential for querying nested data efficiently.

## Example

```sql
-- Define nested data
SELECT
  'order-1' AS order_id,
  STRUCT('Alice' AS name, 'alice@example.com' AS email) AS customer,
  [STRUCT('Widget' AS product, 2 AS qty, 9.99 AS price),
   STRUCT('Gadget' AS product, 1 AS qty, 24.99 AS price)] AS items;

-- Query nested fields
SELECT
  o.order_id,
  o.customer.name,
  item.product,
  item.qty * item.price AS line_total
FROM `project.dataset.orders` AS o
CROSS JOIN UNNEST(o.items) AS item;

-- LEFT JOIN UNNEST preserves rows with empty arrays
SELECT o.order_id, item.product
FROM `project.dataset.orders` AS o
LEFT JOIN UNNEST(o.items) AS item;

-- ARRAY_AGG to re-nest data
SELECT
  customer_id,
  ARRAY_AGG(STRUCT(order_id, total) ORDER BY created_at DESC) AS orders
FROM `project.dataset.orders`
GROUP BY customer_id;

-- EXISTS with UNNEST for filtering
SELECT order_id
FROM `project.dataset.orders` AS o
WHERE EXISTS (
  SELECT 1 FROM UNNEST(o.items) AS item WHERE item.product = 'Widget'
);
```

## Edge Cases / Pitfalls

- **CROSS JOIN vs LEFT JOIN UNNEST:** CROSS JOIN UNNEST drops rows where the array is empty or NULL. Use LEFT JOIN UNNEST to preserve them.
- **ARRAY ordering:** ARRAYs in BigQuery are ordered. Use `OFFSET` or `ORDINAL` for positional access: `items[OFFSET(0)]` (0-based) vs `items[ORDINAL(1)]` (1-based).
- **ARRAY subquery:** You cannot use correlated ARRAY subqueries in some contexts. Use ARRAY_AGG as an alternative.
- **Nesting depth:** BigQuery supports up to 15 levels of nested STRUCTs. Beyond that, flatten your schema.
- **NULL vs empty array:** `NULL` and `[]` behave differently. `ARRAY_LENGTH(NULL)` is NULL, `ARRAY_LENGTH([])` is 0.
