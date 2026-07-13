# 02 — Preprocessing: Scaling & Encoding

Almost no model performs well on raw, unprocessed data — this file covers the standard preprocessing steps that happen right before `.fit()`, and connects directly back to the manual pandas versions of these same operations from the pandas ML/DL notes.

## Why scaling matters
Many algorithms (KNN, SVM, gradient descent-based models, and literally every neural network) are sensitive to the SCALE of features. If one feature ranges 0-1 and another ranges 0-100000, the larger-scale feature can dominate the model's learning process purely due to magnitude, not actual importance. Tree-based models (decision trees, random forests) are a notable exception — they're generally scale-invariant since they split on thresholds, not distances.

## StandardScaler — the most common scaler (z-score normalization)
```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)     # fit AND transform on training data
X_test_scaled = scaler.transform(X_test)              # ONLY transform on test data -- using the SAME scaling learned from train
```
This transforms each feature to have mean=0 and standard deviation=1 — the manual pandas version of this was `(x - mean) / std` from the pandas file, this is that exact formula automated.

## Critical rule: fit on train, transform on test (never fit on test data)
This is genuinely one of the most important rules in the entire ML workflow, and a mistake that silently produces overly-optimistic results if done wrong.
```python
# CORRECT
scaler.fit(X_train)                      # learn mean/std from TRAINING data only
X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)     # apply the SAME learned mean/std to test data

# WRONG -- this is called "data leakage"
scaler.fit(X_test)     # NEVER do this -- lets information from test data leak into preprocessing
```
The reasoning: in real deployment, you won't have access to "future" data to compute a fresh mean/std — you only have what you learned during training. Fitting the scaler on test data gives an unrealistically rosy picture of how well the model will actually perform on genuinely new data.

## MinMaxScaler — scales to a fixed range, usually 0-1
```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
X_scaled = scaler.fit_transform(X_train)     # same fit/transform pattern
```
Manual pandas equivalent: `(x - min) / (max - min)`. Useful specifically for algorithms that expect bounded input (like some neural network activation functions), or when you want to preserve the exact shape of the original distribution rather than centering it around zero.

## RobustScaler — scaling that's resistant to outliers
```python
from sklearn.preprocessing import RobustScaler

scaler = RobustScaler()
X_scaled = scaler.fit_transform(X_train)
```
Uses median and IQR (recap from the pandas outlier detection notes) instead of mean/std — genuinely useful when a dataset has significant outliers that would otherwise skew a standard scaler's mean/std calculation.

## When to use which scaler
- **StandardScaler** — my default choice, works well for most models, especially anything gradient-based (including neural networks)
- **MinMaxScaler** — when you need bounded 0-1 output specifically
- **RobustScaler** — when outliers are present and you don't want them distorting the scaling

## Encoding categorical variables

### LabelEncoder — converts categories to integers (for the TARGET variable, typically)
```python
from sklearn.preprocessing import LabelEncoder

encoder = LabelEncoder()
y_encoded = encoder.fit_transform(y)     # e.g. ['cat', 'dog', 'cat'] -> [0, 1, 0]
encoder.classes_                            # shows the mapping: array(['cat', 'dog'])
```
Important note: LabelEncoder is really meant for the TARGET/label column, not for input features with more than 2 categories — using it on a feature column implies a false ORDERING (like saying "dog" > "cat" numerically), which can genuinely mislead certain models.

### OneHotEncoder — the correct way to encode categorical FEATURES
```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(sparse_output=False)
X_encoded = encoder.fit_transform(X[["category_column"]])     # creates one binary column PER category
```
This is scikit-learn's version of `pd.get_dummies()` from the pandas notes — genuinely does the same job, but as part of the scikit-learn ecosystem so it fits into Pipelines (file 10) cleanly.

### Ordinal encoding — when categories DO have a genuine order
```python
from sklearn.preprocessing import OrdinalEncoder

encoder = OrdinalEncoder(categories=[["low", "medium", "high"]])     # explicit order specified
X_encoded = encoder.fit_transform(X[["size_category"]])
```
Different from LabelEncoder mainly in INTENT and usage location — OrdinalEncoder is meant for features where order genuinely matters (like a size category: low/medium/high), and lets you explicitly specify what that order should be instead of just alphabetical/appearance order.

## Handling missing values — SimpleImputer (scikit-learn's version of pandas' fillna)
```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="mean")     # options: 'mean', 'median', 'most_frequent', 'constant'
X_imputed = imputer.fit_transform(X_train)
```
Same "fit on train, transform on test" rule applies here too — the imputer should learn its fill values (like the mean) from training data only, then apply that same learned value to fill gaps in test data.

## Putting it together — a typical preprocessing sequence
```python
from sklearn.preprocessing import StandardScaler
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")
X_train_imputed = imputer.fit_transform(X_train)
X_test_imputed = imputer.transform(X_test)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train_imputed)
X_test_scaled = scaler.transform(X_test_imputed)
```
Doing this manually step-by-step works, but genuinely gets unwieldy fast — this exact sequence is what Pipelines (file 10) exist to clean up and automate.
