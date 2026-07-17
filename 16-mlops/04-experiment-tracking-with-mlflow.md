# 04 — Experiment Tracking with MLflow

Recap file 03's closing note and file 01's honest "which of these 47 notebook runs was the best one" problem — this file directly solves it. Genuinely one of the highest-value tools in this entire folder for actual day-to-day ML work.

## The problem MLflow solves — genuinely universal, painfully familiar

```python
# Recap the general hyperparameter tuning discussions from scikit-learn/XGBoost/PyTorch notes
# Every training run has: hyperparameters (learning_rate, n_estimators, batch_size...),
# metrics (accuracy, loss, F1...), and an ARTIFACT (the trained model itself)
# WITHOUT tracking: these all live in scattered print statements, notebook cell
# outputs, or (worse) only in memory, GONE the moment the notebook kernel restarts
```

## Setting up MLflow — genuinely minimal to get started

```python
import mlflow

mlflow.set_tracking_uri("sqlite:///mlflow.db")      # recap SQL folder -- MLflow can genuinely
                                                          # use a SQL database as its backend
mlflow.set_experiment("sentiment-classification")      # groups related runs together
```

## Logging a training run — recap scikit-learn's whole workflow directly

```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, f1_score      # recap scikit-learn's metrics folder

with mlflow.start_run():
    # Log HYPERPARAMETERS
    C = 1.0
    mlflow.log_param("C", C)
    mlflow.log_param("model_type", "LogisticRegression")

    # Train -- genuinely just the SAME scikit-learn code from that whole folder
    model = LogisticRegression(C=C)
    model.fit(X_train, y_train)
    predictions = model.predict(X_test)

    # Log METRICS
    accuracy = accuracy_score(y_test, predictions)
    f1 = f1_score(y_test, predictions)
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("f1_score", f1)

    # Log the ARTIFACT -- the trained model itself
    mlflow.sklearn.log_model(model, "model")
```

Genuinely worth recognizing: `with mlflow.start_run():` wraps EXACTLY the same training code already written throughout the scikit-learn folder, just adding explicit logging calls around it. Nothing about the actual ML changes — MLflow is purely an ADDITIVE tracking layer.

## The MLflow UI — genuinely the payoff, seeing every run at a glance

```bash
mlflow ui
# Opens a local web interface (recap the entire Flask folder -- genuinely
# built using the same request/response web-app concepts) at localhost:5000
```

Genuinely worth actually running this and exploring it — every logged run appears in a sortable, filterable TABLE, with parameters and metrics as columns, letting different hyperparameter choices be compared side-by-side visually, directly answering file 01's "which run was best" problem with a genuinely simple, visual answer.

## Logging across a hyperparameter search — recap scikit-learn's GridSearch/XGBoost tuning directly

```python
for C in [0.01, 0.1, 1.0, 10.0]:
    with mlflow.start_run():
        mlflow.log_param("C", C)
        model = LogisticRegression(C=C)
        model.fit(X_train, y_train)
        accuracy = accuracy_score(y_test, model.predict(X_test))
        mlflow.log_metric("accuracy", accuracy)
```

Genuinely the exact real-world use case this tool was built for — recap XGBoost file 08's hyperparameter tuning discussion directly: instead of manually noting down results in a spreadsheet or scrolling through notebook output, EVERY combination tried gets automatically, permanently recorded and comparable in the MLflow UI.

## Autologging — genuinely convenient, minimal-code tracking

```python
import mlflow.sklearn

mlflow.sklearn.autolog()      # genuinely logs params, metrics, AND the model automatically

model = LogisticRegression(C=1.0)
model.fit(X_train, y_train)      # MLflow automatically captures this call's relevant details
```

Genuinely worth knowing autologging exists for several major libraries (`mlflow.sklearn`, `mlflow.pytorch`, `mlflow.xgboost` — recap those exact folders) — a real, practical way to get tracking benefits with almost zero extra code, though explicit logging (as shown earlier) gives more control over EXACTLY what gets recorded.

## Tracking PyTorch training runs — recap PyTorch file 07's training loop directly

```python
import mlflow.pytorch

with mlflow.start_run():
    mlflow.log_param("learning_rate", 0.001)
    mlflow.log_param("epochs", 20)
    mlflow.log_param("batch_size", 32)

    for epoch in range(20):
        # ... recap PyTorch file 07's five-step training loop exactly ...
        train_loss = ...  # computed each epoch
        val_loss = ...

        mlflow.log_metric("train_loss", train_loss, step=epoch)      # genuinely important --
        mlflow.log_metric("val_loss", val_loss, step=epoch)             # step= lets MLflow plot
                                                                            # the loss CURVE over epochs,
                                                                            # not just a single final number

    mlflow.pytorch.log_model(model, "model")
```

Genuinely worth understanding `step=epoch` explicitly — recap PyTorch file 09's train/val divergence PLOT discussion directly: MLflow can visualize metrics AS A CURVE across training, letting the exact same overfitting-detection pattern (train loss dropping while val loss climbs) be spotted directly in the MLflow UI, rather than needing a separate matplotlib plot each time.

## The Model Registry — genuinely important for the "which model is in production" problem

```python
result = mlflow.register_model("runs:/<run_id>/model", "sentiment-classifier")

# Transitioning a model through stages -- genuinely mirrors a real deployment lifecycle
from mlflow.tracking import MlflowClient
client = MlflowClient()
client.transition_model_version_stage(
    name="sentiment-classifier", version=1, stage="Production"
)
```

Directly solves file 01's "which exact model is currently running in production" problem — the Model Registry maintains a genuinely clear, versioned record of every model, and which STAGE (Staging, Production, Archived) each version is currently in, giving a single, unambiguous source of truth.

## Loading a tracked model back for inference — connecting directly to Flask file 06

```python
loaded_model = mlflow.sklearn.load_model("models:/sentiment-classifier/Production")

@api_bp.route('/predict', methods=['POST'])
def predict():
    data = request.get_json()
    prediction = loaded_model.predict([data['features']])
    return jsonify({"prediction": prediction.tolist()})
```

Genuinely the direct, practical payoff connecting this file straight back to the entire Flask folder — instead of a hardcoded `joblib.load('model.pkl')` path (recap Flask file 06), the Flask app can load "whatever model is CURRENTLY marked Production" directly from the MLflow registry, letting model updates happen WITHOUT needing to redeploy the Flask app's code at all.

## Try this

```python
# Take a hyperparameter search already run somewhere in this repo (recap XGBoost
# file 08, or a scikit-learn GridSearch) and re-run it wrapped in MLflow tracking --
# log every hyperparameter combination as a separate run, with its resulting
# metrics
# Open the MLflow UI, sort by your primary metric, and confirm you can immediately
# identify the best-performing run and its exact hyperparameters
# Register the best model in the Model Registry, transition it to "Production",
# then write a small script that loads it back using models:/name/Production
# and confirms it produces the same predictions as the original in-memory model
```

Actually experiencing the MLflow UI answer "which run was best" in seconds — a question that would otherwise require manually scrolling through scattered notebook outputs — is genuinely the moment this tool's real value clicks.

## What's next
File 05 — Experiment Tracking with Weights & Biases, covering the other genuinely major, industry-standard experiment tracking tool, with a particular focus on its especially strong visualization and deep learning-specific features.
