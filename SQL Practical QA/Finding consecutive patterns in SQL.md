### Finding consecutive patterns in SQL often involves identifying sequences of rows that meet a specific condition or criteria. Several techniques can be employed, with the most common being: Using Window Functions (LEAD/LAG).
These functions allow access to data from preceding or succeeding rows within the same result set, enabling comparisons between adjacent rows. For example, to find three consecutive occurrences of a specific value:
```sql
    SELECT DISTINCT
        your_value
    FROM (
        SELECT
            your_value,
            LAG(your_value, 1) OVER (ORDER BY order_column) AS prev_value_1,
            LAG(your_value, 2) OVER (ORDER BY order_column) AS prev_value_2
        FROM
            your_table
    ) AS subquery
    WHERE
        your_value = prev_value_1 AND your_value = prev_value_2;
```
### Gaps and Islands Method.
This technique is useful for identifying continuous sequences (islands) separated by gaps. It typically involves calculating a "grouping" column by subtracting a row number from a sequential identifier (like a date or ID). When the difference remains constant, it indicates a consecutive sequence.
```sql
    WITH cte_gaps AS (
        SELECT
            your_column,
            your_order_column,
            your_order_column - ROW_NUMBER() OVER (ORDER BY your_order_column) AS grp
        FROM
            your_table
    ),
    cte_islands AS (
        SELECT
            your_column,
            grp,
            COUNT(*) AS consecutive_count
        FROM
            cte_gaps
        GROUP BY
            your_column, grp
        HAVING
            COUNT(*) >= N -- N is the minimum consecutive count you are looking for
    )
    SELECT
        your_column,
        consecutive_count
    FROM
        cte_islands;
```
### self-joins.
While less efficient for longer sequences, self-joins can be used to compare a row with its immediate neighbors. This involves joining a table to itself based on a condition that links consecutive rows.
```sql
    SELECT
        t1.your_value
    FROM
        your_table t1
    JOIN
        your_table t2 ON t1.order_column = t2.order_column - 1 AND t1.your_value = t2.your_value
    JOIN
        your_table t3 ON t1.order_column = t3.order_column - 2 AND t1.your_value = t3.your_value;
```

The choice of method depends on the specific pattern being sought, the database system, and performance considerations. Window functions are generally preferred for their conciseness and efficiency in handling consecutive patterns.
