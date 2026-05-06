# Example — Common Patterns

> Recurring SQL patterns for everyday use.

---

## CTE — name a subquery for readability

```sql
WITH active_users AS (
  SELECT id, name FROM users WHERE status = 'active'
)
SELECT * FROM active_users WHERE name LIKE 'A%';
```

---

## Upsert — insert or update if already exists

```sql
-- PostgreSQL
INSERT INTO settings (user_id, theme)
VALUES (1, 'dark')
ON CONFLICT (user_id)
DO UPDATE SET theme = EXCLUDED.theme;
```

---

## Keep one row per group (deduplication)

```sql
-- Keep the most recent order per user
WITH ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
  FROM orders
)
SELECT * FROM ranked WHERE rn = 1;
```

---

## Bulk update from another table

```sql
UPDATE orders o
SET status = 'fulfilled'
FROM shipments s
WHERE o.id = s.order_id
  AND s.delivered_at IS NOT NULL;
```

---

## Conditional count in one query

```sql
SELECT
  COUNT(*) FILTER (WHERE status = 'pending')  AS pending,
  COUNT(*) FILTER (WHERE status = 'paid')     AS paid,
  COUNT(*) FILTER (WHERE status = 'cancelled') AS cancelled
FROM orders;
```
