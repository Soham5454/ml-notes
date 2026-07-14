# 01 — Introduction & The Estimator API

## What is scikit-learn and why it exists
Scikit-learn is THE standard library for classical machine learning in Python — regression, classification, clustering, model evaluation, preprocessing, all under one extremely consistent API. It's built on top of NumPy and pandas (everything from those notes feeds directly in here) and is genuinely the bridge between "I understand my data" and "I have a model that predicts something."

## The Estimator API — the single most important concept in this entire library
Almost EVERY algorithm in scikit-learn — whether it's linear regression, a decision tree, or K-means clustering — follows the exact same interface pattern. Once this clicks, using a new algorithm you've never touched before is genuinely just swapping one class name for another.

```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()     # 1. Instantiate the model (with any hyperparameters)
model.fit(X_train, y_train)      # 2. Fit it to training data
predictions = model.predict(X_test)  # 3. Predict on new data
```
This exact three-step pattern (`instantiate → fit → predict`) is IDENTICAL whether I'm using `LinearRegression`, `RandomForestClassifier`, `KMeans`, or literally any other scikit-learn algorithm. This consistency is genuinely the biggest reason scikit-learn is so widely used — learning one algorithm's syntax basically teaches you all of them.

## The universal method names
```python
model.fit(X, y)          # trains the model on data (supervised: needs X and y; unsupervised: just X)
model.predict(X)            # makes predictions on new data
model.score(X, y)              # quick built-in performance score (accuracy for classifiers, R² for regressors)
model.transform(X)                # for preprocessing tools -- transforms data without predicting anything
model.fit_transform(X)               # fit AND transform in one call -- shortcut, common for preprocessing steps
```

## X and y — the universal input convention
```python
X = df.drop("target", axis=1)     # features -- capital X by convention, since it's typically a 2D matrix
y = df["target"]                     # target/label -- lowercase y by convention, since it's typically a 1D vector
```
This `X`/`y` naming convention (recap from the pandas ML/DL file) is genuinely universal across scikit-learn's entire documentation and virtually every tutorial — worth internalizing since it's not just a preference, it's the expected variable naming almost everywhere.

## X must be 2D, even for a single feature
```python
X = df[["age"]]     # DOUBLE brackets -- keeps it as a DataFrame (2D), which scikit-learn expects
X = df["age"]           # single brackets -- a Series (1D), scikit-learn will ERROR on this for most estimators

# If working with plain NumPy instead of pandas:
X = age_array.reshape(-1, 1)     # recap from NumPy notes -- reshape a 1D array into a 2D column
```
This tripped me up early on — scikit-learn genuinely insists X be 2D (`n_samples, n_features`) even when there's only ONE feature, since the API is built assuming multiple features are the norm.

## Types of learning scikit-learn covers
- **Supervised learning** — you have labeled data (X and y), the model learns to predict y from X
  - **Regression** — y is continuous/numeric (predicting a price, a temperature)
  - **Classification** — y is categorical (predicting a class label, like spam/not-spam)
- **Unsupervised learning** — you only have X, no labels, the model finds structure/patterns on its own
  - **Clustering** — grouping similar data points together
  - **Dimensionality reduction** — compressing many features into fewer, while preserving important structure

## A complete minimal example (to see the full flow before diving into individual pieces)
```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

X = df[["hours_studied"]]
y = df["exam_score"]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)

predictions = model.predict(X_test)
mse = mean_squared_error(y_test, predictions)
print(f"MSE: {mse}")
```
This tiny example already touches train_test_split (file 03), a linear model (file 04), and an evaluation metric (file 08) — genuinely the entire "shape" of an ML project in miniature, before all the individual pieces get their own deep-dive files.

## Model parameters vs hyperparameters (a distinction that matters going forward)
- **Parameters** — values the model LEARNS from data during `.fit()` (e.g. the actual weights/coefficients in linear regression)
- **Hyperparameters** — values YOU set BEFORE training, controlling how the model learns (e.g. `n_estimators` for a random forest, `k` for KNN)
```python
model = LinearRegression()               # no hyperparameters really needed here
model.coef_                                 # the LEARNED parameters, available only after .fit()
model.intercept_                               # also learned

from sklearn.neighbors import KNeighborsClassifier
knn = KNeighborsClassifier(n_neighbors=5)     # n_neighbors is a HYPERPARAMETER, set BEFORE training
```
This distinction becomes genuinely important in file 09 (hyperparameter tuning) — you can't "train" a hyperparameter the normal way, you have to search for good values externally.
