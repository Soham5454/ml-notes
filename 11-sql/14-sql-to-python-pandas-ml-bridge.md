# 14 — SQL → Python/Pandas/ML Bridge

Same purpose as every other folder's closing bridge file — pulling files 01-13 together and mapping every SQL concept explicitly back onto its pandas/ML equivalent, closing the loop on this entire SQL phase.

## The complete concept map — SQL to pandas to ML

| SQL Concept | File | pandas / ML Equivalent |
|---|---|---|
| `SELECT`, `WHERE` | 02 | Column selection, boolean masking (`df[df['col'] > x]`) |
| `ORDER BY`, `LIMIT` | 03 | `.sort_values()`, `.head(n)` |
| `GROUP BY`, aggregates | 04 | `.groupby().agg()` |
| `HAVING` | 04 | `.groupby().filter()` |
| `JOIN` (all types) | 05 | `pd.merge()` with `how='inner'/'left'/'right'/'outer'` |
| Subqueries, CTEs | 06 | Chained/intermediate DataFrame assignments |
| Window functions | 07 | `.groupby().transform()`, `.rank()`, `.shift()` (LAG/LEAD) |
| `UNION`, `INTERSECT` | 08 | `pd.concat()`, set operations on Series/Index |
| Schema/constraints | 09 | DataFrame dtypes, validation logic |
| `INSERT`/`UPDATE`/`DELETE` | 10 | DataFrame row addition/modification (though pandas isn't transactional) |
| Indexes | 11 | DataFrame's own `.index`, sorted lookups |
| String/date/NULL functions | 12 | `.str` accessor methods, `pd.to_datetime()`, `.fillna()` |
| Full pipeline | 13 | `pd.read_sql_query()` → scikit-learn/XGBoost/PyTorch |

## Joins — the deepest, most directly transferable connection

Recap file 05 fully — this mapping is genuinely EXACT, not just similar:

```sql
SELECT s.name, e.course
FROM students s
LEFT JOIN enrollments e ON s.id = e.student_id;
```

```python
pd.merge(students_df, enrollments_df, left_on='id', right_on='student_id', how='left')
```

Genuinely the SAME operation, same NULL-filling behavior for unmatched rows, same underlying relational logic — just SQL syntax versus pandas method syntax. Someone comfortable with one should be able to read the other almost immediately once this mapping clicks.

## GROUP BY / aggregation — recap file 04, the second deepest connection

```sql
SELECT major, AVG(age) AS avg_age, COUNT(*) AS count
FROM students
GROUP BY major
HAVING COUNT(*) > 3;
```

```python
(students_df.groupby('major')
    .agg(avg_age=('age', 'mean'), count=('age', 'size'))
    .query('count > 3'))
```

Genuinely worth noting the STRUCTURAL parallel between SQL's `HAVING` and pandas' post-aggregation `.query()`/filtering — both exist specifically because you can't filter on an aggregate result using the SAME mechanism used for filtering raw rows (`WHERE` vs `HAVING` in SQL; boolean masking on raw rows vs filtering an already-aggregated DataFrame in pandas).

## Window functions — recap file 07, connects to `.transform()` and `.shift()`

```sql
SELECT name, major, age,
    AVG(age) OVER (PARTITION BY major) AS major_avg_age,
    ROW_NUMBER() OVER (PARTITION BY major ORDER BY age DESC) AS rank_in_major
FROM students;
```

```python
students_df['major_avg_age'] = students_df.groupby('major')['age'].transform('mean')
students_df['rank_in_major'] = students_df.groupby('major')['age'].rank(ascending=False, method='first')
```

Recap file 07's core insight (window functions keep every row, unlike GROUP BY's collapsing) — `.transform()` is precisely pandas' equivalent of that exact behavior, genuinely not a coincidence, both solve the identical "annotate every row with group-level context" problem.

## Where SQL wins vs where pandas wins — the honest, practical comparison

**SQL genuinely wins when:**
- Data lives in a database and doesn't need to leave it for simple aggregation/filtering (recap file 13's "push work to the database" point).
- The dataset is too large to comfortably fit in memory as a DataFrame.
- Multiple people/applications need to query the SAME data consistently (a database is a shared, managed resource; a pandas DataFrame is local to one Python process).
- Data integrity constraints (file 09) need enforcing at the storage level, not just validated in application code.

**pandas genuinely wins when:**
- Doing exploratory, iterative analysis — trying different transformations quickly (recap the pandas/Seaborn EDA notes).
- Feature engineering needs custom Python logic (arbitrary functions, complex conditionals) beyond what SQL's built-in functions (file 12) comfortably express.
- The data needs to flow directly into scikit-learn/XGBoost/PyTorch (recap those folders) — pandas DataFrames are the standard interface those libraries expect.

**Honest, realistic answer:** real ML workflows genuinely use BOTH — SQL to extract and pre-aggregate from the source database (file 13), pandas to do the final feature engineering and modeling prep, exactly the round-trip demonstrated in file 13's closing example.

## Data integrity — SQL's structural guarantees vs pandas' lack thereof

Recap file 09's constraints (`NOT NULL`, `UNIQUE`, `CHECK`, foreign keys) — genuinely worth stating explicitly: pandas DataFrames have NO equivalent built-in enforcement. A DataFrame will happily hold duplicate "unique" IDs, NULL values in a column that should never be null, or an age of -50, with zero complaint. This is precisely why real production data pipelines often validate data quality checks (recap file 13's data-quality queries) EITHER at the database level (constraints) OR with a dedicated Python validation step, rather than assuming pandas will catch bad data on its own.

## The complete pipeline, one more time, fully annotated

```python
import sqlite3
import pandas as pd
from sklearn.model_selection import train_test_split

conn = sqlite3.connect('production.db')

# Step 1: SQL does the heavy join + aggregation (files 04, 05, 06, 13)
query = """
    WITH features AS (
        SELECT s.id, s.age,
            COUNT(e.course) AS num_courses,
            COALESCE(AVG(c.credits), 0) AS avg_credits
        FROM students s
        LEFT JOIN enrollments e ON s.id = e.student_id
        LEFT JOIN courses c ON e.course = c.course_name
        GROUP BY s.id
    )
    SELECT * FROM features WHERE age IS NOT NULL;
"""
df = pd.read_sql_query(query, conn)          # Step 2: hand off to pandas

# Step 3: further feature engineering in pandas if needed (recap pandas notes)
df['courses_per_year_of_age'] = df['num_courses'] / df['age']

# Step 4: standard ML pipeline from here (recap scikit-learn/XGBoost/PyTorch notes)
X_train, X_test = train_test_split(df, test_size=0.2, random_state=42)
```

Genuinely the realistic, complete shape of how this SQL folder connects into EVERY prior folder in this repo — the database is where real projects genuinely start, not a CSV file sitting conveniently in a project directory.

## Roadmap position, updated
Python Basics → NumPy → pandas → Matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Math Foundations → DSA → **SQL (done)** → Classical NLP → Hugging Face → OpenCV → Flask → MLOps → Cloud → Frontier GenAI → ML System Design → Interview Prep
