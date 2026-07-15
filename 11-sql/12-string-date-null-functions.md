# 12 — String, Date & NULL Functions

Rounding out the practical SQL toolkit — genuinely the everyday utility functions used constantly in real queries and reports, less conceptually deep than files 04-11 but high-frequency in actual use.

## String functions — genuinely used constantly

```sql
SELECT UPPER(name), LOWER(name) FROM students;

SELECT LENGTH(name) FROM students;                          -- string length

SELECT SUBSTR(name, 1, 3) FROM students;                       -- first 3 characters (1-indexed in SQL, NOT 0-indexed like Python)

SELECT TRIM('  hello  ');                                        -- removes leading/trailing whitespace
SELECT LTRIM('  hello'), RTRIM('hello  ');                          -- one-sided versions

SELECT name || ' - ' || major AS summary FROM students;              -- string concatenation (SQLite/PostgreSQL use ||, MySQL uses CONCAT())

SELECT REPLACE(name, 'oh', '0h') FROM students;                        -- find-and-replace within a string
```

Genuinely important gotcha worth flagging explicitly: SQL string indexing (`SUBSTR`) is typically **1-indexed**, unlike Python's 0-indexed strings (recap the DSA/Python-basics notes) — a real, common off-by-one source of bugs when mentally translating between SQL and Python string logic.

## Date and time functions — genuinely database-specific, SQLite shown here

```sql
SELECT DATE('now');                              -- current date
SELECT DATETIME('now');                            -- current date AND time
SELECT DATE('now', '+7 days');                        -- date arithmetic -- a week from now
SELECT DATE('now', '-1 month');                          -- a month ago

SELECT strftime('%Y', signup_date) AS signup_year FROM students;   -- extract just the year
SELECT strftime('%Y-%m', signup_date) AS signup_month FROM students;  -- year-month, useful for monthly grouping
```

```sql
-- Calculating age from a birthdate, genuinely a common real query
SELECT name,
    (strftime('%Y', 'now') - strftime('%Y', birthdate)) AS age
FROM students;

-- Days between two dates
SELECT julianday('now') - julianday(signup_date) AS days_since_signup FROM students;
```

Genuinely important, real-world-relevant honest note: date/time function SYNTAX varies significantly between database systems — PostgreSQL uses `EXTRACT(YEAR FROM date_column)` and `CURRENT_DATE`, MySQL uses `YEAR(date_column)` and `NOW()`, SQLite uses `strftime()` as shown above. Worth knowing this upfront rather than assuming SQL date handling is fully portable across systems — it genuinely isn't, more than most other SQL features.

## Grouping by time period — a genuinely common real analytics pattern

```sql
-- Recap file 04's GROUP BY, now applied to time-bucketed data
SELECT strftime('%Y-%m', order_date) AS month, COUNT(*) AS order_count, SUM(amount) AS total_revenue
FROM orders
GROUP BY strftime('%Y-%m', order_date)
ORDER BY month;
```

Genuinely one of the most common real business-analytics query shapes — "monthly revenue," "daily active users," "weekly signups" — all reduce to exactly this pattern: extract a time bucket, group by it, aggregate.

## COALESCE — handling NULLs gracefully, genuinely important

```sql
SELECT name, COALESCE(major, 'Undeclared') AS major FROM students;
```

`COALESCE` returns the FIRST non-NULL value from a list of arguments — genuinely the SQL equivalent of pandas' `.fillna()` (recap pandas notes). Extremely common in real reporting queries where NULL values need a sensible default DISPLAY value without actually modifying the underlying stored data.

```sql
-- COALESCE with multiple fallbacks -- tries each in order until a non-NULL is found
SELECT COALESCE(preferred_name, first_name, 'Unknown') AS display_name FROM students;
```

## NULLIF — the reverse idea, turning a specific value INTO NULL

```sql
SELECT NULLIF(age, 0) FROM students;      -- if age is 0 (likely a data-entry placeholder), treat it as NULL instead
```

Genuinely useful for a real, common scenario: a system uses `0` or `-1` as a "no value entered" placeholder instead of true NULL — `NULLIF` converts that placeholder INTO genuine NULL, so it can then be handled correctly by `IS NULL` checks (recap file 02) or `COALESCE` above, rather than being silently included in numeric calculations like `AVG()` where a placeholder zero would incorrectly skew the result.

## CAST — converting between data types

```sql
SELECT CAST(age AS TEXT) FROM students;                -- number to string
SELECT CAST('123' AS INTEGER) + 1;                        -- string to number, genuinely useful when data arrives as text

SELECT AVG(CAST(age AS REAL)) FROM students;                -- forcing float division instead of potential integer division
```

Genuinely a real, common gotcha worth flagging: some databases perform INTEGER division by default (`5 / 2` returning `2`, not `2.5`) unless explicitly cast — recap the general "know your types" caution from file 09; this integer-division behavior varies by database system and is a real, silent source of wrong numeric results if not accounted for.

## Combining string, date, and NULL functions in a realistic query

```sql
SELECT
    COALESCE(UPPER(name), 'UNKNOWN') AS display_name,
    strftime('%Y', signup_date) AS signup_year,
    CASE WHEN major IS NULL THEN 'Undeclared' ELSE major END AS field
FROM students
ORDER BY signup_year DESC;
```

Genuinely the realistic shape of a query feeding an actual report or dashboard — cleaning up display values, extracting meaningful time buckets, and handling missing data gracefully, all in one pass rather than needing post-processing in Python afterward.

## Try this

```sql
-- Given an "orders" table with (id, customer_name, order_date, amount), write a query that:
-- 1. Groups orders by MONTH, showing total revenue and order count per month
-- 2. Uses COALESCE to handle any NULL customer_name values, defaulting to 'Guest'
-- 3. Formats a summary string like "January 2026: 45 orders, $12,340.50" using
--    string concatenation and CAST to combine the numeric and text pieces
```

Building a realistic formatted summary string (combining `strftime`, `CAST`, and `||` concatenation) is genuinely representative of the kind of query that ends up feeding directly into a report or dashboard, tying together everything covered in this file.

## What's next
File 13 — SQL for Data Science / ML Workflows, connecting everything from files 01-12 directly into the actual ML pipeline context — feature engineering, data extraction for training, and the pandas/SQL round-trip.
