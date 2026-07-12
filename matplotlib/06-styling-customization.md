# 06 — Styling & Customization

## Titles and labels — full options
```python
ax.set_title("My Plot", fontsize=16, fontweight="bold", color="navy")
ax.set_xlabel("X Axis", fontsize=12)
ax.set_ylabel("Y Axis", fontsize=12)
```

## Text annotations — labeling specific points on a plot
```python
ax.annotate("Peak value", xy=(5, 10), xytext=(6, 12),
            arrowprops=dict(facecolor="black", shrink=0.05))
```
`xy` is the point being pointed AT, `xytext` is where the label TEXT actually sits, and `arrowprops` draws an arrow connecting them. This is genuinely useful for highlighting a specific interesting point in a plot (like a peak, an outlier, or a specific epoch where something notable happened during training).

## Plain text at a specific location (no arrow)
```python
ax.text(5, 10, "Some note here", fontsize=10, color="red")
```

## Colormaps — for anything that maps a value to a color (scatter, heatmaps, images)
```python
ax.scatter(x, y, c=values, cmap="viridis")     # 'viridis' is matplotlib's modern default, perceptually uniform
ax.scatter(x, y, c=values, cmap="coolwarm")       # good for diverging data (negative to positive)
ax.scatter(x, y, c=values, cmap="plasma")
```
Note to self: 'viridis' and similar "perceptually uniform" colormaps are generally the better choice over the old rainbow-style 'jet' colormap, since jet can visually mislead viewers about which values are actually close together (it doesn't map linearly to perceived brightness).

## Style sheets — quick, complete visual theme changes
```python
plt.style.use("ggplot")           # mimics R's ggplot2 look
plt.style.use("seaborn-v0_8")        # Seaborn-like styling, without needing Seaborn itself
plt.style.use("dark_background")        # dark theme, good for presentations
plt.style.available                        # lists ALL available built-in styles
```
Using `plt.style.use()` once at the top of a script changes the look of EVERY plot that follows — a genuinely fast way to make plots look more polished without manually tweaking colors/grids/fonts every time.

## Custom color palettes (defining your own consistent set of colors)
```python
colors = ["#FF6B6B", "#4ECDC4", "#45B7D1", "#FFA07A"]
for i, category in enumerate(categories):
    ax.bar(category, values[i], color=colors[i])
```

## Font sizes globally (instead of setting them individually every time)
```python
plt.rcParams["font.size"] = 12
plt.rcParams["axes.titlesize"] = 16
plt.rcParams["axes.labelsize"] = 12
```
`rcParams` sets DEFAULT values that apply to every subsequent plot in the session — useful for consistent styling across an entire notebook/script instead of repeating the same size arguments constantly.

## Legend customization
```python
ax.legend(loc="upper right", fontsize=10, frameon=True, shadow=True, title="Categories")
```

## Log scale axes (useful when data spans many orders of magnitude)
```python
ax.set_yscale("log")     # y-axis becomes logarithmic instead of linear
ax.set_xscale("log")
```
Genuinely important for things like loss curves where the loss might start very high and drop rapidly — a log scale makes the LATER, smaller changes actually visible instead of looking like a flat line near zero.

## Twin axes — two different y-scales on the same plot
```python
fig, ax1 = plt.subplots()
ax1.plot(x, y1, color="blue")
ax1.set_ylabel("Y1", color="blue")

ax2 = ax1.twinx()     # creates a second y-axis sharing the same x-axis
ax2.plot(x, y2, color="red")
ax2.set_ylabel("Y2", color="red")
```
Useful when comparing two metrics with very different scales/units on the same x-axis (like loss vs accuracy over training epochs) without one completely dwarfing the other visually.

## Removing/customizing spines (recap from file 02, more detail here)
```python
for spine in ["top", "right"]:
    ax.spines[spine].set_visible(False)

ax.spines["left"].set_color("gray")
ax.spines["bottom"].set_linewidth(2)
```

## Setting aspect ratio (useful for things like confusion matrices where square cells matter)
```python
ax.set_aspect("equal")     # forces x and y units to be visually equal -- important for things like scatter plots of geographic coordinates
```
