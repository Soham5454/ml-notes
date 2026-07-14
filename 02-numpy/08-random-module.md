# 08 — Random Module

Used constantly for generating fake/sample data, shuffling datasets, initializing ML model weights, train-test splitting, etc.

## Setting a seed (for reproducibility)
```python
np.random.seed(42)     # ANY future random calls will now be reproducible/deterministic
```
Why this matters: if you don't set a seed, your "random" results are different every run — which is bad for debugging or comparing model results. Setting a seed means you (or anyone else) gets the EXACT same "random" numbers every time the code runs. This is standard practice in ML for reproducible experiments.

## Random floats between 0 and 1
```python
np.random.rand(3)          # 3 random floats in [0, 1)
np.random.rand(2, 3)         # 2x3 matrix of random floats
```

## Random integers
```python
np.random.randint(0, 10)             # single random int between 0-9
np.random.randint(0, 10, 5)            # array of 5 random ints between 0-9
np.random.randint(0, 10, (2, 3))         # 2x3 matrix of random ints
```

## Random floats from a normal (Gaussian) distribution
```python
np.random.randn(5)         # 5 values from a standard normal distribution (mean=0, std=1)
np.random.normal(loc=50, scale=10, size=5)     # 5 values from a normal distribution with mean=50, std=10
```
This shows up A LOT in ML — weight initialization in neural networks typically samples from a normal distribution.

## Shuffling data
```python
arr = np.array([1, 2, 3, 4, 5])
np.random.shuffle(arr)     # shuffles IN PLACE, modifies the original array
```

## Random choice — sampling from an array
```python
arr = np.array([10, 20, 30, 40, 50])
np.random.choice(arr)                     # picks one random element
np.random.choice(arr, 3)                    # picks 3 random elements (with replacement by default)
np.random.choice(arr, 3, replace=False)       # picks 3 UNIQUE elements (no repeats)
np.random.choice(arr, 3, p=[0.1,0.1,0.1,0.1,0.6])  # weighted probabilities for each element
```

## Practical example — simulating a train-test split manually
```python
data = np.arange(100)
np.random.shuffle(data)

split_point = int(0.8 * len(data))     # 80% train, 20% test
train = data[:split_point]
test = data[split_point:]
```
(In real projects you'd use scikit-learn's `train_test_split` instead, but understanding this manual version helped me actually get what that function is doing underneath.)

## Newer random API (Generator-based, NumPy 1.17+)
```python
rng = np.random.default_rng(seed=42)     # newer, recommended way going forward
rng.random(5)                                # random floats
rng.integers(0, 10, 5)                          # random integers
```
NumPy's docs recommend this newer `Generator` API over the older `np.random.seed()` style for new code, though the older style is still everywhere in tutorials/existing codebases so it's worth knowing both.
