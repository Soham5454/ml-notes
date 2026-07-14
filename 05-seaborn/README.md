# Seaborn — A to Z

Notes on Seaborn, learned right after matplotlib. Seaborn is built directly on top of matplotlib but gives a much higher-level, statistics-focused interface — most plots that would take 10+ lines of manual matplotlib code take 1-2 lines in Seaborn, with better defaults (styling, color palettes) out of the box.

## Index

| # | Topic | File |
|---|-------|------|
| 01 | Introduction & Basics | [01-introduction-basics.md](01-introduction-basics.md) |
| 02 | Relational Plots (scatter, line) | [02-relational-plots.md](02-relational-plots.md) |
| 03 | Distribution Plots (hist, kde, rug) | [03-distribution-plots.md](03-distribution-plots.md) |
| 04 | Categorical Plots (box, violin, bar, strip, swarm) | [04-categorical-plots.md](04-categorical-plots.md) |
| 05 | Regression Plots | [05-regression-plots.md](05-regression-plots.md) |
| 06 | Matrix Plots (heatmap, clustermap) | [06-matrix-plots.md](06-matrix-plots.md) |
| 07 | Multi-Plot Grids (FacetGrid, PairGrid, pairplot, jointplot) | [07-multiplot-grids.md](07-multiplot-grids.md) |
| 08 | Styling & Themes | [08-styling-themes.md](08-styling-themes.md) |
| 09 | EDA Workflow (putting it all together) | [09-eda-workflow.md](09-eda-workflow.md) |
| 10 | ML/DL Use Cases | [10-ml-dl-use-cases.md](10-ml-dl-use-cases.md) |

## Setup
```bash
pip install seaborn
```
```python
import seaborn as sns     # 'sns' is the universal convention (named after West Wing character Samuel Norman Seaborn, apparently)
```

## Roadmap position
Python Basics → NumPy → pandas → matplotlib → **Seaborn (you are here)** → scikit-learn → XGBoost → PyTorch → Hugging Face
