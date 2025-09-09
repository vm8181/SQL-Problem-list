# SQL Joins Explained with Examples

This document explains different types of SQL joins (INNER, LEFT, RIGHT, FULL OUTER, NATURAL, and CROSS) using simple datasets.  
Each join type is demonstrated step by step with counts and reasoning.

---

## 📊 Example Data

We will use two tables, `TableA` and `TableB`.

### Scenario 1 – Same column name (`ID`)
```text
TableA               TableB
+------+             +------+
|  ID  |             |  ID  |
+------+             +------+
|   1  |             |   1  |
|   1  |             |   1  |
|   2  |             |   1  |
|   4  |             |   3  |
|   5  |             |   4  |
| NULL |             |   5  |
+------+             +------+
```

- TableA has **6 rows** (2 rows with `ID=1`)  
- TableB has **6 rows** (3 rows with `ID=1`)  

---

### Scenario 2 – Different column names (`ID` vs `#ID`)
```text
TableA               TableB
+------+             +-------+
|  ID  |             |  #ID  |
+------+             +-------+
|   1  |             |   1   |
|   1  |             |   1   |
|   2  |             |   1   |
|   4  |             |   3   |
|   5  |             |   4   |
| NULL |             |   5   |
+------+             +-------+
```

> ⚠️ Only the **column names differ**; values are the same as Scenario 1.  
This matters for **NATURAL JOIN**.

---

## 🔑 Join Rules Recap

- **INNER JOIN** → matched rows only (`A.ID = B.ID`)  
- **LEFT JOIN** → all rows from A + matches from B  
- **RIGHT JOIN** → all rows from B + matches from A  
- **FULL OUTER JOIN** → all rows from both (matched + unmatched)  
- **CROSS JOIN** → Cartesian product (every row of A × every row of B)  
- **NATURAL JOIN** →  
  - If columns with same name exist → works like INNER JOIN  
  - If no common columns → works like CROSS JOIN  

👉 **NULL never equals NULL** in joins.

---

## 🧮 Scenario 1 – Same Column Name (`ID`)

### INNER JOIN
```sql
SELECT * 
FROM TableA A
JOIN TableB B
  ON A.ID = B.ID;
```
- ID=1 → 2 rows in A × 3 rows in B = **6 rows**  
- ID=4 → 1 × 1 = **1 row**  
- ID=5 → 1 × 1 = **1 row**  

✅ **Total = 8 rows**

---

### LEFT JOIN
```sql
SELECT * 
FROM TableA A
LEFT JOIN TableB B
  ON A.ID = B.ID;
```
- Inner matches: 8 rows  
- Unmatched A rows: `ID=2`, `ID=NULL` → 2 rows  

✅ **Total = 10 rows**

---

### RIGHT JOIN
```sql
SELECT * 
FROM TableA A
RIGHT JOIN TableB B
  ON A.ID = B.ID;
```
- Inner matches: 8 rows  
- Unmatched B row: `ID=3` → 1 row  

✅ **Total = 9 rows**

---

### FULL OUTER JOIN
```sql
SELECT * 
FROM TableA A
FULL OUTER JOIN TableB B
  ON A.ID = B.ID;
```
- Inner matches: 8 rows  
- Unmatched A: 2 rows  
- Unmatched B: 1 row  

✅ **Total = 11 rows**

---

### NATURAL JOIN
```sql
SELECT * 
FROM TableA
NATURAL JOIN TableB;
```
- Since both tables have column `ID`, NATURAL behaves as INNER JOIN.  

✅ **Total = 8 rows**

---

### CROSS JOIN
```sql
SELECT * 
FROM TableA
CROSS JOIN TableB;
```
- 6 rows in A × 6 rows in B = 36  

✅ **Total = 36 rows**

---

## 🧮 Scenario 2 – Different Column Names (`ID` vs `#ID`)

- **INNER, LEFT, RIGHT, FULL OUTER** → behave the same as Scenario 1 (explicit `ON` condition used).  
  - INNER = 8  
  - LEFT = 10  
  - RIGHT = 9  
  - FULL OUTER = 11  

- **NATURAL JOIN** → No common column name → works like CROSS JOIN.  
  - NATURAL = CROSS = 36  

---

## 📌 Key Takeaways
- Always use explicit `ON` conditions in JOINs.  
- NATURAL JOIN can be risky → behavior changes if column names change.  
- Remember multiplicity: if A has `m` rows with a value and B has `n` rows, the join produces `m × n` rows for that value.  
- NULL never matches NULL.  
- CROSS JOIN always = `|A| × |B|`.  
