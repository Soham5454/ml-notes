# Matplotlib — A to Z

Notes on matplotlib, the foundational plotting library everything else (pandas' `.plot()`, Seaborn) is built on top of. Learned this right after pandas, before moving into Seaborn for the more statistical/EDA-focused plotting.

## Index

| # | Topic | File |
|---|-------|------|
| 01 | Introduction & Basics | [01-introduction-basics.md](01-introduction-basics.md) |
| 02 | Figure & Axes Anatomy | [02-figure-axes-anatomy.md](02-figure-axes-anatomy.md) |
| 03 | Line Plots | [03-line-plots.md](03-line-plots.md) |
| 04 | Bar, Scatter & Histogram | [04-bar-scatter-histogram.md](04-bar-scatter-histogram.md) |
| 05 | Subplots & Multiple Plots | [05-subplots-multiple-plots.md](05-subplots-multiple-plots.md) |
| 06 | Styling & Customization | [06-styling-customization.md](06-styling-customization.md) |
| 07 | Advanced Plot Types | [07-advanced-plot-types.md](07-advanced-plot-types.md) |
| 08 | Saving & Exporting | [08-saving-exporting.md](08-saving-exporting.md) |
| 09 | ML/DL Visualization (loss curves, confusion matrix, feature maps) | [09-ml-dl-visualization.md](09-ml-dl-visualization.md) |

## Setup
```bash
pip install matplotlib
```
```python
import matplotlib.pyplot as plt     # 'plt' is the universal convention
```

## Roadmap position
Python Basics → NumPy → pandas → **matplotlib (you are here)** → Seaborn → scikit-learn → XGBoost → PyTorch → Hugging Face
