To find candidates with specific skills using WHERE, GROUP BY, and HAVING, you must use a query that first filters for the relevant skills, then groups the results by candidate, and finally checks that the group count matches the number of required skills. This technique can be adapted for any situation where you need to filter based on properties of a group rather than individual rows. 
Key concepts
Clause 	Purpose	Position in query	Example logic
WHERE	Filters individual rows before they are grouped. You cannot use aggregate functions like COUNT() in a WHERE clause.	Before the GROUP BY clause.	Filter for candidates who have at least one of the desired skills.
GROUP BY	Groups rows that have the same values into summary rows. It is used with aggregate functions to perform calculations on each group.	After the WHERE clause but before HAVING.	Combine all skill rows for a single candidate into one group.
HAVING	Filters groups of rows based on conditions applied to aggregate values. It works like WHERE but for aggregated data.	After the GROUP BY clause.	Check if the count of skills within each candidate's group is equal to the number of required skills.
Example: Find candidates with all 3 skills
Problem: Find all candidate IDs who have exactly the three skills: 'Python', 'Tableau', and 'PostgreSQL'. Assume you have a table named candidates with columns candidate_id and skill.
Step 1: Filter initial rows with WHERE
Start by narrowing down the dataset to only include rows related to the skills you care about. The WHERE clause is the most efficient way to do this, as it filters the data before any grouping occurs. 
sql
SELECT
  candidate_id
FROM candidates
WHERE
  skill IN ('Python', 'Tableau', 'PostgreSQL');
Use code with caution.

Step 2: Group the filtered rows with GROUP BY
To count the number of relevant skills for each candidate, you need to group the result set from Step 1 by candidate_id. 
sql
SELECT
  candidate_id,
  COUNT(skill)
FROM candidates
WHERE
  skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY
  candidate_id;
Use code with caution.

Step 3: Filter the groups with HAVING
With the GROUP BY clause in place, you can now use HAVING to filter these grouped results. You only want the candidates for whom the count of skills equals 3. 
sql
SELECT
  candidate_id
FROM candidates
WHERE
  skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY
  candidate_id
HAVING
  COUNT(skill) = 3;
Use code with caution.

Advanced example: Combining WHERE and HAVING
You can combine WHERE and HAVING in the same query for more complex filtering. 
Problem: Find departments with more than 3 employees, but only count employees whose salary is over $50,000.
Database schema
employees table: id, department, salary
Query
sql
SELECT
  department,
  COUNT(id) AS number_of_employees
FROM
  employees
WHERE
  salary > 50000 -- WHERE filters individual rows before grouping
GROUP BY
  department
HAVING
  COUNT(id) > 3; -- HAVING filters the aggregated groups
Use code with caution.

Explanation
WHERE salary > 50000: The query first filters out any employee with a salary of $50,000 or less.
GROUP BY department: It then groups the remaining high-earning employees by their department.
HAVING COUNT(id) > 3: Finally, it filters those groups, only returning departments that have more than 3 employees who meet the initial salary condition. 
AI responses may include mistakes. Learn more




undefined
undefined
undefined
13 sites
The Difference Between WHERE and HAVING in SQL
Oct 10, 2024 — The Difference Between WHERE and HAVING in SQL. Discover how WHERE filters row-level data in SQL queries, while HAVING filters grouped data after aggregation, a...
favicon
DataCamp
LinkedIn SQL Interview Question - DataLemur
We'll start by using the IN operator to find candidates which have some of the required skills: * SELECT candidate_id FROM candidates WHERE skill IN ('Python', ...
favicon
DataLemur

Use HAVING and WHERE Clauses in the Same Query
Aug 3, 2025 — In this article. ... In some instances, you might want to exclude individual rows from groups (using a WHERE clause) before applying a condition to groups as a ...
favicon
Microsoft Learn
Show all
more tricky questions asked in interview in the same patterns
Building on the WHERE, GROUP BY, and HAVING pattern, tricky SQL interview questions often involve combining these clauses with other advanced concepts like JOINs, subqueries, and conditional logic (CASE). These questions test your ability to think through multiple steps and manage data at different levels of aggregation. 
Schema for examples
The following examples will use this schema:
Employees table:
employee_id
first_name
last_name
department
salary
Projects table:
project_id
project_name
EmployeeProjects table (junction table):
employee_id
project_id
Question 1: Departments meeting multiple conditions
Find the names of all departments that have at least 10 employees and an average salary over $75,000. 
Thinking process:
You need to check conditions on both the number of employees and the average salary. Since both are aggregate functions (COUNT and AVG), you'll need GROUP BY and HAVING.
The GROUP BY column must be department to group employees by their department.
The HAVING clause will contain the two aggregate conditions, combined with AND. 
Solution:
sql
SELECT
  department
FROM Employees
GROUP BY
  department
HAVING
  COUNT(employee_id) >= 10 AND AVG(salary) > 75000;
Use code with caution.

Question 2: Find employees with ALL skills of a specific colleague
Find the names of all employees who possess every skill held by the employee named 'John Doe'.
Thinking process:
Step 1: Identify John Doe's skills. Use a subquery with WHERE to find all skills for 'John Doe'.
Step 2: Aggregate skills by employee. Use a JOIN and GROUP BY to group all the skills by employee.
Step 3: Compare skill counts. The number of skills for any candidate who meets the criteria must equal the number of skills held by 'John Doe'. This is where HAVING comes in. You'll need a separate subquery to count John Doe's skills. 
Schema: Employees (as above) and EmployeeSkills (employee_id, skill).
Solution:
sql
-- Find employees with every skill John Doe has
SELECT
  e.first_name,
  e.last_name
FROM Employees AS e
JOIN EmployeeSkills AS es
  ON e.employee_id = es.employee_id
WHERE
  -- Filter initial rows to only consider John Doe's skills
  es.skill IN (
    SELECT
      skill
    FROM Employees
    JOIN EmployeeSkills
      ON Employees.employee_id = EmployeeSkills.employee_id
    WHERE
      first_name = 'John' AND last_name = 'Doe'
  )
GROUP BY
  e.first_name,
  e.last_name
HAVING
  -- Count the skills and match against the count of John Doe's skills
  COUNT(es.skill) = (
    SELECT
      COUNT(skill)
    FROM Employees
    JOIN EmployeeSkills
      ON Employees.employee_id = EmployeeSkills.employee_id
    WHERE
      first_name = 'John' AND last_name = 'Doe'
  );
Use code with caution.

Question 3: Active employees who haven't worked on a specific project
Find all employees who are currently active (status = 'Active') and have not worked on any projects with the word 'Legacy' in their name.
Thinking process:
This problem has both a row-level filter (status = 'Active') and a group-level filter (HAVING count of legacy projects = 0).
Use a LEFT JOIN to include all employees, even those with no projects.
Filter for active employees with WHERE.
Group by employee to aggregate project information.
Use HAVING with a COUNT(CASE ...) statement to count how many 'Legacy' projects each employee has worked on. 
Schema: Employees and Projects (as above) and EmployeeProjects. Assume Employees also has a status column. 
Solution:
sql
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
  -- Count projects with 'Legacy' in the name. We want this count to be zero.
  COUNT(CASE
      WHEN p.project_name LIKE '%Legacy%' THEN 1
      ELSE NULL
  END) = 0;
Use code with caution.

Question 4: Find projects with employees from multiple departments
Find all projects that have employees from at least two different departments assigned to them. 
Thinking process:
You need to group by project_id and then count the number of distinct departments involved.
A JOIN is needed to connect employees to projects via the junction table.
Use GROUP BY on project_id and HAVING to count the distinct departments for each project. 
Schema: Employees, Projects, and EmployeeProjects (as above). 
Solution:
sql
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