# SQL — Query Optimization

> Understanding and fixing slow queries.

---

## EXPLAIN — read the query plan

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 1;
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1;   -- actually runs the query
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;                    -- also shows cache hits
```

### Key nodes to recognize

| Node | What it means |
|---|---|
| `Seq Scan` | Full table scan — no index used |
| `Index Scan` | Uses an index, fetches rows from table |
| `Index Only Scan` | Uses a covering index — no table access |
| `Nested Loop` | For each row in outer, scan inner — fast for small sets |
| `Hash Join` | Builds a hash table — good for larger sets |
| `Merge Join` | Both inputs sorted — efficient for large sorted sets |
| `Sort` | Explicit sort — expensive without an index |

### Reading costs

```
Seq Scan on orders (cost=0.00..1842.00 rows=50000 width=32)
                          ↑ startup    ↑ total
```

Cost is an arbitrary unit. What matters is relative difference — a cost of 10 vs 10000 is significant.

---

## Common slow patterns and fixes

### 1. Function on a filtered column

```sql
-- Slow — can't use index on created_at
WHERE DATE(created_at) = '2024-01-01'

-- Fast — range query uses the index
WHERE created_at >= '2024-01-01' AND created_at < '2024-01-02'
```

### 2. N+1 queries — fetch once, not per row

```sql
-- Bad — one query per order (N+1)
SELECT * FROM orders;
-- then for each order: SELECT * FROM users WHERE id = ?

-- Good — one JOIN
SELECT o.*, u.name FROM orders o JOIN users u ON o.user_id = u.id;
```

### 3. SELECT * — fetch only what you need

```sql
-- Bad — transfers all columns including large blobs
SELECT * FROM products;

-- Good
SELECT id, name, price FROM products;
```

### 4. Subquery vs JOIN

```sql
-- Slower — correlated subquery runs once per row
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- Faster — JOIN evaluated once
SELECT DISTINCT u.* FROM users u JOIN orders o ON u.id = o.user_id WHERE o.amount > 100;
```

### 5. Large OFFSET pagination

```sql
-- Slow — must scan and discard 100000 rows
SELECT * FROM orders ORDER BY id LIMIT 10 OFFSET 100000;

-- Fast — keyset pagination (cursor-based)
SELECT * FROM orders WHERE id > 100000 ORDER BY id LIMIT 10;
```

---

## Useful diagnostic queries (PostgreSQL)

```sql
-- Slow queries log
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Unused indexes
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0;

-- Table sizes
SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::regclass))
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC;
```

---

## Common pitfalls

- Optimizing before measuring — always EXPLAIN ANALYZE first, don't guess
- Adding indexes blindly — verify the query plan actually uses them after creation
- Ignoring `Seq Scan` on large tables — almost always a sign of a missing or unused index
