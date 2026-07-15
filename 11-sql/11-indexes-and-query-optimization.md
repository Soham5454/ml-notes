# 11 — Indexes & Query Optimization

Recap file 01's complexity-analysis parallel from the DSA folder — this file is genuinely about making queries FAST, not just correct, and directly reuses Big O thinking (recap DSA file 01) to explain why.

## The problem indexes solve — a full table scan

```sql
SELECT * FROM students WHERE name = 'Soham';
```

Without an index, the database genuinely has to check EVERY SINGLE ROW to find matches — a **full table scan**, `O(n)` (recap DSA file 01's exact complexity notation) where `n` is the number of rows. Fine for a small practice table, genuinely catastrophic for a production table with millions of rows.

## What an index actually is

```sql
CREATE INDEX idx_students_name ON students(name);
```

An index is genuinely a separate, sorted data structure (commonly a B-tree, conceptually related to DSA file 07's BST, though typically more balanced/wide) that lets the database jump DIRECTLY to matching rows instead of scanning everything — recap DSA file 12's binary search: an index lookup is genuinely `O(log n)` instead of `O(n)`, the exact same complexity improvement binary search provides over linear search.

```python
# The mental model, directly parallel to a Python dict/hash lookup (recap DSA file 06):
# without an index: like searching a Python LIST for a value -- O(n)
# with an index: like looking up a value in a Python DICT/set -- close to O(1) or O(log n)
```

## When to add an index — genuinely a real tradeoff, not "always index everything"

```sql
-- Good candidates for indexing:
-- - Columns used FREQUENTLY in WHERE clauses
-- - Columns used in JOIN conditions (recap file 05 -- join performance depends heavily on this)
-- - Columns used in ORDER BY, especially on large result sets
-- - Foreign key columns (recap file 09) -- often not auto-indexed, worth adding explicitly

-- Poor candidates:
-- - Columns rarely queried
-- - Columns with very few DISTINCT values (e.g. a boolean is_active column) --
--   an index provides little benefit since it can't narrow the search down much
-- - Small tables -- a full scan on 100 rows is already fast; the index overhead isn't worth it
```

Genuinely the real cost of indexes, worth understanding honestly: every index SPEEDS UP reads (`SELECT`) but SLOWS DOWN writes (`INSERT`/`UPDATE`/`DELETE`), since the index itself has to be updated every time the underlying data changes. Real database design is a genuine balancing act between read and write performance, not "index as much as possible."

## Composite indexes — indexing multiple columns together

```sql
CREATE INDEX idx_major_age ON students(major, age);
```

Genuinely useful for queries that filter on BOTH columns together (`WHERE major = 'AIML' AND age > 20`) — but important, commonly-missed detail: a composite index is most effective when the query's filtering matches the COLUMN ORDER in the index definition. This index helps `WHERE major = 'AIML'` alone, and `WHERE major = 'AIML' AND age > 20` together, but does NOT efficiently help a query filtering ONLY on `age` without also filtering on `major` — the "leftmost prefix" rule, genuinely worth knowing exists even without memorizing every edge case.

## EXPLAIN — seeing what the database ACTUALLY does

```sql
EXPLAIN QUERY PLAN
SELECT * FROM students WHERE name = 'Soham';
```

Genuinely the single most practically useful tool in this whole file — `EXPLAIN` (syntax varies slightly per database: `EXPLAIN ANALYZE` in PostgreSQL) shows whether a query is using an index (`SEARCH` in SQLite's output) or doing a full table scan (`SCAN`), letting real performance problems be DIAGNOSED rather than guessed at. Real database optimization work genuinely starts with running `EXPLAIN`, not with assumptions.

## Query optimization beyond indexing — genuinely practical habits

```sql
-- Avoid SELECT * in production queries -- pulling unnecessary columns wastes I/O
SELECT name, age FROM students WHERE major = 'AIML';   -- better than SELECT *

-- Filter EARLY -- WHERE before JOIN when possible lets the database work with less data
-- (though modern query planners often reorder this automatically -- still good practice to write clearly)

-- Avoid functions on INDEXED columns in WHERE -- this can prevent the index from being used
-- BAD: WHERE UPPER(name) = 'SOHAM'      -- forces a full scan, since the index is on the RAW name values
-- BETTER: store/compare consistently, or use a functional index if the database supports it

-- LIMIT results when the full result set isn't actually needed
SELECT * FROM students ORDER BY age DESC LIMIT 10;   -- genuinely faster than pulling everything then truncating in Python
```

The `UPPER(name)` example is genuinely a real, common performance trap — wrapping an indexed column in a function forces the database to compute that function for EVERY row before it can compare, defeating the whole purpose of having a sorted index available for instant lookups.

## JOIN performance — recap file 05, now with the "why it's fast/slow" explanation

```sql
-- This join is efficient IF both id columns are indexed (student.id is a PRIMARY KEY,
-- automatically indexed; enrollments.student_id should be EXPLICITLY indexed too)
SELECT students.name, enrollments.course
FROM students
INNER JOIN enrollments ON students.id = enrollments.student_id;
```

Genuinely important practical takeaway: primary keys are automatically indexed by most databases, but FOREIGN KEY columns are often NOT automatically indexed — explicitly adding an index on `enrollments.student_id` can genuinely make a huge difference in join performance on a large table, and this is a very common thing that gets missed in real schema design.

## Denormalization — a genuine, honest tradeoff, recap file 09

```sql
-- Sometimes, for READ-HEAVY workloads, deliberately duplicating some data
-- (e.g. storing a student's major directly on the enrollments table too, even though
-- it's technically redundant with students.major) avoids an extra JOIN on a
-- very frequently-run query, at the cost of needing to keep both copies in sync
```

Genuinely a real engineering tradeoff practiced in production systems, not a mistake — worth knowing it's a legitimate technique, applied deliberately and sparingly, after confirming (via `EXPLAIN`, real profiling) that join cost is genuinely a bottleneck, not as a first-resort habit.

## Try this

```sql
-- Create a students table with 1000+ rows (generate synthetic data, e.g. via a loop
-- in Python using sqlite3, or a recursive CTE from file 06)
-- Run EXPLAIN QUERY PLAN on a WHERE name = 'X' query BEFORE adding an index -- note "SCAN"
-- Add CREATE INDEX idx_name ON students(name)
-- Run the SAME EXPLAIN QUERY PLAN again -- confirm it now shows "SEARCH" instead
-- Optionally, time both queries directly (Python's time module) to see a CONCRETE
-- speed difference, not just the query plan's description
```

Actually seeing the `EXPLAIN` output change from `SCAN` to `SEARCH` after adding an index, and ideally measuring a real timing difference, is genuinely the clearest way to make "indexes matter" feel concrete rather than theoretical.

## What's next
File 12 — String, Date & NULL Functions, rounding out the practical SQL toolkit with the everyday utility functions used constantly in real queries and reports.
