# 12 — Dimensionality Reduction (PCA, t-SNE, LDA)

Different from feature SELECTION (file 11, which picks a SUBSET of original features) — dimensionality reduction CREATES entirely new features that are combinations of the originals, compressing many dimensions into fewer while trying to preserve the important structure/information.

## Why reduce dimensions at all
- **Visualization** — humans can only really see 2D/3D plots, so reducing 50 features down to 2 lets you actually PLOT and visually explore high-dimensional data
- **Speed** — fewer dimensions generally means faster training
- **The "curse of dimensionality"** — as the number of features grows, data becomes increasingly SPARSE in that high-dimensional space, and distance-based algorithms (like KNN from file 05, or K-Means from file 07) genuinely struggle as a result, since "nearest neighbor" starts losing meaning when there's so much empty space between points
- **Removing noise/redundancy** — genuinely correlated features (recap Seaborn correlation heatmap) often contain overlapping information that can be compressed without real information loss

## PCA — Principal Component Analysis, the most common technique
```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2)     # reduce down to 2 dimensions
X_reduced = pca.fit_transform(X_scaled)     # IMPORTANT -- PCA is distance-based, so scaling (file 02) first is genuinely essential
```

### The core idea (connects directly to the NumPy linear algebra eigenvalues/eigenvectors note)
PCA finds new axes (called "principal components") that capture the MAXIMUM variance in the data. The first principal component is the direction of greatest spread in the data; the second is the direction of next-greatest spread, PERPENDICULAR to the first; and so on. This is EXACTLY where the eigenvalues/eigenvectors concept I flagged as "haven't needed this practically yet" in the NumPy notes actually gets used — PCA's principal components ARE the eigenvectors of the data's covariance matrix, and the amount of variance each captures corresponds to the eigenvalues.

### Explained variance — understanding how much information survived
```python
pca.explained_variance_ratio_     # array showing the PROPORTION of total variance each component captures
print(pca.explained_variance_ratio_.sum())     # total variance retained after reduction
```
If reducing 50 features down to 2 components only retains, say, 40% of the total variance, that's a genuinely significant amount of information lost — this number is essential for judging whether the reduction is reasonable or too aggressive.

### Choosing the number of components
```python
pca = PCA()     # no n_components specified -- computes ALL components first
pca.fit(X_scaled)

cumulative_variance = np.cumsum(pca.explained_variance_ratio_)
plt.plot(cumulative_variance)
plt.xlabel("Number of Components")
plt.ylabel("Cumulative Explained Variance")
plt.axhline(y=0.95, color="r", linestyle="--")     # common target: retain 95% of variance
```
This "elbow-style" plot (genuinely similar visual pattern to the K-Means elbow method from file 07) helps decide how many components to KEEP — often choosing the smallest number that retains, say, 90-95% of the original variance, rather than arbitrarily picking 2 or 3.

### Visualizing high-dimensional data after PCA
```python
X_reduced = PCA(n_components=2).fit_transform(X_scaled)
plt.scatter(X_reduced[:, 0], X_reduced[:, 1], c=y, cmap="viridis")
plt.xlabel("Principal Component 1")
plt.ylabel("Principal Component 2")
```
This is genuinely one of the most common practical uses of PCA — squashing a dataset with dozens of features down to 2D specifically so it can be plotted and visually inspected for separability between classes, connecting directly back to matplotlib/Seaborn scatter plot skills.

## t-SNE — a different technique, specifically built for visualization
```python
from sklearn.manifold import TSNE

tsne = TSNE(n_components=2, random_state=42)
X_tsne = tsne.fit_transform(X_scaled)
```
Unlike PCA (which is linear and preserves GLOBAL variance/structure), t-SNE focuses on preserving LOCAL structure — points that were close together in the original high-dimensional space stay close together in the 2D result, even if that distorts the overall global shape. This makes t-SNE genuinely better at revealing tight, well-separated CLUSTERS visually, but the actual axes/distances in a t-SNE plot don't carry the same meaningful interpretation that PCA's components do (you can't say "this cluster is 3x more spread out" the way you sometimes can with PCA).

### Important honest limitation of t-SNE
t-SNE is genuinely for VISUALIZATION ONLY — since it doesn't have a clean, reusable `.transform()` for new unseen data the way PCA does (each run essentially recomputes everything from scratch), it's not something you'd use as a genuine preprocessing step before feeding data into another model, unlike PCA which fits naturally into a Pipeline (file 10).

## LDA — Linear Discriminant Analysis, dimensionality reduction that USES the labels
```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis

lda = LinearDiscriminantAnalysis(n_components=2)
X_lda = lda.fit_transform(X_scaled, y)     # NOTICE -- unlike PCA, LDA needs y too, since it's supervised
```
Key difference from PCA: PCA finds directions of maximum VARIANCE regardless of class labels (unsupervised), while LDA specifically finds directions that BEST SEPARATE the known classes (supervised) — genuinely more effective than PCA specifically when the goal is downstream classification, since LDA is directly optimizing for exactly that separability.

## PCA vs t-SNE vs LDA — when to use which
- **PCA** — general-purpose dimensionality reduction, works as a genuine preprocessing step before modeling, fast, reusable via `.transform()` on new data
- **t-SNE** — purely for VISUALIZING/exploring high-dimensional data in 2D/3D, not for feeding into further modeling
- **LDA** — when you specifically have labels AND the downstream goal is classification — uses that label information to reduce dimensions in the most classification-useful way

## Connecting forward to Deep Learning
This entire "compress high-dimensional data into a smaller, more meaningful representation" idea reappears constantly in DL — autoencoders are essentially a neural-network-based, non-linear generalization of what PCA does linearly here. Understanding PCA properly now genuinely sets up an easier entry point into that concept later, since the GOAL (compress while preserving important information) is identical, just achieved through a very different (learned, non-linear) mechanism.
