# SQL — Joins

> Combining rows from multiple tables.

---

## Join types

```sql
-- INNER JOIN — only rows that match in both tables
SELECT u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN — all rows from left, matched rows from right (NULL if no match)
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN — all rows from right, matched rows from left (rare — rewrite as LEFT JOIN)
SELECT u.name, o.amount
FROM orders o
RIGHT JOIN users u ON u.id = o.user_id;

-- FULL OUTER JOIN — all rows from both tables (NULL where no match)
SELECT u.name, o.amount
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN — every combination of rows (cartesian product)
SELECT u.name, p.name
FROM users u
CROSS JOIN products p;
```

---

## Visual reference

```
LEFT JOIN          INNER JOIN         FULL OUTER JOIN
┌───┬───┐         ┌───┬───┐          ┌───┬───┐
│ A │   │         │   │ B │          │ A │ B │
│ A │ B │         │ A │ B │          │ A │ B │
│   │ B │         └───┴───┘          │   │ B │
└───┴───┘                            └───┴───┘
```

---

## Self-join

Join a table to itself — useful for hierarchies or comparing rows within the same table.

```sql
-- Find employees and their manager (same table)
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

## Multiple joins

```sql
SELECT u.name, o.amount, p.name AS product
FROM users u
JOIN orders o      ON u.id = o.user_id
JOIN order_items i ON o.id = i.order_id
JOIN products p    ON i.product_id = p.id
WHERE o.status = 'paid';
```

---

## Common pitfalls

- **Missing join condition** — a forgotten `ON` clause produces a cartesian product (every row × every row)
- **Ambiguous columns** — always qualify with table alias when joining (`u.id` not just `id`)
- **LEFT JOIN + WHERE on right table** — filtering on a nullable column from the right table turns a LEFT JOIN into an INNER JOIN

```sql
-- Bug: WHERE cancels the LEFT JOIN effect
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'paid';   -- users with no orders are excluded

-- Fix: move the condition to ON
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'paid';
```
