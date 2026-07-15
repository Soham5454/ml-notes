# 07 — Text Classification with Classical ML

Recap file 06's closing note — this file applies everything from files 01-06 to an actual TASK, directly connecting this whole NLP folder back to the scikit-learn/XGBoost notes built earlier in the roadmap.

## The full classical text classification pipeline

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix

texts = [
    "I loved this movie, it was fantastic",
    "Terrible film, waste of time",
    "Amazing acting and great story",
    "Boring and way too long",
    "Best movie I've seen this year",
    "Awful, would not recommend"
]
labels = [1, 0, 1, 0, 1, 0]      # 1 = positive, 0 = negative

# Step 1: preprocessing (recap files 01-02)
# Step 2: vectorization (recap file 04's TF-IDF)
vectorizer = TfidfVectorizer(stop_words='english')
X = vectorizer.fit_transform(texts)

# Step 3: standard scikit-learn train/test split (recap that folder's discipline)
X_train, X_test, y_train, y_test = train_test_split(X, labels, test_size=0.33, random_state=42)

# Step 4: train ANY scikit-learn classifier -- text features work like any other numeric features
clf = LogisticRegression()
clf.fit(X_train, y_train)

predictions = clf.predict(X_test)
print(classification_report(y_test, predictions))
```

Genuinely the whole point of files 01-06 — once text becomes a numeric feature matrix (TF-IDF, BoW, or averaged embeddings), it plugs directly into the EXACT SAME scikit-learn workflow as any tabular dataset from that folder. Nothing about `LogisticRegression`, `train_test_split`, or `classification_report` changes — only the FEATURE EXTRACTION step (files 03/04/06) is NLP-specific.

## Which classifier to use — recap scikit-learn's decision framework, now for text

```python
from sklearn.naive_bayes import MultinomialNB
from sklearn.svm import LinearSVC
from sklearn.ensemble import RandomForestClassifier

# Naive Bayes -- genuinely the classic, fast baseline for text (recap file 03/Math Foundations file 11)
nb_clf = MultinomialNB()

# Linear SVM -- often performs GENUINELY well on high-dimensional sparse text features
svm_clf = LinearSVC()

# Logistic Regression -- interpretable, gives probability estimates, a strong general default
lr_clf = LogisticRegression()

# Random Forest / XGBoost -- CAN work, but recap the XGBoost bridge file's honest framework:
# text features are HIGH-DIMENSIONAL and SPARSE, which tree-based models handle less
# naturally than linear models -- genuinely worth trying, but not always the first choice for text
```

Genuinely important, practical honest note connecting back to the XGBoost bridge file's decision framework: high-dimensional SPARSE data (thousands of TF-IDF features, mostly zero per document) tends to favor LINEAR models (Logistic Regression, Linear SVM) over tree-based ones — trees split on individual features one at a time, which is less natural for sparse, high-dimensional spaces than for the dense tabular data XGBoost was originally motivated by.

## Evaluating text classifiers — recap scikit-learn's metrics folder

```python
print(confusion_matrix(y_test, predictions))
# Recap scikit-learn's confusion matrix notes -- same exact interpretation:
# rows = actual class, columns = predicted class

from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
print(f"Accuracy: {accuracy_score(y_test, predictions):.3f}")
print(f"Precision: {precision_score(y_test, predictions):.3f}")
print(f"Recall: {recall_score(y_test, predictions):.3f}")
print(f"F1: {f1_score(y_test, predictions):.3f}")
```

Genuinely nothing new conceptually here — every metric definition, every precision/recall tradeoff discussion from the scikit-learn folder applies IDENTICALLY to text classification. The only NLP-specific consideration: class imbalance (recap that scikit-learn topic) is genuinely COMMON in text data — e.g. spam detection typically has far more "not spam" than "spam" examples, making accuracy alone a misleading metric, same caution as always.

## Multi-class text classification

```python
# Genuinely the same scikit-learn pattern, just with more than 2 label classes
categories = ["sports", "politics", "technology", "sports", "technology"]

vectorizer = TfidfVectorizer()
X = vectorizer.fit_transform(texts_multiclass)      # assume texts_multiclass exists

clf = LogisticRegression(multi_class='ovr')      # one-vs-rest, recap scikit-learn's multi-class notes
clf.fit(X, categories)
```

## Handling class imbalance in text — recap scikit-learn's techniques, applied here

```python
clf = LogisticRegression(class_weight='balanced')      # recap scikit-learn's imbalance handling

# Or explicit oversampling/undersampling techniques (recap that scikit-learn topic)
from imblearn.over_sampling import SMOTE
# Note: SMOTE genuinely works less naturally on sparse TF-IDF features than dense
# numeric ones -- worth being aware this is a real, honest limitation rather than
# assuming every classical ML technique transfers perfectly to text data
```

## Building a complete Pipeline — recap scikit-learn's Pipeline object

```python
from sklearn.pipeline import Pipeline

text_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(stop_words='english', max_features=5000)),
    ('classifier', LogisticRegression())
])

text_pipeline.fit(texts, labels)
new_predictions = text_pipeline.predict(["This was an incredible experience"])
print(new_predictions)
```

Recap scikit-learn's Pipeline notes directly — genuinely the exact same tool, and genuinely ESSENTIAL for text specifically, since it bundles the vectorizer's LEARNED vocabulary together with the trained classifier, preventing the exact "forgot to use `.transform()` instead of `.fit_transform()` on new data" mistake flagged back in file 04.

## Cross-validation for text — recap scikit-learn's cross-validation notes

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(text_pipeline, texts, labels, cv=5)
print(f"Mean accuracy: {scores.mean():.3f}, Std: {scores.std():.3f}")
```

Genuinely identical to cross-validation for any other scikit-learn task — the Pipeline handles re-fitting the vectorizer correctly on each fold automatically, which is precisely why wrapping the vectorizer INSIDE the Pipeline (rather than fitting it once outside and reusing it) matters for getting an honest cross-validation estimate, avoiding data leakage across folds.

## Try this

```python
# Build a genuinely complete spam/not-spam classifier:
# 1. Create ~20 short example texts (10 spam-like: "WIN FREE MONEY NOW", 10 normal)
# 2. Build a full scikit-learn Pipeline with TfidfVectorizer + LogisticRegression
# 3. Use cross_val_score to get an honest performance estimate
# 4. Test the trained pipeline on 3 NEW sentences it hasn't seen, and inspect
#    predict_proba() (recap scikit-learn's probability output notes) to see the
#    model's actual CONFIDENCE, not just its final predicted label
```

Building this small but complete classification pipeline end-to-end is genuinely the clearest demonstration that everything from the scikit-learn folder transfers directly to text — the only genuinely new piece was the vectorization step from files 03/04.

## What's next
File 08 — Sentiment Analysis, a focused, practical deep-dive into the specific classification task used as the running example in this file, including techniques and pitfalls unique to sentiment specifically.
