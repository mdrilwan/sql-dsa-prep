# CROSS JOIN

---

## Concept

A CROSS JOIN returns the **Cartesian product** of two tables:

- Each row from table A is combined with each row from table B.
- Row count = rows_in_A × rows_in_B.

Use with care; mainly for generating combinations or grids.

---

## Basic syntax

```sql
SELECT <columns>
FROM table_a a
CROSS JOIN table_b b;
```

---

## Example 1: All region × category combinations

```sql
SELECT
    r.region_name,
    p.category
FROM regions r
CROSS JOIN (
    SELECT DISTINCT category FROM products
) p;
```

Use case:

- Build a “matrix” of region and category to later left join actual metrics (sales, counts) onto it.

---

## Example 2: Generating a reference grid for reporting

```sql
-- Suppose you want every (region, payment_method) combination
SELECT
    r.region_name,
    pm.payment_method
FROM regions r
CROSS JOIN (
    SELECT DISTINCT payment_method FROM payments
) pm;
```

Later you can LEFT JOIN this grid with aggregated data to ensure you show 0s instead of missing rows.

---

## CROSS JOIN + WHERE

You usually filter either before or after the cross join via subqueries.

```sql
SELECT
    r.region_name,
    p.category
FROM (SELECT * FROM regions WHERE parent_region_id = 1) r   -- India subregions
CROSS JOIN (
    SELECT DISTINCT category FROM products
) p;
```

---

## Practice for CROSS JOIN

### 1. Region × product category matrix

- Build all (region_name, category) combos.
- Then LEFT JOIN with aggregated revenue by region & category.

### 2. Region × payment_method matrix

- Build all (region, payment_method) combos.
- Then LEFT JOIN with total payments to show 0 where no payments exist.