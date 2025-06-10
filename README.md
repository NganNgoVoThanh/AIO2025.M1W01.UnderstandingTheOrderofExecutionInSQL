# AIO2025.M1W01.Blog

# Understanding The Order of Execution In SQL

SQL (Structured Query Language) is a powerful language used to query and manipulate data in relational database management systems. SQL statements are written as `SELECT … FROM … WHERE …`, but understanding the actual execution order helps you **filter**, **group**, and **sort data correctly — and optimize** your queries effectively.

---

## 1. Real Execution Flow

| Step | Clause | Description |
|------|-------------------------|----------------------------------------------|
| **1** | `FROM` | Identify the source table(s), including any `JOIN` operations. |
| **2** | `WHERE` | Filter rows based on the specified condition. |
| **3** | `GROUP BY` | Group rows based on one or more columns, usually for aggregation. |
| **4** | `HAVING` | Filter the resulting groups based on a condition. |
| **5** | `SELECT` | Select the columns to return, including expressions or aggregates. |
| **6** | `DISTINCT` | Remove duplicate rows |
| **7** | `UNION` / `UNION ALL` | Combine results from multiple queries |
| **8** | `ORDER BY` | Sort the results by one or more columns. |
| **9** | `LIMIT` / `OFFSET` | Return a subset of rows |

> *Note*: Some databases evaluate **window functions** right after `SELECT` and before `DISTINCT`.

---

## 2. Window Functions

A **window function** performs calculations across a set of rows (a "window") that are related to the current row. Unlike aggregate functions (`SUM`, `AVG`, etc.) that collapse rows, window functions **retain individual rows** while adding analytical results.

### 🔹 Syntax

```sql
<function>(<expression>) OVER (
    PARTITION BY <column>
    ORDER BY <column>
    ROWS BETWEEN ... AND ...
)
```

### Common Window Functions

| Category | Functions |
|----------|-----------|
| **Ranking / Ordering** | `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()` |
| **Cumulative / Moving Stats** | `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()` … with `OVER (…)` |
| **Lag & Lead Comparisons** | `LAG()`, `LEAD()`, `FIRST_VALUE()`, `LAST_VALUE()` |
| **Percentiles & Distribution** | `PERCENT_RANK()`, `CUME_DIST()` |

### Window vs. Traditional Aggregates

| Traditional Aggregate (`GROUP BY`) | Window Function |
|-----------------------------------|-----------------|
| Collapses many rows → 1 row per group | **Keeps every row** in the result set |
| Cannot show detail **and** aggregate together | Shows detail **plus** aggregated values side-by-side |
| Aggregate result unavailable in `WHERE` | Can appear directly in `SELECT`, `ORDER BY`, `HAVING` |

---

## 3. Why is the function written first but the engine reads differently?

SQL is a **declarative language**: you describe *what* result you want, not *how* to compute it.

When the database runs your query:
- It needs to **access the data first** (`FROM`)  
- Then **filter** it (`WHERE`)  
- Then **group or aggregate**,  
- Before it can **return or calculate** anything in the `SELECT`.

So even though `SELECT` is written first, the **engine executes `FROM` first** under the hood.

## 4. Example Query

### Minimal SQL Example – Whole-Kernel Yield 

This snippet shows how to calculate **% Whole** for each production line on a given day using just two arithmetic formulas and the basic SQL clauses covered in class.

### SQL Script

```sql
/* Goal: compute %Whole per line for a single production date */
SELECT
    prod_date,                       -- production date
    line_id,                         -- line identifier
    SUM(kernel_g) AS kernel_g,       -- total kernel weight (g)
    SUM(whole_g)  AS whole_g,        -- total whole-kernel weight (g)
    SUM(whole_g) / SUM(kernel_g) AS pct_whole   -- ③ %Whole
FROM (
    /* Row-level calculations (two formulas) */
    SELECT
        prod_date,
        line_id,
        -- ① kernel_g = sample_weight × (100 − shell_pct) / 100
        sample_weight * (100 - shell_pct) / 100.0          AS kernel_g,
        -- ② whole_g  = kernel_g × whole_pct / 100
        sample_weight * (100 - shell_pct) / 100.0
            * whole_pct / 100.0                            AS whole_g
    FROM quality_data
) AS t
GROUP BY
    prod_date,
    line_id
ORDER BY
    prod_date,
    line_id;
```

### Logical Execution Order

1. **FROM `quality_data`** – read the base table.  
2. **Column arithmetic** – calculate `kernel_g` and `whole_g` for each row (two formulas).  
3. The sub-query becomes temporary table **`t`**.  
4. **GROUP BY `prod_date`, `line_id`** – aggregate rows per date and line.  
5. **SELECT** – return summed grams and compute `pct_whole`.  
6. **ORDER BY** – sort the final result.

## 5. Key Takeaways

- **Memorize the logical order:** `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`.
- **Write queries for readability:** keep SQL clear and expressive; let the query optimizer decide the actual execution path.
- **Use `EXPLAIN`:** review the execution plan, then refine indexes, filters, and column lists for leaner, faster queries.
