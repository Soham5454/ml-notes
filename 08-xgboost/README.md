# XGBoost — A to Z

Notes on XGBoost (eXtreme Gradient Boosting), learned right after scikit-learn. This is a deeper, more powerful, more optimized implementation of the Gradient Boosting concept already covered in the scikit-learn ensemble methods notes — genuinely the industry-standard tool for structured/tabular data problems, and a common competition-winning choice.

## Index

| # | Topic | File |
|---|-------|------|
| 01 | Introduction & Boosting Recap | [01-introduction-boosting-recap.md](01-introduction-boosting-recap.md) |
| 02 | Installation & Basic Usage | [02-installation-basic-usage.md](02-installation-basic-usage.md) |
| 03 | Core Parameters | [03-xgboost-parameters.md](03-xgboost-parameters.md) |
| 04 | Classification & Regression | [04-classification-regression.md](04-classification-regression.md) |
| 05 | Regularization & Overfitting Control | [05-regularization-overfitting-control.md](05-regularization-overfitting-control.md) |
| 06 | Cross-Validation & Early Stopping | [06-cross-validation-early-stopping.md](06-cross-validation-early-stopping.md) |
| 07 | Feature Importance & Interpretation | [07-feature-importance-interpretation.md](07-feature-importance-interpretation.md) |
| 08 | Hyperparameter Tuning for XGBoost | [08-hyperparameter-tuning-xgboost.md](08-hyperparameter-tuning-xgboost.md) |
| 09 | Handling Missing Data & Imbalanced Classes | [09-handling-missing-imbalanced-data.md](09-handling-missing-imbalanced-data.md) |
| 10 | GPU Acceleration | [10-gpu-acceleration.md](10-gpu-acceleration.md) |
| 11 | ML/DL Bridge | [11-ml-dl-bridge.md](11-ml-dl-bridge.md) |

## Setup
```bash
pip install xgboost
```
```python
import xgboost as xgb     # 'xgb' is the universal convention
```

## Roadmap position
Python Basics → NumPy → pandas → matplotlib → Seaborn → scikit-learn → **XGBoost (you are here)** → PyTorch → Hugging Face
