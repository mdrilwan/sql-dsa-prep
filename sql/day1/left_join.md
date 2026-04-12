# LEFT JOIN (LEFT OUTER JOIN)

---

## Basic syntax

```sql
SELECT <columns>
FROM table_a a
LEFT JOIN table_b b
    ON a.key = b.key;
-- Keep all rows from a, matched rows from b (NULL when no match)
```

## Example: All customers, including those with no orders

```sql
SELECT
    *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id;
```

---

## LEFT JOIN + WHERE clause

```sql
-- Example: all Tamil Nadu customers and any of their orders (if present)
SELECT
    *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN regions r
    ON c.region_id = r.region_id
WHERE r.region_name = 'Tamil Nadu';
```

> Note: Filtering on columns from the left table (customers/regions) keeps even customers without orders.  
> If you filter on right-table columns in WHERE, you may remove NULLs and effectively turn it into an inner join.

Example that **loses** customers with no orders:

```sql
SELECT
    *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.status = 'PAID';   -- This removes rows where o.* is NULL
```

Safer pattern when you truly want a left join:

```sql
SELECT
    *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
   AND o.status = 'PAID';
```

---

## LEFT JOIN + GROUP BY (aggregation)

```sql
-- Example: all customers with their total paid amount (0 if no payments)
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(SUM(p.amount), 0) AS total_paid
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN payments p
    ON o.order_id = p.order_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_paid DESC;
```

---

## LEFT JOIN with CTE

```sql
-- Example: pre-aggregate order_totals, but keep all customers
WITH order_totals AS (
    SELECT
        oi.order_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM order_items oi
    GROUP BY oi.order_id
)
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(SUM(ot.order_total), 0) AS total_order_value
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN order_totals ot
    ON o.order_id = ot.order_id
GROUP BY c.customer_id, c.customer_name;
```

---

## Practice for LEFT JOIN

### 1. Customers with and without orders

- Output: `customer_name`, `order_count`
- Use: `customers`, `orders`
- Include customers with 0 orders (order_count = 0).

### 2. Customers with paid amount or 0

- Output: `customer_name`, `total_paid`
- Use: `customers`, `orders`, `payments`
- Use LEFT JOIN and `COALESCE` so customers with no payments show `0`.

### 3. Tamil Nadu customers and their latest order date (if any)

- Output: `customer_name`, `last_order_date`
- Use: `customers`, `orders`, `regions`
- Include Tamil Nadu customers with no orders (last_order_date = NULL).