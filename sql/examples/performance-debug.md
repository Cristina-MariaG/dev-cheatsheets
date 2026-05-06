# Example — Performance Debug

> Finding and fixing slow queries.

---

## Step 1 — Read the query plan

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1;
```

Look for `Seq Scan` on large tables — it means no index is used.

---

## Step 2 — Add a missing index

```sql
-- Before: Seq Scan (slow)
SELECT * FROM orders WHERE user_id = 1;

-- Add index
CREATE INDEX idx_orders_user_id ON orders (user_id);

-- After: Index Scan (fast)
```

---

## Step 3 — Avoid functions on filtered columns

```sql
-- Slow — index on created_at not used
SELECT * FROM orders WHERE YEAR(created_at) = 2024;

-- Fast — rewrite as a range
SELECT * FROM orders
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

---

## Step 4 — Fix slow pagination

```sql
-- Slow on large tables — scans and discards 10000 rows
SELECT * FROM orders ORDER BY id DESC LIMIT 10 OFFSET 10000;

-- Fast — start from the last seen ID
SELECT * FROM orders WHERE id < :last_seen_id ORDER BY id DESC LIMIT 10;
```

---

## Step 5 — Update statistics if estimates are wrong

```sql
ANALYZE orders;   -- refreshes row count estimates used by the query planner
```
