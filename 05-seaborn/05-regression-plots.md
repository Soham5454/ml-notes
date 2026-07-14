# 05 — Regression Plots

Seaborn can fit and display a simple regression line directly as part of the plotting call — genuinely useful for a fast "is there a linear relationship here" check during EDA, before reaching for scikit-learn for anything more serious.

## regplot — scatter plot + fitted regression line in one call
```python
sns.regplot(data=tips, x="total_bill", y="tip")
plt.show()
```
This single line does THREE things at once: plots the scatter points, fits a linear regression, and shows the confidence interval band around the fitted line — genuinely a lot of statistical work condensed into one function call.

## Understanding what's being shown
- The scattered points = raw data
- The solid line = the best-fit linear regression line
- The shaded band around the line = the confidence interval for that fit (wider band = less certainty, often near the edges where there's less data)

## Turning off the confidence interval band
```python
sns.regplot(data=tips, x="total_bill", y="tip", ci=None)
```

## Fitting higher-order (polynomial) relationships
```python
sns.regplot(data=tips, x="total_bill", y="tip", order=2)     # fits a QUADRATIC curve instead of a straight line
```
Useful when the relationship clearly isn't linear — bumping the order up lets the fitted line curve to match, though going too high risks overfitting the visual fit to noise (same overfitting concept from the matplotlib DL notes, just visible here in a simple 2D fit instead of a full model).

## Logistic regression fit (for binary outcome variables)
```python
titanic = sns.load_dataset("titanic")
sns.regplot(data=titanic, x="age", y="survived", logistic=True)
```
Fits an S-shaped logistic curve instead of a straight line — appropriate when the y-variable is binary (0/1, survived/didn't) rather than continuous. This is a genuinely nice preview of what logistic regression actually LOOKS like visually, before I've formally learned it as an algorithm in scikit-learn.

## lmplot — figure-level version, enables faceting (same pattern as relplot/displot/catplot)
```python
sns.lmplot(data=tips, x="total_bill", y="tip", col="time")     # separate regression plot per time value
sns.lmplot(data=tips, x="total_bill", y="tip", hue="smoker")      # separate regression LINES per smoker/non-smoker, overlaid on the same axes
```
The `hue=` version here is genuinely powerful for a quick "does this relationship differ between groups" check — two separate fitted lines on one plot, instantly comparable.

## residplot — checking how well the linear fit actually worked
```python
sns.residplot(data=tips, x="total_bill", y="tip")
```
Plots the RESIDUALS (actual value minus predicted value) instead of the raw data — if the linear fit was appropriate, residuals should look like a random scattered cloud around zero with no obvious pattern. If there's a visible curve or pattern in the residuals, that's a sign the relationship isn't actually linear and a straight-line fit is a poor choice.

## Honest note on where this fits in the roadmap
`regplot`/`lmplot` are genuinely useful for FAST visual exploration ("does this look linear, roughly") but I know actual regression modeling (fitting on train data, evaluating on test data, getting coefficients, proper metrics) is scikit-learn's job, further down the roadmap. This file is really about pattern-recognition during EDA, not real model building.
