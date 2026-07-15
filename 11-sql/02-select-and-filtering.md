# 02 — SELECT & Filtering

Recap file 01's query structure — this file goes deep on `SELECT` and `WHERE`, genuinely the two clauses used in almost every single SQL query ever written.

## Basic SELECT

```sql
SELECT * FROM students;                       -- every column, every row
SELECT name, age FROM students;                 -- specific columns only
SELECT name AS student_name FROM students;        -- aliasing, full treatment in file 03
SELECT DISTINCT major FROM students;                -- unique values only, recap pandas .unique()
```

`SELECT *` is genuinely fine for quick exploration, but real production queries should explicitly list needed columns — pulling unnecessary columns wastes bandwidth/memory, especially on wide tables, a habit worth building early.

## WHERE — filtering rows

```sql
SELECT * FROM students WHERE age > 20;
SELECT * FROM students WHERE major = 'AIML';
SELECT * FROM students WHERE age >= 18 AND age <= 25;
SELECT * FROM students WHERE major = 'AIML' OR major = 'CSE';
SELECT * FROM students WHERE NOT major = 'CSE';
```

Recap pandas boolean masking (`df[df['age'] > 20]`) — `WHERE` is genuinely the exact same filtering logic, just SQL syntax instead of Python.

## Comparison and logical operators

```sql
=    -- equals
!=   -- or <>, not equals
>    -- greater than
<    -- less than
>=   -- greater than or equal
<=   -- less than or equal
AND  -- both conditions must be true
OR   -- at least one condition true
NOT  -- negates a condition
```

## BETWEEN, IN, LIKE — genuinely high-value shorthand operators

```sql
SELECT * FROM students WHERE age BETWEEN 18 AND 25;      -- inclusive range, cleaner than >= AND <=

SELECT * FROM students WHERE major IN ('AIML', 'CSE', 'ECE');   -- matches ANY value in the list,
                                                                     -- cleaner than chained OR conditions

SELECT * FROM students WHERE name LIKE 'S%';        -- starts with 'S' -- % is a wildcard for ANY sequence
SELECT * FROM students WHERE name LIKE '_oham';      -- _ matches exactly ONE character -- 'Soham' matches
SELECT * FROM students WHERE name LIKE '%ham%';       -- contains 'ham' anywhere
```

`LIKE` with `%`/`_` wildcards is genuinely the SQL equivalent of Python's basic string matching (`'ham' in name` for `%ham%`, `name.startswith('S')` for `'S%'`) — a real, commonly-used pattern-matching tool short of full regex.

## NULL handling — a genuinely important, easy-to-get-wrong topic

```sql
SELECT * FROM students WHERE major IS NULL;         -- correct way to check for NULL
SELECT * FROM students WHERE major IS NOT NULL;

-- WRONG, and a genuinely common mistake:
-- SELECT * FROM students WHERE major = NULL;        -- this NEVER matches anything, always returns empty
```

`NULL` represents "unknown/missing," genuinely NOT the same as an empty string or zero — and standard comparison operators (`=`, `!=`) against `NULL` always evaluate to `NULL` (neither true nor false), which is precisely why `= NULL` silently fails rather than throwing an error. `IS NULL` / `IS NOT NULL` are the only correct way to check. Recap pandas' `isna()`/`notna()` — same underlying concept, same "special handling required" caveat, carried over directly from the pandas notes.

## Combining conditions with parentheses — genuinely important for correctness

```sql
-- Ambiguous without parentheses -- AND has higher precedence than OR, similar to * vs + in math
SELECT * FROM students WHERE major = 'CSE' OR major = 'AIML' AND age > 20;
-- This actually means: major='CSE' OR (major='AIML' AND age>20) -- probably NOT what was intended

-- Explicit, correct version:
SELECT * FROM students WHERE (major = 'CSE' OR major = 'AIML') AND age > 20;
```

Genuinely worth ALWAYS using explicit parentheses when mixing `AND`/`OR` — relying on implicit operator precedence is a real, common source of subtly wrong query results that don't throw any error, just silently return the wrong rows.

## Column and table aliasing — a preview, full treatment in file 03

```sql
SELECT s.name, s.age FROM students AS s WHERE s.age > 20;
```

## CASE — conditional logic inside a query

```sql
SELECT name, age,
    CASE
        WHEN age < 18 THEN 'Minor'
        WHEN age BETWEEN 18 AND 25 THEN 'Young Adult'
        ELSE 'Adult'
    END AS age_group
FROM students;
```

`CASE` is genuinely SQL's equivalent of an if/elif/else chain, or `np.select()`/`pd.cut()` from the pandas notes — used constantly for creating derived/bucketed columns directly within a query, without pulling data into Python first.

## Combining everything into one realistic query

```sql
SELECT name, age, major
FROM students
WHERE (major IN ('AIML', 'CSE')) AND (age BETWEEN 19 AND 23) AND name LIKE 'S%'
ORDER BY age DESC;      -- ORDER BY gets its full treatment in file 03
```

## Try this

```sql
-- Using the students table from file 01's "try this" (plus the courses table you built),
-- write a single query that finds all students who are EITHER majoring in 'AIML'
-- OR are older than 22, but explicitly EXCLUDES anyone whose name starts with 'P'
-- Use parentheses correctly to avoid the AND/OR precedence trap covered in this file
```

Getting the parentheses right here is genuinely the exact skill this file is building — a query that "looks right" but has subtly wrong logic due to operator precedence is one of the most common real-world SQL bugs, and it never throws an error to reveal itself.

## What's next
File 03 — Sorting, Limiting & Aliasing, rounding out the basic query toolkit before moving into aggregation (file 04) and joins (file 05).
