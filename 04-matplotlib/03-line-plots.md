# 03 — Line Plots

The most basic and most common plot type — good starting point since most of the customization options here (color, style, markers) apply to almost every other plot type too.

## Basic line plot
```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

fig, ax = plt.subplots()
ax.plot(x, y)
plt.show()
```

## Multiple lines on the same plot
```python
y1 = np.sin(x)
y2 = np.cos(x)

fig, ax = plt.subplots()
ax.plot(x, y1)
ax.plot(x, y2)     # just calling .plot() again on the SAME axes adds another line
plt.show()
```

## Line color
```python
ax.plot(x, y, color="red")
ax.plot(x, y, color="#FF5733")       # hex codes work too
ax.plot(x, y, color=(0.1, 0.5, 0.8))    # RGB tuple, values between 0 and 1
```

### Shorthand color codes (quick, common ones)
```
'b' -> blue, 'g' -> green, 'r' -> red, 'c' -> cyan,
'm' -> magenta, 'y' -> yellow, 'k' -> black, 'w' -> white
```
```python
ax.plot(x, y, "r")     # shorthand for color="red"
```

## Line style
```python
ax.plot(x, y, linestyle="-")       # solid (default)
ax.plot(x, y, linestyle="--")         # dashed
ax.plot(x, y, linestyle=":")            # dotted
ax.plot(x, y, linestyle="-.")             # dash-dot
```

## Line width
```python
ax.plot(x, y, linewidth=2)     # thicker line, default is usually 1
```

## Markers — showing individual data points on the line
```python
ax.plot(x, y, marker="o")          # circle markers at each data point
ax.plot(x, y, marker="s")             # square markers
ax.plot(x, y, marker="^")                # triangle markers
ax.plot(x, y, marker="x")                   # x markers
ax.plot(x, y, marker="o", markersize=8, markerfacecolor="red")     # customize marker appearance
```

## Combining shorthand format string (color + linestyle + marker in one string)
```python
ax.plot(x, y, "ro--")     # red, circle markers, dashed line -- all in one compact string
```
This shorthand is genuinely common in quick example code online — `'ro--'` reads as color-marker-linestyle, though I still prefer writing it out explicitly (`color=`, `marker=`, `linestyle=`) in my own code since it's clearer to read back later.

## Labels and legend (essential once you have multiple lines)
```python
ax.plot(x, y1, label="sin(x)")
ax.plot(x, y2, label="cos(x)")
ax.legend()     # shows a legend box using the label= values above
```

### Legend positioning
```python
ax.legend(loc="upper right")        # common positions: 'upper left', 'lower right', 'center', 'best'
ax.legend(loc="best")                  # matplotlib picks the position that overlaps the data least -- my usual default
```

## Alpha (transparency)
```python
ax.plot(x, y, alpha=0.5)     # 0 = fully transparent, 1 = fully opaque -- useful when lines overlap
```

## Filling area under a curve
```python
ax.fill_between(x, y, alpha=0.3)     # shades the area between the curve and 0
ax.fill_between(x, y1, y2, alpha=0.3)   # shades the area BETWEEN two curves
```

## Practical example — plotting a training loss curve (preview, real depth in file 09)
```python
epochs = np.arange(1, 21)
loss = 1 / epochs + np.random.normal(0, 0.02, 20)     # fake decreasing loss with a bit of noise

fig, ax = plt.subplots()
ax.plot(epochs, loss, marker="o", color="crimson", label="Training Loss")
ax.set_xlabel("Epoch")
ax.set_ylabel("Loss")
ax.set_title("Training Loss Over Time")
ax.legend()
plt.show()
```
Genuinely excited about this one specifically since this exact pattern is literally what I'll be looking at constantly once I'm actually training models on my RTX 3050 — watching the loss curve go down (or not) is basically the main feedback loop during training.
