# 11 — File I/O & Performance

## Saving and loading NumPy arrays (native format)
```python
arr = np.array([1, 2, 3, 4, 5])

np.save("my_array.npy", arr)         # saves in NumPy's own binary format
loaded = np.load("my_array.npy")       # loads it back exactly as-is
```
`.npy` is the fastest way to save/load a single array — preserves dtype and shape perfectly, much faster than text formats for large arrays.

## Saving multiple arrays together
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.savez("arrays.npz", first=a, second=b)     # save multiple named arrays in one file
data = np.load("arrays.npz")
data["first"]     # [1 2 3]
data["second"]      # [4 5 6]
```

## Working with text/CSV files
```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

np.savetxt("data.csv", arr, delimiter=",")     # save as plain text/CSV
loaded = np.loadtxt("data.csv", delimiter=",")    # load it back
```
Note: this is fine for pure-numeric data, but pandas handles CSVs with headers/mixed types much better — I'll switch to pandas' `read_csv` for anything with column names or text data (that's literally the next thing in my roadmap after this).

## Why NumPy is fast — a quick recap on WHY performance matters here
- Data stored in one contiguous memory block (not scattered like a Python list of objects)
- Operations run in compiled C code, not the Python interpreter
- Vectorization avoids the overhead of Python-level loops entirely

## Measuring performance yourself
```python
import time

arr = np.arange(1_000_000)

start = time.time()
result = arr * 2
print(time.time() - start)     # near-instant

# vs a pure python loop
py_list = list(range(1_000_000))
start = time.time()
result = [x * 2 for x in py_list]
print(time.time() - start)       # noticeably slower
```
Actually running this comparison myself made the "NumPy is fast" claim feel real instead of just something tutorials repeat.

## Memory usage matters too, not just speed
```python
arr = np.array([1, 2, 3], dtype=np.float64)     # 8 bytes per element
arr32 = np.array([1, 2, 3], dtype=np.float32)      # 4 bytes per element -> half the memory
```
This connects directly to something I looked into separately — mixed precision training on my RTX 3050 (4GB VRAM). Using float16/float32 instead of float64 wherever possible is a big part of how you fit larger models/batches into limited VRAM. Seeing the dtype-memory relationship here in plain NumPy made that concept click properly instead of just being an abstract technique name.

## Avoiding unnecessary copies (performance tip)
```python
arr = np.arange(1000000)

view = arr[:100]          # VIEW -> no memory copied, fast
copy = arr[:100].copy()     # COPY -> new memory allocated, use only when actually needed
```
Default to views/slices when you don't need an independent copy — this matters more once you're working with genuinely large datasets where unnecessary copies actually cost real time and memory.

## Vectorize custom functions (when a built-in ufunc doesn't exist)
```python
def custom_func(x):
    return x**2 + 3*x + 1

vectorized_func = np.vectorize(custom_func)
arr = np.array([1, 2, 3])
vectorized_func(arr)     # [5 11 19]
```
Worth noting: `np.vectorize` is mainly a convenience wrapper, not a true performance optimization — it still loops internally in Python. For genuinely large-scale custom math, writing the operation using built-in vectorized NumPy operations directly (or using something like Numba) is the actual fast approach.
