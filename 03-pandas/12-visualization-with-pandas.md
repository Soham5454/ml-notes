# 12 — Quick Visualization with Pandas

Pandas has basic plotting built in (via matplotlib under the hood) — genuinely useful for quick exploratory checks, though I'll use matplotlib/Seaborn directly (already covered those separately) for anything that needs to look properly polished.

## Setup
```python
import matplotlib.pyplot as plt     # pandas plotting uses matplotlib as the backend
```

## Line plot (default plot type — good for time series)
```python
df["sales"].plot()                 # quick line plot of a single column
df.plot(x="date", y="sales")          # explicit x and y columns
plt.show()
```

## Bar plot
```python
df["department"].value_counts().plot(kind="bar")     # frequency bar chart -- extremely common quick check
```

## Histogram — distribution of a numeric column
```python
df["age"].plot(kind="hist", bins=20)     # quick distribution check
```

## Box plot — spotting outliers quickly (connects to the IQR outlier detection from file 04)
```python
df["salary"].plot(kind="box")     # visual version of the same IQR-based outlier logic from data cleaning
```

## Scatter plot — relationship between two numeric columns
```python
df.plot(kind="scatter", x="age", y="salary")
```

## Plotting multiple columns at once
```python
df[["math", "science"]].plot()     # plots both columns on the same chart automatically
```

## Grouped bar plots (combines groupby from file 06 with plotting)
```python
df.groupby("department")["salary"].mean().plot(kind="bar")     # average salary per department, as a bar chart
```

## Pie chart
```python
df["department"].value_counts().plot(kind="pie", autopct="%1.1f%%")     # autopct shows percentage labels on each slice
```

## When to reach for pandas plotting vs matplotlib/Seaborn directly
Honest take based on what I've already learned in the matplotlib/Seaborn notes: pandas' built-in `.plot()` is genuinely great for a 2-second "let me just LOOK at this column real quick" check while exploring data. The moment I need proper styling, multiple subplots, or anything presentation-worthy, I switch to matplotlib/Seaborn directly since I already have those skills — pandas plotting is really just a fast shortcut layered on top of matplotlib, not a separate thing to master deeply.

## Customizing pandas plots (since it's just matplotlib underneath)
```python
df["sales"].plot(title="Monthly Sales", figsize=(10, 5), color="green")
plt.xlabel("Month")
plt.ylabel("Sales")
plt.show()
```
Since pandas plots ARE matplotlib figures underneath, all the regular matplotlib customization (titles, labels, figure size) still works directly on top — nothing new to learn here beyond what's already in the matplotlib notes.
