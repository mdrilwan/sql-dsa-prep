# Non-Recursive CTEs

## 1. Concept

CTE (Common Table Expression) = named temporary result defined with `WITH`, used only in that query.

Advantages:
- Break complex queries into steps.
- Improve readability vs deeply nested subqueries.
- Allow reuse of intermediate results.

We use:
- orders, order_items, payments, customers, regions.

---

## 2. Single CTE example – order_totals

```sql
WITH order_totals AS (
  SELECT
    oi.order_id,
    SUM(oi.quantity * oi.unit_price) AS order_value
  FROM order_items oi
  GROUP BY oi.order_id
)
SELECT
  o.order_id,
  o.customer_id,
  o.order_date,
  ot.order_value
FROM orders o
JOIN order_totals ot
  ON o.order_id = ot.order_id;
```

- `order_totals` is a reusable named result set of `order_id` → `order_value`.

---

## 3. Two-step CTE – customer_totals built on top of order_totals

```sql
WITH order_totals AS (
  SELECT
    oi.order_id,
    SUM(oi.quantity * oi.unit_price) AS order_value
  FROM order_items oi
  GROUP BY oi.order_id
),
customer_totals AS (
  SELECT
    o.customer_id,
    SUM(ot.order_value) AS total_order_value
  FROM orders o
  JOIN order_totals ot
    ON o.order_id = ot.order_id
  GROUP BY o.customer_id
)
SELECT
  c.customer_id,
  c.customer_name,
  ct.total_order_value
FROM customers c
LEFT JOIN customer_totals ct
  ON c.customer_id = ct.customer_id;
```

- Step 1: `order_totals`.  
- Step 2: `customer_totals` aggregates across orders.  
- Final SELECT joins customers to their totals.

---

## 4. Region-level revenue using multiple CTEs

```sql
WITH order_totals AS (
  SELECT
    oi.order_id,
    SUM(oi.quantity * oi.unit_price) AS order_value
  FROM order_items oi
  GROUP BY oi.order_id
),
customer_totals AS (
  SELECT
    o.customer_id,
    SUM(ot.order_value) AS total_order_value
  FROM orders o
  JOIN order_totals ot
    ON o.order_id = ot.order_id
  GROUP BY o.customer_id
),
region_totals AS (
  SELECT
    r.region_id,
    r.region_name,
    SUM(ct.total_order_value) AS region_order_value
  FROM regions r
  JOIN customers c
    ON c.region_id = r.region_id
  JOIN customer_totals ct
    ON ct.customer_id = c.customer_id
  GROUP BY r.region_id, r.region_name
)
SELECT *
FROM region_totals;
```

- Chains: `order_items → orders → customers → regions`.  
- Each step is separated into its own CTE for clarity.

---

## 5. CTE vs subquery equivalence

Subquery version:

```sql
SELECT
  o.customer_id,
  SUM(oi.quantity * oi.unit_price) AS total_order_value
FROM orders o
JOIN order_items oi
  ON o.order_id = oi.order_id
GROUP BY o.customer_id;
```

CTE version:

```sql
WITH order_totals AS (
  SELECT
    order_id,
    SUM(quantity * unit_price) AS order_value
  FROM order_items
  GROUP BY order_id
)
SELECT
  o.customer_id,
  SUM(ot.order_value) AS total_order_value
FROM orders o
JOIN order_totals ot
  ON o.order_id = ot.order_id
GROUP BY o.customer_id;
```

- Same result, but the CTE separates concerns.

---

## 6. Practice (non-recursive CTEs)

Write CTE-based queries for:

1. `WITH order_totals` then select only orders with `order_value > 500`.  
2. Two CTEs: `logically_paid_orders` (orders that have a payment) and `unpaid_orders` (orders with no payment), then select from both to show counts.  
3. `WITH customer_totals` (total_paid from payments), then join with `regions` to show total_paid per region.