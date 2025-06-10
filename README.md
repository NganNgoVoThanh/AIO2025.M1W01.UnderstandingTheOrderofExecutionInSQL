# AIO2025.M1W01.Blog

# Understanding the Order of Execution in SQL

SQL (Structured Query Language) is a powerful language used to query and manipulate data in relational database management systems. SQL statements **are written** as `SELECT … FROM … WHERE …`, but **executed** differently. Understanding this “execution order” helps you filter, group, and sort data correctly — and optimize your queries effectively.

---

## 🔄 1. Real Execution Flow (10 Steps)

| Step | Clause | Description |
|------|-------------------------|----------------------------------------------|
| **1** | `FROM` (+ `JOIN`, `ON`) | Load data sources and perform joins |
| **2** | `WHERE` | Filter individual rows |
| **3** | `GROUP BY` | Group rows into logical sets |
| **4** | `HAVING` | Filter grouped results |
| **5** | `SELECT` | Choose columns, compute expressions |
| **6** | `DISTINCT` | Remove duplicate rows |
| **7** | `UNION` / `UNION ALL` | Combine results from multiple queries |
| **8** | `ORDER BY` | Sort the final output |
| **9** | `LIMIT` / `OFFSET` | Return a subset of rows |
| **10** | `FOR UPDATE`, locking | Final step before return or lock |

> *Note*: Some databases evaluate **window functions** right after `SELECT` and before `DISTINCT`.
> A **window function** performs calculations across a set of rows (a “window”) that are related to the current row. Unlike aggregate functions (`SUM`, `AVG`, etc.) that collapse rows, window functions **retain individual rows** while adding analytical results.
> ### 🔹 Syntax
```sql
<function>(<expression>) OVER (
    PARTITION BY <column>
    ORDER BY <column>
    ROWS BETWEEN ... AND ...
)
