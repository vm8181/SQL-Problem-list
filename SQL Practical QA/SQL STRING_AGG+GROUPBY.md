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

## License

This content is provided as-is for learning, interview prep, and practice.  
