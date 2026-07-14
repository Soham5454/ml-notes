# 03 — Distribution Plots

Showing how a single variable (or the relationship between two) is distributed — this is where Seaborn genuinely outshines plain matplotlib the most, since building these manually in matplotlib is a lot more code for a worse-looking result.

## histplot — histogram, but nicer defaults than matplotlib's
```python
sns.histplot(data=tips, x="total_bill")
plt.show()
```

### Customizing bins and adding a KDE curve on top
```python
sns.histplot(data=tips, x="total_bill", bins=20)
sns.histplot(data=tips, x="total_bill", kde=True)     # overlays a smoothed density curve on top of the histogram bars -- genuinely useful for seeing the underlying shape more clearly than bars alone
```

## kdeplot — Kernel Density Estimate, a smoothed version of a histogram
```python
sns.kdeplot(data=tips, x="total_bill")
```
Think of KDE as "what the histogram would look like if you had infinite data and infinitely thin bins, smoothed out" — it shows the underlying probability distribution shape without the somewhat arbitrary choice of bin width that histograms require.

### Filled KDE plots
```python
sns.kdeplot(data=tips, x="total_bill", fill=True)     # shades the area under the curve, visually closer to a histogram
```

### Comparing distributions with hue
```python
sns.kdeplot(data=tips, x="total_bill", hue="time", fill=True)     # overlapping density curves for lunch vs dinner, auto-colored
```
This overlapping-KDE-by-category pattern is genuinely one of my favorite EDA tools now — instantly shows whether two groups have meaningfully different distributions, which would take real effort to see clearly with overlapping histograms alone (transparency issues, bin-alignment issues).

## displot — figure-level distribution plot (same "figure-level enables faceting" pattern from file 02)
```python
sns.displot(data=tips, x="total_bill", kind="hist")
sns.displot(data=tips, x="total_bill", kind="kde")
sns.displot(data=tips, x="total_bill", col="time")     # separate histogram per time value, auto-faceted
```

## rugplot — small tick marks showing every individual data point along an axis
```python
sns.histplot(data=tips, x="total_bill")
sns.rugplot(data=tips, x="total_bill")     # adds tick marks along the bottom showing EXACTLY where each real data point sits
```
Rug plots are usually layered UNDER another plot (like above) rather than used alone — they show the raw data points directly, which can reveal things a smoothed histogram/KDE might hide, like genuine gaps or clusters in the raw data.

## 2D distributions — joint distribution of two numeric variables
```python
sns.histplot(data=tips, x="total_bill", y="tip")     # 2D histogram -- darker cells mean more data points fall in that (bill, tip) combination
sns.kdeplot(data=tips, x="total_bill", y="tip")          # 2D KDE -- smooth contour version of the same idea
```
This 2D version genuinely surprised me the first time — same function names as the 1D versions (`histplot`, `kdeplot`), just pass both x AND y, and Seaborn automatically switches to the 2D joint-distribution version instead of the 1D marginal version.

## ecdfplot — Empirical Cumulative Distribution Function (less common, but useful for a specific question)
```python
sns.ecdfplot(data=tips, x="total_bill")
```
Instead of showing "how many observations fall in this bucket" (histogram), this shows "what proportion of observations are LESS than or equal to X" — genuinely useful for answering questions like "what percentage of bills are under $20."

## Normalizing histograms — stat parameter
```python
sns.histplot(data=tips, x="total_bill", stat="density")        # normalized so total area = 1, comparable to a KDE's scale
sns.histplot(data=tips, x="total_bill", stat="probability")       # normalized so bars sum to 1 (percentages)
sns.histplot(data=tips, x="total_bill", stat="count")                # raw counts (default)
```
Useful when comparing two groups of very different sample sizes — raw counts would be misleading (the bigger group just LOOKS like it "has more"), but density/probability normalization puts them on a fair, comparable scale.
