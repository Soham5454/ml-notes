# 02 — Array Creation

## From Python lists/tuples
```python
np.array([1, 2, 3])                # from a list
np.array((1, 2, 3))                  # from a tuple
np.array([[1, 2], [3, 4]])             # 2D from nested lists
```

## Arrays of zeros, ones, and empty
```python
np.zeros(5)              # [0. 0. 0. 0. 0.]           -> 1D array of 5 zeros
np.zeros((2, 3))           # 2x3 matrix of zeros
np.ones((3, 3))              # 3x3 matrix of ones
np.empty((2, 2))               # uninitialized values (garbage memory) — faster but risky, rarely used unless you're immediately overwriting every value
np.full((2, 2), 7)                # 2x2 matrix filled with the value 7
```

## Identity matrix
```python
np.eye(3)     # 3x3 identity matrix (1s on diagonal, 0s elsewhere)
```

## Ranges — arange and linspace
```python
np.arange(0, 10)            # [0 1 2 3 4 5 6 7 8 9]     -> like Python's range(), but returns an array
np.arange(0, 10, 2)           # [0 2 4 6 8]                -> with a step
np.arange(0, 1, 0.1)            # works with floats too, unlike Python's range()

np.linspace(0, 1, 5)               # [0. 0.25 0.5 0.75 1.]    -> 5 evenly spaced values between 0 and 1 (inclusive)
```
Difference between the two: `arange` uses a step size, `linspace` uses a count of points. `linspace` is often more useful for plotting since you specify exactly how many points you want.

## Random arrays (quick preview — full detail in file 08)
```python
np.random.rand(3)              # 3 random floats between 0 and 1
np.random.randint(0, 10, 5)      # 5 random integers between 0 and 9
```

## Copying an existing array's shape
```python
arr = np.array([[1, 2], [3, 4]])
np.zeros_like(arr)     # zeros with the SAME shape and dtype as arr
np.ones_like(arr)        # ones with the same shape as arr
```
This is genuinely useful — I use it a lot instead of manually typing out shapes when initializing something to match existing data.

## Converting data types after creation
```python
arr = np.array([1, 2, 3])
arr.astype(float)     # [1. 2. 3.]   -> creates a NEW array with converted dtype, doesn't modify original
```

## reshape right after creation (common combo)
```python
np.arange(12).reshape(3, 4)     # creates 0-11, then arranges into a 3x4 matrix
```
(More on reshape properly in file 04 — this is just the common pattern you'll see everywhere.)

## Quick gotcha I ran into
`np.array()` vs `np.asarray()` — if you pass an existing NumPy array to `np.array()`, it makes a COPY by default. `np.asarray()` won't copy if it's already an array (saves memory). Small thing but good to know if you're debugging why changes aren't reflecting where expected.
```python
a = np.array([1, 2, 3])
b = np.array(a)        # b is a NEW copy
c = np.asarray(a)         # c points to the SAME array as a (no copy made)
```
