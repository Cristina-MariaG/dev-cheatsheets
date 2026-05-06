# Example — Analytics Queries

> Common reporting patterns.

---

## Count orders per user

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
ORDER BY order_count DESC;
```

---

## Revenue per month

```sql
SELECT
  DATE_TRUNC('month', created_at) AS month,
  SUM(amount) AS revenue
FROM orders
GROUP BY 1
ORDER BY 1;
```

---

## Top 5 products by sales

```sql
SELECT product_name, SUM(quantity) AS total_sold
FROM order_items
GROUP BY product_name
ORDER BY total_sold DESC
LIMIT 5;
```

---

## Users who never placed an order

```sql
SELECT u.id, u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

---

## Revenue per country, only countries above 10 000

```sql
SELECT country, SUM(amount) AS revenue
FROM orders
GROUP BY country
HAVING SUM(amount) > 10000
ORDER BY revenue DESC;
```
