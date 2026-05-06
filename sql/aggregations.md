# SQL — Aggregations

> Summarizing data with GROUP BY and aggregate functions.

---

## Aggregate functions

```sql
SELECT COUNT(*)           FROM orders;                 -- total rows
SELECT COUNT(email)       FROM users;                  -- non-NULL values only
SELECT COUNT(DISTINCT country) FROM users;             -- unique values
SELECT SUM(amount)        FROM orders;
SELECT AVG(amount)        FROM orders;
SELECT MIN(created_at)    FROM orders;
SELECT MAX(amount)        FROM orders;
```

> `COUNT(*)` counts all rows including NULLs. `COUNT(column)` skips NULLs.

---

## GROUP BY

```sql
-- Orders per user
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id;

-- Revenue per country per month
SELECT country, DATE_TRUNC('month', created_at) AS month, SUM(amount) AS revenue
FROM orders
GROUP BY country, DATE_TRUNC('month', created_at)
ORDER BY month DESC;
```

> Every column in SELECT must either be in GROUP BY or wrapped in an aggregate function.

---

## HAVING — filter on aggregated results

```sql
-- Users with more than 5 orders
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Countries with average order above 100
SELECT country, AVG(amount) AS avg_order
FROM orders
GROUP BY country
HAVING AVG(amount) > 100
ORDER BY avg_order DESC;
```

> `WHERE` filters rows **before** aggregation. `HAVING` filters **after**. Use WHERE when possible — it's more efficient.

---

## Execution order

```sql
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

This matters when writing queries: you can't use a SELECT alias in WHERE or HAVING (it hasn't been computed yet).

---

## Common pitfalls

- Using WHERE instead of HAVING to filter aggregated results — WHERE runs before GROUP BY and can't see aggregate values
- Selecting a column not in GROUP BY — most databases reject this (MySQL in strict mode, PostgreSQL always)
- `AVG` on integers silently truncates in some databases — cast to float: `AVG(amount::float)`
