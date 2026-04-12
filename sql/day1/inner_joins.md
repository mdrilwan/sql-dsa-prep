# INNER JOIN

---

## Basic syntax
```sql
SELECT <columns>
FROM table_a a
INNER JOIN table_b b
    ON a.key = b.key;
```

## Example: On your schema (customers who have placed orders):
```sql
SELECT
    *
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id;
```

---

## INNER JOIN with multiple tables
```sql
-- You can chain joins; think of it as joining one table at a time.
--Example: order details with customer and product:
SELECT
	*
FROM customers c
INNER JOIN orders o
	ON c.customer_id = o.customer_id
INNER JOIN order_items ot
	ON o.order_id = ot.order_id
INNER JOIN products p
	ON ot.product_id =p.product_id;
```

---

## INNER JOIN + WHERE clause
```sql
-- Example: paid orders only, for customers in ‘Tamil Nadu’:
SELECT
    *
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id 
INNER JOIN regions r
    ON c.region_id=r.region_id
WHERE r.region_name='Tamil Nadu' AND o.status='PAID';
```

---

## INNER JOIN + GROUP BY (aggregation)
```sql
-- Very common pattern: join, then aggregate.
-- Example: total amount paid per customer
SELECT
	c.customer_id,
	c.customer_name,
	SUM(p.amount)
FROM customers c
INNER JOIN orders o
	ON c.customer_id = o.customer_id
INNER JOIN payments p
	ON o.order_id = p.order_id
GROUP BY (c.customer_id, c.customer_name);
```

---

## INNER JOIN with CTE
```sql
-- Use a CTE to pre-aggregate, then join.
-- Example: high-value orders based on order_items:
WITH order_totals AS (
	SELECT
		order_id,
		SUM(quantity * unit_price) AS order_price
	FROM order_items
	GROUP BY order_id
)
SELECT
	*
FROM customers c
INNER JOIN orders o
	ON c.customer_id = o.customer_id
INNER JOIN order_totals ot
	ON o.order_id = ot.order_id
WHERE ot.order_price>1000;
-- Scenario: When the raw join is messy, you calculate an intermediate result (like order_totals) and then join.
```

---

## When to use INNER JOIN


---