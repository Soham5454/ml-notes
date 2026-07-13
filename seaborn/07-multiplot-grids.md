# 07 — Multi-Plot Grids (FacetGrid, PairGrid, pairplot, jointplot)

This is where Seaborn's "figure-level functions automatically create multiple subplots" concept (mentioned in files 01-05) gets its own proper explanation — these tools are specifically about looking at many slices/combinations of data at once, without manually writing a subplot loop.

## FacetGrid — the underlying engine behind col=/row= faceting
```python
g = sns.FacetGrid(tips, col="time", row="smoker")
g.map(sns.scatterplot, "total_bill", "tip")
plt.show()
```
This is genuinely what's happening BEHIND THE SCENES every time I use `col=`/`row=` in relplot/catplot/displot/lmplot — `FacetGrid` creates the grid of subplots based on category combinations, then `.map()` applies a given plot function to EACH subplot using the relevant subset of data automatically.

### Why understanding FacetGrid directly matters
Even though I'll usually use the shortcut versions (`relplot(..., col=...)`), knowing FacetGrid exists underneath means I understand WHAT's actually happening, and I can use FacetGrid directly for plot types that don't have a built-in figure-level shortcut.
```python
g = sns.FacetGrid(tips, col="day", col_wrap=2)     # col_wrap: wrap into a new row after 2 columns, instead of one giant row
g.map(sns.histplot, "total_bill")
```

## pairplot — pairwise relationships between ALL numeric columns at once
```python
iris = sns.load_dataset("iris")
sns.pairplot(iris)
plt.show()
```
This is genuinely one of my favorite single-line EDA tools — it creates a full grid where every numeric column is plotted against every other numeric column (scatter plots off-diagonal), with histograms/KDEs on the diagonal showing each variable's own distribution. One line gives an overview of the ENTIRE dataset's internal relationships at once.

### Adding hue to pairplot
```python
sns.pairplot(iris, hue="species")     # color-codes every single scatter plot AND every diagonal distribution by species
```
This is genuinely one of the very first things I'd run on any new classification dataset — instantly shows which feature PAIRS visually separate the classes well (useful for gut-checking which features might matter most before any actual modeling).

### Controlling which columns to include
```python
sns.pairplot(iris, vars=["sepal_length", "sepal_width"], hue="species")     # only these two columns, instead of ALL numeric columns
```

## PairGrid — the customizable underlying engine behind pairplot (same relationship as FacetGrid to relplot)
```python
g = sns.PairGrid(iris, hue="species")
g.map_diag(sns.histplot)                # what to plot on the diagonal
g.map_offdiag(sns.scatterplot)             # what to plot everywhere else
g.add_legend()
```
Useful when I want DIFFERENT plot types on the diagonal vs off-diagonal cells, something the simpler `pairplot()` shortcut doesn't directly support without extra parameters.

## jointplot — relationship between TWO variables + each variable's own distribution, combined
```python
sns.jointplot(data=tips, x="total_bill", y="tip", kind="scatter")
```
This shows a scatter plot in the middle, with a histogram of `total_bill` along the top and a histogram of `tip` along the side — genuinely a nice combo of the relational (file 02) and distribution (file 03) concepts in one compact figure.

### Different jointplot kinds
```python
sns.jointplot(data=tips, x="total_bill", y="tip", kind="hex")     # hexagonal binning instead of raw scatter points -- better for LARGE datasets where points would overlap too heavily
sns.jointplot(data=tips, x="total_bill", y="tip", kind="kde")        # smoothed 2D density contours instead of scatter points
sns.jointplot(data=tips, x="total_bill", y="tip", kind="reg")           # adds a regression line (recap from file 05) directly into the joint plot
```

## When to reach for each of these
- **pairplot** — fastest first look at an entire dataset's numeric relationships, especially for classification data with a clear category column for hue
- **jointplot** — deep look at exactly TWO variables together, including their individual distributions
- **FacetGrid/PairGrid directly** — only when the shortcut figure-level functions (relplot, pairplot, etc.) don't support the specific customization I need
