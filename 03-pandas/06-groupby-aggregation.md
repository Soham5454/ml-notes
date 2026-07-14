# 06 — GroupBy & Aggregation

This is probably the single most powerful pandas feature for actual data analysis — "group rows by some category, then compute something per group" covers an enormous amount of real analytical work.

## The basic idea — split, apply, combine
GroupBy works in three conceptual steps: **split** the data into groups based on a column's values, **apply** some function to each group independently, then **combine** the results back together.

```python
df.groupby("department")["salary"].mean()     # average salary PER department
```

## Basic groupby aggregations
```python
df.groupby("department")["salary"].sum()          # total salary per department
df.groupby("department")["salary"].count()           # number of employees per department
df.groupby("department")["salary"].min()                # min salary per department
df.groupby("department")["salary"].max()                   # max salary per department
df.groupby("department").size()                                # row count per group (works even without picking a specific column)
```

## Grouping by multiple columns
```python
df.groupby(["department", "gender"])["salary"].mean()     # average salary per department AND gender combination
```
Result is a Series with a MultiIndex (department, gender) — worth knowing this exists since it looks a little different from a normal single-level index the first time you see it.

## Multiple aggregations at once — .agg()
```python
df.groupby("department")["salary"].agg(["mean", "min", "max", "count"])     # multiple stats in ONE call, one row per department
```

### Different aggregations for different columns
```python
df.groupby("department").agg({
    "salary": "mean",
    "age": "max",
    "name": "count"
})
```

### Custom aggregation functions
```python
df.groupby("department")["salary"].agg(lambda x: x.max() - x.min())     # custom range calculation per group
```

## Resetting the index after groupby (very common, since groupby results have the grouped column as the index)
```python
result = df.groupby("department")["salary"].mean().reset_index()     # turns the department index back into a normal column
```
I do this pretty much every time after a groupby now, since the grouped-by column being the index instead of a normal column trips up later steps (like plotting or merging) if you're not expecting it.

## Iterating over groups (rarely needed, but good to know it exists)
```python
for name, group in df.groupby("department"):
    print(name)          # the department name
    print(group)            # the sub-DataFrame for just that department
```

## Filtering groups based on a group-level condition
```python
df.groupby("department").filter(lambda x: x["salary"].mean() > 50000)     # keep only rows belonging to departments where the average salary exceeds 50000
```

## transform() — apply a function per group, but keep the ORIGINAL shape
```python
df["dept_avg_salary"] = df.groupby("department")["salary"].transform("mean")
```
Difference from `.agg()`: `.agg()` collapses each group down to ONE row, `.transform()` broadcasts the group's result back to EVERY row in that group, so the output has the same number of rows as the original DataFrame. This is genuinely useful for creating comparison columns, like "how does this employee's salary compare to their department's average."

## Pivot-table style groupby (covered more fully in file 08)
```python
df.groupby(["department", "gender"])["salary"].mean().unstack()     # reshapes the MultiIndex result into a proper 2D table, department as rows, gender as columns
```

## crosstab — quick frequency tables between two categorical columns
```python
pd.crosstab(df["department"], df["gender"])     # count of each department-gender combination, laid out as a table
pd.crosstab(df["department"], df["gender"], normalize=True)     # as proportions instead of raw counts
```
`crosstab` is basically a specialized, simpler version of groupby+count specifically for comparing two categorical columns against each other — comes up a lot in exploratory data analysis (EDA).
