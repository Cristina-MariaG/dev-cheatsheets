# SQL — Cheatsheet

> Most used queries and patterns, grouped by situation.

---

## Query a table

```sql
SELECT col1, col2 FROM table WHERE condition ORDER BY col DESC LIMIT 10;
SELECT DISTINCT col FROM table;
SELECT COUNT(*), COUNT(col), COUNT(DISTINCT col) FROM table;
```

---

## Filter

```sql
WHERE col = 'value'
WHERE col IN ('a', 'b', 'c')
WHERE col BETWEEN 10 AND 100
WHERE col LIKE 'prefix%'
WHERE col IS NULL / IS NOT NULL
WHERE col1 = 'x' AND col2 > 0
WHERE col1 = 'x' OR col2 = 'y'
```

---

## Aggregate

```sql
SELECT col, COUNT(*), SUM(amount), AVG(amount), MIN(col), MAX(col)
FROM table
GROUP BY col
HAVING COUNT(*) > 5
ORDER BY COUNT(*) DESC;
```

---

## Joins

```sql
-- Keep only matching rows
FROM a INNER JOIN b ON a.id = b.a_id

-- All from left, NULL if no match on right
FROM a LEFT JOIN b ON a.id = b.a_id

-- All rows from both, NULL where no match
FROM a FULL OUTER JOIN b ON a.id = b.a_id
```

---

## Window functions

```sql
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)
RANK()        OVER (ORDER BY score DESC)
LAG(col)      OVER (ORDER BY date)          -- previous row value
LEAD(col)     OVER (ORDER BY date)          -- next row value
SUM(amount)   OVER (ORDER BY date)          -- running total
AVG(amount)   OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)  -- moving avg
```

---

## CTE

```sql
WITH active_users AS (
  SELECT id FROM users WHERE status = 'active'
)
SELECT * FROM orders WHERE user_id IN (SELECT id FROM active_users);
```

---

## Modify data

```sql
INSERT INTO table (col1, col2) VALUES ('a', 1);
UPDATE table SET col1 = 'x' WHERE id = 1;
DELETE FROM table WHERE id = 1;

-- Upsert (PostgreSQL)
INSERT INTO table (id, col) VALUES (1, 'x')
ON CONFLICT (id) DO UPDATE SET col = EXCLUDED.col;
```

---

## Indexes

```sql
CREATE INDEX idx_name ON table (col);
CREATE INDEX idx_name ON table (col1, col2);    -- composite
CREATE UNIQUE INDEX idx_name ON table (col);
DROP INDEX idx_name;
```

---

## Debug

```sql
EXPLAIN ANALYZE SELECT ...;                     -- query plan + actual timing
SELECT * FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;  -- slow queries
```
