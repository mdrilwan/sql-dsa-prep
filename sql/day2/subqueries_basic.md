# Basic Subqueries

## Concept

A subquery is a query inside another query:
- Can appear in `WHERE`, `FROM`, or `SELECT`.
- Often used for filtering or pre-aggregating. [web:309][web:325]

## 1. Subquery in WHERE (filtering)

Example: users whose total events > 10.

```sql
SELECT user_id
FROM events
GROUP BY user_id
HAVING COUNT(*) > 10;
```

Example using subquery: users with more events than average.

```sql
SELECT user_id
FROM events
GROUP BY user_id
HAVING COUNT(*) >
  (
    SELECT AVG(event_count)
    FROM (
      SELECT COUNT(*) AS event_count
      FROM events
      GROUP BY user_id
    ) t
  );
```

- Inner subquery computes `event_count` per user.
- Outer HAVING compares each user to the average. [web:309]

## 2. Subquery in FROM (derived table)

Use when you want to build an intermediate result first.

```sql
SELECT
  t.user_id,
  t.total_events
FROM (
  SELECT user_id, COUNT(*) AS total_events
  FROM events
  GROUP BY user_id
) t
WHERE t.total_events > 5;
```

- The subquery acts like a temporary table `t`. [web:309]

## 3. Subquery in SELECT (scalar subquery)

Example: for each user, show their total events.

```sql
SELECT DISTINCT e.user_id,
  (
    SELECT COUNT(*)
    FROM events e2
    WHERE e2.user_id = e.user_id
  ) AS total_events
FROM events e;
```

- Subquery returns a **single value** per outer row. [web:331]

## Practice

Using your `events(user_id, event_type, event_time)`:

1. Find users whose total events > average events per user.  
2. For each user, show latest event_time (using a subquery in SELECT).  
3. Build a derived table of user + total events, then select only those above 3 events.