# 13 — SQL for Data Science / ML Workflows

Connecting everything from files 01-12 directly into the actual ML pipeline context this whole roadmap is building toward — genuinely where SQL stops being an abstract skill and becomes part of the real day-to-day ML workflow.

## The realistic pipeline position — where SQL actually sits

```
Raw data in a database (SQL) -> Extract/query relevant data (SQL) ->
Load into pandas (Python) -> Clean/feature-engineer (pandas, recap those notes) ->
Train model (scikit-learn/XGBoost/PyTorch, recap those folders)
```

Genuinely the realistic starting point for most real ML projects — data doesn't arrive as a clean CSV, it lives in a database, and pulling the RIGHT subset with the RIGHT joins/aggregations via SQL is often the actual first step, before any pandas or model code gets written at all.

## Querying directly into pandas — the practical round-trip

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect('practice.db')

df = pd.read_sql_query("""
    SELECT s.name, s.age, s.major, COUNT(e.course) AS num_courses
    FROM students s
    LEFT JOIN enrollments e ON s.id = e.student_id
    GROUP BY s.id
""", conn)

print(df.head())
```

Genuinely the single most practically important line in this whole file — `pd.read_sql_query()` runs SQL directly and returns a pandas DataFrame, immediately ready for the ENTIRE pandas/scikit-learn/PyTorch pipeline already built across this repo. This is the real connective tissue between the SQL folder and everything that came before it.

## Feature engineering IN SQL vs IN pandas — a genuine, honest tradeoff

```sql
-- Doing feature engineering directly in SQL, BEFORE pulling into Python
SELECT
    student_id,
    COUNT(*) AS total_courses,
    AVG(credits) AS avg_credits_per_course,
    MAX(credits) AS max_single_course_credits,
    SUM(CASE WHEN credits >= 4 THEN 1 ELSE 0 END) AS num_heavy_courses    -- recap file 02's CASE
FROM enrollments
JOIN courses ON enrollments.course = courses.course_name
GROUP BY student_id;
```

```python
# The SAME feature engineering, done in pandas AFTER pulling raw data
raw_df = pd.read_sql_query("SELECT * FROM enrollments JOIN courses ON ...", conn)
features_df = raw_df.groupby('student_id').agg(
    total_courses=('course', 'count'),
    avg_credits_per_course=('credits', 'mean'),
    max_single_course_credits=('credits', 'max')
)
features_df['num_heavy_courses'] = raw_df[raw_df['credits'] >= 4].groupby('student_id').size()
```

Genuinely both approaches reach the same result — the honest tradeoff: SQL-side aggregation pushes the computational work to the DATABASE (which is often optimized for exactly this, recap file 11's indexing), reducing how much RAW data needs to travel over the network into Python. This matters GENUINELY a lot at real scale (millions of rows) — pulling everything raw into pandas first and aggregating in Python can be dramatically slower and more memory-hungry than letting the database do it. For SMALL data or exploratory work, pandas is often more flexible/iterative. Real production ML pipelines often do BOTH — heavy aggregation in SQL, fine-tuned feature engineering in pandas afterward.

## Train/test split considerations — a genuinely important, easy-to-miss detail

```sql
-- Genuinely important for TIME-SERIES-like ML problems: splitting by DATE in SQL
-- to avoid data leakage (recap the general train/test discipline from scikit-learn notes)
SELECT * FROM transactions WHERE transaction_date < '2026-01-01';   -- training set
SELECT * FROM transactions WHERE transaction_date >= '2026-01-01';    -- test set
```

Recap the scikit-learn/XGBoost/PyTorch notes' repeated emphasis on train/validation/test discipline — when data has a time dimension, a RANDOM split (like plain `train_test_split`) can leak FUTURE information into training, genuinely inflating validation metrics unrealistically. Splitting by date directly in SQL, BEFORE any Python-side splitting happens, is a real, practical safeguard worth building as a habit for time-relevant ML problems.

## Sampling data directly in SQL — for large-scale exploration

```sql
-- SQLite/general approach: random sample of rows
SELECT * FROM large_table ORDER BY RANDOM() LIMIT 1000;

-- PostgreSQL has a more efficient dedicated syntax for large tables:
-- SELECT * FROM large_table TABLESAMPLE SYSTEM (1);   -- roughly 1% sample
```

Genuinely important practical note: `ORDER BY RANDOM()` works but requires sorting the ENTIRE table first (expensive on large data) — for genuinely large-scale sampling, database-specific sampling methods (like PostgreSQL's `TABLESAMPLE`) avoid this cost. Worth knowing the naive approach doesn't scale, even though it's fine for practice-sized data.

## Detecting data quality issues directly in SQL — before they reach a model

```sql
-- Recap file 02's NULL handling and file 13's descriptive stats parallel from Math Foundations
SELECT COUNT(*) AS total_rows, COUNT(age) AS non_null_age, COUNT(*) - COUNT(age) AS missing_age
FROM students;

-- Detect duplicate rows (recap DSA file 06's hashing-based duplicate detection, same idea in SQL)
SELECT name, age, COUNT(*) AS occurrence_count
FROM students
GROUP BY name, age
HAVING COUNT(*) > 1;

-- Detect outliers using the same IQR logic from Math Foundations file 13
WITH stats AS (
    SELECT
        (SELECT age FROM students ORDER BY age LIMIT 1 OFFSET (SELECT COUNT(*)/4 FROM students)) AS q1,
        (SELECT age FROM students ORDER BY age LIMIT 1 OFFSET (SELECT COUNT(*)*3/4 FROM students)) AS q3
)
SELECT * FROM students, stats WHERE age < q1 - 1.5*(q3-q1) OR age > q3 + 1.5*(q3-q1);
```

Genuinely a real, valuable habit — catching missing data, duplicates, and outliers directly in SQL BEFORE pulling data into a training pipeline saves real time versus discovering these issues deep inside a pandas/scikit-learn workflow, and keeps the "garbage in, garbage out" risk in check from the earliest possible stage.

## Building a proper training dataset — a realistic, complete example

```sql
WITH student_features AS (
    SELECT
        s.id,
        s.age,
        COUNT(e.course) AS num_courses,
        COALESCE(AVG(c.credits), 0) AS avg_credits,
        CASE WHEN s.major = 'AIML' THEN 1 ELSE 0 END AS is_aiml_major
    FROM students s
    LEFT JOIN enrollments e ON s.id = e.student_id
    LEFT JOIN courses c ON e.course = c.course_name
    GROUP BY s.id
)
SELECT * FROM student_features WHERE age IS NOT NULL;
```

Genuinely the realistic shape of a "build my feature table" query — combining files 05 (joins), 06 (CTEs), 04 (aggregation), 02 (CASE), and 12 (COALESCE) into one coherent query that hands off a genuinely ML-ready table straight to `pd.read_sql_query()`.

## Try this

```python
# Using the sqlite3 setup from this whole folder, write ONE comprehensive SQL query
# that builds a feature table for a hypothetical "will this student enroll in 4+ courses"
# classification problem -- include at least one aggregate, one CASE-based encoded feature,
# one COALESCE for missing data, and a WHERE clause excluding invalid rows
# Pull it directly into pandas with pd.read_sql_query(), then confirm the DataFrame
# looks ready to hand to a scikit-learn train_test_split call
```

This exercise is genuinely the complete round-trip this whole file has been building toward — SQL query design feeding directly and cleanly into the ML pipeline already built across the rest of this repo.

## What's next
File 14 — the final SQL → Python/Pandas/ML Bridge file, same style as every other folder's closing bridge file, mapping every SQL concept from files 01-13 back onto its pandas/ML equivalent explicitly.
