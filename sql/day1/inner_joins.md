# INNER JOIN

---

## Basic syntax
SELECT <columns>
FROM table_a a
INNER JOIN table_b b
    ON a.key = b.key;

## Example: On your schema (customers who have placed orders):
```sql
SELECT 
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id;
```

---

## INNER JOIN + WHERE clause
```sql
-- Example: paid orders only, for customers in ‘Tamil Nadu’:
SELECT 
    c.customer_name,
    o.order_id,
    o.order_date,
    o.status
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
WHERE 
    o.status = 'PAID'
    AND c.region_id = 6;  -- Tamil Nadu
```

---

## INNER JOIN + GROUP BY (aggregation)
```sql
-- Very common pattern: join, then aggregate.
-- Example: total amount paid per customer (using payments):
SELECT 
    c.customer_id,
    c.customer_name,
    SUM(p.amount) AS total_paid
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
INNER JOIN payments p
    ON o.order_id = p.order_id
GROUP BY 
    c.customer_id,
    c.customer_name
ORDER BY total_paid DESC;
```

---

## INNER JOIN with multiple tables
```sql
-- You can chain joins; think of it as joining one table at a time.
--Example: order details with customer and product:
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id
INNER JOIN order_items oi
    ON o.order_id = oi.order_id
INNER JOIN products p
    ON oi.product_id = p.product_id;
```

---

## INNER JOIN with CTE
```sql
-- Use a CTE to pre-aggregate, then join.
-- Example: high-value orders based on order_items:
WITH order_totals AS (
    SELECT 
        oi.order_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM order_items oi
    GROUP BY oi.order_id
)
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    ot.order_total
FROM orders o
INNER JOIN order_totals ot
    ON o.order_id = ot.order_id
INNER JOIN customers c
    ON o.customer_id = c.customer_id
WHERE ot.order_total > 1000;
-- Scenario: When the raw join is messy, you calculate an intermediate result (like order_totals) and then join.
```

---

## When to use INNER JOIN


---