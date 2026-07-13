# 08 — Styling & Themes

Seaborn genuinely improves on matplotlib's default look right out of the box, but there's still real control available for further customization.

## Themes — sns.set_theme() / sns.set_style()
```python
sns.set_theme()                       # applies Seaborn's default improved styling globally -- I run this pretty much at the top of every notebook now
sns.set_style("whitegrid")               # white background with grid lines -- my personal most-used style
sns.set_style("darkgrid")                   # gray background with grid lines (Seaborn's actual default)
sns.set_style("white")                         # plain white background, no grid
sns.set_style("dark")                             # plain gray background, no grid
sns.set_style("ticks")                               # white background, small tick marks instead of a grid
```
Once `sns.set_theme()` or `sns.set_style()` is called, EVERY subsequent plot (even plain matplotlib ones, since Seaborn's styling affects the shared matplotlib rcParams) picks up the new look — genuinely convenient for keeping a whole notebook visually consistent without repeating style code per plot.

## Color palettes
```python
sns.color_palette()                        # shows the current default palette
sns.color_palette("pastel")                    # softer, muted colors
sns.color_palette("bright")                       # more vivid colors
sns.color_palette("Set2")                            # a specific qualitative palette, good for categorical data
sns.set_palette("pastel")                               # sets a palette GLOBALLY for all subsequent plots
```

### Types of palettes and when to use each
- **Qualitative** (`Set1`, `Set2`, `pastel`, `bright`) — for CATEGORICAL data with no inherent order (like day names, departments)
- **Sequential** (`Blues`, `viridis`, `rocket`) — for numeric data with a clear low-to-high order (like age, price)
- **Diverging** (`coolwarm`, `RdBu`, `vlag`) — for numeric data with a meaningful CENTER point (like correlation values from -1 to +1, covered in file 06)

Getting this palette TYPE right genuinely matters for how easily a viewer can correctly interpret the plot — using a sequential palette on unordered categories (or vice versa) can visually imply a false ordering/relationship that isn't actually there in the data.

## Custom color palettes
```python
custom_palette = ["#FF6B6B", "#4ECDC4", "#45B7D1"]
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="day", palette=custom_palette)
```

## despine — removing the top/right border lines (recap of the concept from matplotlib file 02, Seaborn has a dedicated shortcut)
```python
sns.scatterplot(data=tips, x="total_bill", y="tip")
sns.despine()     # removes top and right spines automatically -- one line instead of manually doing it per spine like in raw matplotlib
```
Seaborn already removes these by default in most of its styling themes, but `sns.despine()` is there for when I've combined Seaborn with raw matplotlib and want that same clean look applied.

## Scaling plot elements (font sizes, line widths) all at once
```python
sns.set_context("notebook")     # default -- balanced for viewing in a notebook
sns.set_context("paper")           # smaller elements, good for academic paper figures
sns.set_context("talk")               # larger elements, good for presentation slides
sns.set_context("poster")                # largest elements, good for poster printouts
```
This `set_context` is genuinely useful and something I didn't expect — instead of manually scaling font sizes and line widths for different output purposes (a notebook vs a presentation slide vs a printed poster), one line adjusts everything proportionally.

## Figure size for Seaborn plots (since axes-level functions still respect matplotlib's figure setup)
```python
plt.figure(figsize=(10, 6))
sns.scatterplot(data=tips, x="total_bill", y="tip")
```
For figure-level functions (relplot, catplot, etc.), sizing works slightly differently since they manage their own figure:
```python
sns.relplot(data=tips, x="total_bill", y="tip", height=6, aspect=1.5)     # height in inches, aspect = width/height ratio
```

## Setting everything at once with sns.set_theme() parameters
```python
sns.set_theme(style="whitegrid", palette="pastel", font_scale=1.2)
```
This single call combines style, palette, and font scaling all together — genuinely my actual go-to first line now at the start of any new analysis notebook, before any actual plotting begins.
