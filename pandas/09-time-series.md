# 09 — Time Series

Dates/times show up in an enormous number of real datasets (sales data, sensor logs, stock prices), so pandas has a huge amount of built-in support specifically for this.

## Converting strings to actual datetime objects
```python
df["date"] = pd.to_datetime(df["date"])     # THE most important line -- without this, dates are just plain strings and you can't do date math
```
Note to self, again since it matters: skipping this step means sorting by date might sort ALPHABETICALLY instead of chronologically (since it's just treating dates as text), which is a genuinely sneaky bug if you don't catch it.

### Handling different date formats
```python
pd.to_datetime(df["date"], format="%d-%m-%Y")     # explicitly tell pandas the format if auto-detection guesses wrong
pd.to_datetime(df["date"], errors="coerce")           # invalid dates become NaT (Not a Time) instead of throwing an error
```

## Extracting parts of a date (once it's a proper datetime column)
```python
df["year"] = df["date"].dt.year
df["month"] = df["date"].dt.month
df["day"] = df["date"].dt.day
df["day_of_week"] = df["date"].dt.day_name()     # 'Monday', 'Tuesday', etc.
df["is_weekend"] = df["date"].dt.dayofweek >= 5     # Saturday=5, Sunday=6
```
The `.dt` accessor is the key thing here — same idea as the `.str` accessor for strings (file 10), just for datetime-specific operations.

## Date arithmetic
```python
df["date"] + pd.Timedelta(days=7)     # add 7 days to every date
df["days_since"] = (pd.Timestamp.today() - df["date"]).dt.days     # how many days ago something happened
```

## Setting a datetime column as the index (common for time series analysis)
```python
df.set_index("date", inplace=True)
```

## Resampling — aggregating time series data into different time buckets
```python
df.resample("M").mean()      # monthly average
df.resample("W").sum()          # weekly sum
df.resample("D").count()          # daily count
```
Requires the DataFrame to have a datetime index (from `set_index` above). This is essentially "groupby, but specifically for time" — instead of grouping by a category column, you're grouping by time buckets (day/week/month/year).

### Common resample frequency codes
```
'D'   -> day
'W'   -> week
'M'   -> month end
'MS'  -> month start
'Q'   -> quarter
'Y'   -> year
'H'   -> hour
'T' or 'min'  -> minute
```

## Rolling windows — moving averages, common in time series analysis
```python
df["sales"].rolling(window=7).mean()     # 7-day moving average
df["sales"].rolling(window=7).sum()         # 7-day rolling sum
```
Rolling windows are the standard technique for smoothing out noisy time series data — I've seen this come up specifically in stock price / sales trend analysis examples.

## Shifting data — comparing a value to a previous/future period
```python
df["previous_day_sales"] = df["sales"].shift(1)     # shift down by 1 -- each row now shows the PREVIOUS row's value
df["sales_change"] = df["sales"] - df["sales"].shift(1)     # day-over-day change
```

## Creating a date range manually (useful for generating test data or filling gaps)
```python
pd.date_range(start="2026-01-01", end="2026-01-10")           # every day between two dates
pd.date_range(start="2026-01-01", periods=10, freq="D")           # 10 days starting from a date
pd.date_range(start="2026-01-01", periods=12, freq="M")             # 12 months
```

## Filtering by date range
```python
df[df["date"] >= "2026-01-01"]                                 # simple comparison works directly on datetime columns
df[(df["date"] >= "2026-01-01") & (df["date"] <= "2026-06-30")]   # range filter
df.loc["2026-01-01":"2026-06-30"]                                    # if date is set as the index, can slice directly like this
```
