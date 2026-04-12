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
FROM regions r
INNER JOIN customers c
	ON r.region_id = c.region_id
LEFT JOIN orders o
	ON c.customer_id = o.customer_id
WHERE r.region_name = 'Tamil Nadu';
```

---

## LEFT JOIN + GROUP BY (aggregation)

```sql
-- Example: all customers with their total paid amount (0 if no payments)
SELECT
	c.customer_id,
	c.customer_name,
	COALESCE(SUM(p.amount),0) as total_paid
FROM customers c
LEFT JOIN orders o
	ON c.customer_id = o.customer_id
LEFT JOIN payments p
	ON o.order_id = p.order_id
GROUP BY c.customer_id, c.customer_name;
```

---

## LEFT JOIN with CTE

```sql
-- Example: pre-aggregate order_totals, but keep all customers
WITH order_totals AS (
	SELECT
		order_id,
		SUM(quantity*unit_price) as total_price
	FROM order_items
	GROUP bY order_id
)
SELECT
	c.customer_id,
	c.customer_name,
	COALESCE(SUM(ot.total_price), 0)
FROM customers c
LEFT JOIN orders o
	ON c.customer_id=o.customer_id
LEFT JOIN order_totals ot
	ON o.order_id=ot.order_id
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
