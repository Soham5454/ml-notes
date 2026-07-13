# 10 — Pipelines & Workflow

Recap the problem from file 02's ending — manually chaining imputation, scaling, and model training step by step works, but gets messy and error-prone fast, especially remembering to apply `fit_transform` on train and only `transform` on test EVERY time. Pipelines solve this cleanly.

## The basic Pipeline
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression())
])

pipeline.fit(X_train, y_train)          # automatically: fit_transform scaler on X_train, then fit the model on the scaled result
predictions = pipeline.predict(X_test)     # automatically: transform X_test using the ALREADY-FITTED scaler, then predict
```
This genuinely eliminates an entire category of mistakes — you literally cannot accidentally fit the scaler on test data, since the pipeline handles the correct fit/transform sequence internally, automatically, every time.

## Why this matters beyond just convenience
1. **Prevents data leakage** — the exact mistake warned about in file 02 becomes structurally impossible
2. **Cleaner code** — one `.fit()` and one `.predict()` call instead of manually chaining multiple steps
3. **Works seamlessly with cross-validation and GridSearchCV** — the ENTIRE pipeline (preprocessing + model) gets properly refit on each fold/combination, which is actually necessary for correct cross-validation (if you scale BEFORE splitting into folds, that's its own subtle data leakage — pipelines handle this correctly automatically)

## Multi-step pipelines
```python
from sklearn.impute import SimpleImputer

pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
    ("model", RandomForestClassifier())
])
pipeline.fit(X_train, y_train)
```
Any number of preprocessing steps can be chained before the final model — each step's `fit_transform` output feeds directly into the next step.

## ColumnTransformer — applying DIFFERENT preprocessing to DIFFERENT columns
Real datasets almost always have a MIX of numeric and categorical columns needing entirely different treatment — this is where it gets genuinely necessary beyond a simple linear Pipeline.
```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

numeric_features = ["age", "salary"]
categorical_features = ["department", "gender"]

preprocessor = ColumnTransformer([
    ("num", StandardScaler(), numeric_features),
    ("cat", OneHotEncoder(), categorical_features)
])

full_pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", RandomForestClassifier())
])

full_pipeline.fit(X_train, y_train)
```
This is genuinely the realistic, practical shape of most real preprocessing pipelines — numeric columns get scaled, categorical columns get one-hot encoded, all handled correctly and automatically in one combined object.

## Using Pipelines with GridSearchCV — tuning preprocessing AND model hyperparameters together
```python
param_grid = {
    "model__n_estimators": [50, 100, 200],           # note the DOUBLE UNDERSCORE -- 'model' refers to the pipeline step name, 'n_estimators' is that step's actual parameter
    "model__max_depth": [5, 10, None],
    "preprocessor__num__with_mean": [True, False]       # can even tune PREPROCESSING hyperparameters this way
}

grid_search = GridSearchCV(full_pipeline, param_grid, cv=5)
grid_search.fit(X_train, y_train)
```
That `step_name__parameter_name` double-underscore syntax genuinely confused me the first time I saw it — it's how GridSearchCV knows WHICH step inside the pipeline a given hyperparameter belongs to, since a pipeline can have multiple steps each with their own hyperparameters.

## FunctionTransformer — wrapping custom functions into the pipeline
```python
from sklearn.preprocessing import FunctionTransformer
import numpy as np

log_transformer = FunctionTransformer(np.log1p)     # log1p = log(1+x), handles zero values safely unlike plain log

pipeline = Pipeline([
    ("log_transform", log_transformer),
    ("scaler", StandardScaler()),
    ("model", LinearRegression())
])
```
Genuinely useful for custom preprocessing steps (like log-transforming a skewed feature, recap from the Seaborn EDA distribution-checking notes) that aren't covered by scikit-learn's built-in transformers, while still keeping everything inside the same clean pipeline structure.

## Inspecting pipeline steps
```python
pipeline.named_steps["model"]              # access a specific step by name
pipeline.named_steps["model"].coef_           # access that step's learned attributes directly
```

## Saving an entire pipeline (recap connects to file 13)
```python
import joblib
joblib.dump(full_pipeline, "model_pipeline.pkl")     # saves the WHOLE pipeline -- preprocessing steps AND trained model together, as one object
```
This is genuinely one of the biggest practical advantages of pipelines for real deployment — saving just the trained model alone would be useless without also saving the EXACT preprocessing steps (scaler's learned mean/std, encoder's learned categories) used during training. A pipeline bundles all of that together as one coherent, reloadable unit.

## Honest summary — why I'm treating this as genuinely essential, not optional
Early on, doing preprocessing manually step-by-step feels fine for small experiments. But the moment cross-validation, hyperparameter tuning, or actual deployment enters the picture, manual preprocessing becomes a genuine liability (easy to introduce data leakage, easy to forget a step when reusing code). I'm planning to just default to building a Pipeline from the very start of any real project going forward, rather than treating it as an optional "clean-up" step done later.
