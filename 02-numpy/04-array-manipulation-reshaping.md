# 04 — Array Manipulation & Reshaping

## reshape — changing the shape without changing the data
```python
arr = np.arange(12)          # [0 1 2 3 4 5 6 7 8 9 10 11]
arr.reshape(3, 4)               # reshapes into 3 rows, 4 columns
arr.reshape(4, 3)                 # 4 rows, 3 columns
arr.reshape(2, 2, 3)                 # reshape into 3D: 2 blocks of 2x3
```
Rule: the total number of elements must stay the same. `reshape(3,4)` needs exactly 12 elements — trying `reshape(3,5)` on a 12-element array throws an error.

## Using -1 as a placeholder dimension
```python
arr.reshape(3, -1)     # NumPy figures out the missing dimension automatically -> becomes (3, 4)
arr.reshape(-1, 1)         # turns any array into a single column -> very common when feeding data into ML models that expect a specific shape
```
I use `reshape(-1, 1)` a lot conceptually already from reading ML tutorials — scikit-learn functions often expect 2D input even for a single feature column, and this is the standard fix.

## Flatten — converting any shape back to 1D
```python
matrix = np.array([[1,2],[3,4]])
matrix.flatten()      # [1 2 3 4]   -> always returns a COPY
matrix.ravel()          # [1 2 3 4]    -> returns a VIEW when possible (faster, but can affect original)
```

## Transpose — flipping rows and columns
```python
matrix = np.array([[1,2,3],[4,5,6]])
matrix.T          # transposes: (2,3) shape becomes (3,2)
matrix.transpose()   # same thing, more explicit name
```

## Concatenation — joining arrays
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])       # [1 2 3 4 5 6]

m1 = np.array([[1,2],[3,4]])
m2 = np.array([[5,6],[7,8]])
np.concatenate([m1, m2], axis=0)     # stacks vertically (adds rows)
np.concatenate([m1, m2], axis=1)     # stacks horizontally (adds columns)
```

## vstack and hstack (shortcuts for common concatenation)
```python
np.vstack([m1, m2])     # stack vertically -> same as axis=0
np.hstack([m1, m2])       # stack horizontally -> same as axis=1
```

## Splitting arrays
```python
arr = np.arange(9)
np.split(arr, 3)      # splits into 3 equal parts: [array([0,1,2]), array([3,4,5]), array([6,7,8])]

matrix = np.arange(16).reshape(4,4)
np.hsplit(matrix, 2)     # split into 2 pieces horizontally (column-wise)
np.vsplit(matrix, 2)       # split into 2 pieces vertically (row-wise)
```

## Adding/removing dimensions
```python
arr = np.array([1, 2, 3])           # shape (3,)
arr[np.newaxis, :]                     # shape (1, 3) -> adds a dimension
arr[:, np.newaxis]                       # shape (3, 1) -> adds a dimension the other way

np.expand_dims(arr, axis=0)               # same as np.newaxis, more explicit
np.squeeze(arr)                              # removes dimensions of size 1
```
This "adding a dimension" thing looks pointless at first but it's genuinely important once you get to broadcasting (next file) and later feeding data into ML models that expect a specific number of dimensions (batch dimension, channel dimension, etc.)

## Sorting
```python
arr = np.array([3, 1, 4, 1, 5])
np.sort(arr)         # [1 1 3 4 5]  -> returns a new sorted array
arr.sort()             # sorts in place, modifies arr directly

matrix = np.array([[3,1],[2,4]])
np.sort(matrix, axis=0)   # sort each COLUMN independently
np.sort(matrix, axis=1)   # sort each ROW independently
```

## argsort — get the indices that WOULD sort the array
```python
arr = np.array([3, 1, 4])
np.argsort(arr)     # [1 0 2]   -> index 1 (value 1) should come first, then index 0 (value 3), then index 2 (value 4)
```
Useful when you need to sort one array based on the order of another (e.g. sort names by their corresponding scores).

## Unique values
```python
arr = np.array([1, 2, 2, 3, 3, 3])
np.unique(arr)               # [1 2 3]
np.unique(arr, return_counts=True)     # ([1 2 3], [1 2 3])  -> values and their frequency counts
```
