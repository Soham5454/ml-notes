# Pandas — A to Z

Notes on pandas, written after NumPy and before matplotlib/Seaborn in the roadmap. Pandas is basically NumPy with labels and a much friendlier interface for real-world messy data — most of what I already know about NumPy (vectorization, broadcasting, boolean masking) carries over directly since DataFrames are built on top of NumPy arrays internally.

## Index

| # | Topic | File |
|---|-------|------|
| 01 | Introduction — Series & DataFrame | [01-introduction-series-dataframe.md](01-introduction-series-dataframe.md) |
| 02 | Data Creation & I/O (CSV, Excel, JSON, SQL) | [02-data-creation-io.md](02-data-creation-io.md) |
| 03 | Indexing & Selection (loc, iloc, boolean) | [03-indexing-selection.md](03-indexing-selection.md) |
| 04 | Data Cleaning (missing data, duplicates, dtypes) | [04-data-cleaning.md](04-data-cleaning.md) |
| 05 | Data Manipulation (apply, map, sort, rename) | [05-data-manipulation.md](05-data-manipulation.md) |
| 06 | GroupBy & Aggregation | [06-groupby-aggregation.md](06-groupby-aggregation.md) |
| 07 | Merging, Joining & Concatenating | [07-merging-joining-concatenating.md](07-merging-joining-concatenating.md) |
| 08 | Pivot Tables & Reshaping | [08-pivot-tables-reshaping.md](08-pivot-tables-reshaping.md) |
| 09 | Time Series | [09-time-series.md](09-time-series.md) |
| 10 | String Operations | [10-string-operations.md](10-string-operations.md) |
| 11 | Performance & Vectorization | [11-performance-vectorization.md](11-performance-vectorization.md) |
| 12 | Quick Visualization with Pandas | [12-visualization-with-pandas.md](12-visualization-with-pandas.md) |
| 13 | ML/DL Use Cases (feature engineering, data pipelines) | [13-ml-dl-use-cases.md](13-ml-dl-use-cases.md) |

## Setup
```bash
pip install pandas
```
```python
import pandas as pd     # 'pd' is the universal convention
```

## Roadmap position
Python Basics → NumPy → **pandas (you are here)** → matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Hugging Face
