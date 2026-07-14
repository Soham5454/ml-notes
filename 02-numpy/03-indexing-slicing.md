# 03 — Indexing & Slicing

## Basic 1D indexing (same as Python lists)
```python
arr = np.array([10, 20, 30, 40, 50])
arr[0]        # 10
arr[-1]        # 50
arr[1:4]         # [20 30 40]
arr[::-1]          # [50 40 30 20 10]   -> reversed
```

## 2D indexing — this is where it gets different from plain lists
```python
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

matrix[0]           # [1 2 3]        -> entire first row
matrix[0][1]          # 2               -> row 0, then column 1 (works, but slower)
matrix[0, 1]            # 2                -> row 0, column 1 (the NumPy way — preferred, faster)
matrix[:, 0]              # [1 4 7]           -> entire first COLUMN (this is the useful part lists can't do easily)
matrix[1, :]                # [4 5 6]            -> entire second row
matrix[0:2, 0:2]              # [[1 2], [4 5]]      -> top-left 2x2 sub-matrix
```
`matrix[row_slice, col_slice]` — this comma-based syntax is the big thing to get used to coming from plain Python lists, where you'd need `matrix[0][1]` style nested indexing every time.

## Slicing returns a VIEW, not a copy (important gotcha)
```python
arr = np.array([1, 2, 3, 4, 5])
sub = arr[1:3]      # [2 3]
sub[0] = 100          # modifies sub...
print(arr)              # [1 100 3 4 5]  -> original array changed too!
```
This tripped me up at first coming from Python lists, where slicing always makes a copy. NumPy slices are VIEWS into the same memory for performance reasons. If you actually want an independent copy:
```python
sub = arr[1:3].copy()     # now modifying sub won't touch arr
```

## Negative indexing works the same as lists
```python
arr = np.array([1, 2, 3, 4, 5])
arr[-1]      # 5
arr[-3:]       # [3 4 5]
```

## Step slicing
```python
arr = np.arange(10)
arr[::2]       # [0 2 4 6 8]     -> every 2nd element
arr[1::2]        # [1 3 5 7 9]     -> every 2nd element, starting from index 1
```

## Fancy indexing — passing a LIST of indices
```python
arr = np.array([10, 20, 30, 40, 50])
arr[[0, 2, 4]]      # [10 30 50]    -> pick specific indices at once
```
Unlike basic slicing, fancy indexing always returns a COPY, not a view.

## Fancy indexing on 2D arrays
```python
matrix = np.array([[1,2,3],[4,5,6],[7,8,9]])
matrix[[0, 2]]      # picks row 0 and row 2
matrix[[0, 1], [1, 2]]     # picks matrix[0,1] and matrix[1,2] -> [2, 6]
```

## Modifying via slicing
```python
arr = np.array([1, 2, 3, 4, 5])
arr[1:3] = [100, 200]     # replace a slice
print(arr)                    # [1 100 200 4 5]

arr[:] = 0     # sets EVERY element to 0, keeps same array object (useful when other variables reference it)
```
