# 04 — Data Cleaning

Real datasets are messy — missing values, duplicate rows, wrong data types. This is probably the single most important practical skill in pandas since almost no real dataset arrives clean.

## Detecting missing values
```python
df.isnull()          # DataFrame of True/False, True where value is missing
df.isnull().sum()       # count of missing values PER COLUMN -- extremely common first check
df.isnull().sum().sum()    # total missing values in the entire DataFrame
df.notnull()               # opposite of isnull()
```
`df.isnull().sum()` is genuinely one of the first commands I run on any new dataset now, right after `.info()`.

## Dropping missing values
```python
df.dropna()                    # drops ANY row with at least one missing value
df.dropna(axis=1)                 # drops COLUMNS with any missing value instead
df.dropna(subset=["age"])           # only drop rows where 'age' specifically is missing
df.dropna(thresh=2)                    # keep rows with at least 2 non-null values
```
Careful with plain `df.dropna()` on real data — it's easy to accidentally drop way more rows than expected if even one column has scattered missing values across many rows.

## Filling missing values
```python
df.fillna(0)                              # replace ALL missing values with 0
df["age"].fillna(df["age"].mean(), inplace=True)     # fill missing ages with the column's mean -- very common
df.fillna(method="ffill")                    # forward fill -- use the previous valid value
df.fillna(method="bfill")                      # backward fill -- use the next valid value
df.fillna({"age": 0, "name": "Unknown"})          # different fill values per column
```
Filling with the mean/median is the standard go-to for numeric columns in early-stage data cleaning, though which strategy is "correct" really depends on the dataset and what the missing values actually represent.

## Checking and removing duplicates
```python
df.duplicated()          # boolean Series, True where a row is a duplicate of an earlier row
df.duplicated().sum()      # count of duplicate rows
df.drop_duplicates()          # removes duplicate rows, keeps the FIRST occurrence by default
df.drop_duplicates(subset=["name"])     # consider only the 'name' column when checking for duplicates
df.drop_duplicates(keep="last")            # keep the LAST occurrence instead of the first
```

## Changing data types
```python
df["age"] = df["age"].astype(int)          # convert column to a specific type
df["age"] = df["age"].astype(float)
df["date"] = pd.to_datetime(df["date"])       # convert a string column to actual datetime objects (huge deal, covered more in file 09)
df["category"] = df["category"].astype("category")   # 'category' dtype saves memory for columns with few unique repeated values
```
Note to self: forgetting to convert date strings to actual datetime objects means you can't do date math (subtracting dates, extracting month/year, etc.) — this tripped me up before I understood `pd.to_datetime` was necessary.

## Renaming and cleaning column names (common with real-world messy headers)
```python
df.columns = df.columns.str.strip()          # remove leading/trailing whitespace from column names
df.columns = df.columns.str.lower()             # lowercase all column names
df.columns = df.columns.str.replace(" ", "_")      # replace spaces with underscores
```

## Replacing specific values
```python
df["gender"].replace({"M": "Male", "F": "Female"}, inplace=True)     # replace specific values with mapped values
df.replace(-1, np.nan, inplace=True)                                    # treat -1 as a missing value marker (common in older datasets)
```

## Detecting and handling outliers (a lighter cleaning task)
```python
Q1 = df["marks"].quantile(0.25)
Q3 = df["marks"].quantile(0.75)
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers = df[(df["marks"] < lower_bound) | (df["marks"] > upper_bound)]
df_clean = df[(df["marks"] >= lower_bound) & (df["marks"] <= upper_bound)]     # filter them out
```
This is the standard IQR (interquartile range) method for flagging outliers — a genuinely common first-pass technique before deciding whether to remove, cap, or investigate them further.

## Checking for consistency issues (things .info() and .describe() won't catch on their own)
```python
df["gender"].unique()          # see all distinct values -- catches typos like 'Male', 'male', 'M ' (trailing space) being treated as different categories
df["gender"].value_counts()      # frequency of each unique value -- also useful for spotting rare/weird entries
```
