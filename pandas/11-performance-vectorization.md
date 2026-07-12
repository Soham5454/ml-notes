# 11 — Performance & Vectorization

Same underlying philosophy as NumPy — avoid explicit Python loops, let pandas/NumPy handle the iteration internally in compiled code. This matters more and more as datasets get bigger.

## Why loops over DataFrames are slow
```python
# SLOW -- iterating row by row
total = 0
for index, row in df.iterrows():
    total += row["salary"]

# FAST -- vectorized
total = df["salary"].sum()
```
`iterrows()` exists and works, but it's genuinely one of the slowest ways to do anything in pandas — it recreates a Series object for every single row, which has real overhead at scale.

## Vectorized operations vs apply() vs iterrows() — the actual speed hierarchy
From fastest to slowest, roughly:
1. **Vectorized operations** (`df["col"] * 2`, built-in pandas/NumPy methods) — fastest, always prefer this
2. **`.apply()`** — slower than vectorized, but faster than manual loops, use when there's no built-in vectorized equivalent
3. **`.iterrows()` / manual for-loops** — slowest, avoid unless genuinely necessary (complex row-by-row logic that can't be vectorized)

```python
# Vectorized (fastest)
df["bonus"] = df["salary"] * 0.1

# apply (slower, but still reasonable)
df["bonus"] = df["salary"].apply(lambda x: x * 0.1)

# iterrows (slowest, avoid for this kind of task)
for i, row in df.iterrows():
    df.at[i, "bonus"] = row["salary"] * 0.1
```
I default to checking "is there a vectorized way to do this" FIRST before reaching for `.apply()`, and only reach for a real loop if I genuinely can't figure out a vectorized approach.

## itertuples() — a faster alternative to iterrows() IF you truly need to iterate
```python
for row in df.itertuples():
    print(row.name, row.age)     # noticeably faster than iterrows(), though still slower than proper vectorization
```

## Memory usage — checking and reducing it
```python
df.memory_usage(deep=True)          # memory usage per column, deep=True accounts for actual string content size too
df.info(memory_usage="deep")           # summary including accurate memory usage
```

### Reducing memory with better dtypes
```python
df["category_col"] = df["category_col"].astype("category")     # huge memory savings for columns with few repeated unique values (like 'department', 'gender')
df["small_number_col"] = df["small_number_col"].astype("int8")    # use smaller int types when the value range allows it, instead of the default int64
```
This connects to the exact same dtype/memory concept from NumPy (file 11 there) — smaller dtypes mean less RAM used, which matters a lot with genuinely large datasets on a machine with limited RAM.

## Reading large files efficiently — chunking
```python
chunks = pd.read_csv("huge_file.csv", chunksize=100000)     # reads the file in chunks of 100,000 rows instead of loading everything into memory at once
for chunk in chunks:
    process(chunk)     # process each chunk separately, e.g. filtering, aggregating partial results
```
Genuinely useful technique for files too large to fit in RAM all at once — process piece by piece instead of loading the whole thing.

## Using query() and eval() for faster filtering/computation on large DataFrames
```python
df.query("age > 18 and marks > 80")          # can be faster than boolean indexing on very large DataFrames
df.eval("total = marks + bonus")                # computes and assigns in one step, can be faster than a normal assignment on big data
```

## Avoiding chained indexing (a performance AND correctness issue)
```python
# BAD -- chained indexing, can be slow AND trigger SettingWithCopyWarning
df[df["age"] > 18]["status"] = "adult"

# GOOD -- single .loc call
df.loc[df["age"] > 18, "status"] = "adult"
```
This connects back to the `.loc` assignment habit from file 03 — beyond just being "the correct way," it's also genuinely faster since it avoids creating an intermediate temporary DataFrame.

## Profiling — actually measuring what's slow instead of guessing
```python
import time
start = time.time()
result = df.groupby("department")["salary"].mean()
print(time.time() - start)
```
Same simple timing approach from the NumPy notes — worth actually measuring before assuming something needs optimizing, since premature optimization on small dataframes is often wasted effort.
