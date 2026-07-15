# 05 — Joins

Recap file 01's foreign key note — joins are precisely the mechanism that reconnects data spread across separate, RELATED tables. Genuinely the single most important SQL topic for any real multi-table database, and a very common interview/practical topic.

## Why data gets split across tables at all — normalization, briefly

```sql
-- Instead of one giant table repeating course info for every enrolled student:
-- enrollments: (student_id, student_name, student_age, course_name, course_credits, ...)
-- -- this DUPLICATES course_name/credits for every student taking that course

-- Split into two related tables instead:
CREATE TABLE students (id INTEGER PRIMARY KEY, name TEXT, age INTEGER);
CREATE TABLE enrollments (student_id INTEGER, course_name TEXT, credits INTEGER,
                           FOREIGN KEY (student_id) REFERENCES students(id));
```

This splitting (called **normalization**) avoids data duplication/inconsistency — genuinely the real reason relational databases exist as multiple linked tables rather than one giant flat spreadsheet. Joins are what RECOMBINE this data back together when a query actually needs both pieces at once.

## Setting up example data for this whole file

```sql
CREATE TABLE students (id INTEGER PRIMARY KEY, name TEXT);
INSERT INTO students VALUES (1, 'Soham'), (2, 'Priya'), (3, 'Amit');

CREATE TABLE enrollments (student_id INTEGER, course TEXT);
INSERT INTO enrollments VALUES (1, 'ML Basics'), (1, 'DSA'), (2, 'Web Dev');
-- Note: Amit (id=3) has NO enrollments -- deliberately, to demonstrate join behavior below
```

## INNER JOIN — the default, most common join

```sql
SELECT students.name, enrollments.course
FROM students
INNER JOIN enrollments ON students.id = enrollments.student_id;

-- Result:
-- Soham | ML Basics
-- Soham | DSA
-- Priya | Web Dev
-- (Amit does NOT appear -- he has no matching row in enrollments)
```

`INNER JOIN` returns ONLY rows where the join condition matches in BOTH tables — genuinely the "intersection" of the two tables based on the `ON` condition. Amit disappearing from the result is the key behavior to understand: no matching enrollment row means no output row for him.

## LEFT JOIN — keep everything from the left table, regardless of matches

```sql
SELECT students.name, enrollments.course
FROM students
LEFT JOIN enrollments ON students.id = enrollments.student_id;

-- Result:
-- Soham | ML Basics
-- Soham | DSA
-- Priya | Web Dev
-- Amit  | NULL          <- Amit NOW appears, with NULL for course since he has no enrollment
```

Genuinely the most commonly needed join type after INNER JOIN — "show me ALL students, and their courses IF they have any" is an extremely common real requirement (e.g. "show all customers, including those with zero orders" for a business report).

## RIGHT JOIN — the mirror image of LEFT JOIN

```sql
SELECT students.name, enrollments.course
FROM students
RIGHT JOIN enrollments ON students.id = enrollments.student_id;
-- Keeps everything from enrollments (the RIGHT table), NULL-fills unmatched students
```

Genuinely rarely used in practice compared to LEFT JOIN — most people just swap table order and use LEFT JOIN instead, since it reads more naturally ("all students, plus their courses" rather than "all enrollments, plus their students"). Worth knowing it exists and behaves as the mirror of LEFT JOIN, but LEFT JOIN is the one to reach for by default. Note: SQLite doesn't support RIGHT JOIN directly — worth knowing this is a real database-specific limitation.

## FULL OUTER JOIN — keep everything from BOTH tables

```sql
-- PostgreSQL syntax (SQLite doesn't support FULL OUTER JOIN natively)
SELECT students.name, enrollments.course
FROM students
FULL OUTER JOIN enrollments ON students.id = enrollments.student_id;
-- Returns matched rows, PLUS unmatched students (NULL course), PLUS unmatched enrollments (NULL student)
```

Genuinely useful for "show me everything from both sides, matched where possible" — e.g. reconciling two datasets to find records present in only one of them.

## CROSS JOIN — every combination of both tables, genuinely rare but worth knowing

```sql
SELECT students.name, courses.course_name
FROM students
CROSS JOIN courses;      -- every student paired with EVERY course -- no ON condition needed/used
```

Produces the full Cartesian product — `n * m` rows for `n` students and `m` courses. Genuinely rare in real queries (usually a mistake if it happens accidentally — forgetting an `ON` condition on a regular join can silently produce a cross join instead, a real and confusing bug), but occasionally intentional for generating all possible combinations (e.g. all products × all sizes for an inventory template).

## SELF JOIN — joining a table to itself

```sql
CREATE TABLE employees (id INTEGER, name TEXT, manager_id INTEGER);
INSERT INTO employees VALUES (1, 'Alice', NULL), (2, 'Bob', 1), (3, 'Carol', 1);

SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Bob   | Alice
-- Carol | Alice
-- Alice | NULL          <- Alice has no manager
```

Genuinely a real, common pattern — any HIERARCHICAL relationship stored within one table (employees/managers, categories/subcategories, comment/reply threads) needs a self join, aliasing the SAME table twice (`e` and `m` here) to represent "this row" and "the related row" simultaneously.

## Multiple joins in one query — genuinely common in real reporting queries

```sql
SELECT students.name, enrollments.course, courses.credits
FROM students
INNER JOIN enrollments ON students.id = enrollments.student_id
INNER JOIN courses ON enrollments.course = courses.course_name;
```

Genuinely just chaining the same pattern — each additional `JOIN` connects one more related table, and real production queries routinely join 4-5+ tables together to assemble a complete report.

## The honest join-type decision guide

```
Need only matched rows from both tables?         -> INNER JOIN
Need ALL rows from the main/primary table,
  with related data attached where it exists?      -> LEFT JOIN
Need ALL rows from BOTH tables, matched where possible? -> FULL OUTER JOIN
Need a hierarchical self-relationship?                -> SELF JOIN (with LEFT JOIN typically)
Need every possible combination (rare)?                 -> CROSS JOIN
```

## Try this

```sql
-- Using the students/enrollments setup from this file, write a query that finds
-- students who are NOT enrolled in ANY course (hint: LEFT JOIN, then WHERE the
-- joined column IS NULL -- a genuinely common real pattern called an "anti-join")
-- Then write a query using a self join on the employees table to find all
-- employee-manager PAIRS where the employee is OLDER than their manager
-- (you'll need to add an age column to make this work)
```

The "LEFT JOIN + IS NULL" anti-join pattern is genuinely one of the most useful, slightly-non-obvious techniques in practical SQL — used constantly for "find records with no matching related record" style business questions.

## What's next
File 06 — Subqueries & CTEs, techniques for building queries out of OTHER queries — genuinely essential once questions get more complex than a single SELECT/JOIN can answer directly.
