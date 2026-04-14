# Correlated Subqueries

## 1. Concept

A correlated subquery:
- References columns from the outer query.
- Is evaluated per outer row.

We will use:
- products (category-level comparisons)
- customers + regions (if you join them)
- orders + payments.

---

## 2. Classic pattern: product vs category average

```sql
SELECT
  p.product_id,
  p.product_name,
  p.category,
  p.price
FROM products p
WHERE p.price >
  (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category = p.category
  );
```

- Inner query uses `p.category` from outer row.
- Returns products priced above their own category’s average.

---

## 3. Customer vs all customers (total paid)

First build total paid per customer using a derived table, then correlated compare:

```sql
-- total per customer as a derived table
SELECT
  t.customer_id,
  t.total_paid
FROM (
  SELECT
    o.customer_id,
    SUM(p.amount) AS total_paid
  FROM orders o
  JOIN payments p
    ON o.order_id = p.order_id
  GROUP BY o.customer_id
) t;
```

Now “greater than average of all customers” using a correlated style:

```sql
SELECT
  t.customer_id,
  t.total_paid
FROM (
  SELECT
    o.customer_id,
    SUM(p.amount) AS total_paid
  FROM orders o
  JOIN payments p
    ON o.order_id = p.order_id
  GROUP BY o.customer_id
) t
WHERE t.total_paid >
  (
    SELECT AVG(total_paid)
    FROM (
      SELECT
        o2.customer_id,
        SUM(p2.amount) AS total_paid
      FROM orders o2
      JOIN payments p2
        ON o2.order_id = p2.order_id
      GROUP BY o2.customer_id
    ) x
  );
```

- Outer query row `t` is compared against an average computed over all customers.

---

## 4. Order value vs customer’s average order value

```sql
-- First compute order_value per order using subquery or CTE
SELECT
  o.order_id,
  o.customer_id,
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

Now use a correlated subquery to pick orders whose value > that customer’s average order value:

```sql
SELECT
  o.order_id,
  o.customer_id,
  ot.order_value
FROM orders o
JOIN (
  SELECT
    order_id,
    SUM(quantity * unit_price) AS order_value
  FROM order_items
  GROUP BY order_id
) ot
  ON o.order_id = ot.order_id
WHERE ot.order_value >
  (
    SELECT AVG(ot2.order_value)
    FROM orders o2
    JOIN (
      SELECT
        order_id,
        SUM(quantity * unit_price) AS order_value
      FROM order_items
      GROUP BY order_id
    ) ot2
      ON o2.order_id = ot2.order_id
    WHERE o2.customer_id = o.customer_id
  );
```

- Inner query uses `o.customer_id` from outer query.

---

## 5. Practice (correlated subqueries)

Write your own queries for:

1. Products whose price is **below** the average price of their category.  
2. Customers whose total number of orders is **greater than** the average number of orders per customer.  
3. Orders whose payment amount (from `payments`) is **greater than** the average payment amount for that customer.