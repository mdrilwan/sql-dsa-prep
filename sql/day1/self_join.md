# SELF JOIN

---

## Concept

A self join is a join where a table is joined to itself. You use **aliases** to distinguish the two roles.

---

## Basic syntax

```sql
SELECT <columns>
FROM table_a a
JOIN table_a b
    ON <relationship between a and b>;
```

---

## Example 1: Region hierarchy (parent-child)

```sql
SELECT
    child.region_id AS child_region_id,
    child.region_name AS child_region,
    parent.region_id AS parent_region_id,
    parent.region_name AS parent_region
FROM regions child
LEFT JOIN regions parent
    ON child.parent_region_id = parent.region_id;
```

Use cases:

- Display full hierarchy: child region and its parent.
- Good base for recursive CTEs later.

---

## Example 2: Customers in the same region

```sql
-- All pairs of customers in the same region
SELECT
    c1.customer_name AS customer_1,
    c2.customer_name AS customer_2,
    r.region_name
FROM customers c1
JOIN customers c2
    ON c1.region_id = c2.region_id
   AND c1.customer_id < c2.customer_id
JOIN regions r
    ON c1.region_id = r.region_id;
```

Use case:

- Find “pairs” of related entities inside the same group (same region, same department, etc.).

---

## SELF JOIN + WHERE

```sql
-- Child regions only (those that have a parent)
SELECT
    child.region_name AS child_region,
    parent.region_name AS parent_region
FROM regions child
JOIN regions parent
    ON child.parent_region_id = parent.region_id;
```

---

## Practice for SELF JOIN

### 1. Regions with and without parents

- Output: `region_name`, `parent_region_name` (NULL if no parent).
- Use: `regions`, self join.

### 2. Customers with “same-region neighbours”

- Output: `customer_name`, `other_customer_name`, `region_name`
- Use: `customers` (self join) + `regions`.