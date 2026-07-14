# 16 — Correlation & Regression Theory

The statistical closing chapter — connecting file 12's correlation, file 15's MLE, and the linear/logistic regression already used throughout the scikit-learn notes into one complete picture.

## Correlation, revisited with full rigor

Recap file 12's correlation formula — Pearson correlation coefficient `r` measures the strength and direction of a LINEAR relationship, ranging from -1 to 1.

```python
import numpy as np

x = np.array([1, 2, 3, 4, 5])
y_linear = 2*x + 1                         # perfect linear relationship
y_nonlinear = x**2                          # strong relationship, but NOT linear
y_random = np.array([3, 1, 4, 1, 5])        # essentially unrelated

print(np.corrcoef(x, y_linear)[0,1])     # ~1.0
print(np.corrcoef(x, y_nonlinear)[0,1])  # notably less than 1.0, despite a very strong TRUE relationship
print(np.corrcoef(x, y_random)[0,1])     # close to 0
```

Genuinely important, commonly-missed point: `y_nonlinear = x^2` has a PERFECT deterministic relationship with x, yet Pearson correlation doesn't fully capture it, because that relationship is curved, not straight. Correlation specifically measures LINEAR association — a low correlation value never proves "no relationship," only "no strong LINEAR relationship." This is precisely why EDA (recap Seaborn scatterplot notes) should always include visual inspection, not just a correlation number.

## Correlation does not imply causation — the classic caveat, with a concrete mechanism

```python
# Classic example: ice cream sales and drowning incidents are strongly correlated
# NEITHER causes the other -- both are driven by a THIRD variable: hot weather
# This is called a "confounding variable"
```

Genuinely worth having a concrete mental example ready (like this one) rather than just repeating the phrase "correlation isn't causation" without a working explanation of HOW two unrelated things can still correlate strongly — a confounding variable driving both is the most common real mechanism.

## Simple linear regression — deriving it properly, connecting files 06/07/15

Recap scikit-learn's `LinearRegression()` — here's what it's actually solving:

```
y = β0 + β1*x + ε
```

`β0` is the intercept, `β1` is the slope, `ε` is the error term (assumed normally distributed, recap file 15's MLE-MSE connection). The best-fit line minimizes the sum of squared residuals — exactly the MSE loss from PyTorch file 06, exactly the MLE-derived "correct" loss under Gaussian error assumptions from file 15.

```python
from sklearn.linear_model import LinearRegression

x = np.array([1, 2, 3, 4, 5]).reshape(-1, 1)
y = np.array([2.1, 3.9, 6.2, 7.8, 10.1])

model = LinearRegression()
model.fit(x, y)
print(model.coef_, model.intercept_)   # beta1, beta0
```

## The closed-form solution — where file 02's matrix inverse finally gets used

Recap file 02's matrix inverse note ("shows up in the closed-form solution to linear regression") — here it is, fully:

```
β = (X^T X)^-1 X^T y
```

```python
X = np.column_stack([np.ones(len(x)), x.flatten()])   # add intercept column of 1s
y_vec = y

beta = np.linalg.inv(X.T @ X) @ X.T @ y_vec
print(beta)   # [intercept, slope] -- should match sklearn's model.intercept_ and model.coef_
```

This is genuinely the "Normal Equation" — an exact, closed-form way to solve linear regression WITHOUT gradient descent at all, because the loss surface for linear regression is a simple convex bowl (recap file 08's convexity discussion) with a solvable minimum. Neural networks can't use this shortcut (their loss surfaces are non-convex, recap file 08), which is precisely why they need iterative gradient descent instead — this closes the loop on why linear regression gets a "cheat" that deep learning doesn't.

## R-squared — how well does the model actually explain the data

```python
from sklearn.metrics import r2_score

y_pred = model.predict(x)
r2 = r2_score(y, y_pred)
print(r2)   # close to 1.0 for this near-perfect linear data
```

`R²` represents the proportion of variance in `y` that's explained by the model — `R² = 1` means perfect fit, `R² = 0` means the model does no better than just predicting the mean every time. Genuinely useful, standard regression evaluation metric, complementing MSE (which is in the target's units and harder to interpret as a "quality percentage").

## Multicollinearity — when correlated FEATURES cause real problems

Recap file 03's rank/determinant discussion — when two or more input features are highly correlated with EACH OTHER (not with the target), the `(X^T X)` matrix in the Normal Equation above becomes close to singular (low rank, near-zero determinant) — genuinely causing unstable, unreliable coefficient estimates (small changes in data cause big swings in the fitted coefficients).

```python
# Two nearly-identical (highly correlated) features
X_multicollinear = np.column_stack([
    np.ones(5),
    x.flatten(),
    x.flatten() * 1.001 + np.random.randn(5)*0.01   # almost a duplicate of the previous column
])
det = np.linalg.det(X_multicollinear.T @ X_multicollinear)
print(det)   # very close to 0 -- numerically unstable to invert
```

This is genuinely the concrete, provable reason why redundant/highly-correlated features are flagged as a problem during EDA (recap the correlation heatmap notes) — not just a vague "it's bad practice," but a specific, derivable numerical instability rooted in file 03's determinant/rank theory.

## Logistic regression — the classification analog, tying back to file 09/15

Recap scikit-learn's `LogisticRegression()` — genuinely NOT "regression" in the linear-regression sense; it models `P(y=1 | x)` using the sigmoid function (recap PyTorch file 05), and its parameters are estimated via MLE (recap file 15) under a Bernoulli likelihood (recap file 10) instead of a Gaussian one — which is precisely why its loss is cross-entropy/log-loss, not MSE.

```python
from sklearn.linear_model import LogisticRegression

# Same MLE logic from file 15, just with a Bernoulli likelihood instead of Gaussian
# loss = -log(P(y | x)) = binary cross-entropy, recap PyTorch file 06
```

## Try this

```python
# Generate x and y with a genuine QUADRATIC relationship (y = x^2 + small noise)
# 1. Fit a LinearRegression model anyway and check its R^2 -- it should be noticeably imperfect
# 2. Compute Pearson correlation between x and y -- confirm it's NOT close to 1, despite
#    a very strong (just non-linear) true relationship
# 3. Now fit a model using x^2 as the feature instead of x, and confirm R^2 improves dramatically
```

This exercise genuinely ties together the whole file: correlation's blindness to non-linear relationships, why the "right" features matter as much as the right model, and how a simple feature transformation can turn a bad linear fit into a great one — all provable with the same tools used throughout this entire Math Foundations folder.

## What's next
File 17 — the final Math → ML Bridge file, pulling every concept from files 01-16 together into one map connecting straight back to the scikit-learn, XGBoost, and PyTorch folders already built.
