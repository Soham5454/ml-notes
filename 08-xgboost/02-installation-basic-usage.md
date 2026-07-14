# 02 — Installation & Basic Usage

## Installing XGBoost
```bash
pip install xgboost
```
For GPU support specifically (relevant to file 10, and my RTX 3050 setup), the standard pip install usually includes GPU support automatically in modern versions, though it's worth verifying against XGBoost's official docs for the exact CUDA version compatibility with my installed CUDA Toolkit.

## Basic classification example (recap of the pattern from file 01, expanded)
```python
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split
from sklearn.datasets import load_breast_cancer

data = load_breast_cancer()     # a genuinely useful built-in scikit-learn dataset for practicing binary classification
X, y = data.data, data.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = XGBClassifier(random_state=42)
model.fit(X_train, y_train)

predictions = model.predict(X_test)
probabilities = model.predict_proba(X_test)     # SAME predict_proba pattern from scikit-learn's LogisticRegression notes
```

## Basic regression example
```python
from xgboost import XGBRegressor

model = XGBRegressor(random_state=42)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```
Same estimator pattern, just swapping the class name — this consistency is genuinely the whole point of the scikit-learn-compatible API from file 01.

## Working directly with pandas DataFrames (XGBoost handles this natively, no manual NumPy conversion needed)
```python
import pandas as pd

X = df.drop("target", axis=1)     # recap from the scikit-learn/pandas ML use cases notes
y = df["target"]

model = XGBClassifier()
model.fit(X, y)     # XGBoost accepts a DataFrame directly, genuinely convenient
```

## Inspecting the trained model
```python
model.get_booster()     # access the underlying "booster" object (the actual trained ensemble of trees)
model.n_estimators          # how many trees were actually used
```

## Making predictions with a specific number of trees (useful for inspecting the boosting process)
```python
predictions_partial = model.predict(X_test, iteration_range=(0, 50))     # predict using only the first 50 trees instead of all of them
```
Genuinely useful for understanding HOW the ensemble's predictions evolve/improve as more trees get added — connects to the visualization of training progress covered more in file 06.

## A complete workflow combining XGBoost with the Pipeline concept (recap from scikit-learn file 10)
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from xgboost import XGBClassifier

pipeline = Pipeline([
    ("scaler", StandardScaler()),     # note: tree-based models like XGBoost are generally SCALE-INVARIANT (recap from scikit-learn preprocessing notes), so scaling isn't strictly necessary for XGBoost itself -- but doesn't hurt, and matters if combining with other scale-sensitive steps
    ("model", XGBClassifier())
])
pipeline.fit(X_train, y_train)
```
Worth noting honestly: unlike KNN or SVM, XGBoost (being tree-based) doesn't actually NEED feature scaling to perform well, since tree splits are threshold-based rather than distance-based. I still often include a scaler in the pipeline anyway for consistency when comparing against other model types, but it's genuinely optional specifically for XGBoost.

## Saving and loading an XGBoost model (XGBoost has its own native save format too, beyond joblib from the scikit-learn notes)
```python
model.save_model("my_xgb_model.json")     # XGBoost's own native format -- genuinely portable across languages (can be loaded in XGBoost's R, C++, Java bindings too, not just Python)
model.load_model("my_xgb_model.json")

# joblib still works too (recap from scikit-learn file 13), since XGBClassifier IS a scikit-learn-compatible estimator
import joblib
joblib.dump(model, "my_xgb_model.pkl")
```
This native `.json`/`.save_model()` format is genuinely a nice bonus over plain joblib — useful specifically if a trained model ever needs to be used outside a Python environment.

## Quick sanity check — comparing against scikit-learn's GradientBoostingClassifier directly
```python
from sklearn.ensemble import GradientBoostingClassifier
from xgboost import XGBClassifier

sklearn_gb = GradientBoostingClassifier(random_state=42)
xgb_model = XGBClassifier(random_state=42)

sklearn_gb.fit(X_train, y_train)
xgb_model.fit(X_train, y_train)

print("sklearn GB:", sklearn_gb.score(X_test, y_test))
print("XGBoost:", xgb_model.score(X_test, y_test))
```
Genuinely worth running this kind of side-by-side comparison on real projects — while XGBoost often wins on speed and sometimes on raw accuracy, it's not GUARANTEED to always outperform plain scikit-learn gradient boosting on every single dataset, so verifying rather than assuming is a good habit.
