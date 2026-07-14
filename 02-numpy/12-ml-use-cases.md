# 12 — ML Use Cases (why all of this actually matters)

Writing this last file as a way to tie everything above back to the actual reason I'm learning NumPy — not just "because it's next on the roadmap," but because it's genuinely the foundation everything else sits on.

## Where NumPy shows up in the ML stack, concretely

### 1. Data representation
Every dataset eventually becomes a NumPy array (or a pandas DataFrame, which is built on NumPy underneath) before it's fed into any model. A dataset of images is a 4D array: `(num_images, height, width, channels)`. A dataset of tabular data is a 2D array: `(num_rows, num_features)`.

### 2. Feature normalization (uses broadcasting, file 05)
```python
data = np.array([[10, 200], [20, 300], [30, 400]])
mean = data.mean(axis=0)
std = data.std(axis=0)
normalized = (data - mean) / std     # broadcasting handles this cleanly, no loops needed
```
Almost every ML model trains better/faster on normalized data — this exact pattern is what scikit-learn's `StandardScaler` does internally.

### 3. Splitting data (uses random module, file 08)
```python
indices = np.arange(len(data))
np.random.shuffle(indices)
split = int(0.8 * len(data))
train_idx, test_idx = indices[:split], indices[split:]
```
The actual mechanism behind scikit-learn's `train_test_split`.

### 4. The math of a neural network layer (uses linear algebra, file 09)
A single dense/fully-connected layer is literally:
```python
output = inputs @ weights + bias
```
Where `inputs` is shape `(batch_size, num_features)`, `weights` is `(num_features, num_neurons)`, and the `@` does the matrix multiplication across the whole batch at once — this is the vectorization concept from file 01 applied at model scale.

### 5. Activation functions (uses ufuncs, file 06)
```python
def relu(x):
    return np.maximum(0, x)

def sigmoid(x):
    return 1 / (1 + np.exp(-x))
```
Both apply element-wise across an entire array of numbers in one line — no loops, thanks to vectorization.

### 6. Loss calculation (uses aggregation, file 07)
```python
predictions = np.array([0.8, 0.4, 0.9])
actual = np.array([1, 0, 1])
mse_loss = np.mean((predictions - actual) ** 2)     # mean squared error, one line
```

### 7. Filtering predictions / evaluating accuracy (uses masking, file 10)
```python
predictions = np.array([1, 0, 1, 1, 0])
actual = np.array([1, 0, 0, 1, 0])
accuracy = np.mean(predictions == actual)     # fraction of correct predictions
```

## Honest note to self
A lot of this I'm connecting from stuff I've READ about ML (PyTorch, model training) rather than having built yet — but going through NumPy properly first means when I actually get to PyTorch tensors later, the operations (`.reshape()`, `@`, broadcasting, `.mean(axis=...)`) will already feel familiar instead of being new syntax to memorize on top of new concepts. That's really the whole point of doing this step properly instead of rushing to pandas.

## Next in the roadmap
NumPy (done) → **pandas** (next — already have fundamentals, moving to deeper practice) → matplotlib (done) → Seaborn (done) → scikit-learn → XGBoost → PyTorch → Hugging Face
