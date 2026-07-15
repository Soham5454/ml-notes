# 03 — Sorting, Limiting & Aliasing

Rounding out the basic query toolkit from files 01-02 before moving into aggregation (file 04) and joins (file 05).

## ORDER BY — sorting results

```sql
SELECT * FROM students ORDER BY age;              -- ascending by default
SELECT * FROM students ORDER BY age DESC;           -- descending
SELECT * FROM students ORDER BY major, age DESC;      -- multi-column: sort by major first, then age within each major
```

Recap pandas `.sort_values()` — genuinely the exact same operation, and the multi-column behavior matches too: `ORDER BY major, age DESC` sorts primarily by major (ascending), and WITHIN each major group, sorts by age descending — a direct parallel to pandas' `sort_values(['major', 'age'], ascending=[True, False])`.

## LIMIT and OFFSET — restricting result size

```sql
SELECT * FROM students ORDER BY age DESC LIMIT 5;              -- top 5 oldest students
SELECT * FROM students ORDER BY age DESC LIMIT 5 OFFSET 5;       -- the NEXT 5 (skip first 5) -- genuinely how pagination works
```

`LIMIT`/`OFFSET` together are precisely the mechanism behind "page 2 of results" in any real application — genuinely worth knowing this pattern by name, since "implement pagination" is a common practical/interview task.

## Aliasing — renaming columns and tables for clarity

```sql
SELECT name AS student_name, age AS student_age FROM students;

SELECT s.name, s.age FROM students AS s WHERE s.age > 20;      -- table aliasing, genuinely essential once joins (file 05) are involved

SELECT COUNT(*) AS total_students FROM students;                  -- aliasing an aggregate result (file 04) for a readable column name
```

Genuinely important practical reason for aliasing: without it, aggregate/computed columns get ugly default names (like `COUNT(*)`), which is awkward to reference in application code or further queries. Table aliases (`students AS s`) become close to non-negotiable once multiple tables are joined together (file 05) — writing `students.name` repeatedly versus `s.name` is a real readability difference in longer queries.

## Combining ORDER BY with computed/aliased columns

```sql
SELECT name, age,
    CASE WHEN age < 21 THEN 'Under 21' ELSE '21+' END AS age_category
FROM students
ORDER BY age_category, name;
```

Genuinely useful, sometimes-missed capability — `ORDER BY` can reference an ALIAS defined in the `SELECT` clause (not universally true across every database system, but works in SQLite, PostgreSQL, MySQL), avoiding the need to repeat the full `CASE` expression in the `ORDER BY` clause.

## Sorting NULLs — a genuinely important, database-specific detail

```sql
SELECT * FROM students ORDER BY major;
-- Where do rows with a NULL major end up? Behavior GENUINELY differs by database:
-- PostgreSQL: NULLs sort LAST by default (for ASC)
-- SQLite/MySQL: NULLs sort FIRST by default (for ASC)

-- Explicit control, when supported:
SELECT * FROM students ORDER BY major NULLS LAST;      -- PostgreSQL syntax
```

Genuinely worth knowing this varies by database system — a query that "works correctly" in one system's default NULL-sorting behavior can silently behave differently when the same SQL runs against a different database engine, a real gotcha when migrating between systems.

## Sorting by column POSITION — a real but risky shorthand

```sql
SELECT name, age, major FROM students ORDER BY 2 DESC;      -- sorts by the 2nd selected column (age)
```

Genuinely works, but honestly fragile — if the column order in `SELECT` ever changes, this silently sorts by the WRONG column with no error. Worth knowing it exists (it shows up in older codebases), but explicit column names in `ORDER BY` are the safer, recommended practice for new code.

## Putting it together — a realistic query using files 01-03

```sql
SELECT name AS student_name, age, major
FROM students
WHERE major IN ('AIML', 'CSE')
ORDER BY age DESC
LIMIT 3;
```

Genuinely worth reading this out loud in execution order (recap file 01): FROM students → WHERE filters to AIML/CSE → results get ORDERED by age descending → LIMIT keeps only the top 3 → THEN the SELECT clause determines which columns/aliases actually get returned.

## Try this

```sql
-- Using the students table, write a query that returns the 2nd and 3rd oldest students
-- (using LIMIT + OFFSET), showing their name and age, with the age column aliased as
-- "years_old"
-- Then write a SEPARATE query that sorts students first by major (alphabetically),
-- then by age (youngest first) within each major
```

Getting comfortable combining `ORDER BY`, `LIMIT`, `OFFSET`, and aliasing in one query is genuinely the exact combination used constantly in real applications — leaderboards, paginated tables, "top N by category" reports.

## What's next
File 04 — Aggregate Functions & GROUP BY/HAVING, moving from "filter and sort individual rows" into "summarize data across groups of rows" — genuinely where SQL starts doing things pandas' `.groupby()` does, but as a first-class query feature.
