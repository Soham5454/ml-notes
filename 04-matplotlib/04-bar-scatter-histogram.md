# 04 — Bar, Scatter & Histogram

## Bar plots
```python
categories = ["A", "B", "C", "D"]
values = [23, 45, 12, 36]

fig, ax = plt.subplots()
ax.bar(categories, values)
plt.show()
```

### Horizontal bar plots
```python
ax.barh(categories, values)     # horizontal bars -- better when category names are long and would overlap on the x-axis
```

### Customizing bar colors
```python
ax.bar(categories, values, color="skyblue")
ax.bar(categories, values, color=["red", "green", "blue", "orange"])     # different color per bar
```

### Grouped bar charts (comparing two things side by side per category)
```python
x = np.arange(len(categories))     # numeric positions for the categories
width = 0.35                          # width of each bar

fig, ax = plt.subplots()
ax.bar(x - width/2, values1, width, label="Group A")
ax.bar(x + width/2, values2, width, label="Group B")
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()
plt.show()
```
This grouped bar pattern (shifting bars left/right by half the width) is genuinely the trickiest "basic" plot to get right the first time — took me a bit to understand why the `width/2` offset is needed (it's just centering each group's bars around the category's tick position).

### Stacked bar charts
```python
ax.bar(categories, values1, label="Group A")
ax.bar(categories, values2, bottom=values1, label="Group B")     # 'bottom' stacks the second set ON TOP of the first
ax.legend()
```

## Scatter plots
```python
x = np.random.rand(50)
y = np.random.rand(50)

fig, ax = plt.subplots()
ax.scatter(x, y)
plt.show()
```

### Customizing scatter points
```python
ax.scatter(x, y, color="red", s=100, alpha=0.6)     # s = marker size, alpha = transparency
```

### Color-coding points by a third variable (very useful for showing an extra dimension of data)
```python
colors = np.random.rand(50)     # a value per point
sizes = np.random.rand(50) * 200

scatter = ax.scatter(x, y, c=colors, s=sizes, alpha=0.6, cmap="viridis")
plt.colorbar(scatter)     # adds a color scale legend showing what each color value means
```
This is genuinely one of my favorite matplotlib tricks once I saw it — encoding a THIRD variable as color (and even a FOURTH as size) means a simple 2D scatter plot can show 4 dimensions of data at once.

## Histograms — showing the distribution of a single numeric variable
```python
data = np.random.normal(0, 1, 1000)     # 1000 samples from a normal distribution

fig, ax = plt.subplots()
ax.hist(data, bins=30)
plt.show()
```

### Customizing histograms
```python
ax.hist(data, bins=30, color="teal", edgecolor="black", alpha=0.7)
ax.hist(data, bins=30, density=True)     # normalize so the area sums to 1 (shows probability density instead of raw counts)
```

### Overlapping multiple histograms (comparing two distributions)
```python
ax.hist(data1, bins=30, alpha=0.5, label="Group A")
ax.hist(data2, bins=30, alpha=0.5, label="Group B")
ax.legend()
```
The `alpha=0.5` transparency is essential here — without it, the second histogram just completely covers the first one and you can't see the overlap at all.

## Choosing bin count for histograms (a genuine judgment call)
Too few bins hide the actual shape of the distribution, too many bins make it look noisy/spiky even for smooth underlying data. I generally start with `bins=30` as a reasonable default and adjust based on how the plot looks — there's no single "correct" number, it depends on the dataset size and what you're trying to show.

## Pie charts (brief mention — genuinely rare in serious data analysis, but comes up occasionally)
```python
sizes = [30, 25, 25, 20]
labels = ["A", "B", "C", "D"]
ax.pie(sizes, labels=labels, autopct="%1.1f%%")
```
Worth noting: pie charts are widely considered a weaker visualization choice compared to bar charts for most real comparisons (harder for the human eye to compare angles/areas than bar heights), so I don't reach for these much beyond quick, simple proportion displays.
