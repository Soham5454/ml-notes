# 01 — Introduction & Basics

## What is Seaborn and why it exists
Seaborn sits on top of matplotlib (everything from the matplotlib notes still applies underneath) but is built specifically for statistical visualization and works directly with pandas DataFrames — you pass column NAMES instead of manually extracting arrays, which removes a lot of the boilerplate matplotlib requires.

## The core philosophy — "long-form" data
Seaborn works best with data in "long/tidy" format (recap from the pandas notes, file 08 — melt) — one row per observation, columns represent variables. This is DIFFERENT from how you might naturally lay out spreadsheet-style "wide" data, and getting this right early saves a lot of confusion later.
```python
import seaborn as sns
import pandas as pd

# Seaborn ships with built-in example datasets, genuinely useful for practicing
tips = sns.load_dataset("tips")
print(tips.head())
#    total_bill   tip     sex smoker  day    time  size
# 0       16.99  1.01  Female     No  Sun  Dinner     2
# 1       10.34  1.66    Male     No  Sun  Dinner     3
```
This `tips` dataset is long-form: each row is ONE observation (one meal), each column is ONE variable (bill amount, tip, day, etc.) — this shape is what basically every Seaborn function expects.

## The simplest possible plot
```python
sns.scatterplot(data=tips, x="total_bill", y="tip")
```
Notice the pattern: `data=` (the DataFrame), then `x=` and `y=` as COLUMN NAMES (strings), not raw arrays. This `data=`, `x=`, `y=` pattern repeats across almost every single Seaborn function — once it clicks, most Seaborn plots become predictable to write even for plot types I haven't used before.

## Seaborn's built-in datasets (great for practicing before using real data)
```python
sns.load_dataset("tips")            # restaurant tips data
sns.load_dataset("iris")               # classic flower measurements dataset
sns.load_dataset("titanic")               # Titanic passenger survival data
sns.load_dataset("penguins")                 # penguin species measurements
sns.get_dataset_names()                         # lists ALL available built-in datasets
```
I've been using `tips` and `iris` mainly while practicing since they're clean and genuinely well-known reference datasets — good for testing a new plot type without worrying about data cleaning first.

## The "hue" parameter — the single most powerful Seaborn concept
Adding a THIRD (categorical) variable to almost any plot by color-coding it, with zero extra plotting code needed.
```python
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="sex")     # color-codes points by 'sex' automatically, adds a legend automatically too
```
This `hue=` parameter is genuinely the thing that sold me on Seaborn over plain matplotlib — the equivalent in matplotlib would need manually looping through groups and plotting each with a different color. Here it's one extra argument.

### hue works across almost EVERY seaborn plot type
```python
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="day")
sns.boxplot(data=tips, x="day", y="total_bill", hue="smoker")
sns.histplot(data=tips, x="total_bill", hue="time")
```
This consistency across plot types is a big part of why Seaborn feels so much faster to work with once the core pattern clicks.

## Figure-level vs Axes-level functions (an important distinction, confused me at first)
Seaborn functions come in two families:
- **Axes-level functions** (`sns.scatterplot`, `sns.boxplot`, `sns.histplot`) — plot onto a single matplotlib Axes, can be combined with `plt.subplots()` like any matplotlib plot
- **Figure-level functions** (`sns.relplot`, `sns.catplot`, `sns.displot`) — manage their OWN figure, and can automatically create multiple subplots via `col=`/`row=` parameters (this connects to file 07)

```python
# Axes-level -- works with matplotlib subplots directly
fig, ax = plt.subplots()
sns.scatterplot(data=tips, x="total_bill", y="tip", ax=ax)     # note the ax= parameter

# Figure-level -- creates its own figure, can facet automatically
sns.relplot(data=tips, x="total_bill", y="tip", col="day")     # automatically creates ONE subplot PER day value
```
The `col="day"` bit on a figure-level function is genuinely powerful — it automatically creates a grid of subplots, one per category, without any manual subplot loop. Full depth on this in file 07.

## Combining Seaborn with matplotlib customization (since it IS matplotlib underneath)
```python
sns.scatterplot(data=tips, x="total_bill", y="tip")
plt.title("Tip Amount vs Bill")     # regular matplotlib functions still work directly on top
plt.xlabel("Total Bill ($)")
plt.show()
```
Since axes-level Seaborn functions return the underlying matplotlib Axes object, everything from the matplotlib notes (titles, labels, saving, styling) applies directly without needing to learn anything new.
