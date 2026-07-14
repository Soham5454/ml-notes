# 11 — Feature Selection & Engineering

Recap the honest note from the pandas ML/DL file: the modeling algorithm often matters less than whether the input features are actually informative. This file covers the scikit-learn-specific tools for deciding WHICH features to keep, on top of the manual feature-creation techniques already covered in pandas.

## Why feature selection matters
- **Fewer irrelevant features** = less noise for the model to filter through, often better generalization
- **Fewer features** = faster training, especially relevant given my hardware constraints
- **Removing redundant/correlated features** (recap from the Seaborn correlation heatmap notes) = simpler, more interpretable models without losing real information

## Filter methods — selecting features based on statistical properties, independent of any model

### VarianceThreshold — removing features with little to no variation
```python
from sklearn.feature_selection import VarianceThreshold

selector = VarianceThreshold(threshold=0.01)     # removes features with variance below this threshold
X_selected = selector.fit_transform(X)
```
A feature that's almost the SAME value for every single row genuinely carries little useful information — this is a fast, simple first-pass filter before anything more sophisticated.

### SelectKBest — keeping the K features most correlated with the target
```python
from sklearn.feature_selection import SelectKBest, f_classif

selector = SelectKBest(score_func=f_classif, k=10)     # f_classif for classification, f_regression for regression targets
X_selected = selector.fit_transform(X_train, y_train)

selected_features = X.columns[selector.get_support()]     # which original column names were actually kept
```
Uses a statistical test (like ANOVA F-value for classification) to score each feature's individual relationship with the target, then keeps only the top K scorers. Genuinely fast since it evaluates each feature independently, though it misses feature INTERACTIONS (two features that are only useful when combined might each individually score low).

## Wrapper methods — selecting features by actually testing model performance

### Recursive Feature Elimination (RFE)
```python
from sklearn.feature_selection import RFE
from sklearn.ensemble import RandomForestClassifier

selector = RFE(estimator=RandomForestClassifier(), n_features_to_select=10)
X_selected = selector.fit_transform(X_train, y_train)

selected_features = X.columns[selector.support_]
```
Trains the model repeatedly, each time removing the LEAST important feature (based on the model's own feature_importances_ or coefficients), until reaching the target number of features. More computationally expensive than filter methods (since it retrains the model multiple times), but accounts for how features actually interact within that specific model.

## Embedded methods — feature selection built INTO the model training itself

### Using Lasso's automatic feature selection (recap from file 04)
```python
from sklearn.linear_model import Lasso

model = Lasso(alpha=0.1)
model.fit(X_train, y_train)

selected_features = X.columns[model.coef_ != 0]     # features Lasso didn't shrink to exactly zero
```
Recap from the linear models file — Lasso's L1 regularization naturally zeros out unhelpful features AS PART of training, no separate feature selection step needed.

### Using tree-based feature importances directly (recap from file 06)
```python
model = RandomForestClassifier()
model.fit(X_train, y_train)

importance_df = pd.DataFrame({"feature": X.columns, "importance": model.feature_importances_}).sort_values("importance", ascending=False)
top_features = importance_df.head(10)["feature"]
```

## Feature Engineering — creating NEW features (extends the pandas feature engineering notes)

### PolynomialFeatures — automatically generating interaction/polynomial terms (recap from file 04)
```python
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, interaction_only=False)
X_poly = poly.fit_transform(X)
```
`interaction_only=True` would give ONLY interaction terms (like feature1 × feature2) without also squaring individual features — useful when you specifically suspect feature COMBINATIONS matter, without wanting the extra squared terms cluttering the feature space.

### Binning numeric features (scikit-learn's version of pandas' pd.cut, file 05)
```python
from sklearn.preprocessing import KBinsDiscretizer

binner = KBinsDiscretizer(n_bins=5, encode="ordinal", strategy="quantile")
X_binned = binner.fit_transform(X[["age"]])
```

## A genuinely important caution — feature selection and data leakage (connects to file 02/10)
Feature selection that uses the TARGET variable (like SelectKBest, RFE) must ONLY be fit on training data, exactly like scalers/encoders — fitting a feature selector on the full dataset (including test data) before splitting is a subtle form of data leakage, since the selection process would have "seen" test data's relationship with the target. The safest way to guarantee this is correct: put the feature selector INSIDE a Pipeline (file 10), same as any other preprocessing step.
```python
pipeline = Pipeline([
    ("selector", SelectKBest(k=10)),
    ("model", RandomForestClassifier())
])
```

## A realistic feature selection workflow
```python
# 1. Start broad -- remove obviously useless features (near-zero variance, near-duplicate columns)
# 2. Check correlation heatmap (Seaborn) -- manually drop or combine heavily redundant features
# 3. Fit a baseline model, check feature_importances_ or Lasso coefficients
# 4. If needed, use RFE or SelectKBest for a more systematic narrowing
# 5. Compare model performance WITH vs WITHOUT the selected subset -- more features isn't always worse, so verify selection actually helped
```
This last step (step 5) is genuinely something I want to remember to always do — feature selection is a hypothesis ("these features matter more"), and it needs to be validated against actual held-out performance, not just assumed correct because a statistical test or importance score said so.
