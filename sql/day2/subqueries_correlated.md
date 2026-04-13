# Day 2 – Correlated Subqueries

## Concept

A correlated subquery:
- Uses columns from the outer query.
- Runs once per row of the outer query. [web:331][web:333]

## Example pattern

```sql
SELECT ...
FROM outer_table o
WHERE some_column >
  (
    SELECT AVG(x)
    FROM inner_table i
    WHERE i.key = o.key
  );
```

## Example: events – user with above-average events

```sql
SELECT e1.user_id
FROM events e1
GROUP BY e1.user_id
HAVING COUNT(*) >
  (
    SELECT AVG(user_event_count)
    FROM (
      SELECT e2.user_id, COUNT(*) AS user_event_count
      FROM events e2
      GROUP BY e2.user_id
    ) t
  );
```

More classic example: salary greater than department average. [web:331]

```sql
SELECT
  e1.employee_id,
  e1.salary,
  e1.department_id
FROM employees e1
WHERE e1.salary >
  (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
  );
```

## When to use

- Row-by-row comparisons (per user, per department, per region).  
- Filtering based on a value calculated within each group context. [web:333][web:336]

## Practice

1. For each user, select them only if their event count is greater than the average count of users in the same region (assuming `user_region` table).  
2. Using your `events`, pick users whose number of `purchase` events is greater than their number of `login` events (subquery inside WHERE that counts per user).