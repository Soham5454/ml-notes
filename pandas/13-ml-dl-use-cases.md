# 13 — ML/DL Use Cases (Feature Engineering & Data Pipelines)

Tying everything above back to why this actually matters for the ML/DL roadmap — pandas is basically the bridge between "raw messy data" and "clean numeric arrays a model can actually train on."

## The general pipeline pandas fits into
```
raw CSV/data → pandas cleaning (file 04) → feature engineering (this file) → NumPy/scikit-learn arrays → model training (scikit-learn / PyTorch)
```
Everything I've learned in files 01-12 exists to get data INTO the shape a model expects — pandas rarely trains anything itself, it prepares data for the tools that do.

## Handling missing data BEFORE modeling (recap + why it matters here specifically)
Most ML/DL models genuinely cannot handle NaN values directly — training will error out or silently produce garbage. So the `.fillna()` / `.dropna()` work from file 04 isn't just "cleaning for cleaning's sake," it's a hard requirement before feeding data to scikit-learn or PyTorch.
```python
df["age"].fillna(df["age"].median(), inplace=True)     # median is often preferred over mean for skewed data with outliers
```

## Encoding categorical variables — the manual pandas version
This is literally what scikit-learn's LabelEncoder/OneHotEncoder automate, but understanding the manual pandas version made those tools click properly for me.

### Label encoding (assign each category an integer)
```python
df["gender_encoded"] = df["gender"].map({"M": 0, "F": 1})
df["category_encoded"] = df["category"].astype("category").cat.codes     # automatic integer codes for any categorical column
```

### One-hot encoding (a column per category, 0/1 values) — pandas has this built in
```python
pd.get_dummies(df, columns=["department"])     # creates department_Sales, department_HR, etc. as separate 0/1 columns
```
This `pd.get_dummies()` function is genuinely one of the most-used lines in real feature engineering — turns a single categorical column into multiple binary columns a model can actually use, since most models (especially neural networks) can't directly interpret text categories.

## Feature scaling — pandas-level preview (real depth is in scikit-learn's StandardScaler/MinMaxScaler)
```python
df["salary_normalized"] = (df["salary"] - df["salary"].mean()) / df["salary"].std()     # manual standardization (z-score)
df["salary_scaled"] = (df["salary"] - df["salary"].min()) / (df["salary"].max() - df["salary"].min())     # manual min-max scaling to 0-1 range
```
I already know scikit-learn's StandardScaler/MinMaxScaler do exactly this — writing the manual pandas version helped confirm I actually understand the formula, not just the tool name.

## Feature engineering — creating new columns from existing ones
```python
df["bmi"] = df["weight_kg"] / (df["height_m"] ** 2)                    # derived feature from two existing columns
df["age_group"] = pd.cut(df["age"], bins=[0,18,35,60,100], labels=["teen","young_adult","adult","senior"])     # recap from file 05
df["is_weekend"] = df["date"].dt.dayofweek >= 5                             # recap from file 09
```
This is genuinely where a lot of real "data science skill" actually lives — the modeling algorithm often matters less than whether the input features are actually informative, and that's a pandas-and-domain-knowledge problem, not a modeling-library problem.

## Splitting features (X) and target (y) — the standard ML setup
```python
X = df.drop("target_column", axis=1)     # all columns EXCEPT the one we're trying to predict
y = df["target_column"]                     # just the target/label column
```
This exact `X`/`y` split is the universal starting point before calling scikit-learn's `train_test_split` — I've seen this pattern in basically every ML tutorial I've come across, now I understand precisely what it's doing structurally.

## Converting a DataFrame to a NumPy array (the final handoff point)
```python
X_array = X.values          # older syntax, still works
X_array = X.to_numpy()         # newer, preferred syntax
```
This is the literal moment pandas hands off to NumPy — scikit-learn and PyTorch ultimately want NumPy arrays (or PyTorch tensors, which are built the same conceptual way), not DataFrames, for actual model training. Everything before this line is data preparation; everything after is modeling.

## Preparing data specifically for PyTorch (a preview, since PyTorch is further down the roadmap)
```python
import torch
X_tensor = torch.tensor(X.to_numpy(), dtype=torch.float32)     # DataFrame -> NumPy -> PyTorch tensor, the standard chain
y_tensor = torch.tensor(y.to_numpy(), dtype=torch.float32)
```
Noting this here even though PyTorch is a few steps away in my roadmap — the chain is always DataFrame → NumPy array → tensor, so understanding pandas and NumPy properly first means this final step is just a formality by the time I actually reach PyTorch, not a new concept.

## Batch-loading large datasets for DL (a preview of what's coming)
```python
for chunk in pd.read_csv("huge_dataset.csv", chunksize=1000):
    X_batch = chunk.drop("target", axis=1).to_numpy()
    y_batch = chunk["target"].to_numpy()
    # feed X_batch, y_batch to a model incrementally
```
Connects directly to the chunking technique from file 11 — for datasets too large to fit in memory (or, relevant to my situation, too large to fit comfortably alongside a 4GB VRAM training loop), this same chunking pattern is the foundation of how DataLoader-style batch feeding works in PyTorch, just done manually here at the pandas level first.

## Honest summary
Pandas' actual job in the ML/DL pipeline is narrow but critical: take messy real-world data and turn it into clean, numeric, model-ready arrays. Every file in this folder (cleaning, indexing, groupby, merging, string ops, time series) ultimately serves that one goal — nothing here trains a model directly, but skipping any of it usually means the model either fails to train or trains on garbage.

## Next in the roadmap
Python Basics → NumPy → pandas (done) → **matplotlib** (already covered) → **Seaborn** (already covered) → scikit-learn → XGBoost → PyTorch → Hugging Face
