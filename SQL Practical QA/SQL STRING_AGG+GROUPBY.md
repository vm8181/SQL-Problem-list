# SQL Aggregations with STRING_AGG, GROUP BY, and ORDER BY

This guide demonstrates how to use `STRING_AGG` alongside other aggregate functions like `SUM`, `COUNT`, `AVG`, and `MAX` with `GROUP BY` and `ORDER BY` clauses for complex analysis and presentation.

---

## 1. `STRING_AGG` with `ORDER BY`

The `STRING_AGG` function concatenates string values from multiple rows into a single string, with a specified separator.  
The `WITHIN GROUP (ORDER BY ...)` clause defines the order of concatenation.

```sql
SELECT
    Category,
    STRING_AGG(ProductName, ', ') WITHIN GROUP (ORDER BY ProductName ASC) AS ProductList
FROM
    Products
GROUP BY
    Category;
```

👉 Groups products by category, concatenates product names alphabetically.

---

## 2. Combining `SUM`, `COUNT`, and `STRING_AGG`

Multiple aggregate functions can be combined to give a comprehensive view.

```sql
SELECT
    Department,
    COUNT(EmployeeID) AS NumberOfEmployees,
    SUM(Salary) AS TotalSalary,
    STRING_AGG(EmployeeName, '; ') WITHIN GROUP (ORDER BY EmployeeName ASC) AS EmployeeNames
FROM
    Employees
GROUP BY
    Department
ORDER BY
    TotalSalary DESC;
```

👉 Groups by department, counts employees, sums salaries, lists employee names, orders by salary total.

---

## 3. `STRING_AGG` with `DISTINCT`

To avoid duplicates, wrap values in `DISTINCT`.

```sql
SELECT
    Department,
    STRING_AGG(DISTINCT EmployeeName, ', ') WITHIN GROUP (ORDER BY EmployeeName) AS UniqueEmployeeList
FROM
    Employees
GROUP BY
    Department;
```

👉 Produces unique names per department.

---

## 4. Combining `AVG`, `MAX`, and `STRING_AGG`

```sql
SELECT
    Department,
    AVG(Salary) AS AvgSalary,
    MAX(Salary) AS HighestSalary,
    STRING_AGG(EmployeeName, ' | ') WITHIN GROUP (ORDER BY Salary DESC) AS EmployeesBySalary
FROM
    Employees
GROUP BY
    Department
ORDER BY
    AvgSalary DESC;
```

👉 Shows average, maximum, and employee names sorted by salary.

---

## 5. Grouping by Multiple Columns

```sql
SELECT
    Department,
    Role,
    COUNT(EmployeeID) AS EmployeeCount,
    STRING_AGG(EmployeeName, ', ') WITHIN GROUP (ORDER BY EmployeeName) AS Employees
FROM
    Employees
GROUP BY
    Department, Role
ORDER BY
    Department, Role;
```

👉 Breaks down employees by department and role.

---

## 6. Conditional Aggregation with `CASE`

```sql
SELECT
    Department,
    SUM(CASE WHEN Salary > 80000 THEN 1 ELSE 0 END) AS HighEarners,
    SUM(CASE WHEN Salary <= 80000 THEN 1 ELSE 0 END) AS MidOrLowEarners,
    STRING_AGG(EmployeeName, ', ') WITHIN GROUP (ORDER BY EmployeeName) AS AllEmployees
FROM
    Employees
GROUP BY
    Department;
```

👉 Counts employees in salary brackets within the same query.

---

## 7. `STRING_AGG` with Dates (Timelines)

```sql
SELECT
    EmployeeID,
    STRING_AGG(CONVERT(VARCHAR(10), ProjectDate, 120), ' → ') 
        WITHIN GROUP (ORDER BY ProjectDate ASC) AS ProjectTimeline
FROM
    EmployeeProjects
GROUP BY
    EmployeeID;
```

👉 Produces project timelines per employee (e.g., `2023-01-10 → 2023-05-15 → 2024-02-01`).

---

how to use `STRING_AGG` alongside other aggregate functions (e.g., `SUM`, `COUNT`, `AVG`, `MAX`) with `GROUP BY` and `ORDER BY` — and how to compute **top page flows** (session path analysis) such as:

```
P1-P2-P3-P2 - 200
P2-P3-P6-P2-P7 - 180
```

---

## Schema (example)

Table: `PageVisits`
- `user_id`
- `session_id`
- `visit_time` (timestamp)
- `page_id` (e.g., 'P1', 'P2', ...)

---

## A. Full-session flows (one row per session)

### 1) Basic approach (concatenate pages in visit order)
**Postgres-style**
```sql
WITH OrderedPages AS (
    SELECT
        session_id,
        string_agg(page_id, '-' ORDER BY visit_time) AS page_flow
    FROM PageVisits
    GROUP BY session_id
)
SELECT
    page_flow,
    COUNT(*) AS flow_count
FROM OrderedPages
GROUP BY page_flow
ORDER BY flow_count DESC
LIMIT 3;
```

**SQL Server / Standard SQL (WITHIN GROUP)**
```sql
WITH OrderedPages AS (
    SELECT
        session_id,
        STRING_AGG(page_id, '-') WITHIN GROUP (ORDER BY visit_time) AS page_flow
    FROM PageVisits
    GROUP BY session_id
)
SELECT
    page_flow,
    COUNT(*) AS flow_count
FROM OrderedPages
GROUP BY page_flow
ORDER BY flow_count DESC
FETCH FIRST 3 ROWS ONLY;
```

**Sample output**
```
P1-P2-P3-P2 - 200
P2-P3-P6-P2-P7 - 180
P3-P4-P5     - 120
```

---

### 2) Remove consecutive duplicate page hits (optional but often useful)
If the same `page_id` is recorded multiple times in a row (e.g., refresh), remove consecutive duplicates before concatenation:

```sql
WITH Ordered AS (
  SELECT
    session_id,
    page_id,
    visit_time,
    LAG(page_id) OVER (PARTITION BY session_id ORDER BY visit_time) AS prev_page
  FROM PageVisits
),
Filtered AS (
  SELECT
    session_id,
    page_id,
    visit_time
  FROM Ordered
  WHERE prev_page IS NULL OR page_id <> prev_page
)
,Flows AS (
  SELECT
    session_id,
    string_agg(page_id, '-' ORDER BY visit_time) AS page_flow
  FROM Filtered
  GROUP BY session_id
)
SELECT page_flow, COUNT(*) AS flow_count
FROM Flows
GROUP BY page_flow
ORDER BY flow_count DESC
LIMIT 3;
```

---

## B. n-step (sliding window) sub-flows — e.g., top 3-page sequences

Sometimes you want the top **n-grams** within sessions (e.g., P1→P2→P3, P2→P3→P6). Use `LEAD()`:

```sql
WITH Seq AS (
  SELECT
    session_id,
    page_id,
    visit_time,
    LEAD(page_id, 1) OVER (PARTITION BY session_id ORDER BY visit_time) AS p1,
    LEAD(page_id, 2) OVER (PARTITION BY session_id ORDER BY visit_time) AS p2
  FROM PageVisits
)
SELECT
  CONCAT(page_id, '-', p1, '-', p2) AS flow3,
  COUNT(*) AS flow_count
FROM Seq
WHERE p1 IS NOT NULL AND p2 IS NOT NULL
GROUP BY flow3
ORDER BY flow_count DESC
LIMIT 3;
```

**Sample output**
```
P1-P2-P3 - 600
P2-P3-P6 - 420
P3-P6-P2 - 310
```

**Notes**
- To get 4-step sub-flows, add more `LEAD()` columns.
- Consider deduping consecutive duplicates (see section A.2) before generating n-grams.

---

## C. Additional considerations & performance tips

- **Sessionization**: Ensure `session_id` is already calculated (or compute it using timeouts) so page hits from different visits aren't combined.
- **Ordering**: Always use an event timestamp (`visit_time`) to order pages; ties may need tie-breakers (e.g., an `event_id`).
- **Normalization**: Remove consecutive duplicates if you don’t want refreshes or repeated hits to skew flows.
- **Flow length**: Full session flows may be long; consider capping maximum length or using n-grams for analysis.
- **Storage & indexing**: Large datasets benefit from indexes on `(session_id, visit_time)` and partitioning by date.
- **Approximate counting**: For huge traffic volumes, consider approximate counting (HyperLogLog) or sampling before aggregation.
- **Presentation**: If you want to store flows as arrays (Postgres `array_agg`) instead of strings, it can be easier to post-process.

---

## Quick recipes

- **Top 3 full session page flows** — use `STRING_AGG` per `session_id`, then `GROUP BY` the flow and `ORDER BY COUNT DESC`.
- **Top 3 3-page sequences (n-grams)** — use `LEAD()` with `PARTITION BY session_id` and count occurrences.
- **Remove refresh/duplicate noise** — use `LAG()` to filter out consecutive duplicates before aggregating.

---

## Example: Final output format
When you run the "top full-session flows" query you can print results as:

```
P1-P2-P3-P2 - 200
P2-P3-P6-P2-P7 - 180
P3-P4-P5     - 120
```

## Explanation of Clauses

- **`SELECT`** → Columns and aggregate outputs to retrieve  
- **`FROM`** → Table(s) source  
- **`GROUP BY`** → Groups rows for aggregates  
- **`WITHIN GROUP (ORDER BY ...)`** → Defines order for `STRING_AGG` concatenation  
- **`ORDER BY`** → Orders final result set after aggregation  

---

## Tips & Best Practices

- Use `DISTINCT` with `STRING_AGG` to avoid duplicate entries.  
- Combine multiple aggregates (`SUM`, `COUNT`, `AVG`, `MAX`, `MIN`) for richer insights.  
- Use conditional `CASE` inside aggregates for flexible reporting.  
- Use `WITHIN GROUP (ORDER BY ...)` to ensure meaningful ordering of concatenated strings.  

---
