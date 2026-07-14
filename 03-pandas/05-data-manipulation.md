# 05 — Data Manipulation

## Sorting
```python
df.sort_values("age")                          # sort by a single column, ascending by default
df.sort_values("age", ascending=False)            # descending
df.sort_values(["age", "marks"])                     # sort by multiple columns (age first, then marks as tiebreaker)
df.sort_index()                                         # sort by the index instead of a column value
```

## apply() — running a custom function across rows/columns
```python
df["age"].apply(lambda x: x + 1)          # apply to a single column (a Series)

def categorize_age(age):
    if age < 18:
        return "minor"
    return "adult"

df["age_group"] = df["age"].apply(categorize_age)     # apply a named function, not just lambda
```

### apply() on an entire DataFrame (axis matters here too, same concept as NumPy)
```python
df.apply(lambda row: row["marks"] * 2, axis=1)     # axis=1 -> apply function ACROSS each row
df.apply(lambda col: col.max(), axis=0)               # axis=0 -> apply function DOWN each column (default)
```

## map() — element-wise transformation, Series only (not full DataFrames)
```python
df["gender"].map({"M": "Male", "F": "Female"})     # maps each value using a dictionary
```
Difference from `.replace()` (file 04): `.map()` sets anything NOT in the dictionary to NaN, while `.replace()` leaves unmatched values unchanged. This distinction bit me once — used `.map()` expecting unmatched values to stay the same, and they silently became NaN instead.

## applymap() — element-wise transformation across the ENTIRE DataFrame
```python
df.applymap(lambda x: x * 2)     # applies to every single cell, not just one column
```
Note: in newer pandas versions this is being replaced by `df.map()` at the DataFrame level too — worth checking the pandas version you're using if this throws a deprecation warning.

## Renaming (recap + more detail from file 01)
```python
df.rename(columns={"old_name": "new_name"}, inplace=True)
df.rename(index={0: "first_row"}, inplace=True)     # can rename index labels too, not just columns
```

## Dropping columns/rows
```python
df.drop("age", axis=1)                # drop a column (axis=1)
df.drop(["age", "marks"], axis=1)        # drop multiple columns
df.drop(0, axis=0)                          # drop a row by index label (axis=0)
df.drop(columns=["age"])                       # alternative syntax, arguably more readable
```

## Adding/removing columns quickly
```python
df["new_col"] = 0                      # add a column, same value for every row
df["total"] = df["marks"] + df["bonus"]   # add a computed column
del df["new_col"]                            # delete a column (older style, still works)
df.pop("total")                                 # deletes AND returns the column, useful if you need the values you're removing
```

## Combining conditions to create new categorical columns (very common pattern)
```python
import numpy as np
df["grade"] = np.where(df["marks"] >= 90, "A",
                np.where(df["marks"] >= 75, "B", "C"))
```
Nested `np.where` for multi-level categorization — this is a genuinely common pattern once you have more than a simple two-way condition. For anything with many categories, `pd.cut` (covered below) is cleaner.

## Binning continuous data into categories — pd.cut
```python
df["age_group"] = pd.cut(df["age"], bins=[0, 12, 18, 60, 100],
                           labels=["child", "teen", "adult", "senior"])
```
This turns a continuous numeric column into clean categorical buckets in one line — much cleaner than writing a chain of if-elif logic manually.

## Value counts and unique values (quick exploration, also touched in file 04)
```python
df["grade"].value_counts()          # frequency of each category
df["grade"].value_counts(normalize=True)     # as proportions/percentages instead of raw counts
df["grade"].nunique()                             # count of DISTINCT values
```

## Chaining operations (a style you'll see everywhere in real pandas code)
```python
result = (df[df["age"] > 18]
            .sort_values("marks", ascending=False)
            .head(10))
```
Wrapping the whole thing in parentheses lets you chain `.method()` calls across multiple lines cleanly — this "method chaining" style is extremely common in real pandas codebases once you get comfortable with it, though it can get hard to debug if a chain gets too long (better to break it into steps while learning).
