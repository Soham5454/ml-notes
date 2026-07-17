# 08 — CI/CD for ML Pipelines

Recap file 07's closing note — extending general CI/CD with genuinely ML-SPECIFIC automation: model evaluation gates, data validation checks, and automated retraining triggers. This is precisely where file 01's "code, data, model" three-way risk gets addressed with real automated tooling.

## Why standard software CI genuinely isn't enough for ML

```python
# Recap file 07's pytest tests -- these verify CODE correctness
# (does the /predict endpoint return a 200, does it reject invalid input)
# But they DON'T verify: is the MODEL actually good enough to deploy?
# A model that passes every code test could STILL be a genuinely terrible
# model (e.g. accuracy dropped from 90% to 60% after a bad retraining run)
```

Genuinely the core gap this file fills — ML pipelines need an ADDITIONAL layer of automated checks specifically about MODEL QUALITY and DATA VALIDITY, beyond standard code correctness testing.

## Model evaluation gates — genuinely the core new concept in this file

```python
# tests/test_model_quality.py
import mlflow      # recap MLOps file 04 directly

MINIMUM_ACCURACY = 0.85
MINIMUM_F1 = 0.80

def test_model_meets_minimum_performance():
    model = mlflow.sklearn.load_model("models:/sentiment-classifier/Staging")
    X_test, y_test = load_held_out_test_set()      # a GENUINELY fixed, versioned test set (recap file 06's DVC)

    predictions = model.predict(X_test)
    accuracy = accuracy_score(y_test, predictions)      # recap scikit-learn's metrics folder
    f1 = f1_score(y_test, predictions)

    assert accuracy >= MINIMUM_ACCURACY, f"Accuracy {accuracy} below minimum {MINIMUM_ACCURACY}"
    assert f1 >= MINIMUM_F1, f"F1 {f1} below minimum {MINIMUM_F1}"
```

Genuinely worth recognizing this as a REAL pytest test (recap Flask file 08's exact `assert` pattern), just testing MODEL QUALITY instead of API behavior — this can be run in the exact same GitHub Actions workflow structure from file 07, adding a genuinely essential "is this model actually good enough" gate before any deployment happens.

## Comparing against the CURRENT production model — genuinely important, not just an absolute threshold

```python
def test_new_model_beats_current_production():
    new_model = mlflow.sklearn.load_model("models:/sentiment-classifier/Staging")
    current_model = mlflow.sklearn.load_model("models:/sentiment-classifier/Production")

    X_test, y_test = load_held_out_test_set()

    new_accuracy = accuracy_score(y_test, new_model.predict(X_test))
    current_accuracy = accuracy_score(y_test, current_model.predict(X_test))

    assert new_accuracy >= current_accuracy, "New model does not outperform current production model"
```

Genuinely important, real practice beyond a fixed threshold — an absolute minimum (like `MINIMUM_ACCURACY` above) can become STALE as a model improves over time (recap the general "don't just set it and forget it" caution). Comparing a NEW candidate model directly against the CURRENT production model ensures every deployment is a genuine, provable IMPROVEMENT, not just "good enough" by some fixed historical bar.

## Data validation — genuinely essential, using Great Expectations conceptually

```python
def validate_training_data(df):      # recap pandas folder directly
    assert df['age'].min() >= 0, "Found negative age values"
    assert df['age'].max() <= 120, "Found unrealistic age values"
    assert df['label'].isnull().sum() == 0, "Found missing labels"      # recap pandas .isna() discussion
    assert df['label'].nunique() == 2, "Expected exactly 2 classes"      # recap scikit-learn's classification setup

    expected_columns = {'age', 'income', 'label'}
    assert set(df.columns) == expected_columns, f"Unexpected columns: {set(df.columns) - expected_columns}"
```

Genuinely worth recognizing this as the EXACT SAME data-quality checks from Classical NLP file 13's "detecting data quality issues in SQL" discussion, and the general pandas/EDA discipline from throughout this roadmap — now AUTOMATED as assertions that run EVERY TIME new training data arrives, rather than a manual, one-time EDA check. Tools like **Great Expectations** genuinely formalize this pattern with a dedicated framework, but the underlying IDEA — codify data quality assumptions as testable assertions — is precisely what matters, regardless of tooling.

## A complete ML CI/CD workflow — recap file 07's GitHub Actions structure

```yaml
# .github/workflows/ml-pipeline.yml
name: ML Pipeline

on:
  push:
    paths:
      - 'data/**'        # genuinely important -- trigger specifically when DATA changes
      - 'src/train.py'      # or when training CODE changes

jobs:
  validate-data:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - run: python -m pytest tests/test_data_validation.py

  train-and-evaluate:
    needs: validate-data      # recap file 07's needs: dependency-chain discussion directly
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: dvc pull      # recap MLOps file 06 -- pull the exact VERSIONED training data
      - run: python src/train.py      # trains and logs to MLflow (recap file 04)
      - run: python -m pytest tests/test_model_quality.py      # the model quality gate from this file

  deploy:
    needs: train-and-evaluate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'      # genuinely important -- only deploy from the main branch
    steps:
      - name: Promote model to production
        run: python scripts/promote_model.py      # recap file 04's MLflow stage transition
```

Genuinely the complete, realistic picture this entire folder has been building toward — `paths:` triggers specifically on data/training-code changes (not every unrelated code push), data validation runs FIRST, training only proceeds if data is valid, and DEPLOYMENT only happens if the newly trained model genuinely PASSES its quality gate — a fully automated, safety-checked pipeline from raw data change to production deployment.

## Automated retraining triggers — genuinely closing the full MLOps loop

```yaml
on:
  schedule:
    - cron: '0 2 * * 0'      # genuinely runs automatically every Sunday at 2 AM
  workflow_dispatch:            # ALSO allows manually triggering it on demand
```

Genuinely worth knowing this pattern exists for real, ongoing ML systems — rather than only retraining when a human remembers to, a SCHEDULED workflow can automatically re-run the entire validate → train → evaluate → deploy pipeline periodically, directly connecting to file 01's full lifecycle diagram's final "retraining & iteration" loop-back step.

## Shadow testing — a genuinely important, safer alternative to trusting evaluation metrics alone

```python
# Genuinely worth knowing this concept exists, full treatment in file 10:
# run the NEW model alongside the CURRENT production model on REAL live traffic,
# logging both predictions but only ACTUALLY using the current model's output --
# compare their real-world agreement/performance before fully switching over
```

## Try this

```python
# Take a scikit-learn model already tracked via MLflow (recap file 04's exercise)
# Write a test_model_quality.py with a minimum accuracy threshold AND a
# "beats current production" comparison test, both using pytest assertions
# Write a separate test_data_validation.py checking for nulls, expected value
# ranges, and expected column names on your training data
# Build a GitHub Actions workflow that runs data validation FIRST, then model
# training/evaluation ONLY if data validation passes, mirroring the dependency
# structure shown in this file's complete example
# Deliberately corrupt your training data (introduce a null value in a
# required column) and confirm the pipeline correctly stops at the data
# validation stage, never reaching model training at all
```

Deliberately corrupting the data and confirming the pipeline stops EARLY (rather than training a model on bad data and only failing later) is genuinely the clearest demonstration of why data validation belongs at the START of the pipeline, not as an afterthought.

## What's next
File 09 — Model Monitoring & Drift Detection, covering what happens AFTER a model is successfully deployed — directly solving file 01's "the model has been silently getting worse for weeks and nobody noticed" problem.
