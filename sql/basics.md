# SQL — Basics

> Core statements for querying and modifying data.

---

## SELECT

```sql
SELECT *                          FROM users;
SELECT id, name, email            FROM users;
SELECT DISTINCT country           FROM users;           -- deduplicate results
SELECT name, age * 12 AS age_months FROM users;        -- computed column with alias
```

---

## Filtering — WHERE

```sql
SELECT * FROM orders WHERE status = 'active';
SELECT * FROM orders WHERE amount > 100 AND status = 'paid';
SELECT * FROM orders WHERE status IN ('pending', 'active');
SELECT * FROM orders WHERE status NOT IN ('cancelled');
SELECT * FROM users  WHERE name LIKE 'A%';             -- starts with A
SELECT * FROM users  WHERE name ILIKE 'a%';            -- case-insensitive (PostgreSQL)
SELECT * FROM users  WHERE email IS NULL;
SELECT * FROM users  WHERE age BETWEEN 18 AND 65;
```

---

## Sorting & limiting

```sql
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY last_name ASC, first_name ASC;
SELECT * FROM users LIMIT 10;
SELECT * FROM users LIMIT 10 OFFSET 20;               -- page 3 of 10 results
```

---

## INSERT

```sql
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- Insert multiple rows
INSERT INTO users (name, email) VALUES
  ('Alice', 'alice@example.com'),
  ('Bob',   'bob@example.com');

-- Insert from a query
INSERT INTO archive SELECT * FROM orders WHERE status = 'closed';
```

---

## UPDATE

```sql
UPDATE users SET status = 'inactive' WHERE last_login < '2024-01-01';
UPDATE users SET name = 'Alice', email = 'alice@new.com' WHERE id = 1;
```

> **Pitfall:** Always include a WHERE clause. `UPDATE users SET status = 'inactive'` updates every row.

---

## DELETE

```sql
DELETE FROM users WHERE id = 1;
DELETE FROM sessions WHERE expires_at < NOW();
```

> **Pitfall:** Same as UPDATE — a DELETE without WHERE wipes the entire table. Test with SELECT first using the same WHERE clause.

---

## Common pitfalls

- `NULL` comparisons — `WHERE email = NULL` never matches. Use `IS NULL` / `IS NOT NULL`
- `LIKE` is case-sensitive in most databases (except PostgreSQL's `ILIKE`)
- `LIMIT` without `ORDER BY` returns rows in undefined order — results are non-deterministic
