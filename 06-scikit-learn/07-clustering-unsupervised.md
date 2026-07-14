# 07 — Clustering & Unsupervised Learning

Everything up to this point (files 04-06) has been SUPERVISED learning — we had labeled data (y) telling the model the "correct answer." This file covers UNSUPERVISED learning, where there's no y at all — the algorithm has to find structure in the data purely on its own.

## K-Means Clustering — the most common clustering algorithm
```python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, random_state=42)
model.fit(X)     # notice -- NO y here, since this is unsupervised
labels = model.labels_     # which cluster each data point was assigned to
```

### How K-Means actually works
1. Randomly place `k` "centroids" (cluster centers) in the feature space
2. Assign every data point to its NEAREST centroid (using distance, same concept as KNN from file 05)
3. Move each centroid to the actual MEAN position of all points assigned to it
4. Repeat steps 2-3 until the centroids stop moving significantly

## Predicting cluster assignment for new data
```python
new_point_cluster = model.predict(X_new)     # predict() works here too, assigns new points to the nearest existing centroid
```

## Choosing k — the Elbow Method (since k is a hyperparameter YOU must choose)
```python
inertias = []
for k in range(1, 11):
    model = KMeans(n_clusters=k, random_state=42)
    model.fit(X)
    inertias.append(model.inertia_)     # inertia = sum of squared distances from points to their assigned centroid -- lower is "tighter" clusters

plt.plot(range(1, 11), inertias, marker="o")
plt.xlabel("Number of Clusters (k)")
plt.ylabel("Inertia")
plt.title("Elbow Method")
plt.show()
```
As k increases, inertia always decreases (more clusters = tighter fit, trivially true), but there's usually a point where the improvement sharply slows down — that "elbow" bend in the curve is the suggested choice for k. Genuinely a judgment call based on where the curve visibly bends, not a hard mathematical rule — this connects directly back to the matplotlib line plot skills for actually visualizing and reading this curve.

## Silhouette Score — a more rigorous way to evaluate clustering quality
```python
from sklearn.metrics import silhouette_score

score = silhouette_score(X, model.labels_)     # ranges from -1 to 1, higher is better
```
Measures how similar each point is to its own cluster compared to other clusters — genuinely more principled than just eyeballing the elbow curve, and directly comparable across different k values.

## Hierarchical Clustering — builds a full tree of nested clusters
```python
from sklearn.cluster import AgglomerativeClustering

model = AgglomerativeClustering(n_clusters=3)
labels = model.fit_predict(X)
```
Starts with every point as its own cluster, then repeatedly merges the two CLOSEST clusters together until reaching the target number of clusters (or a single cluster containing everything, if you don't specify a target). This connects directly to Seaborn's `clustermap` from the Seaborn matrix plots notes — that dendrogram visualization IS the tree this algorithm builds.

### Visualizing the dendrogram
```python
from scipy.cluster.hierarchy import dendrogram, linkage

linked = linkage(X, method="ward")
dendrogram(linked)
plt.show()
```
Genuinely useful for deciding how many clusters make sense — you can visually see at what "height" the tree naturally splits into distinct groups, rather than needing to pre-specify k like K-Means requires.

## DBSCAN — density-based clustering, genuinely different approach
```python
from sklearn.cluster import DBSCAN

model = DBSCAN(eps=0.5, min_samples=5)
labels = model.fit_predict(X)     # points labeled -1 are considered NOISE/outliers, not part of any cluster
```
Unlike K-Means, DBSCAN doesn't require specifying the number of clusters upfront — it finds dense regions of points and treats sparse/isolated points as noise/outliers automatically. Genuinely better than K-Means for clusters with irregular (non-spherical) shapes, and has the useful side effect of automatic outlier detection built in.

### K-Means vs DBSCAN vs Hierarchical — when to use which
- **K-Means** — fast, scales well to large datasets, but assumes roughly spherical/similarly-sized clusters and requires choosing k upfront
- **DBSCAN** — handles irregular cluster shapes and noise/outliers naturally, but sensitive to the `eps`/`min_samples` hyperparameters and struggles when clusters have very different densities
- **Hierarchical** — gives the most INTERPRETABLE structure (the full dendrogram tree), but genuinely slow on large datasets (computationally expensive)

## Practical example — customer segmentation (a genuinely common real-world clustering use case)
```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

X = customer_df[["annual_spending", "visit_frequency"]]
X_scaled = StandardScaler().fit_transform(X)     # recap -- scaling matters for K-Means too, since it's distance-based like KNN

model = KMeans(n_clusters=4, random_state=42)
customer_df["segment"] = model.fit_predict(X_scaled)

sns.scatterplot(data=customer_df, x="annual_spending", y="visit_frequency", hue="segment", palette="Set2")
```
This pattern — cluster customers/users into segments based on behavior, then treat the cluster label as a genuinely new categorical feature — is a real, common business use case, and directly ties back to the Seaborn scatterplot skills for visualizing the resulting groups.

## Connection to Dimensionality Reduction (file 12)
Clustering and dimensionality reduction (PCA especially) are both unsupervised techniques and are often used TOGETHER — reduce dimensions first to make clustering faster/more meaningful in high-dimensional data, then cluster on the reduced representation. Full detail on this combo comes in file 12.
