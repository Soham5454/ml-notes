# 04 — Aggregate Functions & GROUP BY / HAVING

Moving from "filter and sort individual rows" (files 02-03) into "summarize data ACROSS groups of rows" — genuinely the SQL equivalent of pandas' `.groupby()`, and one of the most practically important skills in this whole folder.

## Aggregate functions — summarizing a whole column into one value

```sql
SELECT COUNT(*) FROM students;                  -- total number of rows
SELECT COUNT(major) FROM students;                -- count of NON-NULL majors specifically
SELECT AVG(age) FROM students;                      -- average age
SELECT SUM(age) FROM students;                       -- sum of ages (not usually meaningful, but the function works)
SELECT MIN(age) FROM students;                        -- youngest
SELECT MAX(age) FROM students;                          -- oldest
```

Recap file 12 in the Math Foundations folder — `AVG()` is literally computing the sample mean (expectation), same formula, just as a SQL aggregate instead of a Python calculation. Genuinely important distinction worth being precise about: `COUNT(*)` counts ALL rows including NULLs in other columns, while `COUNT(column_name)` counts only rows where THAT specific column is non-NULL — a real, common source of off-by-some-amount confusion.

## GROUP BY — the core of this whole file

```sql
SELECT major, COUNT(*) AS student_count
FROM students
GROUP BY major;
```

Recap pandas' `.groupby('major').size()` — genuinely the exact same operation: split the table into GROUPS based on the `major` column's distinct values, then compute an aggregate (here, `COUNT(*)`) WITHIN each group separately.

```sql
SELECT major, AVG(age) AS avg_age, MIN(age) AS youngest, MAX(age) AS oldest
FROM students
GROUP BY major;
```

Directly parallels pandas' `.groupby('major').agg({'age': ['mean', 'min', 'max']})` — multiple aggregates computed per group in a single query.

## The genuinely critical rule — every non-aggregated column MUST be in GROUP BY

```sql
-- This is INVALID in strict SQL (and will error in PostgreSQL, though SQLite/MySQL
-- may silently allow it with undefined/misleading behavior):
-- SELECT major, name, COUNT(*) FROM students GROUP BY major;
-- 'name' isn't aggregated AND isn't in GROUP BY -- which specific name would even be shown per group?

-- Correct:
SELECT major, COUNT(*) FROM students GROUP BY major;
```

Genuinely the most common GROUP BY mistake — every column in `SELECT` must EITHER be wrapped in an aggregate function (`COUNT`, `AVG`, etc.) OR be one of the columns listed in `GROUP BY`. If a column is neither, SQL has no defined way to decide which single value to show for that group, since a group can contain MULTIPLE rows with different values in that column.

## GROUP BY on multiple columns

```sql
SELECT major, age, COUNT(*) AS count
FROM students
GROUP BY major, age;
```

Genuinely creates one group per UNIQUE COMBINATION of major AND age — directly parallel to pandas' `.groupby(['major', 'age'])`.

## HAVING — filtering on AGGREGATED results, the piece WHERE genuinely can't do

```sql
SELECT major, COUNT(*) AS student_count
FROM students
GROUP BY major
HAVING COUNT(*) > 5;         -- only majors with MORE than 5 students
```

Recap file 01's execution order note — `WHERE` filters rows BEFORE grouping happens, so it genuinely CANNOT reference an aggregate result (the aggregate doesn't exist yet at that point in execution). `HAVING` filters AFTER grouping/aggregation, which is precisely why it's the correct (and only) tool for "show me groups where the COUNT/AVG/SUM meets some condition."

```sql
-- WRONG -- this will error, COUNT(*) doesn't exist yet when WHERE runs
-- SELECT major, COUNT(*) FROM students WHERE COUNT(*) > 5 GROUP BY major;

-- CORRECT
SELECT major, COUNT(*) FROM students GROUP BY major HAVING COUNT(*) > 5;
```

## Combining WHERE and HAVING — both filtering at their correct stage

```sql
SELECT major, AVG(age) AS avg_age
FROM students
WHERE age > 18                    -- filters INDIVIDUAL rows first (e.g. exclude minors)
GROUP BY major
HAVING AVG(age) < 25              -- THEN filters GROUPS based on the aggregated result
ORDER BY avg_age DESC;
```

Genuinely a realistic, common query shape — `WHERE` for row-level filtering, `HAVING` for group-level filtering, both in the same query serving genuinely different purposes.

## Aggregate functions with DISTINCT

```sql
SELECT COUNT(DISTINCT major) AS unique_majors FROM students;
```

Recap pandas' `.nunique()` — counts the number of UNIQUE values, not total rows, genuinely a common real requirement ("how many different majors are represented," not "how many students have a major").

## GROUP_CONCAT / STRING_AGG — aggregating strings, not just numbers

```sql
-- SQLite / MySQL syntax
SELECT major, GROUP_CONCAT(name) AS students_list
FROM students
GROUP BY major;

-- PostgreSQL syntax (different function name, same idea)
-- SELECT major, STRING_AGG(name, ', ') AS students_list FROM students GROUP BY major;
```

Genuinely useful, sometimes-overlooked aggregate — combines all the TEXT values within a group into one comma-separated string, rather than a numeric summary. Useful for reports/summaries where "list every student in each major" is the actual desired output.

## Try this

```sql
-- Using the students table, write a query that shows each major, the number of students
-- in it, and the average age, but ONLY for majors with 3 or more students, ordered by
-- student count descending
-- Then extend it: add a WHERE clause that first excludes any student under 18,
-- BEFORE the grouping happens -- confirm the average age changes appropriately
```

Combining `WHERE` (row-level) and `HAVING` (group-level) filtering correctly in one query is genuinely the exact skill this file exists to build — a very common real-world query shape, and a frequent point of confusion for people newer to SQL.

## What's next
File 05 — Joins, genuinely the single most important SQL topic for real, multi-table databases — combining related data spread across separate tables (recap file 01's foreign key note) into one unified result.
