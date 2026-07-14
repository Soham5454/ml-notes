# 07 — Merging, Joining & Concatenating

Real data is rarely in one single table — you often need to combine multiple DataFrames (like joining tables in SQL). Pandas has three main tools for this, and picking the right one depends on what you're actually trying to do.

## concat() — stacking DataFrames together (no relationship needed, just combining)
```python
df1 = pd.DataFrame({"name": ["Soham", "Alex"], "age": [18, 20]})
df2 = pd.DataFrame({"name": ["Priya", "Raj"], "age": [19, 21]})

pd.concat([df1, df2])                     # stacks rows on top of each other (like appending)
pd.concat([df1, df2], ignore_index=True)     # resets the index to a clean 0,1,2,3... instead of keeping duplicate 0,1,0,1
pd.concat([df1, df2], axis=1)                  # stacks side by side (adds columns instead of rows)
```
`ignore_index=True` is something I forget constantly and then wonder why my index has duplicates after combining files — worth remembering as a default habit when concatenating.

## merge() — SQL-style joins based on a common column (the real power tool)
```python
students = pd.DataFrame({"student_id": [1, 2, 3], "name": ["Soham", "Alex", "Priya"]})
marks = pd.DataFrame({"student_id": [1, 2, 4], "marks": [90, 85, 70]})

pd.merge(students, marks, on="student_id")     # joins on the shared 'student_id' column
```

### Types of joins (this is the part that actually matters — same concept as SQL joins)
```python
pd.merge(students, marks, on="student_id", how="inner")     # only rows where student_id exists in BOTH tables (default)
pd.merge(students, marks, on="student_id", how="left")         # ALL rows from students, matching marks where available (NaN if missing)
pd.merge(students, marks, on="student_id", how="right")           # ALL rows from marks, matching students where available
pd.merge(students, marks, on="student_id", how="outer")              # ALL rows from BOTH, NaN wherever there's no match
```
Picking the right `how` genuinely matters — `inner` silently drops unmatched rows (student_id=3 and 4 both disappear in the example above), which caused me confusion once when row counts didn't add up as expected until I realized `inner` was quietly excluding non-matches.

### Merging when column names differ between the two tables
```python
pd.merge(students, marks, left_on="student_id", right_on="id")     # when the join column has different names in each DataFrame
```

### Merging on multiple columns
```python
pd.merge(df1, df2, on=["student_id", "year"])     # requires BOTH columns to match
```

## join() — merging using the INDEX instead of a column
```python
students_indexed = students.set_index("student_id")
marks_indexed = marks.set_index("student_id")

students_indexed.join(marks_indexed)     # joins using the shared index, similar result to merge but index-based
```
`.join()` is essentially a shortcut for merging on the index specifically — I mostly use `.merge()` since column-based joining feels more explicit and readable, but `.join()` shows up in code that's already index-heavy.

## Checking for duplicate keys before merging (a genuinely useful habit)
```python
students["student_id"].duplicated().sum()     # check for duplicate IDs BEFORE merging -- duplicates on the join key can silently multiply row counts unexpectedly
```
This one's a real gotcha: if the join column has duplicate values on either side, a merge can produce far more rows than either original table had (every match creates a new row) — checking for duplicates first avoids a confusing "why did my row count explode" moment.

## Indicator column — seeing WHERE each row came from
```python
pd.merge(students, marks, on="student_id", how="outer", indicator=True)
```
Adds a `_merge` column showing whether each row came from `left_only`, `right_only`, or `both` — genuinely useful for debugging an outer join and understanding exactly which rows didn't match.

## Suffix handling for overlapping column names
```python
pd.merge(df1, df2, on="id", suffixes=("_left", "_right"))     # if both DataFrames have a column with the same name (besides the join key), pandas auto-adds suffixes to disambiguate
```
