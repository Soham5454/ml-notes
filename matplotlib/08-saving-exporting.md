# 08 — Saving & Exporting

## Basic saving
```python
fig.savefig("my_plot.png")
```
Note: `savefig()` should be called BEFORE `plt.show()` in some environments, since `plt.show()` can clear the figure from memory afterward depending on the backend. Safest habit: always save first, then show.

## Common formats
```python
fig.savefig("plot.png")       # raster image, most common, good for general use
fig.savefig("plot.jpg")          # also raster, generally used for photos, slightly worse for line art/text due to compression artifacts
fig.savefig("plot.pdf")             # vector format -- scales to ANY size with no quality loss, great for reports/papers
fig.savefig("plot.svg")                # vector format, good for web use / further editing in tools like Illustrator
```
For anything going into an actual report or academic-style document, I'd reach for PDF or SVG since they stay crisp at any zoom level — PNG/JPG are fine for quick sharing (like dropping into a Slack message or a GitHub README) but will look blurry if someone zooms in a lot.

## Controlling resolution (DPI — dots per inch)
```python
fig.savefig("plot.png", dpi=300)     # 300 DPI is a common "print quality" standard
fig.savefig("plot.png", dpi=72)         # lower DPI, smaller file size, fine for quick web use
```
Higher DPI = sharper image but bigger file size. For anything meant to be zoomed into or printed, 300 DPI is the standard target; for quick informal sharing, the default (usually 100) is fine.

## Removing extra whitespace around the plot
```python
fig.savefig("plot.png", bbox_inches="tight")     # trims excess white border around the actual plot content
```
This is one I now add to almost every `savefig()` call — without it, there's often a noticeable amount of wasted white space around the actual chart.

## Transparent background (useful for overlaying onto slides/websites with their own background color)
```python
fig.savefig("plot.png", transparent=True)
```

## Saving with a specific figure size
```python
fig, ax = plt.subplots(figsize=(10, 6))
# ... plotting code ...
fig.savefig("plot.png", dpi=300, bbox_inches="tight")
```

## Saving multiple plots to a single PDF (multi-page)
```python
from matplotlib.backends.backend_pdf import PdfPages

with PdfPages("report.pdf") as pdf:
    fig1, ax1 = plt.subplots()
    ax1.plot([1, 2, 3])
    pdf.savefig(fig1)

    fig2, ax2 = plt.subplots()
    ax2.bar(["a", "b"], [3, 5])
    pdf.savefig(fig2)
```
Genuinely useful for generating a full multi-chart report as a single PDF file instead of exporting each chart separately — noting this since it seems relevant to a "reports/documentation" kind of workflow.

## Saving directly to a buffer (in-memory, without writing to disk) — useful for web apps
```python
import io
buf = io.BytesIO()
fig.savefig(buf, format="png")
buf.seek(0)
# buf now holds the PNG image data in memory, ready to be served/displayed without a temp file
```
Haven't needed this yet myself, but it's the standard way plots get served dynamically in web applications (like a Flask/Django app generating charts on the fly) instead of saving temporary files to disk.

## Closing the figure after saving (memory management, recap from file 02)
```python
fig.savefig("plot.png")
plt.close(fig)     # frees the memory -- important if generating MANY plots in a loop (e.g. one plot per epoch during training, or one per image in a dataset)
```
This connects directly to something relevant for my actual future workflow — if I'm generating a plot per training epoch or per batch during model development, forgetting to close figures in the loop would genuinely accumulate and slow things down or eat RAM over a long training run.
