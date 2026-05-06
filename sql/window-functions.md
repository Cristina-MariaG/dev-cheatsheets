# SQL — Window Functions

> Perform calculations across a set of rows related to the current row, without collapsing them into a single result like GROUP BY does.

---

## Syntax

```sql
function() OVER (
  PARTITION BY <column>   -- divide rows into groups (optional)
  ORDER BY <column>       -- order within each group
  ROWS/RANGE ...          -- frame definition (optional)
)
```

---

## Ranking functions

```sql
SELECT
  name,
  salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,   -- unique rank, no ties
  RANK()       OVER (ORDER BY salary DESC) AS rank,      -- ties share rank, next rank skipped
  DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank -- ties share rank, no gap after
FROM employees;
```

| name  | salary | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|---|
| Alice | 9000   | 1          | 1    | 1          |
| Bob   | 8000   | 2          | 2    | 2          |
| Carol | 8000   | 3          | 2    | 2          |
| Dave  | 7000   | 4          | 4    | 3          |

---

## PARTITION BY — reset per group

```sql
-- Rank employees within each department
SELECT
  name,
  department,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

---

## LAG / LEAD — access adjacent rows

```sql
-- Compare each month's revenue to the previous month
SELECT
  month,
  revenue,
  LAG(revenue)  OVER (ORDER BY month) AS prev_month,
  LEAD(revenue) OVER (ORDER BY month) AS next_month,
  revenue - LAG(revenue) OVER (ORDER BY month) AS growth
FROM monthly_revenue;
```

---

## Running totals and moving averages

```sql
-- Running total
SELECT
  created_at,
  amount,
  SUM(amount) OVER (ORDER BY created_at) AS running_total
FROM orders;

-- 7-day moving average
SELECT
  day,
  revenue,
  AVG(revenue) OVER (ORDER BY day ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7d
FROM daily_revenue;
```

---

## FIRST_VALUE / LAST_VALUE / NTH_VALUE

```sql
-- Highest salary in each department, shown on every row
SELECT
  name,
  department,
  salary,
  FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) AS top_salary
FROM employees;
```

---

## Common pitfalls

- Window functions cannot be used in WHERE or HAVING — wrap in a subquery or CTE
- `LAST_VALUE` needs an explicit frame (`ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`) — the default frame stops at the current row, making LAST_VALUE return the current value
- `RANK()` skips numbers after ties — use `DENSE_RANK()` if you need consecutive ranks
