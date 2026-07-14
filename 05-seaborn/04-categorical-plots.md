# 04 — Categorical Plots

Plots specifically for when one variable is categorical (like day of week, gender, department) and you want to compare a numeric variable ACROSS those categories. This is genuinely one of the most-used plot families in real EDA work.

## boxplot — distribution summary per category (recap from matplotlib file 07, much easier syntax here)
```python
sns.boxplot(data=tips, x="day", y="total_bill")
plt.show()
```
Same IQR/median/outlier concept as the matplotlib version, but here it's ONE line with column names instead of manually splitting data into groups first.

### Adding hue for a second categorical split
```python
sns.boxplot(data=tips, x="day", y="total_bill", hue="smoker")     # side-by-side boxes per day, split further by smoker/non-smoker
```

## violinplot — box plot + distribution shape combined (recap from matplotlib file 07)
```python
sns.violinplot(data=tips, x="day", y="total_bill")
sns.violinplot(data=tips, x="day", y="total_bill", hue="smoker", split=True)     # split=True puts each hue category on ONE HALF of the violin instead of side-by-side -- genuinely clever way to save horizontal space while still comparing two groups
```
The `split=True` trick is one of those small Seaborn touches that would be a real pain to replicate manually in matplotlib — each violin gets cut in half, smoker on one side, non-smoker on the other, directly comparable at a glance.

## barplot — shows a summary statistic (mean by default) per category, WITH error bars
```python
sns.barplot(data=tips, x="day", y="total_bill")     # bar height = MEAN total_bill per day, black line = confidence interval
```
Important distinction from a plain count: `sns.barplot` is NOT just counting rows — it's aggregating (mean by default) a numeric column per category, and showing uncertainty via the error bar. This confused me initially since a "bar plot" sounds like it should just be counts.

### Changing the aggregation function
```python
sns.barplot(data=tips, x="day", y="total_bill", estimator="median")     # bar height = median instead of mean
```

## countplot — the ACTUAL simple "count how many rows per category" plot
```python
sns.countplot(data=tips, x="day")     # just counts rows per day, no y= needed since it's counting, not aggregating
```
This is the one I actually reach for when I want plain frequency counts — `barplot` for aggregated numeric summaries, `countplot` for simple row counts. Mixing these two up was a genuine early confusion for me.

## stripplot — shows EVERY individual point, jittered so they don't overlap
```python
sns.stripplot(data=tips, x="day", y="total_bill", jitter=True)
```
Unlike box/violin plots which summarize, stripplot shows every raw data point directly, with small random horizontal jitter so overlapping points remain visible instead of stacking into a single line.

## swarmplot — like stripplot, but arranges points to NEVER overlap (more precise, can get slow on huge datasets)
```python
sns.swarmplot(data=tips, x="day", y="total_bill")
```
Genuinely nicer-looking than stripplot for smaller datasets since points never visually collide, but noticeably slower to render on datasets with many thousands of points — stripplot scales better for large data.

## Combining a summary plot with raw points (a genuinely great EDA combo)
```python
sns.boxplot(data=tips, x="day", y="total_bill", color="lightgray")
sns.stripplot(data=tips, x="day", y="total_bill", color="black", alpha=0.5)
plt.show()
```
Layering a stripplot ON TOP of a boxplot (just calling both functions back to back on the same axes) shows both the summary statistics AND the actual raw data distribution simultaneously — this combo genuinely changed how I think about EDA, since a box plot alone can hide things like bimodal distributions or genuinely small sample sizes per group.

## catplot — figure-level version (same faceting pattern as relplot/displot)
```python
sns.catplot(data=tips, x="day", y="total_bill", kind="box", col="time")     # separate boxplot subplot per time value
sns.catplot(data=tips, x="day", y="total_bill", kind="violin", col="sex", hue="smoker")
```
`kind=` here can be `"strip"`, `"swarm"`, `"box"`, `"violin"`, `"bar"`, `"count"`, `"point"` — same "one figure-level entry point, kind decides the actual plot type" pattern as relplot and displot.

## pointplot — shows means with connecting lines between categories (good for spotting trends across ordered categories)
```python
sns.pointplot(data=tips, x="day", y="total_bill", hue="sex")
```
Useful specifically when categories have a natural order (like days of the week) and you want to visually emphasize the TREND across them, not just compare isolated bars.
