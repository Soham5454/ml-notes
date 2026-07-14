# 03 — Indexing & Selection

This is the part of pandas that confused me the most initially, mainly because there are multiple ways to select data and it's not always obvious which one to use. Writing this one carefully.

## Selecting columns (recap from file 01)
```python
df["age"]              # single column -> Series
df[["age", "marks"]]      # multiple columns -> DataFrame
```

## .loc — label-based selection
Use `.loc` when you want to select by LABEL (row index name, column name).
```python
df.loc[0]                    # row with index label 0
df.loc[0:2]                     # rows with index labels 0 through 2 -- INCLUSIVE of 2! (different from Python slicing)
df.loc[0, "name"]                  # specific cell: row 0, column 'name'
df.loc[0:2, ["name", "age"]]         # rows 0-2, specific columns
df.loc[:, "age"]                        # all rows, just the age column
```
Important gotcha: `.loc` slicing is INCLUSIVE of the end label, unlike normal Python slicing (and unlike `.iloc`, below) which excludes the end. This confused me the first time — `df.loc[0:2]` gives you 3 rows (0, 1, 2), not 2.

## .iloc — position-based selection
Use `.iloc` when you want to select by POSITION (like a regular Python list), regardless of what the labels actually are.
```python
df.iloc[0]              # first row by position
df.iloc[0:2]               # first 2 rows -- EXCLUSIVE of end, like normal Python slicing
df.iloc[0, 1]                 # row 0, column 1 (by position)
df.iloc[:, 0]                    # all rows, first column
df.iloc[-1]                        # last row
```

## When labels and positions are different (this is WHY .loc vs .iloc matters)
```python
df2 = df.set_index("name")     # now the index is names, not 0,1,2...
df2.loc["Soham"]                  # works -- selecting by the actual label
df2.iloc[0]                          # ALSO works -- still selects by position, first row regardless of label
df2.loc[0]                              # ERRORS -- there's no row labeled 0 anymore!
```

## Boolean indexing (filtering rows by condition — same masking concept from NumPy)
```python
df[df["age"] > 18]                          # rows where age > 18
df[(df["age"] > 18) & (df["marks"] > 80)]     # multiple conditions -- use & and |, same as NumPy, NOT 'and'/'or'
df[df["name"].isin(["Soham", "Alex"])]          # rows where name is in a list of values
df[~(df["age"] > 18)]                              # NOT condition
```

## .at and .iat — fast single-value access
```python
df.at[0, "name"]      # fastest way to get/set a SINGLE value by label
df.iat[0, 1]             # fastest way to get/set a SINGLE value by position
```
Use these instead of `.loc`/`.iloc` when you specifically only need ONE cell — genuinely faster for that narrow case, though `.loc`/`.iloc` work fine too if performance isn't critical.

## Selecting rows AND filtering columns together
```python
df.loc[df["age"] > 18, ["name", "marks"]]     # filter rows by condition, then pick specific columns
```
This combo (condition inside `.loc`, plus a column list) is genuinely one of THE most common patterns in real pandas code — filter then select.

## query() — an alternative, more readable syntax for filtering
```python
df.query("age > 18 and marks > 80")     # same result as the boolean indexing above, sometimes more readable
```

## Setting values using .loc (safe way to modify data)
```python
df.loc[df["age"] > 18, "status"] = "adult"     # sets 'status' column to 'adult' ONLY for rows matching the condition
```
Using `.loc` for assignment like this avoids the "SettingWithCopyWarning" that pops up when pandas isn't sure if you're modifying the original DataFrame or a temporary copy of it — a warning I ran into a few times before understanding why `.loc` is the safer way to assign values.
