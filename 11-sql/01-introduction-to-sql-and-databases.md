# 01 — Introduction to SQL & Databases

Starting the SQL phase — genuinely a real gap, since every ML/DL folder so far assumed data already arrives as a clean CSV/DataFrame. SQL is how that data actually gets stored, queried, and pulled out of a real system in the first place.

## What a relational database actually is

Data organized into **tables** (rows and columns, genuinely just like a pandas DataFrame — recap pandas notes), where tables can be RELATED to each other through shared keys. A **RDBMS** (Relational Database Management System — PostgreSQL, MySQL, SQLite, SQL Server) is the software that stores, manages, and lets you query this data.

```sql
-- A table is genuinely just this, conceptually:
-- students
-- | id | name  | age | major |
-- |----|-------|-----|-------|
-- | 1  | Soham | 20  | AIML  |
-- | 2  | Priya | 21  | CSE   |
```

## Why SQL matters even for an ML-focused profile

Recap the original honest gap-analysis conversation — genuinely every real ML pipeline touches a database eventually: training data lives in a warehouse, feature stores are often SQL-queryable, and "pull the last 90 days of user activity, aggregated by day" is a genuinely common first step before any pandas/PyTorch work even begins. SQL is the interface for that step.

## SQL vs pandas — the same mental operations, different syntax

```sql
-- SQL: get all students majoring in AIML
SELECT * FROM students WHERE major = 'AIML';
```

```python
# pandas equivalent (recap pandas notes)
students_df[students_df['major'] == 'AIML']
```

Genuinely worth internalizing this parallel from day one — SQL isn't a brand new way of thinking, it's the SAME filtering/grouping/joining logic already used in pandas, expressed in a declarative query language instead of imperative Python code. "Declarative" here means SQL describes WHAT result is wanted, not the step-by-step HOW to compute it — the database engine figures out the actual execution plan.

## Setting up SQLite — the easiest way to practice locally

```python
import sqlite3

conn = sqlite3.connect('practice.db')      # creates a local database file
cursor = conn.cursor()

cursor.execute('''
    CREATE TABLE students (
        id INTEGER PRIMARY KEY,
        name TEXT,
        age INTEGER,
        major TEXT
    )
''')

cursor.execute("INSERT INTO students VALUES (1, 'Soham', 20, 'AIML')")
cursor.execute("INSERT INTO students VALUES (2, 'Priya', 21, 'CSE')")
conn.commit()

cursor.execute("SELECT * FROM students")
print(cursor.fetchall())   # [(1, 'Soham', 20, 'AIML'), (2, 'Priya', 21, 'CSE')]
```

Genuinely the most practical way to run every example in this whole SQL folder — SQLite needs zero server setup, the whole database is just a local file, and it supports the vast majority of standard SQL syntax covered here.

## Basic SQL syntax structure — the shape every query shares

```sql
SELECT column1, column2          -- WHAT columns to return
FROM table_name                   -- WHICH table
WHERE condition                    -- FILTER rows (file 02)
GROUP BY column                     -- AGGREGATE grouping (file 04)
HAVING condition                     -- FILTER on aggregated groups (file 04)
ORDER BY column                       -- SORT results (file 03)
LIMIT n;                               -- RESTRICT row count (file 03)
```

Genuinely important to know the EXECUTION order differs from the WRITTEN order above — SQL actually processes roughly: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT. This explains a real, common confusion: WHERE can't filter on an aggregate result (since aggregation hasn't happened yet when WHERE runs), which is precisely why HAVING exists as a separate clause — covered fully in file 04.

## Primary keys and foreign keys — the relationships in "relational"

```sql
CREATE TABLE students (
    id INTEGER PRIMARY KEY,        -- uniquely identifies each row in THIS table
    name TEXT
);

CREATE TABLE enrollments (
    student_id INTEGER,
    course TEXT,
    FOREIGN KEY (student_id) REFERENCES students(id)   -- links to another table's primary key
);
```

A **primary key** uniquely identifies each row within its own table (genuinely similar to a pandas DataFrame's index, recap pandas notes). A **foreign key** is a column in one table that REFERENCES a primary key in another table — this is precisely the mechanism that lets file 05's JOIN operations connect related data living in separate tables, rather than needing everything duplicated into one giant flat table.

## SQL data types — a quick preview, fully covered in file 09

```sql
-- Genuinely the core types worth knowing upfront:
-- INTEGER, REAL/FLOAT, TEXT/VARCHAR, DATE/DATETIME, BOOLEAN
```

## The four categories of SQL commands — worth knowing the taxonomy

- **DQL** (Data Query Language) — `SELECT`, reading data. Files 02-08 focus almost entirely here.
- **DDL** (Data Definition Language) — `CREATE`, `ALTER`, `DROP`, defining table structure (file 09).
- **DML** (Data Manipulation Language) — `INSERT`, `UPDATE`, `DELETE`, changing data (file 10).
- **DCL** (Data Control Language) — `GRANT`, `REVOKE`, permissions — genuinely a "know it exists" topic for this roadmap, not deep-diving into database administration.

## Try this

```python
# Using the sqlite3 setup above, create a second table "courses" with columns
# (id, course_name, credits), insert 3 rows, then write a query to SELECT all
# courses with more than 3 credits
# This is genuinely the smallest possible complete "create, insert, query" cycle --
# worth running end-to-end before moving into the deeper filtering/joining files
```

Getting comfortable with this create-insert-query cycle in SQLite is genuinely the foundation every subsequent file builds on — all the SELECT/JOIN/aggregate syntax in files 02-14 assumes this basic setup is already comfortable.

## What's next
File 02 — SELECT & Filtering, the actual query-reading skill that makes up the bulk of real day-to-day SQL usage.
