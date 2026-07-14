# 05 — Broadcasting

This concept confused me the most when I first read about it, so I'm writing this one extra carefully for future me.

## What is broadcasting?
Broadcasting is NumPy's way of performing operations on arrays of DIFFERENT shapes, by automatically "stretching" the smaller one to match the bigger one — without actually copying any data in memory. It's what lets you write `array + 5` instead of manually looping to add 5 to every element.

## The simplest case — scalar broadcasting
```python
arr = np.array([1, 2, 3])
arr + 5     # [6 7 8]
```
The scalar `5` is conceptually "stretched" to `[5, 5, 5]` and added element-wise. You never actually see this stretched array — NumPy just does the math efficiently.

## Broadcasting between a 1D array and a 2D array
```python
matrix = np.array([[1, 2, 3],
                    [4, 5, 6]])          # shape (2, 3)

vector = np.array([10, 20, 30])            # shape (3,)

matrix + vector
# [[11 22 33]
#  [14 25 36]]
```
Here, `vector` gets applied to EACH ROW of the matrix. This works because their shapes are "compatible" for broadcasting.

## The broadcasting rules (this is the part that actually matters)
When comparing shapes of two arrays, NumPy checks dimensions from the RIGHT side:
1. If the dimensions are equal, they're compatible
2. If one of the dimensions is 1, it gets "stretched" to match the other
3. If neither of the above is true, it's an error (shapes are incompatible)

```python
matrix.shape   # (2, 3)
vector.shape     # (3,)
```
Comparing right to left: `3` matches `3` ✓. The matrix has an extra leading dimension (2) which is just kept as-is since the vector doesn't have anything to compare against there. So this works.

## When broadcasting FAILS
```python
a = np.array([[1, 2, 3], [4, 5, 6]])    # shape (2, 3)
b = np.array([1, 2])                        # shape (2,)

a + b     # ERROR: shapes (2,3) and (2,) not compatible
```
Comparing right to left: `3` vs `2` — neither matches nor is either dimension 1, so it fails. To fix this, you'd need to reshape `b` to `(2, 1)` first so it broadcasts down the columns instead:
```python
b_reshaped = b.reshape(2, 1)     # shape (2, 1)
a + b_reshaped
# [[2 3 4]
#  [6 7 8]]
```
This is exactly why `reshape` and `np.newaxis` (from file 04) actually matter in practice — broadcasting errors are usually fixed by adding/adjusting a dimension.

## Practical example — normalizing data (a real ML use case)
```python
data = np.array([[1, 2, 3],
                  [4, 5, 6],
                  [7, 8, 9]])

column_means = data.mean(axis=0)      # [4. 5. 6.]  -> mean of each column
normalized = data - column_means         # broadcasting subtracts the mean from every row automatically
```
This is genuinely how feature normalization works in real ML preprocessing — subtract the mean, divide by the standard deviation, all done via broadcasting instead of manual loops.

## Visualizing it mentally
Think of broadcasting like this: the smaller array is being "copy-pasted" across the missing dimension, but NumPy is smart enough to do this without wasting memory — it's a virtual expansion, not an actual one.
