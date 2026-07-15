# 06 — Subqueries & CTEs

Genuinely essential once a question is too complex for a single SELECT/JOIN — subqueries and CTEs let queries be built out of OTHER queries, breaking complex logic into manageable pieces.

## Subqueries in WHERE — the most common use case

```sql
-- Find students older than the AVERAGE age
SELECT name, age
FROM students
WHERE age > (SELECT AVG(age) FROM students);
```

The inner query `(SELECT AVG(age) FROM students)` runs FIRST, produces a single value, and the outer query uses that value in its `WHERE` condition — genuinely just "a query whose result feeds into another query," nested directly inline.

## Subqueries with IN — matching against a SET of values

```sql
-- Find students enrolled in ANY course (recap file 05's anti-join, this is the flip side)
SELECT name FROM students
WHERE id IN (SELECT student_id FROM enrollments);

-- Find students NOT enrolled in any course
SELECT name FROM students
WHERE id NOT IN (SELECT student_id FROM enrollments);
```

Genuinely a real, common alternative to file 05's JOIN-based approach for these exact same questions — worth knowing subqueries and joins can often solve the SAME problem, and file 11 (indexing/optimization) touches on WHICH approach a database engine tends to handle more efficiently in practice.

## Correlated subqueries — the inner query DEPENDS on the outer query

```sql
-- Find students who are older than the AVERAGE age WITHIN THEIR OWN MAJOR
SELECT s1.name, s1.age, s1.major
FROM students s1
WHERE s1.age > (
    SELECT AVG(s2.age) FROM students s2 WHERE s2.major = s1.major
);
```

Genuinely important distinction from the first example: this inner query REFERENCES `s1.major` from the OUTER query — meaning the inner query runs ONCE PER ROW of the outer query (a fresh average computed for each student's specific major), rather than once total. This is called a **correlated subquery**, and it's genuinely more computationally expensive than a non-correlated one (recap file 01's complexity concerns) since it can't just be computed once and reused.

## Subqueries in SELECT — computing a value per row

```sql
SELECT name, age,
    (SELECT AVG(age) FROM students) AS overall_avg_age,
    age - (SELECT AVG(age) FROM students) AS diff_from_avg
FROM students;
```

Genuinely useful for adding a COMPARISON column — showing each row's value alongside an aggregate reference point, directly parallels a common pandas pattern (`df['age'] - df['age'].mean()`).

## Subqueries in FROM — treating a query's result AS a table

```sql
SELECT major, avg_age
FROM (
    SELECT major, AVG(age) AS avg_age
    FROM students
    GROUP BY major
) AS major_averages
WHERE avg_age > 20;
```

Genuinely useful when a query needs to FILTER or further process an already-aggregated result — recap file 04's HAVING clause, which handles simpler versions of this; a subquery in FROM becomes necessary once the logic gets more complex than HAVING alone can express (e.g. needing to join the aggregated result against another table).

## CTEs (Common Table Expressions) — the cleaner, more readable alternative

```sql
WITH major_averages AS (
    SELECT major, AVG(age) AS avg_age
    FROM students
    GROUP BY major
)
SELECT major, avg_age
FROM major_averages
WHERE avg_age > 20;
```

Genuinely the SAME result as the subquery-in-FROM example above, but the `WITH` clause defines the intermediate result FIRST, with a readable name (`major_averages`), THEN the main query references it cleanly — a real, significant readability improvement once queries have multiple layers of logic. Most experienced SQL users genuinely prefer CTEs over deeply nested subqueries specifically for this reason.

## Multiple CTEs — chaining intermediate steps

```sql
WITH enrolled_students AS (
    SELECT DISTINCT student_id FROM enrollments
),
enrolled_details AS (
    SELECT students.name, students.age
    FROM students
    INNER JOIN enrolled_students ON students.id = enrolled_students.student_id
)
SELECT * FROM enrolled_details WHERE age > 20;
```

Genuinely a real, common pattern in production queries — building up a complex report through several clearly-named, readable steps rather than one deeply nested, hard-to-debug query. Each CTE can reference the ones defined before it, reading top to bottom like a sequence of named intermediate variables.

## Recursive CTEs — a genuinely powerful, less commonly known feature

```sql
-- Find an employee and ALL their subordinates at every level (recap file 05's self join)
WITH RECURSIVE subordinates AS (
    SELECT id, name, manager_id FROM employees WHERE id = 1        -- base case: start with employee id=1
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    INNER JOIN subordinates s ON e.manager_id = s.id                -- recursive case: find their direct reports
)
SELECT * FROM subordinates;
```

Recap file 02's recursion from the DSA folder — this is GENUINELY the same recursive idea (base case + recursive case building on previous results), expressed in SQL. Used for hierarchical data — org charts, category trees, any "find all descendants" style question that a plain JOIN can't answer since the depth of the hierarchy isn't known in advance.

## EXISTS — a genuinely efficient alternative to IN for existence checks

```sql
SELECT name FROM students s
WHERE EXISTS (SELECT 1 FROM enrollments e WHERE e.student_id = s.id);
```

`EXISTS` checks only whether the subquery returns ANY row at all — genuinely can be more efficient than `IN` for large datasets, since the database can stop as soon as ONE matching row is found, rather than needing to build the full list of values `IN` would compare against. `SELECT 1` (rather than `SELECT *`) is a real, common convention here, since the actual selected value doesn't matter — only whether a row exists.

## Try this

```sql
-- Write a CTE that computes, for each major, the number of students AND the average age
-- Then, in the SAME query, filter to majors where the student count is above the
-- OVERALL average student-count-per-major (this genuinely requires a second CTE or
-- subquery referencing the first CTE's aggregated result)
-- Then rewrite the SAME logic using nested subqueries instead of CTEs, and compare
-- readability honestly
```

Doing the same query both ways (CTE vs nested subquery) is genuinely the best way to feel firsthand why CTEs are generally preferred for anything beyond the simplest single-level subquery — the readability difference becomes obvious once there's more than one layer of logic.

## What's next
File 07 — Window Functions, a genuinely powerful and distinct category of SQL feature — computing values ACROSS a set of related rows without collapsing them into a single aggregated row the way GROUP BY does.
