# 08 — Pivot Tables & Reshaping

## pivot_table() — Excel-style pivot tables in pandas
```python
df.pivot_table(values="salary", index="department", columns="gender", aggfunc="mean")
```
This is genuinely just a more flexible, more readable version of the `groupby(...).unstack()` pattern from file 06 — `pivot_table` is usually the more intuitive tool to reach for first when you want a proper cross-tabulated summary table.

### Multiple aggregation functions in a pivot table
```python
df.pivot_table(values="salary", index="department", columns="gender", aggfunc=["mean", "count"])
```

### Handling missing combinations
```python
df.pivot_table(values="salary", index="department", columns="gender", aggfunc="mean", fill_value=0)     # fill missing group combinations with 0 instead of NaN
```

### Adding row/column totals
```python
df.pivot_table(values="salary", index="department", columns="gender", aggfunc="mean", margins=True)     # adds an 'All' row and column showing overall totals/averages
```

## pivot() — simpler reshaping, no aggregation (data must already be unique per combination)
```python
df.pivot(index="date", columns="city", values="temperature")
```
Difference from `pivot_table`: `pivot()` assumes there's exactly ONE value per index-column combination already — if there are duplicates, it errors. `pivot_table()` handles duplicates by aggregating them (mean by default). I mix these up sometimes — the rule I use: if I'm not 100% sure the data is already unique per combination, just use `pivot_table` since it's more forgiving.

## melt() — the OPPOSITE of pivot, turning wide data into long/tidy format
```python
wide_df = pd.DataFrame({
    "student": ["Soham", "Alex"],
    "math": [90, 85],
    "science": [88, 92]
})

pd.melt(wide_df, id_vars=["student"], value_vars=["math", "science"],
         var_name="subject", value_name="score")
#   student  subject  score
# 0   Soham     math     90
# 1    Alex     math     85
# 2   Soham  science     88
# 3    Alex  science     92
```
This "long format" is often what's actually needed for plotting with Seaborn (each row is one observation) — this connects directly to something I noticed while learning Seaborn, where wide data needed melting first before certain plot types would work cleanly.

## stack() and unstack() — moving between columns and a MultiIndex
```python
df_multi = df.groupby(["department", "gender"])["salary"].mean()     # MultiIndex Series
df_multi.unstack()      # turns the inner index level (gender) into columns -- produces a proper 2D table
df_multi.unstack().stack()     # stack() reverses it, back to the MultiIndex Series form
```

## Wide vs Long format — why this distinction actually matters
- **Wide format**: each category gets its own column (easy for humans to read, e.g. a column per subject)
- **Long/tidy format**: each row is a single observation, with category as a value in a column (better for plotting libraries, groupby, and most analysis functions)

Most real analysis work (and virtually all Seaborn plotting) expects long/tidy format, so `melt()` is a tool I expect to use constantly once actual project work starts.

## Cross-tabulation (recap from file 06, relevant here as reshaping too)
```python
pd.crosstab(df["department"], df["gender"], values=df["salary"], aggfunc="mean")     # crosstab can also aggregate a specific value column, making it similar to a simple pivot_table
```

## Transposing a DataFrame (swap rows and columns entirely)
```python
df.T     # same concept as NumPy's .T from earlier -- flips rows into columns and vice versa
```
