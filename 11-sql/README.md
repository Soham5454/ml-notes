# SQL — A to Z

Notes on SQL, filling the database-querying gap in the roadmap. Every real ML/data pipeline touches a database eventually — this folder covers querying, joining, and reasoning about relational data, cross-referenced back to the pandas notes throughout since it's genuinely the same operations, different syntax.

## Index

| # | Topic | File |
|---|---|---|
| 01 | Introduction to SQL & Databases | [01-introduction-to-sql-and-databases.md](01-introduction-to-sql-and-databases.md) |
| 02 | SELECT & Filtering | [02-select-and-filtering.md](02-select-and-filtering.md) |
| 03 | Sorting, Limiting & Aliasing | [03-sorting-limiting-aliasing.md](03-sorting-limiting-aliasing.md) |
| 04 | Aggregate Functions & GROUP BY / HAVING | [04-aggregate-functions-group-by-having.md](04-aggregate-functions-group-by-having.md) |
| 05 | Joins | [05-joins.md](05-joins.md) |
| 06 | Subqueries & CTEs | [06-subqueries-and-ctes.md](06-subqueries-and-ctes.md) |
| 07 | Window Functions | [07-window-functions.md](07-window-functions.md) |
| 08 | Set Operations (UNION, INTERSECT, EXCEPT) | [08-set-operations.md](08-set-operations.md) |
| 09 | Schema Design, Data Types & Constraints | [09-schema-design-data-types-constraints.md](09-schema-design-data-types-constraints.md) |
| 10 | INSERT, UPDATE, DELETE & Transactions | [10-insert-update-delete-transactions.md](10-insert-update-delete-transactions.md) |
| 11 | Indexes & Query Optimization | [11-indexes-and-query-optimization.md](11-indexes-and-query-optimization.md) |
| 12 | String, Date & NULL Functions | [12-string-date-null-functions.md](12-string-date-null-functions.md) |
| 13 | SQL for Data Science / ML Workflows | [13-sql-for-data-science-ml-workflows.md](13-sql-for-data-science-ml-workflows.md) |
| 14 | SQL → Python/Pandas/ML Bridge | [14-sql-to-python-pandas-ml-bridge.md](14-sql-to-python-pandas-ml-bridge.md) |

## How to read these
Files 01-07 build directly on each other in order — SELECT/filtering, then aggregation, then joins, then subqueries/CTEs, then window functions. Files 08-12 round out the practical toolkit (set operations, schema design, transactions, optimization, utility functions). File 13 connects everything to real ML workflows, and file 14 closes the loop with a full concept map back to pandas. Every file includes runnable SQLite examples.

## Roadmap position
Python Basics → NumPy → pandas → Matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Math Foundations → DSA → **SQL (done)** → Classical NLP → Hugging Face → OpenCV → Flask → MLOps → Cloud → Frontier GenAI → ML System Design → Interview Prep
