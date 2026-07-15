# 04 — The Datasets Library

Recap file 03's closing note — moving from single-example inference into properly loading and processing data AT SCALE, the necessary foundation before file 05's fine-tuning.

## Loading a dataset from the Hub

```python
from datasets import load_dataset

dataset = load_dataset("imdb")      # a classic movie review sentiment dataset
print(dataset)
# DatasetDict({
#     train: Dataset({features: ['text', 'label'], num_rows: 25000})
#     test: Dataset({features: ['text', 'label'], num_rows: 25000})
# })
```

Genuinely the direct analog of `pd.read_csv()` (recap the pandas notes) or SQL file 13's `pd.read_sql_query()` — one function call pulls a fully-prepared, ready-to-use dataset, no manual downloading/parsing needed.

## Inspecting a dataset — genuinely similar to pandas EDA habits

```python
print(dataset['train'][0])
# {'text': 'This movie was...', 'label': 0}

print(dataset['train'].features)
# {'text': Value(dtype='string'), 'label': ClassLabel(names=['neg', 'pos'])}

print(len(dataset['train']))   # 25000
```

Recap the pandas notes' `.head()`/`.info()` habits directly — genuinely the same "look at the data before doing anything else" discipline, carried over into this new library's specific API.

## Converting to/from pandas — the practical bridge

```python
df = dataset['train'].to_pandas()
print(df.head())            # now it's a familiar pandas DataFrame

from datasets import Dataset
new_dataset = Dataset.from_pandas(df)      # and back again
```

Genuinely useful for exploratory work — do quick EDA in pandas (recap that whole folder's toolkit), then convert back to a `Dataset` object for the efficient, Hugging-Face-native processing pipeline this file covers.

## The `.map()` method — genuinely the most important Datasets library function

```python
def tokenize_function(examples):
    return tokenizer(examples['text'], padding='max_length', truncation=True, max_length=256)

tokenized_dataset = dataset.map(tokenize_function, batched=True)
print(tokenized_dataset['train'][0].keys())
# dict_keys(['text', 'label', 'input_ids', 'attention_mask'])
```

Recap DSA file 01's Big O concerns and pandas' `.apply()` — `.map()` applies a function across the ENTIRE dataset, genuinely optimized for large-scale data (much of the real work happens outside Python's GIL, using Apache Arrow under the hood for efficient columnar storage). `batched=True` processes many examples at once rather than one at a time — genuinely important for speed, since tokenizing one example at a time in a plain Python loop would be dramatically slower for large datasets.

## Filtering a dataset

```python
short_reviews = dataset['train'].filter(lambda example: len(example['text']) < 500)
print(len(short_reviews))
```

Recap pandas' boolean masking (`df[df['col'] < x]`, recap SQL file 02's `WHERE` too) — genuinely the same filtering concept, applied via the Datasets library's own efficient implementation rather than pandas or SQL syntax.

## Shuffling and splitting

```python
shuffled = dataset['train'].shuffle(seed=42)      # recap the general reproducibility habit (random_state/seed) from scikit-learn

small_train = dataset['train'].shuffle(seed=42).select(range(1000))      # genuinely useful for fast prototyping
                                                                             # before committing to full-dataset fine-tuning

split_dataset = dataset['train'].train_test_split(test_size=0.2)          # recap scikit-learn's exact function name/behavior
```

Genuinely worth noting the DELIBERATE naming parallel — `train_test_split` here works almost identically in spirit to scikit-learn's function of the same name, just operating on a `Dataset` object instead of a NumPy array/pandas DataFrame.

## Loading a CUSTOM dataset — genuinely the realistic use case for personal projects

```python
from datasets import load_dataset

# From local CSV/JSON files
custom_dataset = load_dataset('csv', data_files='my_data.csv')
custom_dataset_json = load_dataset('json', data_files='my_data.json')

# From a pandas DataFrame directly (recap the to_pandas/from_pandas bridge above)
from datasets import Dataset
custom_dataset = Dataset.from_pandas(my_dataframe)
```

Genuinely the most practically relevant pattern for real projects — most personal/work datasets won't already exist on the Hub, and loading from a local CSV (recap SQL file 13's `pd.read_sql_query()` → pandas → here workflow) is the realistic starting point.

## DataCollator — the Datasets-library equivalent of PyTorch's DataLoader collation

```python
from transformers import DataCollatorWithPadding

data_collator = DataCollatorWithPadding(tokenizer=tokenizer)
```

Recap PyTorch file 08's `DataLoader` and file 02's padding notes — a `DataCollator` handles DYNAMIC padding at BATCH-CREATION time (pad each batch only to ITS longest sequence, rather than a fixed global max length) — genuinely more memory-efficient than padding every example to the same fixed length upfront, since different batches can have different actual padding needs.

## Streaming datasets — for genuinely large-scale data

```python
streamed_dataset = load_dataset("imdb", split="train", streaming=True)
for example in streamed_dataset.take(3):
    print(example)
```

Genuinely important for datasets too large to fit in memory or disk comfortably — `streaming=True` loads examples on-the-fly as needed, rather than downloading/materializing the entire dataset upfront. Directly connects to the general "data too big for memory" concern flagged back in SQL file 11's indexing/optimization notes — same underlying constraint, a different tool's solution to it.

## Try this

```python
# Load the "imdb" dataset, take a small shuffled subset (500 examples) for fast iteration
# Tokenize it using .map() with batched=True and a DistilBERT tokenizer
# Convert the tokenized subset to a pandas DataFrame and inspect the input_ids column --
# confirm the lengths are consistent (due to padding/truncation) and match what's expected
# Then use .filter() to find and count how many reviews in your subset are LABELED positive
# vs negative, confirming it roughly matches the known ~50/50 balance of the IMDB dataset
```

Round-tripping between `Dataset` and pandas, and confirming the tokenization output shape makes sense, is genuinely the exact preparation workflow needed before file 05's actual fine-tuning can begin.

## What's next
File 05 — Fine-tuning Text Classification, finally putting files 01-04 together into an actual TRAINING run — taking a pretrained base model and adapting it to a specific task with the Trainer API.
