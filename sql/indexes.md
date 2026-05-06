# SQL — Indexes

> Data structures that speed up queries by avoiding full table scans.

---

## How indexes work

Without an index, the database scans every row to find matches (full table scan). An index is a separate data structure (usually a B-tree) that maps column values to row locations — like a book index.

Trade-off: indexes speed up reads but slow down writes (INSERT/UPDATE/DELETE must update the index too) and consume disk space.

---

## Creating indexes

```sql
CREATE INDEX idx_users_email ON users (email);                    -- single column
CREATE INDEX idx_orders_user_date ON orders (user_id, created_at); -- composite
CREATE UNIQUE INDEX idx_users_email_unique ON users (email);      -- enforce uniqueness
CREATE INDEX idx_users_email_lower ON users (LOWER(email));       -- expression index
```

```sql
DROP INDEX idx_users_email;
```

---

## Index types (PostgreSQL)

| Type | Use case |
|---|---|
| `B-tree` | Default — equality and range queries (`=`, `<`, `>`, `BETWEEN`, `LIKE 'abc%'`) |
| `Hash` | Equality only (`=`) — faster than B-tree for pure equality |
| `GIN` | Full-text search, arrays, JSONB |
| `GiST` | Geometric types, full-text search |
| `BRIN` | Very large tables with naturally ordered data (timestamps, sequential IDs) |

---

## Composite indexes — column order matters

```sql
CREATE INDEX idx_orders_user_status ON orders (user_id, status);
```

This index is used for:
- `WHERE user_id = 1`
- `WHERE user_id = 1 AND status = 'paid'`

But **not** for:
- `WHERE status = 'paid'` (leading column missing)

> Put the most selective column first, or the column used in equality conditions before range conditions.

---

## Covering indexes

An index that contains all columns needed by a query — no need to access the table at all.

```sql
CREATE INDEX idx_orders_covering ON orders (user_id, status, amount);

-- This query is served entirely from the index
SELECT status, amount FROM orders WHERE user_id = 1;
```

---

## When indexes are NOT used

```sql
-- Function on indexed column — index on email won't be used
WHERE LOWER(email) = 'alice@example.com'
-- Fix: create an expression index
CREATE INDEX ON users (LOWER(email));

-- Leading wildcard — can't use B-tree index
WHERE name LIKE '%alice%'
-- Fix: use full-text search (GIN index + tsvector)

-- OR condition across different columns — may skip indexes
WHERE email = 'x' OR phone = 'y'
-- Fix: rewrite as UNION

-- Implicit type cast — index on integer id won't be used if comparing to a string
WHERE id = '123'
```

---

## Common pitfalls

- Over-indexing — every index slows down writes. Index columns used in WHERE, JOIN, and ORDER BY, not every column
- Unused indexes — check with `pg_stat_user_indexes` in PostgreSQL and drop indexes with 0 scans
- Forgetting that `NULL` values are indexed in PostgreSQL (but not in some other databases)
