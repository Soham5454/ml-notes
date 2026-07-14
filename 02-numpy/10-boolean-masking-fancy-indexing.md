# 10 — Boolean Masking & Fancy Indexing

This is one of the most genuinely useful NumPy skills for real data work — filtering data without writing a single explicit loop.

## Creating a boolean mask
```python
arr = np.array([1, 2, 3, 4, 5, 6])
mask = arr > 3
print(mask)     # [False False False True True True]
```
A mask is just a boolean array, same shape as the original, marking which elements meet a condition.

## Using the mask to filter data
```python
arr = np.array([1, 2, 3, 4, 5, 6])
arr[arr > 3]     # [4 5 6]   -> only the elements where the mask is True
```
This single line replaces what would otherwise be a for-loop with an if-check and a manual list append. Once this clicked for me it changed how I think about filtering data entirely.

## Combining multiple conditions
Important: use `&` (and), `|` (or), `~` (not) — NOT Python's regular `and`/`or`/`not`, since those don't work element-wise on arrays.
```python
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8])

arr[(arr > 2) & (arr < 7)]     # [3 4 5 6]    -> AND condition
arr[(arr < 3) | (arr > 6)]       # [1 2 7 8]     -> OR condition
arr[~(arr > 3)]                    # [1 2 3]         -> NOT condition
```
Each condition MUST be wrapped in parentheses — Python's operator precedence with `&`/`|` is different from `and`/`or`, and skipping parentheses causes errors. Learned this one the hard way.

## Modifying values using a mask
```python
arr = np.array([1, 2, 3, 4, 5, 6])
arr[arr > 3] = 0     # replaces all values greater than 3 with 0
print(arr)              # [1 2 3 0 0 0]
```
Extremely common in data cleaning — e.g. capping outliers, replacing invalid sensor readings, etc.

## Masking on 2D arrays
```python
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
matrix[matrix > 5]     # [6 7 8 9]   -> returns a FLATTENED 1D array of matches, loses the original 2D shape
```
Worth remembering: masking a 2D array always gives you back a 1D result, since there's no guarantee the True values form a nice rectangular sub-shape.

## Fancy indexing recap (also touched briefly in file 03)
```python
arr = np.array([10, 20, 30, 40, 50])
indices = [0, 2, 4]
arr[indices]     # [10 30 50]
```
Difference from boolean masking: fancy indexing uses a list of specific POSITIONS, boolean masking uses a list of True/False the SAME LENGTH as the array. Both are useful in different scenarios — masking when you have a condition, fancy indexing when you already know exact positions you want.

## np.where for finding indices (not just filtering values)
```python
arr = np.array([1, 5, 2, 8, 3])
np.where(arr > 3)     # (array([1, 3]),)   -> the INDICES where the condition is True
```
Note this returns a tuple of arrays (one per dimension) — for a 1D array you'd typically do `np.where(arr > 3)[0]` to get just the indices directly.

## Practical example — a pattern I'll use constantly with real data
```python
scores = np.array([45, 88, 92, 34, 67, 78])
passing_scores = scores[scores >= 50]      # [88 92 67 78]
failing_count = np.sum(scores < 50)          # 2
average_of_passing = passing_scores.mean()     # mean of only the passing scores
```
This exact pattern — mask, filter, then aggregate — is basically the backbone of a huge amount of real data analysis work, and it's all one or two lines instead of loops.
