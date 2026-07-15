# 09 — Schema Design, Data Types & Constraints

Recap file 01's brief data-type preview and DDL mention — this file goes deep into actually DESIGNING tables properly, not just querying ones that already exist.

## Common SQL data types

```sql
CREATE TABLE example (
    id INTEGER,             -- whole numbers
    price REAL,               -- floating point (also FLOAT, DOUBLE depending on database)
    name TEXT,                  -- variable-length string (VARCHAR(n) in many databases, with a length limit)
    is_active BOOLEAN,            -- true/false (SQLite stores this as 0/1 internally)
    signup_date DATE,               -- date only, no time
    created_at DATETIME,              -- date AND time
    bio TEXT                            -- large text blocks
);
```

Genuinely important, database-specific note: SQLite is loosely typed (it mostly stores whatever you give it regardless of declared type), while PostgreSQL/MySQL enforce types much more strictly. Worth knowing this varies — code written and tested purely against SQLite can behave differently when deployed against a stricter database engine.

## Choosing appropriate types — genuinely matters for correctness and performance

```sql
-- Genuinely important: NEVER store numbers that need EXACT precision (money, financial data)
-- as REAL/FLOAT -- floating point has known precision issues (recap the general "floating point
-- isn't exact" caution that shows up in the Math Foundations/NumPy notes)

-- WRONG for money:
-- price REAL   -- can silently accumulate tiny rounding errors

-- CORRECT for money:
price DECIMAL(10, 2)   -- exact decimal with 10 total digits, 2 after the decimal point
```

Genuinely a real, practical gotcha worth internalizing early — using floating point for financial calculations is a well-known category of bug, precisely because floating point numbers can't represent most decimal fractions exactly in binary.

## Primary keys — proper syntax and auto-increment

```sql
CREATE TABLE students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,      -- SQLite auto-generates a unique id per row
    name TEXT NOT NULL,
    age INTEGER
);

INSERT INTO students (name, age) VALUES ('Soham', 20);      -- id is auto-assigned, don't need to specify it
```

`AUTOINCREMENT` (or `SERIAL`/`IDENTITY` in PostgreSQL, `AUTO_INCREMENT` in MySQL — genuinely different syntax per database, same underlying idea) means the database handles generating unique IDs automatically, rather than the application needing to track "what's the next available ID" manually.

## Constraints — enforcing data integrity AT THE DATABASE LEVEL

```sql
CREATE TABLE students (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,                    -- cannot be NULL, ever
    email TEXT UNIQUE,                        -- no two rows can have the same email
    age INTEGER CHECK (age >= 0),               -- enforces a validation rule
    major TEXT DEFAULT 'Undeclared'               -- auto-fills this value if none provided
);
```

Genuinely important philosophy worth internalizing: these constraints enforce data quality at the DATABASE level, not just in application code — even if a buggy application tries to insert bad data (a negative age, a duplicate email), the DATABASE ITSELF rejects it. This is a real, meaningful layer of protection beyond just validating input in Python/application code, since it catches errors regardless of WHICH application or script is writing to the database.

```sql
-- Testing constraint enforcement:
INSERT INTO students (id, name, age) VALUES (1, 'Test', -5);
-- This FAILS with a constraint violation error, genuinely by design
```

## Foreign key constraints — enforcing relational integrity

```sql
CREATE TABLE enrollments (
    id INTEGER PRIMARY KEY,
    student_id INTEGER,
    course TEXT,
    FOREIGN KEY (student_id) REFERENCES students(id)
);

-- SQLite requires this to be explicitly turned on:
PRAGMA foreign_keys = ON;

-- Now this FAILS, since student_id=999 doesn't exist in the students table:
INSERT INTO enrollments (student_id, course) VALUES (999, 'ML Basics');
```

Genuinely a real, important safeguard — without foreign key enforcement, it's entirely possible to insert "orphaned" data referencing a student that doesn't exist, silently corrupting the relational integrity that file 05's joins depend on.

## Normalization — organizing tables to avoid redundancy, briefly

Recap file 05's normalization preview. The core normal forms, genuinely worth knowing conceptually even without memorizing formal definitions:

- **1NF (First Normal Form):** each column holds a single, atomic value (no comma-separated lists crammed into one field).
- **2NF:** builds on 1NF, plus every non-key column depends on the WHOLE primary key (relevant mainly for tables with composite/multi-column keys).
- **3NF:** builds on 2NF, plus no non-key column depends on ANOTHER non-key column (eliminates indirect dependencies).

```sql
-- Violates 1NF -- multiple values crammed into one column:
-- students: (id, name, courses)  where courses = "ML Basics, DSA, Web Dev"

-- Properly normalized (1NF+) -- separate rows in a related table instead:
-- students: (id, name)
-- enrollments: (student_id, course)   -- recap file 05's exact setup
```

Genuinely the practical payoff of normalization: avoiding DATA ANOMALIES — if a course name changes, it only needs updating in ONE place, not searched-and-replaced across every student's comma-separated list. The real-world tradeoff (briefly, fully explored once actual performance work happens) is that heavily normalized schemas sometimes need more JOINs (file 05) to reassemble data, which can cost query performance — some systems deliberately "denormalize" for read-heavy workloads, a genuine engineering tradeoff rather than a rule violation.

## ALTER TABLE — modifying an existing table's structure

```sql
ALTER TABLE students ADD COLUMN email TEXT;
ALTER TABLE students RENAME COLUMN major TO field_of_study;
-- Note: SQLite has more LIMITED ALTER TABLE support than PostgreSQL/MySQL --
-- e.g. dropping a column wasn't supported in older SQLite versions, worth checking
-- the specific database's capabilities when working with production schemas
```

## DROP TABLE — genuinely destructive, worth extra caution

```sql
DROP TABLE IF EXISTS old_table;      -- IF EXISTS avoids an error if the table's already gone
```

Genuinely worth flagging with real caution: `DROP TABLE` deletes the table AND all its data, irreversibly (barring a backup) — a real, common source of catastrophic mistakes when run against a production database instead of a test/practice one. Always double-check which database connection is active before running destructive DDL commands.

## Try this

```sql
-- Design a small schema for a simple blog: a "users" table (id, username, email UNIQUE,
-- joined_date) and a "posts" table (id, user_id FOREIGN KEY, title, content, created_at)
-- Add appropriate constraints: NOT NULL where it matters, a CHECK constraint ensuring
-- title isn't empty, and a DEFAULT value for created_at
-- Then deliberately try to violate each constraint with a bad INSERT, and confirm
-- the database correctly rejects it
```

Deliberately trying to BREAK the constraints (rather than only testing valid inserts) is genuinely the best way to confirm the schema is actually enforcing what it's supposed to, rather than assuming it works.

## What's next
File 10 — INSERT, UPDATE, DELETE & Transactions, moving from table DESIGN into actually MODIFYING data safely, including the transaction/ACID concepts that keep data consistent when things go wrong mid-operation.
