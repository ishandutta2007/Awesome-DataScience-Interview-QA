# 🗃️ SQL

[← Back to main README](../README.md)

---

### Q: What is the difference between `WHERE` and `HAVING`?

**Answer:**
`WHERE` filters individual rows **before** grouping/aggregation happens, and cannot reference aggregate functions. `HAVING` filters **after** `GROUP BY` has produced aggregated results, and is used specifically to filter on aggregate conditions (e.g., `HAVING COUNT(*) > 5`). If you don't need to filter on an aggregate, use `WHERE` — it's more efficient since it reduces rows before the (often expensive) grouping step.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
WHERE order_date >= '2026-01-01'
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

---

### Q: Explain the difference between `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL OUTER JOIN`.

**Answer:**
- **INNER JOIN**: returns only rows with matches in both tables.
- **LEFT JOIN**: returns all rows from the left table, with `NULL`s for unmatched right-table columns.
- **RIGHT JOIN**: mirror of LEFT JOIN, keeps all rows from the right table.
- **FULL OUTER JOIN**: returns all rows from both tables, with `NULL`s where there's no match on either side.

A common gotcha: putting a filter on the right table in the `WHERE` clause after a `LEFT JOIN` silently turns it into an `INNER JOIN` — the filter should go in the `ON` clause if you want to preserve unmatched left rows.

---

### Q: What is a window function, and how does it differ from a `GROUP BY` aggregation?

**Answer:**
`GROUP BY` collapses multiple rows into one row per group, losing row-level detail. A **window function** computes an aggregate (or ranking) **across a set of rows related to the current row (a "window")** while still returning one output row per input row — using `OVER (PARTITION BY ... ORDER BY ...)`. This makes it possible to, e.g., compute each employee's salary alongside their department average, without collapsing the individual rows.

```sql
SELECT employee_id, department, salary,
       AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

---

### Q: Write a SQL query to find the second-highest salary in an `employees` table.

**Answer:**
Several valid approaches; the most robust handles ties/gaps correctly using `DENSE_RANK`:

```sql
SELECT salary AS second_highest_salary
FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM employees
) ranked
WHERE rnk = 2;
```

A simpler (but ties-naive) alternative: `SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);` — this correctly handles duplicate max salaries but is less generalizable to "Nth highest."

---

### Q: What's the difference between `UNION` and `UNION ALL`?

**Answer:**
`UNION` combines result sets from two queries and **removes duplicate rows**, which requires an implicit sort/hash step and is therefore slower. `UNION ALL` simply concatenates result sets **without deduplication**, making it faster. Use `UNION ALL` whenever you know there won't be duplicates or don't care about them — it's a common performance win.

---

### Q: Explain the difference between a clustered and a non-clustered index.

**Answer:**
A **clustered index** determines the **physical order** in which data rows are stored on disk — a table can have only one (since rows can only be physically ordered one way). A **non-clustered index** is a separate structure that stores pointers back to the actual rows, with the table's data remaining in its original order — a table can have many. Clustered indexes are typically fastest for range queries on the indexed column; non-clustered indexes add overhead per write (since the pointer structure must also be updated) but allow efficient lookups on other columns.

---

### Q: Write a query to find duplicate rows in a table based on a set of columns.

**Answer:**
```sql
SELECT email, COUNT(*) AS occurrences
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

To retrieve the actual duplicate *rows* (not just the count) including a way to delete extras while keeping one:

```sql
WITH duplicates AS (
  SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
  FROM users
)
DELETE FROM users WHERE id IN (SELECT id FROM duplicates WHERE rn > 1);
```

---

### Q: What is a CTE (Common Table Expression), and how does it differ from a subquery?

**Answer:**
A CTE (`WITH cte_name AS (...)`) is a named, temporary result set defined before the main query, improving readability and enabling **recursion** (which subqueries can't do). Functionally, a non-recursive CTE is often equivalent to an inline subquery — the query planner may or may not materialize it depending on the database engine — but CTEs are preferred for complex queries because they can be **referenced multiple times** and broken into readable logical steps, unlike deeply nested subqueries.

---

### Q: How would you find customers who placed orders in every month of 2025 (a "gaps and islands" style problem)?

**Answer:**
```sql
SELECT customer_id
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2025
GROUP BY customer_id
HAVING COUNT(DISTINCT EXTRACT(MONTH FROM order_date)) = 12;
```

This counts *distinct* months a customer ordered in during 2025 and filters to those with exactly 12 — a common pattern for "did X happen in every period" questions.

---

### Q: What is the difference between `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`?

**Answer:**
All three assign a sequential rank within a window (`OVER (ORDER BY ...)`), but differ on ties: **`ROW_NUMBER()`** always assigns unique, sequential integers even for tied values (arbitrary tie-break). **`RANK()`** assigns the same rank to ties but **skips** subsequent rank numbers (e.g., 1, 2, 2, 4). **`DENSE_RANK()`** assigns the same rank to ties **without gaps** (e.g., 1, 2, 2, 3). Choose based on whether downstream logic needs no-gaps (`DENSE_RANK`) or explicit gap accounting for tie groups (`RANK`).

---

### Q: How do you optimize a slow SQL query? Walk through your general approach.

**Answer:**
1. **`EXPLAIN`/`EXPLAIN ANALYZE`** the query to see the actual execution plan — look for full table scans where an index scan should be possible.
2. Check **indexes** on columns used in `JOIN`, `WHERE`, and `ORDER BY` clauses; add composite indexes matching common filter/sort patterns.
3. Avoid `SELECT *` — only pull needed columns to reduce I/O.
4. Rewrite correlated subqueries as `JOIN`s or window functions where possible, since correlated subqueries often re-execute per row.
5. Check for implicit type conversions in `WHERE` clauses that prevent index usage.
6. For very large tables, consider **partitioning** and pre-aggregated/materialized views for repeated heavy queries.

---

### Q: Write a query to compute a running total (cumulative sum) of daily revenue.

**Answer:**
```sql
SELECT order_date,
       daily_revenue,
       SUM(daily_revenue) OVER (ORDER BY order_date
                                 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM daily_sales;
```

---

### Q: What is a self-join, and give a practical use case.

**Answer:**
A self-join joins a table to itself (using aliases to distinguish the two references), useful for comparing rows within the same table — e.g., finding employees who earn more than their manager:

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

### Q: Explain the difference between `NULL`-handling in `COUNT(*)` vs. `COUNT(column_name)`.

**Answer:**
`COUNT(*)` counts **all rows**, regardless of `NULL` values in any column. `COUNT(column_name)` counts only rows where that **specific column is non-NULL**. This distinction commonly trips people up when computing rates (e.g., "% of users who provided an email") — you'd use `COUNT(email) / COUNT(*)`, not `COUNT(*) / COUNT(*)`.

---

### Q: How would you pivot rows into columns in SQL (e.g., turning monthly sales rows into one row per product with a column per month)?

**Answer:**
Standard SQL approach uses conditional aggregation:

```sql
SELECT product_id,
       SUM(CASE WHEN month = 1 THEN revenue ELSE 0 END) AS jan_revenue,
       SUM(CASE WHEN month = 2 THEN revenue ELSE 0 END) AS feb_revenue
FROM sales
GROUP BY product_id;
```

Some engines (SQL Server, Snowflake) also support a native `PIVOT` operator, but the `CASE WHEN` + aggregation pattern is portable across databases and often more transparent for interviewers to follow.

---
