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

## Example 1: Region hierarchy (parent-child) without top level parent

```sql
SELECT
	r.region_id,
	r.region_name,
	r.parent_region_id,
	pr.region_name as parent_region_name
FROM regions r
INNER JOIN regions pr
	ON r.parent_region_id=pr.region_id;
```

## Example 2: Region hierarchy (parent-child) with top level parent

```sql
SELECT
	r.region_id,
	r.region_name,
	r.parent_region_id,
	pr.region_name as parent_region_name
FROM regions r
LEFT JOIN regions pr
	ON r.parent_region_id=pr.region_id;
```

Use cases:

- Display full hierarchy: child region and its parent.
- Good base for recursive CTEs later.

---

## Example 3: Customers in the same region

```sql
-- All pairs of customers in the same region
SELECT
	c1.customer_name,
	c2.customer_name,
	r.region_name
FROM customers c1
INNER JOIN customers c2
	ON c1.region_id = c2.region_id
	AND c1.customer_id < c2.customer_id
INNER JOIN regions r
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