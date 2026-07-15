# 07 — Window Functions

Genuinely a distinct and powerful SQL feature category — computing a value ACROSS a set of related rows, WITHOUT collapsing them into a single row the way `GROUP BY` (file 04) does. This is precisely the gap between "summarize per group" and "annotate every row with group context."

## The core difference from GROUP BY — genuinely the most important thing to understand first

```sql
-- GROUP BY collapses rows -- one output row PER major
SELECT major, AVG(age) AS avg_age FROM students GROUP BY major;
-- Result: 2-3 rows total (one per unique major)

-- Window function KEEPS every row, just adds a computed column
SELECT name, major, age, AVG(age) OVER (PARTITION BY major) AS major_avg_age
FROM students;
-- Result: ALL original rows, each annotated with its major's average age
```

This is genuinely the entire point of window functions: get an aggregate-style calculation WITHOUT losing row-level detail — directly parallels pandas' `.groupby('major')['age'].transform('mean')`, recap the pandas notes' transform vs aggregate distinction if that's familiar.

## The OVER clause — the syntax that defines a "window"

```sql
function_name() OVER (
    PARTITION BY column       -- optional: groups rows, like GROUP BY but without collapsing
    ORDER BY column             -- optional: defines row ORDER within each partition
)
```

`PARTITION BY` divides rows into groups (window functions compute independently within each partition — recap file 04's GROUP BY, but here rows stay separate). `ORDER BY` inside `OVER()` matters specifically for RANKING and RUNNING TOTAL functions, covered below.

## ROW_NUMBER — a genuinely common, simple ranking function

```sql
SELECT name, major, age,
    ROW_NUMBER() OVER (PARTITION BY major ORDER BY age DESC) AS age_rank
FROM students;
```

Assigns a UNIQUE sequential number (1, 2, 3, ...) within each partition, based on the specified order — genuinely the SQL way to answer "who's the oldest, 2nd oldest, 3rd oldest student IN EACH MAJOR."

## RANK and DENSE_RANK — handling TIES differently

```sql
SELECT name, age,
    RANK() OVER (ORDER BY age DESC) AS rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY age DESC) AS rank_no_gaps
FROM students;

-- If two students are tied for 1st place:
-- RANK():       1, 1, 3, 4, ...   -- SKIPS rank 2 after a tie
-- DENSE_RANK(): 1, 1, 2, 3, ...    -- does NOT skip
```

Genuinely important, commonly-tested distinction: `RANK()` leaves a GAP after ties (matching, e.g., Olympic medal ranking conventions — two golds means the next place is bronze, skipping silver), while `DENSE_RANK()` keeps ranks CONSECUTIVE regardless of ties. Choosing the wrong one for a given business requirement is a real, common mistake.

## LAG and LEAD — accessing PREVIOUS/NEXT row values

```sql
SELECT name, age,
    LAG(age) OVER (ORDER BY age) AS previous_age,
    LEAD(age) OVER (ORDER BY age) AS next_age,
    age - LAG(age) OVER (ORDER BY age) AS age_diff_from_previous
FROM students;
```

Genuinely one of the most practically useful window functions — directly parallels pandas' `.shift()` (recap pandas notes if covered) — used constantly for "compare this row to the previous/next one" questions: month-over-month growth, day-over-day change, detecting consecutive duplicate events.

## Running totals and moving averages — genuinely high-value business queries

```sql
SELECT name, age,
    SUM(age) OVER (ORDER BY age) AS running_total,
    AVG(age) OVER (ORDER BY age ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3row
FROM students;
```

`ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` defines a **frame** — the specific WINDOW of rows (relative to the current one) the calculation should consider — genuinely the exact mechanism behind "3-day moving average" style calculations seen constantly in finance/analytics dashboards. Without an explicit frame, `ORDER BY` inside `OVER()` defaults to "all rows from the start up to the current row," which is precisely why the running total above works with no extra frame clause needed.

## NTILE — splitting rows into equal buckets

```sql
SELECT name, age,
    NTILE(4) OVER (ORDER BY age) AS age_quartile
FROM students;
```

Splits rows into N roughly-equal groups based on the ordering — genuinely the SQL equivalent of pandas' `pd.qcut()` (recap pandas notes) — commonly used for quartile/percentile bucketing in analytics (e.g. "which spending quartile is this customer in").

## FIRST_VALUE and LAST_VALUE

```sql
SELECT name, major, age,
    FIRST_VALUE(name) OVER (PARTITION BY major ORDER BY age DESC) AS oldest_in_major
FROM students;
```

Genuinely useful for "attach the value from the top/bottom of each group to every row in that group" — e.g. tagging every employee's row with their department's highest earner's name, without a separate join.

## Combining window functions with CTEs (recap file 06) — a genuinely realistic pattern

```sql
WITH ranked_students AS (
    SELECT name, major, age,
        ROW_NUMBER() OVER (PARTITION BY major ORDER BY age DESC) AS rank_in_major
    FROM students
)
SELECT * FROM ranked_students WHERE rank_in_major = 1;      -- oldest student in EACH major
```

Genuinely important, commonly-needed pattern worth flagging explicitly: window function RESULTS cannot be filtered directly in the SAME query's `WHERE` clause (similar restriction in spirit to file 04's WHERE-vs-HAVING issue) — the window function needs to be computed FIRST inside a CTE or subquery, THEN filtered in an outer query referencing that computed column.

## Try this

```sql
-- Using the students table (with a major column), write a query using window functions
-- to show, for each student: their age, the average age of their major, the DIFFERENCE
-- between their age and their major's average, and their rank (oldest=1) within their major
-- Then use a CTE to filter down to ONLY the youngest student in each major
-- (rank = the MAX rank in each partition, using ORDER BY age ASC this time)
```

Combining `PARTITION BY`, a ranking function, an aggregate window function, and a CTE-based filter in one exercise is genuinely the realistic shape of an analytics query used in real dashboards/reports — a strong test of whether this file's concepts have actually clicked together.

## What's next
File 08 — Set Operations (UNION, INTERSECT, EXCEPT), a smaller but genuinely useful topic for combining or comparing entire RESULT SETS from separate queries.
