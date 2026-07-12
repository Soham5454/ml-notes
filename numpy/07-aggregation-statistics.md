# 07 — Aggregation & Statistics

## Basic aggregations
```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])

arr.sum()        # 31
arr.mean()         # 3.875
arr.min()            # 1
arr.max()              # 9
arr.std()                # standard deviation
arr.var()                  # variance
```

## Index of min/max (not just the value)
```python
arr = np.array([3, 1, 4, 1, 5])
arr.argmin()     # 1   -> index of the smallest value
arr.argmax()       # 4    -> index of the largest value
```
Genuinely useful pattern: this is how you'd find WHICH class a model predicted from an array of probabilities, e.g. `predictions.argmax()` gives you the predicted class index — this is literally how classification models pick their answer.

## Aggregations on 2D arrays with axis
```python
matrix = np.array([[1, 2, 3],
                    [4, 5, 6]])

matrix.sum()          # 21          -> sums EVERYTHING
matrix.sum(axis=0)      # [5 7 9]      -> sum of each column
matrix.sum(axis=1)        # [6 15]        -> sum of each row

matrix.mean(axis=0)         # [2.5 3.5 4.5]  -> mean of each column
matrix.max(axis=1)             # [3 6]           -> max of each row
```

## Median and percentiles
```python
arr = np.array([1, 3, 5, 7, 9])
np.median(arr)          # 5
np.percentile(arr, 50)     # 5   -> 50th percentile = median
np.percentile(arr, 25)       # 25th percentile (Q1)
```

## Correlation (relevant later for feature analysis)
```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([2, 4, 6, 8, 10])
np.corrcoef(a, b)     # correlation matrix — 1.0 means perfectly correlated
```

## Handling NaN values (missing data — very common in real datasets)
```python
arr = np.array([1, 2, np.nan, 4])

arr.sum()          # nan -> a single NaN poisons the whole result
np.nansum(arr)       # 7.0  -> ignores NaN, sums the rest
np.nanmean(arr)        # ignores NaN when computing mean
np.isnan(arr)             # [False False True False]  -> boolean mask of where NaNs are
```
This is exactly the kind of thing that comes up constantly once real (messy) data enters the picture — pandas builds heavily on this NaN-handling behavior.

## Weighted average
```python
values = np.array([80, 90, 70])
weights = np.array([0.5, 0.3, 0.2])
np.average(values, weights=weights)     # weighted mean, different from plain .mean()
```

## Counting values matching a condition
```python
arr = np.array([1, 2, 3, 4, 5, 6])
np.sum(arr > 3)         # 3   -> counts how many elements are True (True acts as 1)
np.count_nonzero(arr > 3)  # same result, more explicit intent
```
This pattern (`sum of a boolean condition`) is genuinely one of the most-used tricks in NumPy/pandas — counting how many rows match a condition without writing a single loop.
