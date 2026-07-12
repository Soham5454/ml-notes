# 02 — Data Creation & I/O

## Creating DataFrames manually

### From a dictionary of lists (most common way for quick test data)
```python
data = {"name": ["Soham", "Alex"], "age": [18, 20]}
df = pd.DataFrame(data)
```

### From a list of dictionaries (one dict per row — common when data comes from an API/JSON)
```python
data = [
    {"name": "Soham", "age": 18},
    {"name": "Alex", "age": 20}
]
df = pd.DataFrame(data)
```

### From a NumPy array
```python
import numpy as np
arr = np.array([[1, 2], [3, 4]])
df = pd.DataFrame(arr, columns=["col1", "col2"])
```

### Empty DataFrame with defined columns (useful for building up data row by row)
```python
df = pd.DataFrame(columns=["name", "age"])
```

## Reading files — the real everyday use case

### CSV (the most common format you'll deal with)
```python
df = pd.read_csv("data.csv")
df = pd.read_csv("data.csv", index_col=0)          # use first column as index instead of a new one
df = pd.read_csv("data.csv", usecols=["name", "age"])  # load only specific columns (saves memory on huge files)
df = pd.read_csv("data.csv", nrows=100)               # load only first 100 rows (useful for quickly peeking at huge files)
df = pd.read_csv("data.csv", na_values=["NA", "?"])     # tell pandas which strings should be treated as missing/NaN
```

### Excel
```python
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
```
Needs `openpyxl` installed: `pip install openpyxl`

### JSON
```python
df = pd.read_json("data.json")
```

### SQL (reading directly from a database)
```python
import sqlite3
conn = sqlite3.connect("mydatabase.db")
df = pd.read_sql("SELECT * FROM students", conn)
```

## Writing files (saving your work)
```python
df.to_csv("output.csv", index=False)     # index=False -> don't write the row index as a column, usually what you want
df.to_excel("output.xlsx", index=False)
df.to_json("output.json")
```
Forgetting `index=False` on `to_csv` is a classic mistake — you end up with an extra unnamed column full of 0,1,2... every time you re-read the file, since pandas saved the index as a real column by default.

## Quickly peeking at a large file without loading all of it
```python
df = pd.read_csv("huge_file.csv", nrows=5)     # just check the structure/columns first before loading everything
```
Genuinely useful habit before loading something huge — I do this now before committing to loading a full dataset, especially with limited RAM.

## Handling encoding issues (comes up more than you'd expect)
```python
df = pd.read_csv("data.csv", encoding="utf-8")        # default, usually fine
df = pd.read_csv("data.csv", encoding="latin1")          # fallback if utf-8 throws a UnicodeDecodeError
```

## Reading multiple files and combining them
```python
import glob
files = glob.glob("data/*.csv")
df = pd.concat([pd.read_csv(f) for f in files], ignore_index=True)
```
Common pattern when a dataset is split across multiple monthly/daily CSV files — read each one, then stack them all together (concat covered properly in file 07).
