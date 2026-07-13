# 09 — EDA Workflow (Putting It All Together)

This file is less about NEW Seaborn functions and more about how everything from files 01-08 actually gets used together, in order, on a real dataset. Writing this as the "here's my actual process" reference for future me.

## My general EDA sequence on a brand new dataset

### Step 1 — basic pandas checks first (recap from pandas notes)
```python
df.head()
df.info()
df.describe()
df.isnull().sum()
```
Before ANY plotting, I want to know the shape, types, and missing-data situation.

### Step 2 — set the Seaborn theme once
```python
sns.set_theme(style="whitegrid")
```

### Step 3 — distribution of each numeric column (file 03)
```python
numeric_cols = df.select_dtypes(include="number").columns
for col in numeric_cols:
    sns.histplot(data=df, x=col, kde=True)
    plt.title(f"Distribution of {col}")
    plt.show()
```
Looking for: skewness, outliers, unexpected multiple peaks (bimodal distributions), or anything that looks wrong (like a column that should be continuous but shows weirdly discrete clusters).

### Step 4 — correlation heatmap (file 06)
```python
corr = df[numeric_cols].corr()
plt.figure(figsize=(10, 8))
sns.heatmap(corr, annot=True, cmap="coolwarm", center=0)
plt.show()
```
Looking for: strongly correlated feature pairs (possible redundancy), and which features correlate most with the target variable if I already know what I'm predicting.

### Step 5 — categorical variable counts (file 04)
```python
categorical_cols = df.select_dtypes(include="object").columns
for col in categorical_cols:
    sns.countplot(data=df, x=col)
    plt.title(f"Count of {col}")
    plt.xticks(rotation=45)
    plt.show()
```
Looking for: class imbalance (one category vastly outnumbering others), unexpected categories (typos, inconsistent capitalization — recap from pandas cleaning notes), rare categories that might need grouping.

### Step 6 — relationship between features and target (files 02, 04, 05)
```python
# If target is numeric:
sns.scatterplot(data=df, x="feature", y="target")
sns.regplot(data=df, x="feature", y="target")

# If target is categorical:
sns.boxplot(data=df, x="target", y="feature")
sns.violinplot(data=df, x="target", y="feature")
```

### Step 7 — pairwise overview (file 07)
```python
sns.pairplot(df, hue="target")     # if the dataset isn't too large -- pairplot gets slow/cluttered with too many columns or rows
```

## When pairplot gets impractical (a real limitation to know about)
If a dataset has many numeric columns (say, more than 8-10), `pairplot` creates a huge, cluttered grid that's genuinely hard to read and slow to render. In that case, I'd fall back to: correlation heatmap first to identify the most interesting feature PAIRS, then use targeted `jointplot`/`scatterplot` calls on just those specific pairs instead of plotting everything against everything.

## A realistic honest note on this whole process
This sequence isn't a rigid rulebook — it's a starting checklist. Real EDA is iterative: something in step 3 (like an odd distribution) often sends me back to check something in pandas cleaning that I missed, or something in step 6 raises a new question that needs another plot type entirely. The value of having this sequence written out isn't "always do exactly these 7 steps," it's "here's a reasonable default starting point so I'm not staring at a new dataset wondering what to even look at first."

## Connecting EDA back to feature engineering (bridges to pandas file 13)
Everything found during this EDA process directly feeds into the feature engineering decisions from the pandas ML/DL notes — a skewed distribution found in step 3 might need a log transform, a highly correlated pair found in step 4 might mean dropping one feature, a class imbalance found in step 5 might need special handling during model training. EDA isn't just "looking at pretty charts," it's the actual evidence base for every data preparation decision that follows.
