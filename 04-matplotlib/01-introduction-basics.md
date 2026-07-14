# 01 — Introduction & Basics

## What is matplotlib and why it exists
Matplotlib is the original, foundational plotting library in Python — basically every other plotting library people use (pandas' `.plot()`, Seaborn) is either built directly on top of it or heavily inspired by it. It gives you low-level, full control over every visual element, which makes it a bit more verbose than Seaborn but also means you can customize literally anything if you need to.

## The two interfaces — pyplot vs object-oriented (this confused me at first)
Matplotlib actually has two different ways to write the same plot, and tutorials mix both constantly without explaining why, which is genuinely confusing when you're starting out.

### 1. pyplot interface (simpler, quick plots)
```python
import matplotlib.pyplot as plt

plt.plot([1, 2, 3], [4, 5, 6])
plt.title("My Plot")
plt.xlabel("X axis")
plt.ylabel("Y axis")
plt.show()
```
This style implicitly tracks a "current figure" and "current axes" behind the scenes — quick to write, fine for simple one-off plots.

### 2. Object-oriented interface (more control, better for anything complex)
```python
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])
ax.set_title("My Plot")
ax.set_xlabel("X axis")
ax.set_ylabel("Y axis")
plt.show()
```
This style explicitly creates `fig` (the whole figure/canvas) and `ax` (the actual plot area) as objects you control directly. This is the version that actually scales — once you need multiple subplots or fine-grained control, the pyplot shortcuts start getting confusing about which plot they're referring to.

**My takeaway after going through both**: I'm defaulting to the object-oriented style (`fig, ax = plt.subplots()`) for basically everything now, even simple plots, since it's genuinely not much more typing and it means I'm not relearning a different mental model later when things get more complex.

## The absolute simplest plot
```python
plt.plot([1, 2, 3, 4])     # if you only give ONE list, matplotlib assumes it's the y-values, and uses 0,1,2,3 as x automatically
plt.show()
```

## plt.show() — why you need it
`plt.show()` actually renders and displays the plot window. In Jupyter notebooks it's sometimes optional (plots auto-display), but in a regular `.py` script run from the terminal, forgetting this line means literally nothing shows up — the plot exists in memory but is never displayed.

## Working with NumPy arrays directly (the usual real-world case)
```python
import numpy as np

x = np.linspace(0, 10, 100)     # 100 points between 0 and 10, recap from NumPy notes
y = np.sin(x)

plt.plot(x, y)
plt.show()
```
This is genuinely the most common real pattern — generate/load your x and y data as NumPy arrays or pandas columns, then hand them straight to `plt.plot()`.

## Basic anatomy terms (quick preview, full detail in file 02)
- **Figure** — the entire window/canvas, can contain multiple plots
- **Axes** — a single plot area within the figure (confusingly NOT the same as "axis," which refers to the x/y lines themselves)
- **Axis** — the actual x-axis or y-axis line, with ticks and labels

I mixed up "Axes" (the plot) and "axis" (the line) constantly at first — worth remembering these are deliberately different things in matplotlib's vocabulary.

## Multiple ways to create a figure
```python
plt.figure()                    # creates a blank figure using pyplot style
fig = plt.figure()                 # object-oriented equivalent, gives you the actual figure object
fig, ax = plt.subplots()              # creates BOTH a figure and a single axes at once -- this is genuinely the one I use almost every time
fig.set_size_inches(10, 6)               # setting figure size after creation
```

## Setting figure size directly (very commonly needed)
```python
fig, ax = plt.subplots(figsize=(10, 6))     # width, height in inches
```
This is one of those parameters I use in almost every plot now, since the default size is often too small to actually read clearly, especially once labels/legends get added.
