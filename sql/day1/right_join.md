# RIGHT JOIN (RIGHT OUTER JOIN)

---

## Basic syntax

```sql
SELECT <columns>
FROM table_a a
RIGHT JOIN table_b b
    ON a.key = b.key;
-- Keep all rows from b, matched rows from a
```

In practice, RIGHT JOIN is used less; most queries can be rewritten as a LEFT JOIN by swapping table order.

---

## Example: All orders, even if customer info is missing

```sql
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email
FROM customers c
RIGHT JOIN orders o
    ON c.customer_id = o.customer_id;
```

Equivalent LEFT JOIN version (recommended style):

```sql
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id;
```

---

## RIGHT JOIN + WHERE clause

```sql
-- All orders in January 2026, with customers if available
SELECT
    o.order_id,
    o.order_date,
    c.customer_name
FROM customers c
RIGHT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_date >= '2026-01-01'
  AND o.order_date <  '2026-02-01';
```

---

## RIGHT JOIN + GROUP BY

```sql
-- Total payments per order, including orders that might not have customer details
SELECT
    o.order_id,
    o.order_date,
    SUM(p.amount) AS total_paid
FROM customers c
RIGHT JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN payments p
    ON o.order_id = p.order_id
GROUP BY o.order_id, o.order_date;
```

---

## Practice for RIGHT JOIN

> Try to write them first as RIGHT JOIN, then rewrite as LEFT JOIN.

### 1. All orders with customer info if present

- Output: `order_id`, `order_date`, `customer_name`
- Use: `customers`, `orders`

### 2. Orders with no customer row

- Output: `order_id`, `order_date`
- Use: `customers`, `orders`
- Use RIGHT JOIN and filter to only rows where customer is NULL.