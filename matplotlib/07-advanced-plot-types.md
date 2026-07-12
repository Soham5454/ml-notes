# 07 — Advanced Plot Types

## Box plots — showing distribution and outliers at a glance
```python
data = [np.random.normal(0, std, 100) for std in range(1, 4)]

fig, ax = plt.subplots()
ax.boxplot(data, labels=["Group A", "Group B", "Group C"])
plt.show()
```
The box shows the interquartile range (IQR — same concept from the pandas outlier detection notes), the line inside is the median, whiskers extend to the typical range, and individual points beyond the whiskers are flagged outliers. Seeing this plotted made the whole IQR outlier concept click visually instead of just being a formula.

## Violin plots (a fancier alternative to box plots, shows the full distribution shape)
```python
fig, ax = plt.subplots()
ax.violinplot(data)
```
Combines a box-plot-like summary with a mirrored, smoothed distribution shape (like a rotated histogram) — genuinely more informative than a box plot since it shows if a distribution is bimodal (two peaks) or skewed, which a simple box plot would hide.

## Heatmaps via imshow — visualizing a 2D matrix as colored cells
```python
matrix = np.random.rand(10, 10)

fig, ax = plt.subplots()
im = ax.imshow(matrix, cmap="viridis")
plt.colorbar(im)
plt.show()
```
This is the SAME underlying tool used to actually display images (since an image genuinely IS just a 2D — or 3D for color — array of numbers), and also how confusion matrices get visualized in file 09.

### Adding text values on top of heatmap cells (common for confusion matrices)
```python
fig, ax = plt.subplots()
im = ax.imshow(matrix, cmap="Blues")
for i in range(matrix.shape[0]):
    for j in range(matrix.shape[1]):
        ax.text(j, i, f"{matrix[i,j]:.2f}", ha="center", va="center", color="black")
plt.show()
```
This loop-and-annotate pattern is exactly how confusion matrix plots typically display the actual count/percentage inside each cell instead of relying purely on color.

## Contour plots — showing 3D-like data (a surface) in 2D using contour lines
```python
x = np.linspace(-5, 5, 100)
y = np.linspace(-5, 5, 100)
X, Y = np.meshgrid(x, y)     # creates a 2D grid of x,y coordinate pairs -- connects to the meshgrid concept I noted as a gap in the NumPy notes
Z = np.sin(np.sqrt(X**2 + Y**2))

fig, ax = plt.subplots()
contour = ax.contourf(X, Y, Z, cmap="viridis")     # filled contour plot
plt.colorbar(contour)
plt.show()
```
`np.meshgrid` genuinely only made sense to me once I saw it used HERE — it's building the (x,y) coordinate grid that Z values get mapped onto for the plot.

## 3D plots (basic, since matplotlib does have limited 3D support)
```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure()
ax = fig.add_subplot(111, projection="3d")

x = np.random.rand(50)
y = np.random.rand(50)
z = np.random.rand(50)
ax.scatter(x, y, z)
plt.show()
```
Honest note: matplotlib's 3D plotting is functional but genuinely a bit clunky/limited compared to dedicated libraries like Plotly — fine for quick exploratory 3D scatter plots, but I wouldn't rely on it for anything that needs to look polished.

## Error bars — showing uncertainty/variance on a plot
```python
x = np.arange(5)
y = np.array([10, 15, 13, 18, 16])
errors = np.array([1, 2, 1.5, 1, 2])

fig, ax = plt.subplots()
ax.errorbar(x, y, yerr=errors, fmt="o-", capsize=5)
plt.show()
```
Common when showing results with confidence intervals or standard deviation — genuinely relevant for reporting model performance across multiple runs (e.g. average accuracy ± standard deviation across 5 training runs).

## Stack plots — cumulative area over a category (like x-axis often being time)
```python
x = np.arange(5)
y1 = np.array([1, 2, 3, 4, 5])
y2 = np.array([2, 3, 4, 5, 6])

fig, ax = plt.subplots()
ax.stackplot(x, y1, y2, labels=["A", "B"])
ax.legend()
```

## Polar plots (specialized, rare, but noting it exists)
```python
fig, ax = plt.subplots(subplot_kw={"projection": "polar"})
theta = np.linspace(0, 2*np.pi, 100)
r = np.abs(np.sin(3*theta))
ax.plot(theta, r)
```
Haven't needed this in practice, but useful to know it exists for anything genuinely angular/directional (compass data, cyclical patterns).
