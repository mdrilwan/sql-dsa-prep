# Basic Subqueries

## 1. Concept

A subquery is a query inside another query:
- Can appear in `WHERE`, `FROM`, or `SELECT`.
- Often used for filtering and pre-aggregation.

---

## 2. Subqueries in WHERE

### 2.1 Products priced above overall average

```sql
SELECT
  product_id,
  product_name,
  category,
  price
FROM products
WHERE price >
  (
    SELECT AVG(price)
    FROM products
  );
```

- Inner query calculates overall average price.
- Outer query filters products above that value.

---

## 3. Subqueries in FROM (derived tables)

### 3.1 Order totals as derived table

```sql
SELECT
  ot.order_id,
  ot.order_value
FROM (
  SELECT
    order_id,
    SUM(quantity * unit_price) AS order_value
  FROM order_items
  GROUP BY order_id
) ot;
```

- Subquery builds an “order_totals” table on the fly.

### 3.2 Join derived table with orders

```sql
SELECT
  o.order_id,
  o.customer_id,
  o.order_date,
  ot.order_value
FROM orders o
JOIN (
  SELECT
    order_id,
    SUM(quantity * unit_price) AS order_value
  FROM order_items
  GROUP BY order_id
) ot
  ON o.order_id = ot.order_id;
```

---

## 4. Subqueries in SELECT (scalar subqueries)

### 4.1 Total amount paid per customer (scalar)

```sql
SELECT
	c.customer_id,
	c.customer_name,
	(	SELECT
			COALESCE(SUM(p.amount), 0)
		FROM orders o
		LEFT JOIN payments p
			ON o.order_id = p.order_id
		WHERE o.customer_id = c.customer_id
	) AS total_paid
FROM customers c;
```

- Subquery runs per customer row, returning a single numeric value.

### 4.2 Number of orders per customer

```sql
SELECT
	c.customer_id,
	c.customer_name,
	(	SELECT
			COUNT(*)
		FROM orders o
		WHERE o.customer_id = c.customer_id
	) AS order_count
FROM customers c;
```

---

## 5. Practice (basic subqueries)

Write queries yourself for:

1. Products whose price is **above the average price in the same category**.  
2. Customers who have placed **more than 2 orders**.  
3. For each product, show `product_id, product_name, total_quantity_sold` using a subquery in SELECT on `order_items`.