# 02 — Relational Plots (Scatter & Line)

"Relational" plots show the relationship between two numeric variables — this is the category `scatterplot` and `lineplot` both belong to.

## scatterplot — the basic relationship plot
```python
sns.scatterplot(data=tips, x="total_bill", y="tip")
plt.show()
```

## Adding more dimensions with hue, size, and style
```python
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="day")               # color by category
sns.scatterplot(data=tips, x="total_bill", y="tip", size="size")                # point SIZE encodes another numeric variable
sns.scatterplot(data=tips, x="total_bill", y="tip", style="smoker")                # point SHAPE encodes another category
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="day", size="size", style="smoker")     # all four dimensions at once (x, y, color, size, shape)
```
This "stack multiple encodings" ability is genuinely wild once you see it work — a single 2D scatter plot showing 5 dimensions of information (x, y, color, size, marker shape) without needing separate subplots. Definitely don't overdo this in practice though — readability drops fast past 2-3 encoded dimensions.

## Controlling the color palette
```python
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="day", palette="viridis")
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="size", palette="coolwarm")     # good for numeric hue variables (sequential/diverging data)
```

## lineplot — for trends, typically over time or an ordered variable
```python
flights = sns.load_dataset("flights")
sns.lineplot(data=flights, x="year", y="passengers")
plt.show()
```

## The automatic confidence interval — a genuinely unique Seaborn feature
```python
sns.lineplot(data=flights, x="year", y="passengers")     # if there are MULTIPLE y values per x, Seaborn automatically shows a shaded confidence interval band around the line
```
This confidence interval shading happens automatically whenever there's more than one observation per x-value — matplotlib's plain `plt.plot()` would just connect points and give you no sense of the underlying variability, but Seaborn shows you the uncertainty visually without extra code. This tripped me up pleasantly the first time — I didn't ask for it, it just appeared, and once I understood what it meant it became genuinely useful.

## Turning off the confidence interval (if you don't want it)
```python
sns.lineplot(data=flights, x="year", y="passengers", errorbar=None)
```

## Multiple lines using hue
```python
sns.lineplot(data=flights, x="year", y="passengers", hue="month")     # one line PER month, auto-colored, auto-legend
```

## relplot — the figure-level version, enables faceting (recap of the concept from file 01)
```python
sns.relplot(data=tips, x="total_bill", y="tip", kind="scatter", col="time")     # separate subplot for lunch vs dinner
sns.relplot(data=tips, x="total_bill", y="tip", kind="line", col="day", hue="sex")     # a full grid: one subplot per day, colored by sex within each
```
`relplot` is essentially the shared entry point for BOTH scatter and line plots at the figure level — `kind="scatter"` or `kind="line"` decides which one you get, and it's the same function either way. This "kind=" parameter pattern shows up in other figure-level functions too (catplot in file 04, displot in file 03).

## When to use scatterplot/lineplot directly vs relplot
Rule I've settled on: use the direct axes-level function (`sns.scatterplot`, `sns.lineplot`) when I just need ONE plot, especially if I'm combining it with matplotlib subplots manually. Reach for `relplot` specifically when I want Seaborn to automatically handle faceting into multiple subplots via `col=`/`row=` — no point using the figure-level version for a single simple plot since it adds unnecessary complexity for that case.
