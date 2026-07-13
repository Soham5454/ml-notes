# 04 — Linear Models & Regression

## Linear Regression — the simplest, most foundational algorithm
```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

## What's actually happening mathematically (connects directly to the NumPy linear algebra notes)
Linear regression finds the best-fit line (or hyperplane, with multiple features): `y = w1*x1 + w2*x2 + ... + wn*xn + b`. The `.fit()` step is finding the weights (w1, w2, ...) and bias (b) that minimize the squared error between predictions and actual values. This is EXACTLY the same `output = inputs @ weights + bias` pattern from the NumPy ML use cases file — linear regression genuinely IS a single-layer neural network with no activation function, which is a connection that made a lot of DL fundamentals click faster once I saw it explicitly.

## Inspecting the learned parameters
```python
model.coef_          # the learned weights, one per feature
model.intercept_         # the learned bias term
```

## Simple linear regression (recap of the Seaborn regplot visual, now as an actual trained model)
```python
X = df[["hours_studied"]]     # single feature, must still be 2D (recap from file 01)
y = df["exam_score"]

model = LinearRegression()
model.fit(X, y)
print(f"Score = {model.coef_[0]:.2f} * hours + {model.intercept_:.2f}")
```

## Ridge Regression — linear regression + L2 regularization
```python
from sklearn.linear_model import Ridge

model = Ridge(alpha=1.0)     # alpha controls regularization strength
model.fit(X_train, y_train)
```
Regularization adds a penalty for large weight values, which helps prevent overfitting — genuinely the same overfitting concept from the matplotlib DL notes, just addressed here through a mathematical penalty term instead of watching a validation curve. Higher `alpha` = stronger regularization = simpler model (but risk of underfitting if too high).

## Lasso Regression — linear regression + L1 regularization
```python
from sklearn.linear_model import Lasso

model = Lasso(alpha=1.0)
model.fit(X_train, y_train)
```
Key difference from Ridge: Lasso can shrink some weights all the way to EXACTLY zero, effectively removing those features from the model entirely. This makes Lasso genuinely useful for automatic feature selection (recap connects to file 11) — if a feature's weight becomes zero, Lasso is telling you that feature wasn't contributing useful information.

## ElasticNet — combining both Ridge and Lasso
```python
from sklearn.linear_model import ElasticNet

model = ElasticNet(alpha=1.0, l1_ratio=0.5)     # l1_ratio controls the mix between L1 and L2 penalties
```

## Polynomial Regression — fitting curves, not just straight lines
```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

poly = PolynomialFeatures(degree=2)
X_poly = poly.fit_transform(X_train)     # creates x, x^2 (and x1*x2 interaction terms if multiple features)

model = LinearRegression()
model.fit(X_poly, y_train)
```
Genuinely clever trick: this is STILL linear regression underneath (linear in terms of the WEIGHTS), it's just fed transformed features (squared terms, interaction terms) that let it fit curved relationships. Connects directly to the `order=2` parameter from Seaborn's `regplot` — that was fitting exactly this kind of polynomial relationship visually, this is the actual trainable model version.

## Logistic Regression — despite the name, this is for CLASSIFICATION
```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)              # returns actual class labels
probabilities = model.predict_proba(X_test)          # returns probability for EACH class
```
Genuinely confused me at first given the name — "logistic regression" is a classification algorithm, not a regression one. It predicts the PROBABILITY of belonging to a class using the sigmoid/logistic function, then a threshold (usually 0.5) turns that probability into an actual class prediction. Connects directly to the `logistic=True` parameter from Seaborn's `regplot` — that visual S-curve IS what this model actually learns.

## predict_proba — genuinely important, worth understanding well
```python
probabilities = model.predict_proba(X_test)
# array([[0.8, 0.2], [0.3, 0.7], ...])   -- probability of class 0, probability of class 1, per row
```
Having access to the actual PROBABILITY (not just the final predicted class) matters a lot in real applications — e.g. a medical diagnosis model saying "73% confident" is much more useful/actionable than just a flat "yes/no."

## Multi-class logistic regression (more than 2 classes)
```python
model = LogisticRegression(multi_class="multinomial")     # handles 3+ classes directly
```
Modern scikit-learn versions often handle this automatically without needing to specify `multi_class` explicitly, but worth knowing the parameter exists.

## Regularization in Logistic Regression too
```python
model = LogisticRegression(C=1.0)     # NOTE: C is the INVERSE of regularization strength here -- smaller C = STRONGER regularization (opposite direction from Ridge/Lasso's alpha)
```
This inverted convention (`C` instead of `alpha`, and inverted direction) is a genuine gotcha I want to remember — easy to assume it works the same way as Ridge/Lasso's alpha and get the direction backwards.

## When to use linear/logistic regression vs more complex models
Honest take: linear models are fast, interpretable (you can literally read the weights and understand what's driving predictions), and a great BASELINE to compare more complex models against. If a complex model like a random forest only slightly outperforms a simple logistic regression, that's genuinely useful information — sometimes the added complexity isn't worth the loss of interpretability. I plan to always fit a simple linear/logistic model FIRST as a baseline before reaching for anything fancier.
