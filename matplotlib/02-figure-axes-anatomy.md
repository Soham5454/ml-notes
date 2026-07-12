# 02 — Figure & Axes Anatomy

Understanding this properly is what made the rest of matplotlib actually make sense, instead of copy-pasting code and hoping it works.

## The hierarchy
```
Figure (the whole window/canvas)
  └── Axes (one or more plot areas within the figure)
        ├── x-axis (with ticks, tick labels, axis label)
        ├── y-axis (with ticks, tick labels, axis label)
        ├── title
        ├── legend
        └── the actual plotted data (lines, bars, points, etc.)
```

## Creating and inspecting the figure/axes objects
```python
fig, ax = plt.subplots()

print(type(fig))     # <class 'matplotlib.figure.Figure'>
print(type(ax))         # <class 'matplotlib.axes._axes.Axes'>
```

## Setting titles and labels — object-oriented way (the "set_" prefix pattern)
```python
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])

ax.set_title("Sample Plot")
ax.set_xlabel("X Axis Label")
ax.set_ylabel("Y Axis Label")
plt.show()
```
Note to self: on the Axes object, it's always `.set_something()` — `set_title()`, `set_xlabel()`, `set_ylabel()`, `set_xlim()`, etc. This consistent naming pattern made a lot of matplotlib's API predictable once I noticed it, instead of needing to memorize each method separately.

## The pyplot equivalents (no "set_" prefix, since it's operating on the "current" axes implicitly)
```python
plt.plot([1, 2, 3], [4, 5, 6])
plt.title("Sample Plot")     # no 'set_' here -- this is the pyplot shortcut style
plt.xlabel("X Axis Label")
plt.ylabel("Y Axis Label")
plt.show()
```

## Axis limits — controlling what range is shown
```python
ax.set_xlim(0, 10)         # x-axis shows only 0 to 10
ax.set_ylim(-1, 1)            # y-axis shows only -1 to 1
```

## Ticks — controlling exactly where the marks appear
```python
ax.set_xticks([0, 2, 4, 6, 8, 10])                    # specific tick positions
ax.set_xticklabels(["a", "b", "c", "d", "e", "f"])       # custom labels at those positions (must match length)
ax.tick_params(axis="x", rotation=45)                       # rotate tick labels -- genuinely useful for long category names that overlap otherwise
```

## Grid lines
```python
ax.grid(True)                       # turn on grid lines
ax.grid(True, linestyle="--", alpha=0.5)     # dashed, semi-transparent grid -- looks cleaner than solid default
```

## Spines — the border lines around the plot area
```python
ax.spines["top"].set_visible(False)         # remove the top border line
ax.spines["right"].set_visible(False)          # remove the right border line
```
This top/right spine removal is a small but genuinely common styling touch — makes plots look less "boxed in" and closer to how Seaborn plots look by default (Seaborn actually does this automatically, which is one of the small things I appreciated once I saw the matplotlib version manually).

## Multiple axes on one figure (brief preview — full detail in file 05)
```python
fig, (ax1, ax2) = plt.subplots(1, 2)     # 1 row, 2 columns -- creates TWO axes objects at once, side by side
ax1.plot([1, 2, 3])
ax2.plot([3, 2, 1])
plt.show()
```

## Getting the current figure/axes (pyplot style, useful when mixing styles)
```python
fig = plt.gcf()     # "get current figure"
ax = plt.gca()        # "get current axes"
```
Useful when you're mostly using pyplot-style shortcuts but need to grab the actual Axes object briefly for something specific — a bridge between the two interfaces from file 01.

## Clearing and closing figures (important for scripts/loops generating many plots)
```python
plt.clf()      # clear the current FIGURE
plt.cla()        # clear the current AXES
plt.close()        # close the current figure window entirely, frees memory
plt.close("all")     # close ALL open figures
```
This matters more than it seems — if you're generating plots in a loop without closing them, matplotlib keeps every figure in memory, which can genuinely eat up RAM on a long-running script (like something processing many images/batches during training, which connects to file 09's DL context).
