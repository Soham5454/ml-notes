# 01 — Introduction: Series & DataFrame

## What is pandas and why it exists
NumPy gives you fast array math, but real-world data almost never comes as clean, uniformly-typed numbers — it has column names, mixed data types (numbers + text + dates), missing values, and needs filtering/grouping/joining. Pandas builds on top of NumPy to handle exactly this kind of messy, labeled, tabular data.

## The two core objects

### Series — a single labeled column (1D)
```python
import pandas as pd

s = pd.Series([10, 20, 30, 40])
print(s)
# 0    10
# 1    20
# 2    30
# 3    40
# dtype: int64
```
That left column (0,1,2,3) is the **index** — automatically created, but can be customized.
```python
s = pd.Series([10, 20, 30], index=["a", "b", "c"])
s["b"]     # 20   -> access by label instead of just position
```

### DataFrame — a full table (2D), like an Excel sheet or SQL table
```python
data = {
    "name": ["Soham", "Alex", "Priya"],
    "age": [18, 20, 19],
    "marks": [90, 85, 88]
}
df = pd.DataFrame(data)
print(df)
#     name  age  marks
# 0  Soham   18     90
# 1   Alex   20     85
# 2  Priya   19     88
```
Think of a DataFrame as a dictionary of Series — each column IS a Series, and they all share the same index.

## Basic exploration — the first things you run on ANY new dataset
```python
df.head()          # first 5 rows (default), df.head(10) for first 10
df.tail()             # last 5 rows
df.shape                # (rows, columns) tuple
df.columns                 # column names
df.index                     # the row index
df.dtypes                       # data type of each column
df.info()                          # summary: column names, non-null counts, dtypes, memory usage
df.describe()                         # statistical summary (mean, std, min, max, quartiles) for numeric columns
```
`df.info()` and `df.describe()` are genuinely the first two things I run on any dataset now — info tells you about missing data and types, describe gives you a quick statistical feel for the numbers.

## Selecting a single column vs multiple columns
```python
df["name"]              # returns a Series (single column)
df[["name", "age"]]       # returns a DataFrame (subset of columns) -- note the DOUBLE brackets
```
This double-bracket thing tripped me up initially — single brackets with one column name gives a Series, but to select MULTIPLE columns (even conceptually "just picking two"), you pass a LIST inside the brackets, hence the double `[[ ]]`.

## Column data types
```python
df.dtypes
# name      object     (pandas calls strings 'object' dtype)
# age        int64
# marks        int64
```

## Adding a new column
```python
df["passed"] = df["marks"] >= 50     # vectorized, no loop needed -- same NumPy-style thinking carries over
```

## Renaming columns
```python
df.rename(columns={"name": "student_name"}, inplace=True)     # inplace=True modifies the original DataFrame directly
```
Without `inplace=True`, `.rename()` returns a NEW DataFrame instead of modifying the original — same "returns new vs modifies in place" pattern I saw in NumPy's `.sort()` vs `sorted()`.

## Index — set a column as the index
```python
df.set_index("student_name", inplace=True)     # now you can access rows by name instead of 0,1,2...
df.reset_index(inplace=True)                      # undo it, back to default 0,1,2... index
```

## Series and DataFrame share most NumPy-style operations
```python
df["marks"].mean()       # works just like a NumPy array
df["marks"] * 2             # vectorized multiply, no loop
df["marks"] > 80              # boolean Series, same masking concept from NumPy
```
This is the reassuring part about learning NumPy first — almost everything about vectorization and boolean masking transfers directly, pandas just adds labels and messier-data handling on top.
