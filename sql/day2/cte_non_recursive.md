# Non-Recursive CTEs

## Concept

CTE (Common Table Expression) = named temporary result defined with `WITH`, visible only to that query.
- Improves readability vs deeply nested subqueries.
- Can be referenced multiple times. [web:328][web:335][web:332]

## Basic syntax

```sql
WITH cte_name AS (
  SELECT ...
  FROM ...
  WHERE ...
)
SELECT *
FROM cte_name;
```

## Example 1: Active users CTE

Step 1: define active users.

```sql
WITH active_users AS (
  SELECT DISTINCT user_id
  FROM events
  WHERE event_type = 'login'
)
SELECT *
FROM active_users;
```

Step 2: join CTE with orders/payments.

```sql
WITH active_users AS (
  SELECT DISTINCT user_id
  FROM events
  WHERE event_type = 'login'
)
SELECT
  a.user_id,
  o.order_id
FROM active_users a
LEFT JOIN orders o
  ON a.user_id = o.user_id;
```

## Example 2: CTE vs subquery equivalence

Using subquery:

```sql
SELECT t.user_id, t.total_events
FROM (
  SELECT user_id, COUNT(*) AS total_events
  FROM events
  GROUP BY user_id
) t
WHERE t.total_events > 5;
```

Using CTE:

```sql
WITH user_events AS (
  SELECT user_id, COUNT(*) AS total_events
  FROM events
  GROUP BY user_id
)
SELECT user_id, total_events
FROM user_events
WHERE total_events > 5;
```

- Same logic, CTE is more readable and reusable. [web:326][web:335]

## Multiple CTEs

```sql
WITH user_events AS (
  SELECT user_id, COUNT(*) AS total_events
  FROM events
  GROUP BY user_id
),
purchasers AS (
  SELECT DISTINCT user_id
  FROM events
  WHERE event_type = 'purchase'
)
SELECT
  ue.user_id,
  ue.total_events,
  CASE WHEN p.user_id IS NOT NULL THEN 1 ELSE 0 END AS is_purchaser
FROM user_events ue
LEFT JOIN purchasers p
  ON ue.user_id = p.user_id;
```

## Practice

1. CTE that gets user + total events, then select only users with total_events > 3.  
2. Two CTEs: (1) logins per user, (2) purchases per user, then join them.  
3. Replace one of your previous subquery solutions (e.g., users with login but no purchase) using a CTE for clarity.