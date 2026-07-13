# 06 — Matrix Plots (Heatmap & Clustermap)

## heatmap — visualizing a 2D matrix of values, color-coded (recap concept from matplotlib's imshow, much cleaner syntax here)
```python
import numpy as np

data = np.random.rand(10, 12)
sns.heatmap(data)
plt.show()
```

## Adding the actual values as text inside each cell
```python
sns.heatmap(data, annot=True, fmt=".2f")     # annot=True shows the number in each cell, fmt controls decimal formatting
```
This `annot=True` is genuinely one of the most-used parameters — without it, a heatmap only gives a rough visual sense; with it, you get the exact number AND the color-coded visual pattern together.

## The single most common real use case — correlation matrices
```python
correlation_matrix = tips[["total_bill", "tip", "size"]].corr()     # recap from pandas -- .corr() computes pairwise correlation between numeric columns
sns.heatmap(correlation_matrix, annot=True, cmap="coolwarm")
plt.show()
```
This exact pattern (`.corr()` then `sns.heatmap()`) is probably THE most common EDA step I've seen across every tutorial/example — instantly shows which features are correlated with each other, which matters a lot for feature selection before modeling (highly correlated features can be redundant, and understanding relationships with the target variable helps pick useful features).

## Choosing a colormap for correlation data specifically
```python
sns.heatmap(correlation_matrix, annot=True, cmap="coolwarm", center=0, vmin=-1, vmax=1)
```
Since correlation values range from -1 to +1, using a DIVERGING colormap (`coolwarm`, `RdBu`) with `center=0` genuinely matters — it makes strong positive correlations, strong negative correlations, and near-zero correlations all visually distinct, rather than a sequential colormap that would make -1 and +1 look confusingly similar.

## Masking the upper triangle (a common polish for correlation heatmaps)
```python
mask = np.triu(np.ones_like(correlation_matrix, dtype=bool))     # creates a mask for the upper triangle
sns.heatmap(correlation_matrix, annot=True, cmap="coolwarm", mask=mask)
```
Since a correlation matrix is symmetric (correlation of A-with-B equals B-with-A), showing both halves is genuinely redundant — masking the upper triangle removes that duplicate information and makes the plot cleaner to read. Took me a bit to fully understand `np.triu` here, but it directly connects back to boolean masking concepts from the NumPy notes.

## Customizing cell borders and appearance
```python
sns.heatmap(data, linewidths=0.5, linecolor="white")     # adds thin white lines between cells, makes individual cells easier to distinguish
```

## clustermap — heatmap + automatic hierarchical clustering
```python
sns.clustermap(data, cmap="viridis", annot=True)
```
This is genuinely a step beyond a plain heatmap — `clustermap` automatically REORDERS the rows and columns so that similar rows/columns end up next to each other, and shows dendrograms (tree diagrams) on the edges illustrating that clustering. Useful for spotting groups of similar features/samples that wouldn't be obvious in the original row/column order.

## Practical example — a full correlation analysis workflow
```python
numeric_cols = df.select_dtypes(include="number")     # recap from pandas -- grab only numeric columns
corr = numeric_cols.corr()

plt.figure(figsize=(10, 8))
sns.heatmap(corr, annot=True, cmap="coolwarm", center=0, fmt=".2f")
plt.title("Feature Correlation Matrix")
plt.tight_layout()
plt.show()
```
This is genuinely close to a template I expect to reuse on basically every new dataset I work with going forward — correlation heatmap as a standard early EDA step, right alongside `.info()` and `.describe()` from the pandas notes.
