# 06 — Vectorized Operations & Math

## Element-wise arithmetic
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

a + b     # [5 7 9]
a - b       # [-3 -3 -3]
a * b         # [4 10 18]     -> element-wise multiply, NOT matrix multiplication
a / b           # [0.25 0.4 0.5]
a ** 2            # [1 4 9]
a % 2               # [1 0 1]
```
Important: `*` between two arrays is element-wise multiplication, NOT matrix multiplication. That's a separate operation covered in file 09 (linear algebra) — this confused me initially since I expected `*` to behave like matrix math from what I'd read about ML.

## Universal functions (ufuncs) — math functions applied element-wise
```python
arr = np.array([1, 4, 9, 16])

np.sqrt(arr)      # [1. 2. 3. 4.]
np.exp(arr)         # e^x for each element
np.log(arr)           # natural log of each element
np.log2(arr)            # log base 2
np.abs(np.array([-1, -2, 3]))     # [1 2 3]

np.sin(arr)      # trig functions, all element-wise
np.cos(arr)
np.tan(arr)

np.round(np.array([1.234, 5.678]), 2)     # [1.23 5.68]  -> round to 2 decimal places
np.floor(np.array([1.7, 2.3]))               # [1. 2.]
np.ceil(np.array([1.2, 2.8]))                  # [2. 3.]
```

## Comparison operations (return boolean arrays)
```python
arr = np.array([1, 2, 3, 4, 5])
arr > 3         # [False False False True True]
arr == 3          # [False False True False False]
```
This is the foundation for boolean masking (covered properly in file 10) — comparisons give you a boolean array you can then use to filter data.

## Clipping values into a range
```python
arr = np.array([1, 5, 10, 15, 20])
np.clip(arr, 5, 15)     # [5 5 10 15 15]   -> caps values below 5 up to 5, above 15 down to 15
```
Useful for things like clipping gradients or pixel values in ML preprocessing.

## where — conditional element selection
```python
arr = np.array([1, 2, 3, 4, 5])
np.where(arr > 3, arr, 0)     # [0 0 0 4 5]   -> keep value if condition True, else use 0
```
Syntax: `np.where(condition, value_if_true, value_if_false)`. This is basically a vectorized ternary operator, extremely useful once you get used to it instead of writing loops with if-else.

## Cumulative operations
```python
arr = np.array([1, 2, 3, 4])
np.cumsum(arr)      # [1 3 6 10]     -> running total
np.cumprod(arr)        # [1 2 6 24]     -> running product
```

## Applying operations along axes (this connects to aggregation in file 07 too)
```python
matrix = np.array([[1,2,3],[4,5,6]])
matrix.sum(axis=0)     # [5 7 9]     -> sum DOWN each column
matrix.sum(axis=1)       # [6 15]        -> sum ACROSS each row
```
Note to self — I kept mixing up axis=0 vs axis=1 at first. The way I finally remembered it: axis=0 moves DOWN the rows (so you get one result per column), axis=1 moves ACROSS the columns (so you get one result per row).
