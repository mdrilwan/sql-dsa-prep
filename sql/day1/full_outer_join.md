# FULL OUTER JOIN

---

## Basic syntax

```sql
SELECT <columns>
FROM table_a a
FULL OUTER JOIN table_b b
    ON a.key = b.key;
-- Keep all rows from both a and b, matched and unmatched
```

---

## Example: Customers and orders, including mismatches

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date
FROM customers c
FULL OUTER JOIN orders o
    ON c.customer_id = o.customer_id;
```

- Rows where both match → normal joined row.
- Customers with no orders → `order_id` / `order_date` are NULL.
- Orders with no valid customer → `customer_id` / `customer_name` are NULL.

---

## FULL OUTER JOIN + WHERE clause (finding mismatches)

```sql
-- Only mismatched rows: customers without orders OR orders without customers
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date
FROM customers c
FULL OUTER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL
   OR o.order_id IS NULL;
```

Use case: data quality checks, referential integrity audits.

---

## FULL OUTER JOIN + GROUP BY (rare but possible)

```sql
-- Count how many customers have orders vs how many orders have missing customers
SELECT
    CASE
        WHEN c.customer_id IS NULL THEN 'Orphan Order'
        WHEN o.order_id IS NULL THEN 'Customer Without Orders'
        ELSE 'Matched'
    END AS record_type,
    COUNT(*) AS record_count
FROM customers c
FULL OUTER JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY record_type;
```

---

## Practice for FULL OUTER JOIN

### 1. Customers vs payments audit

- FULL JOIN `customers`→`orders`→`payments` (step by step or via CTE).  
- Identify:
  - Customers without orders.
  - Orders without payments.

### 2. Regions vs customers

- FULL JOIN `regions` and `customers` on `region_id`.  
- Find:
  - Regions with no customers.
  - Customers assigned to non-existent regions.