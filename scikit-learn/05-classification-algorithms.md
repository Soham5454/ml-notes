# 05 — Classification Algorithms

Beyond logistic regression (file 04), scikit-learn offers several other classification approaches, each with a genuinely different underlying mechanism — worth understanding WHY each one works, not just how to call `.fit()`.

## K-Nearest Neighbors (KNN)
```python
from sklearn.neighbors import KNeighborsClassifier

model = KNeighborsClassifier(n_neighbors=5)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

### How it actually works (genuinely the simplest algorithm conceptually)
KNN doesn't really "learn" anything during `.fit()` in the traditional sense — it just STORES the training data. To predict a new point, it finds the `k` closest training points (using distance, recap the `norm()` concept from the NumPy linear algebra notes — this IS that Euclidean distance formula) and takes a majority vote among their labels.

### Why scaling matters ENORMOUSLY for KNN specifically
Since KNN is entirely distance-based, features on a larger numeric scale will completely dominate the distance calculation — this connects directly back to WHY scaling (file 02) genuinely matters, and KNN is probably the clearest example of an algorithm that breaks badly without it.

### Choosing k (the hyperparameter)
Small k (like k=1) = very flexible, sensitive to noise, higher risk of overfitting. Large k = smoother decision boundary, more stable, but risks underfitting/oversimplifying. This exact bias-variance tradeoff shows up again later in DL as model capacity — same underlying tension, different algorithm.

## Support Vector Machines (SVM)
```python
from sklearn.svm import SVC

model = SVC(kernel="linear")     # or kernel="rbf" for non-linear boundaries
model.fit(X_train, y_train)
```

### The core idea
SVM tries to find the decision boundary (a line/hyperplane) that maximizes the MARGIN — the distance between the boundary and the nearest points of each class (called "support vectors"). Intuitively: instead of just any line that separates classes, find the one with the most "breathing room" on both sides.

### The kernel trick — handling non-linear boundaries
```python
model = SVC(kernel="rbf")     # Radial Basis Function -- can create curved/complex decision boundaries
model = SVC(kernel="poly", degree=3)     # polynomial kernel
```
The kernel trick lets SVM implicitly work in a higher-dimensional space (where a curved boundary in the original space becomes a straight one) WITHOUT actually computing that transformation explicitly — genuinely elegant mathematically, though the full derivation is deeper linear algebra than I've gone into yet.

## Naive Bayes
```python
from sklearn.naive_bayes import GaussianNB

model = GaussianNB()
model.fit(X_train, y_train)
```

### Why "naive"
It assumes ALL features are independent of each other given the class — an assumption that's almost never actually true in real data (features usually correlate with each other somewhat), but the algorithm works surprisingly well in practice anyway, especially for text classification (spam detection is a classic use case). Genuinely fast to train and a good baseline, especially on high-dimensional data.

## Decision Trees
```python
from sklearn.tree import DecisionTreeClassifier

model = DecisionTreeClassifier(max_depth=5)
model.fit(X_train, y_train)
```

### How it works
A decision tree asks a series of yes/no questions about features (e.g. "is age > 30?") splitting the data at each step, trying to create increasingly "pure" groups (mostly one class) at each split. This is genuinely intuitive since it mirrors how humans often reason through decisions.

### Visualizing a tree (genuinely useful for understanding what it learned)
```python
from sklearn.tree import plot_tree
import matplotlib.pyplot as plt

plt.figure(figsize=(15, 10))
plot_tree(model, feature_names=X.columns, class_names=["No", "Yes"], filled=True)
plt.show()
```
Connects directly to the matplotlib skills from earlier — being able to literally SEE every decision the tree made is a genuine advantage over more "black box" models like neural networks, at least for smaller trees.

### Overfitting risk — genuinely high for decision trees specifically
```python
model = DecisionTreeClassifier(max_depth=5, min_samples_leaf=10)     # constraining the tree's growth
```
An unconstrained decision tree will happily grow until every single training example is perfectly classified — which usually means severe overfitting (memorizing training data, recap of the overfitting concept from matplotlib DL notes). `max_depth`, `min_samples_leaf`, and `min_samples_split` are the standard hyperparameters used to constrain tree growth and improve generalization.

## Comparing all classification algorithms honestly
- **Logistic Regression** — fast, interpretable, good baseline, assumes roughly linear decision boundary
- **KNN** — simple, no real training phase, but slow at prediction time on large datasets and sensitive to scaling
- **SVM** — powerful for complex boundaries via kernels, but can be slow to train on large datasets
- **Naive Bayes** — very fast, works well on text/high-dimensional data despite its "naive" independence assumption
- **Decision Trees** — interpretable, handles non-linear relationships naturally, but prone to overfitting alone (this is exactly why ensemble methods in file 06 exist — combining many trees fixes this weakness)

## A practical habit — comparing multiple algorithms quickly
```python
from sklearn.linear_model import LogisticRegression
from sklearn.neighbors import KNeighborsClassifier
from sklearn.tree import DecisionTreeClassifier

models = {
    "Logistic Regression": LogisticRegression(),
    "KNN": KNeighborsClassifier(),
    "Decision Tree": DecisionTreeClassifier()
}

for name, model in models.items():
    model.fit(X_train, y_train)
    score = model.score(X_test, y_test)
    print(f"{name}: {score:.3f}")
```
This loop-based comparison is genuinely how I plan to start any real classification project — quick baseline check across several algorithms before committing to deeper tuning on whichever looks most promising.
