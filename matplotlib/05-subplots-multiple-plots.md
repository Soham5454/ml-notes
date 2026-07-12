# 05 — Subplots & Multiple Plots

Once you need to show more than one chart at a time (very common when comparing things, or showing multiple metrics), this is where the object-oriented style from file 01 really starts paying off.

## Basic grid of subplots
```python
fig, axes = plt.subplots(2, 2)     # 2 rows, 2 columns -- creates a 2x2 grid of axes
```
`axes` here is actually a 2D NumPy array of Axes objects, accessed like `axes[0, 0]`, `axes[0, 1]`, etc.

```python
axes[0, 0].plot([1, 2, 3])
axes[0, 1].bar(["a", "b"], [3, 5])
axes[1, 0].scatter([1, 2, 3], [4, 5, 6])
axes[1, 1].hist(np.random.randn(100))
plt.show()
```

## 1D grid (a single row or column — simpler indexing)
```python
fig, axes = plt.subplots(1, 3)     # 1 row, 3 columns
axes[0].plot([1, 2, 3])
axes[1].bar(["a", "b"], [3, 5])
axes[2].scatter([1, 2], [3, 4])
```
When there's only 1 row OR 1 column, `axes` is a simple 1D array, so single index (`axes[0]`) works instead of needing the `[row, col]` pair.

## Unpacking axes directly (cleaner syntax when you know the exact layout ahead of time)
```python
fig, (ax1, ax2) = plt.subplots(1, 2)     # unpack directly into named variables
ax1.plot([1, 2, 3])
ax2.bar(["a", "b"], [3, 5])
```
I actually prefer this unpacking style over `axes[0]`/`axes[1]` indexing when I know there's a fixed small number of subplots — reads more clearly, similar to how tuple unpacking works elsewhere in Python.

## Setting the overall figure size for a grid
```python
fig, axes = plt.subplots(2, 2, figsize=(12, 8))     # sets the size of the WHOLE figure containing all 4 subplots
```

## Adding a title to the WHOLE figure (not just individual subplots)
```python
fig.suptitle("Overview of Multiple Metrics", fontsize=16)     # one big title spanning the entire figure
```
Different from `ax.set_title()` which only labels a single subplot — `fig.suptitle()` sits above everything.

## Sharing axes between subplots
```python
fig, axes = plt.subplots(1, 2, sharey=True)     # both subplots use the SAME y-axis scale, making visual comparison fairer
```
Genuinely useful when comparing two plots where you want the reader to trust that equal heights/positions actually mean equal values — without `sharey`, matplotlib auto-scales each subplot independently which can be visually misleading when comparing.

## tight_layout — fixing overlapping labels/titles
```python
plt.tight_layout()     # automatically adjusts spacing so titles/labels don't overlap between subplots
```
This is one I now add almost automatically to any multi-subplot figure — without it, labels and titles frequently overlap or get cut off, especially with `figsize` on the smaller side.

## Looping to create subplots dynamically (useful when the number of plots isn't fixed ahead of time)
```python
data_columns = ["age", "salary", "score"]

fig, axes = plt.subplots(1, len(data_columns), figsize=(15, 4))
for ax, col in zip(axes, data_columns):
    ax.hist(df[col])
    ax.set_title(col)
plt.tight_layout()
plt.show()
```
This loop-based pattern is genuinely how I'd handle plotting, say, a histogram for EVERY numeric column in a dataset during EDA, without manually writing out a separate `ax.hist()` call for each one.

## GridSpec — for irregular/complex layouts (a more advanced tool, noting it exists)
```python
fig = plt.figure(figsize=(10, 6))
gs = fig.add_gridspec(2, 2)
ax1 = fig.add_subplot(gs[0, :])       # spans the ENTIRE top row
ax2 = fig.add_subplot(gs[1, 0])          # bottom-left
ax3 = fig.add_subplot(gs[1, 1])             # bottom-right
```
Haven't needed this much yet since regular `subplots()` grids cover most of what I've done so far, but noting it exists for whenever I need a layout that ISN'T a clean uniform grid (like one plot spanning multiple cells).
