# 08 — Set Operations (UNION, INTERSECT, EXCEPT)

A smaller but genuinely useful topic — combining or comparing entire RESULT SETS from separate queries, rather than combining COLUMNS (joins, file 05) or summarizing WITHIN one result (aggregates, file 04).

## UNION — combining rows from two queries, removing duplicates

```sql
SELECT name FROM students WHERE major = 'AIML'
UNION
SELECT name FROM students WHERE age > 22;
```

Genuinely requires both queries to return the SAME number of columns, with COMPATIBLE data types — `UNION` stacks the results vertically and automatically removes duplicate rows across the combined set (recap file 02's `DISTINCT` — `UNION` behaves similarly by default).

## UNION ALL — the same idea, but KEEPS duplicates

```sql
SELECT name FROM students WHERE major = 'AIML'
UNION ALL
SELECT name FROM students WHERE age > 22;
```

Genuinely important practical distinction: `UNION ALL` is FASTER than `UNION`, since it skips the deduplication step entirely — worth defaulting to `UNION ALL` whenever duplicates are genuinely acceptable or known not to occur, rather than paying the deduplication cost unnecessarily.

## INTERSECT — rows that appear in BOTH queries

```sql
SELECT name FROM students WHERE major = 'AIML'
INTERSECT
SELECT name FROM students WHERE age > 22;
-- Returns only students who are BOTH majoring in AIML AND older than 22
```

Genuinely the SET-operation equivalent of combining two conditions with `AND` — this specific example could also be written as a single `WHERE major='AIML' AND age>22` query. `INTERSECT` becomes more valuable when the two source queries pull from genuinely DIFFERENT tables or have very different structures that can't cleanly combine into one `WHERE` clause.

## EXCEPT (or MINUS in some databases) — rows in the FIRST query, NOT in the second

```sql
SELECT name FROM students
EXCEPT
SELECT name FROM students WHERE major = 'AIML';
-- All students EXCEPT those majoring in AIML -- genuinely equivalent to WHERE major != 'AIML' here,
-- but EXCEPT shines when comparing across separate queries/tables
```

Note: Oracle uses `MINUS` instead of `EXCEPT` for this exact same operation — genuinely just a naming difference between database systems, worth knowing both terms exist.

## A genuinely realistic use case — finding data discrepancies

```sql
-- Find student IDs that exist in the 'students' table but NOT in 'enrollments'
-- (recap file 05's LEFT JOIN + IS NULL anti-join -- this is an alternative approach to the SAME question)
SELECT id FROM students
EXCEPT
SELECT student_id FROM enrollments;
```

Genuinely worth recognizing this as ANOTHER valid way to solve the exact anti-join problem from file 05's "try this" exercise — a good example of how SQL often offers multiple correct paths to the same result, and picking between them often comes down to readability or the specific database engine's performance characteristics (touched on more in file 11).

## Column names and ordering in combined results

```sql
SELECT name AS person_name FROM students
UNION
SELECT course AS person_name FROM enrollments;
-- The FINAL result's column names come from the FIRST query in the UNION chain
```

Genuinely a real, sometimes-confusing detail — even though the second query's column is literally named `course`, the combined result uses the FIRST query's alias (`person_name`). Worth being deliberate about aliasing the first query clearly, since it determines the output column names for the whole combined result.

## Ordering a UNION result — ORDER BY applies to the WHOLE combined set

```sql
SELECT name, age FROM students WHERE major = 'AIML'
UNION
SELECT name, age FROM students WHERE major = 'CSE'
ORDER BY age DESC;      -- applies to the FINAL combined result, not either query individually
```

Genuinely important: `ORDER BY` can only appear ONCE, at the very end of the whole `UNION` chain — it sorts the FINAL combined output, not each individual query's result before combining.

## Try this

```sql
-- Using the students and enrollments tables, write a query using INTERSECT to find
-- students who are enrolled in BOTH 'ML Basics' AND 'DSA' (hint: two separate SELECTs
-- against enrollments, filtered by course, combined with INTERSECT)
-- Then rewrite the SAME logic using a self-join on enrollments instead, and compare
-- which approach reads more clearly for this specific question
```

Solving the same "enrolled in both X and Y" question two different ways (set operations vs self-join) is genuinely a good exercise for building judgment about which SQL tool fits a given question most naturally — there's rarely only one correct way to write a query.

## What's next
File 09 — Schema Design, Data Types & Constraints, moving from QUERYING existing data into actually DESIGNING the tables that hold it well in the first place.
