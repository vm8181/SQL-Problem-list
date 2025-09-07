# SQL: Using `WHERE`, `GROUP BY`, and `HAVING` for row- vs group-level filtering

A concise guide and example collection showing how to filter rows **before** aggregation (`WHERE`) and how to filter **after** aggregation (`HAVING`). Useful for interview prep and real-world queries where you must apply conditions to groups (e.g., “candidates who have *all* required skills”, departments meeting aggregated criteria, projects involving multiple departments, etc.).

---

## Key concepts

| Clause | Purpose | Position in query | Example logic |
|---|---:|---|---|
| `WHERE` | Filters individual rows before they are grouped. Cannot use aggregate functions (e.g., `COUNT()`) here. | Before `GROUP BY` | Filter for rows with any of the desired skills. |
| `GROUP BY` | Groups rows with same values into summary rows; used with aggregate functions. | After `WHERE`, before `HAVING` | Combine all skill rows for a single candidate into one group. |
| `HAVING` | Filters groups based on aggregate conditions. Works like `WHERE` but for aggregated data. | After `GROUP BY` | Keep candidate groups whose skill count equals the number of required skills. |

---

## Example: Find candidates with **all 3** skills

**Problem:** Find all `candidate_id`s who have exactly the three skills `'Python'`, `'Tableau'`, and `'PostgreSQL'`. Table: `candidates(candidate_id, skill)`.

### Step 1 — Filter initial rows with `WHERE`
Filter rows to only those skills we care about (efficient; done before grouping).

```sql
SELECT
  candidate_id
FROM candidates
WHERE
  skill IN ('Python', 'Tableau', 'PostgreSQL');
```

### Step 2 — Group filtered rows with `GROUP BY`
Count relevant skills per candidate.

```sql
SELECT
  candidate_id,
  COUNT(skill) AS skill_count
FROM candidates
WHERE
  skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY
  candidate_id;
```

### Step 3 — Filter groups with `HAVING`
Only keep candidates whose count of those skills equals `3`.

```sql
SELECT
  candidate_id
FROM candidates
WHERE
  skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY
  candidate_id
HAVING
  COUNT(skill) = 3;
```

---

## Advanced example: `WHERE` + `HAVING` together

**Problem:** Find departments with more than 3 employees, but only count employees whose salary is over $50,000. Table: `employees(id, department, salary)`.

```sql
SELECT
  department,
  COUNT(id) AS number_of_employees
FROM
  employees
WHERE
  salary > 50000           -- filters individual rows first
GROUP BY
  department
HAVING
  COUNT(id) > 3;           -- filters aggregated groups
```

**Explanation**
- `WHERE salary > 50000`: exclude lower-salary employees before grouping.
- `GROUP BY department`: group remaining rows by department.
- `HAVING COUNT(id) > 3`: keep only departments with more than 3 high-earning employees.

---

## Interview-style & tricky examples (with thinking process and solutions)

> **Schema used across examples**
- `Employees(employee_id, first_name, last_name, department, salary, status)`
- `Projects(project_id, project_name)`
- `EmployeeProjects(employee_id, project_id)` — junction table
- `EmployeeSkills(employee_id, skill)` (used where applicable)

### Q1 — Departments that meet multiple aggregate conditions
**Find departments with at least 10 employees and average salary > $75,000.**

```sql
SELECT
  department
FROM Employees
GROUP BY
  department
HAVING
  COUNT(employee_id) >= 10
  AND AVG(salary) > 75000;
```

---

### Q2 — Employees who have **all skills** of a specific colleague
**Find employees who possess every skill held by `John Doe`.**

```sql
-- Find employees with every skill John Doe has
SELECT
  e.first_name,
  e.last_name
FROM Employees AS e
JOIN EmployeeSkills AS es
  ON e.employee_id = es.employee_id
WHERE
  es.skill IN (
    SELECT skill
    FROM Employees
    JOIN EmployeeSkills
      ON Employees.employee_id = EmployeeSkills.employee_id
    WHERE first_name = 'John' AND last_name = 'Doe'
  )
GROUP BY
  e.first_name,
  e.last_name
HAVING
  COUNT(es.skill) = (
    SELECT COUNT(skill)
    FROM Employees
    JOIN EmployeeSkills
      ON Employees.employee_id = EmployeeSkills.employee_id
    WHERE first_name = 'John' AND last_name = 'Doe'
  );
```

---

### Q3 — Active employees who **haven’t** worked on a "Legacy" project
**Find active employees (`status = 'Active'`) who have not worked on any project with `'Legacy'` in the name.**

```sql
SELECT
  e.employee_id,
  e.first_name,
  e.last_name
FROM Employees AS e
LEFT JOIN EmployeeProjects AS ep
  ON e.employee_id = ep.employee_id
LEFT JOIN Projects AS p
  ON ep.project_id = p.project_id
WHERE
  e.status = 'Active'
GROUP BY
  e.employee_id,
  e.first_name,
  e.last_name
HAVING
  COUNT(CASE WHEN p.project_name LIKE '%Legacy%' THEN 1 END) = 0;
```

---

### Q4 — Projects involving employees from **multiple departments**
**Find projects that have employees from at least two different departments.**

```sql
SELECT
  p.project_name
FROM Projects AS p
JOIN EmployeeProjects AS ep
  ON p.project_id = ep.project_id
JOIN Employees AS e
  ON ep.employee_id = e.employee_id
GROUP BY
  p.project_name
HAVING
  COUNT(DISTINCT e.department) >= 2;
```

---

## Tips & best practices
- Use `WHERE` whenever you can to reduce rows before aggregation — it’s more efficient.
- Use `HAVING` only when your condition depends on aggregated values (e.g., `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`) or when you need to filter groups after `GROUP BY`.
- You can combine `WHERE` and `HAVING` in a single query to apply both row-level and group-level filters.
- For existence/non-existence checks, consider using `LEFT JOIN` + `HAVING` or `NOT EXISTS` subqueries depending on readability and performance needs.
- Beware of duplicates — `COUNT(*)` vs `COUNT(column)` vs `COUNT(DISTINCT column)` behave differently.

---

## Common interview variations & challenges
- Use `HAVING` with multiple aggregated conditions (e.g., `COUNT(...) >= x AND AVG(...) > y`).
- Nested queries that count a reference set (e.g., “employees who have all skills of another employee”).
- Combining `JOIN`s, window functions, CTEs, and `HAVING` for more complex analytics.
- Handling NULLs and duplicates appropriately while aggregating.

---

## Quick reference — where to use what
- Row filters that reference non-aggregated columns → `WHERE`.
- Grouping and aggregating → `GROUP BY`.
- Filters that reference aggregates → `HAVING`.

---

## License
This content is provided as-is for study, interview prep, and learning. Feel free to reuse and adapt.  
