# 10 — INSERT, UPDATE, DELETE & Transactions

Recap file 01's DML category — moving from designing tables (file 09) and querying them (files 02-08) into actually MODIFYING data, plus the transaction concepts that keep those modifications safe.

## INSERT — adding new rows

```sql
INSERT INTO students (name, age, major) VALUES ('Soham', 20, 'AIML');

-- Multiple rows in one statement -- genuinely more efficient than separate INSERTs
INSERT INTO students (name, age, major) VALUES
    ('Priya', 21, 'CSE'),
    ('Amit', 22, 'ECE'),
    ('Neha', 20, 'AIML');

-- Inserting from ANOTHER query's results -- genuinely useful for copying/archiving data
INSERT INTO alumni (name, graduation_year)
SELECT name, 2026 FROM students WHERE major = 'AIML';
```

## UPDATE — modifying existing rows

```sql
UPDATE students SET age = 21 WHERE name = 'Soham';

UPDATE students SET major = 'AIML', age = age + 1 WHERE id = 1;      -- multiple columns, and a computed value
```

**Genuinely the single most dangerous mistake in all of SQL, worth extreme caution:**

```sql
-- FORGETTING the WHERE clause updates EVERY ROW in the table:
-- UPDATE students SET major = 'Undeclared';    -- this wipes EVERY student's major, not just one
```

Real, practical habit worth building permanently: ALWAYS write and verify the `WHERE` clause with a `SELECT` FIRST, confirm it targets exactly the intended rows, THEN convert it to an `UPDATE`/`DELETE`.

```sql
-- SAFE WORKFLOW:
SELECT * FROM students WHERE name = 'Soham';          -- 1. verify this returns EXACTLY the right row(s)
UPDATE students SET age = 21 WHERE name = 'Soham';       -- 2. only then run the actual update
```

## DELETE — removing rows

```sql
DELETE FROM students WHERE age < 18;

-- Same genuinely dangerous mistake pattern as UPDATE:
-- DELETE FROM students;    -- no WHERE clause -- deletes EVERY row in the table
```

Genuinely the same safe-workflow discipline applies: `SELECT` first to verify, THEN `DELETE`. This habit is honestly one of the most practically important things in this entire SQL folder — a missing `WHERE` clause is a real, common cause of serious production incidents.

## TRUNCATE — a faster, different way to remove ALL rows (where supported)

```sql
-- PostgreSQL/MySQL syntax (not standard SQLite)
TRUNCATE TABLE students;
```

Genuinely faster than `DELETE FROM students` (no `WHERE`) for removing ALL rows, since it doesn't log each individual row deletion — but it also typically can't be rolled back in a transaction the same way `DELETE` can in most systems, and resets auto-increment counters. Worth knowing it exists and behaves differently, not simply a faster synonym for unconditional DELETE.

## Transactions — genuinely one of the most important concepts in this file

A transaction groups multiple statements into ONE atomic unit — either ALL of them succeed, or NONE of them do (if something fails partway, everything gets rolled back).

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;      -- withdraw from account 1
UPDATE accounts SET balance = balance + 100 WHERE id = 2;       -- deposit into account 2

COMMIT;      -- only NOW are both changes permanently saved
```

Genuinely the exact real-world motivation: a bank transfer needs BOTH updates to happen together — if the first succeeds but the second fails (server crash, network error, whatever), the money would simply VANISH without a transaction wrapping both statements. `ROLLBACK` undoes everything since the last `BEGIN` if something goes wrong:

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Something goes wrong here (application detects an error, insufficient funds, etc.)
ROLLBACK;      -- undoes the balance change above -- as if it never happened
```

## ACID — the formal properties transactions guarantee

- **Atomicity** — all operations in a transaction succeed together, or none do (recap the bank transfer example directly).
- **Consistency** — a transaction moves the database from one VALID state to another, never violating constraints (recap file 09) partway through.
- **Isolation** — concurrent transactions don't interfere with each other's intermediate states — genuinely important in real multi-user systems where many people are reading/writing simultaneously.
- **Durability** — once a transaction is COMMITTED, it's permanently saved, even if the system crashes immediately after.

Genuinely worth knowing this acronym by name — it's a standard interview/practical-knowledge topic, and understanding WHY each property matters (using the bank transfer example as the anchor) is more valuable than memorizing the definitions alone.

## Isolation levels — a genuinely deeper topic, worth knowing exists

```sql
-- Different databases/settings allow controlling HOW isolated concurrent transactions are:
-- READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
-- (from least to most strict isolation, with corresponding tradeoffs in performance vs consistency)
```

Genuinely a "know it exists" flag rather than something to master here — real production database work eventually requires understanding these tradeoffs (e.g. "dirty reads," "phantom reads"), but it's a deeper topic than this A-to-Z foundational pass needs to cover exhaustively.

## UPSERT — INSERT or UPDATE depending on whether a row exists

```sql
-- SQLite/PostgreSQL syntax
INSERT INTO students (id, name, age) VALUES (1, 'Soham', 21)
ON CONFLICT(id) DO UPDATE SET age = 21;
```

Genuinely a common real pattern — "insert this record, but if it already exists (based on a unique constraint), update it instead" — avoids needing a separate `SELECT` to check existence first, then branching between `INSERT` and `UPDATE` manually in application code.

## Try this

```sql
-- Using the students table, write a full transaction that: (1) inserts a new student,
-- (2) updates an existing student's age, (3) deletes a student who is under 18
-- Wrap all three in BEGIN TRANSACTION / COMMIT
-- Then deliberately write a version where step 2 references a non-existent condition
-- (or otherwise would fail), and use ROLLBACK to confirm NONE of the three changes
-- persist afterward -- verify with a SELECT * FROM students before and after
```

Actually watching a `ROLLBACK` correctly undo ALL three operations (not just the failed one) is genuinely the clearest way to internalize what "atomic" means in ACID — a multi-statement operation behaving as one indivisible unit.

## What's next
File 11 — Indexes & Query Optimization, moving from "writing correct queries" into "writing FAST queries" — genuinely essential once tables grow beyond toy-sized practice data.
