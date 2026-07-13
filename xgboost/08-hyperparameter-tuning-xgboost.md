# 08 — Hyperparameter Tuning for XGBoost

Recap the general hyperparameter tuning tools from scikit-learn file 09 (GridSearchCV, RandomizedSearchCV) — this file is about applying those SAME tools specifically to XGBoost's particular set of hyperparameters (recap from file 03), plus some XGBoost-specific tuning wisdom.

## Why XGBoost tuning is genuinely trickier than Random Forest's
Random Forest's hyperparameters (n_estimators, max_depth) are relatively INDEPENDENT of each other. XGBoost's hyperparameters genuinely INTERACT more — learning_rate and n_estimators trade off directly (recap file 05), max_depth interacts with min_child_weight and gamma, subsample/colsample_bytree interact with regularization. This makes a full exhaustive GridSearchCV across everything genuinely expensive.

## Using GridSearchCV with XGBoost (recap the pattern from scikit-learn file 09)
```python
from sklearn.model_selection import GridSearchCV
from xgboost import XGBClassifier

param_grid = {
    "max_depth": [3, 4, 5, 6],
    "learning_rate": [0.01, 0.05, 0.1],
    "n_estimators": [100, 200, 300]
}

grid_search = GridSearchCV(XGBClassifier(random_state=42), param_grid, cv=5, scoring="accuracy")
grid_search.fit(X_train, y_train)

print(grid_search.best_params_)
```
Works identically to how it worked for Random Forest — same GridSearchCV class, same pattern, just a different set of parameters and a different underlying model.

## Using RandomizedSearchCV instead (recap from scikit-learn file 09, genuinely more practical here given the interaction complexity)
```python
from sklearn.model_selection import RandomizedSearchCV

param_distributions = {
    "max_depth": [3, 4, 5, 6, 7, 8],
    "learning_rate": [0.01, 0.03, 0.05, 0.1, 0.2],
    "n_estimators": [100, 200, 300, 500],
    "subsample": [0.6, 0.7, 0.8, 0.9, 1.0],
    "colsample_bytree": [0.6, 0.7, 0.8, 0.9, 1.0],
    "gamma": [0, 0.1, 0.2, 0.3],
    "reg_alpha": [0, 0.01, 0.1, 1],
    "reg_lambda": [0.5, 1, 1.5, 2]
}

random_search = RandomizedSearchCV(
    XGBClassifier(random_state=42), param_distributions,
    n_iter=50, cv=5, scoring="accuracy", random_state=42
)
random_search.fit(X_train, y_train)
```
Given how many hyperparameters genuinely interact here, an exhaustive GridSearch across all of these would be enormously expensive (thousands of combinations × 5 folds each) — RandomizedSearch's sampling approach is genuinely the practical choice for XGBoost tuning specifically, more so than it even was for the simpler Random Forest case.

## A staged tuning approach — a genuinely common practical strategy for XGBoost
Rather than tuning EVERYTHING at once, a staged approach tunes groups of related parameters sequentially:

```python
# Stage 1: Fix a reasonably small learning_rate and n_estimators, tune tree structure first
param_grid_1 = {"max_depth": [3, 4, 5, 6, 7], "min_child_weight": [1, 3, 5]}
# ... GridSearchCV with fixed learning_rate=0.1, n_estimators=100 ...

# Stage 2: Fix the best tree structure found, tune sampling parameters
param_grid_2 = {"subsample": [0.6, 0.7, 0.8, 0.9], "colsample_bytree": [0.6, 0.7, 0.8, 0.9]}
# ... GridSearchCV using the best max_depth/min_child_weight from stage 1 ...

# Stage 3: Fix everything found so far, tune regularization
param_grid_3 = {"reg_alpha": [0, 0.01, 0.1, 1], "reg_lambda": [0.5, 1, 1.5, 2]}

# Stage 4: Finally, lower the learning_rate and increase n_estimators (or better, use early stopping from file 06) for the final model
```
This staged approach is genuinely more computationally tractable than one giant search, and reflects how the actual XGBoost documentation and many practitioners approach tuning — each stage narrows the search space for the next, rather than searching the full combinatorial space at once.

## Combining tuning with early stopping (recap from file 06 — a genuinely efficient combination)
```python
model = XGBClassifier(n_estimators=1000, max_depth=4, learning_rate=0.05, random_state=42)
model.fit(
    X_train, y_train,
    eval_set=[(X_val, y_val)],
    early_stopping_rounds=20,
    verbose=False
)
```
Genuinely a practical habit: use RandomizedSearchCV/staged tuning to find good STRUCTURAL hyperparameters (max_depth, subsample, regularization), then use a SEPARATE final training run with a low learning_rate, high n_estimators, and early stopping to find the actual best number of trees — rather than including n_estimators as just another grid search dimension.

## Visualizing tuning results (recap from scikit-learn file 09)
```python
results_df = pd.DataFrame(random_search.cv_results_)
sns.scatterplot(data=results_df, x="param_max_depth", y="mean_test_score", hue="param_learning_rate")     # recap Seaborn scatter with hue
```

## A genuinely reasonable, realistic tuning workflow for XGBoost
```python
# 1. Baseline with default parameters
baseline = XGBClassifier(random_state=42)
baseline_score = cross_val_score(baseline, X_train, y_train, cv=5).mean()

# 2. RandomizedSearchCV across the main structural parameters (max_depth, subsample, colsample_bytree, gamma, regularization)
random_search = RandomizedSearchCV(XGBClassifier(random_state=42), param_distributions, n_iter=50, cv=5, random_state=42)
random_search.fit(X_train, y_train)
best_params = random_search.best_params_

# 3. Final model: use the best structural params found, LOW learning_rate, HIGH n_estimators, with early stopping
final_model = XGBClassifier(**best_params, learning_rate=0.01, n_estimators=2000, random_state=42)
final_model.fit(X_train, y_train, eval_set=[(X_val, y_val)], early_stopping_rounds=30, verbose=False)

# 4. Evaluate ONCE on the truly held-out test set
final_score = final_model.score(X_test, y_test)
```
This is genuinely the realistic end-to-end tuning path I plan to follow for actual XGBoost projects, combining the systematic search tools from scikit-learn with XGBoost-specific practices (early stopping, staged tuning) rather than treating either approach in isolation.
