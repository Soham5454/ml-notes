# 05 — Fine-tuning Text Classification

Recap files 01-04 — everything so far has been building toward this: taking a pretrained BASE model and adapting it to a specific task with real training. Directly continues PyTorch file 12's transfer learning and file 19's `Trainer` preview.

## The full setup — model, tokenizer, dataset

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
from datasets import load_dataset

model_name = "distilbert-base-uncased"      # a BASE model, NOT already fine-tuned for sentiment
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name, num_labels=2)

dataset = load_dataset("imdb")
small_train = dataset['train'].shuffle(seed=42).select(range(2000))      # recap file 04, fast prototyping
small_test = dataset['test'].shuffle(seed=42).select(range(500))
```

Genuinely important detail: `num_labels=2` tells `AutoModelForSequenceClassification` how many OUTPUT classes the new classification head needs (recap file 03's task-specific-head explanation) — the base DistilBERT weights are pretrained, but this final classification layer is RANDOMLY initialized and genuinely needs training from scratch on this specific task.

## Tokenizing the dataset

```python
def tokenize_function(examples):
    return tokenizer(examples['text'], padding='max_length', truncation=True, max_length=256)

tokenized_train = small_train.map(tokenize_function, batched=True)
tokenized_test = small_test.map(tokenize_function, batched=True)
```

Recap file 04's `.map()` pattern directly — nothing new here, just applying it to the actual fine-tuning data.

## The Trainer API — recap PyTorch file 19's preview, now the full picture

```python
from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,                     # recap PyTorch file 07's epoch concept
    per_device_train_batch_size=8,             # recap PyTorch file 08's batching
    per_device_eval_batch_size=8,
    learning_rate=2e-5,                          # recap PyTorch file 19's note -- smaller than typical fine-tuning LR
    weight_decay=0.01,                              # recap PyTorch file 09's regularization
    evaluation_strategy="epoch",                       # run evaluation after every epoch
    save_strategy="epoch",                                # save a checkpoint after every epoch, recap PyTorch file 14
    logging_dir="./logs",
    load_best_model_at_end=True                              # recap PyTorch file 09's "load the BEST checkpoint" discipline
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_train,
    eval_dataset=tokenized_test,
    tokenizer=tokenizer,
)

trainer.train()
```

Genuinely important to recognize: EVERY argument in `TrainingArguments` maps directly onto a concept already covered in PyTorch — `num_train_epochs` (file 07), `learning_rate`/`weight_decay` (files 07/09), `evaluation_strategy` (the train/val loop from file 07), `save_strategy`/`load_best_model_at_end` (file 09/14's checkpointing discipline). `Trainer` is genuinely just automating the exact five-step training loop (zero_grad → forward → loss → backward → step) from PyTorch file 07, plus validation and checkpointing, into a single high-level object.

## Adding a real evaluation metric — recap the `evaluate` library

```python
import numpy as np
import evaluate

accuracy_metric = evaluate.load("accuracy")

def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)      # recap PyTorch file 05's softmax/argmax pattern
    return accuracy_metric.compute(predictions=predictions, references=labels)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_train,
    eval_dataset=tokenized_test,
    compute_metrics=compute_metrics,
    tokenizer=tokenizer,
)
```

Genuinely worth connecting directly to scikit-learn's `accuracy_score` (recap that folder) — `evaluate.load("accuracy")` computes the EXACT same metric, just through Hugging Face's standardized evaluation interface, which genuinely matters more once file 10 covers metrics beyond simple accuracy (F1, precision/recall for imbalanced classification, recap that scikit-learn caution).

## What's happening during `.train()` — the honest, full picture

```python
# Internally, Trainer.train() is genuinely doing this, per batch (recap PyTorch file 07):
# 1. optimizer.zero_grad()
# 2. outputs = model(**batch)                 -- forward pass
# 3. loss = outputs.loss                        -- Hugging Face models compute loss internally
#                                                    when labels are passed (recap PyTorch file 19)
# 4. loss.backward()                               -- backward pass
# 5. optimizer.step()                                -- weight update
# PLUS: periodic evaluation, checkpointing, logging, learning rate scheduling
```

Genuinely worth internalizing this explicitly rather than treating `Trainer` as a black box — every "magic" step is precisely a PyTorch concept already understood deeply from that whole folder.

## Freeze/unfreeze — recap PyTorch file 12's transfer learning decision directly

```python
# Feature-extraction style: freeze the base model, only train the classification head
for param in model.base_model.parameters():
    param.requires_grad = False

# Full fine-tuning (the DEFAULT, and what was done above): everything trainable
# Genuinely the same freeze/unfreeze decision framework from PyTorch file 12 applies
# here identically -- small dataset -> lean toward freezing; larger dataset -> full fine-tune
```

## Running inference with the fine-tuned model

```python
trainer.save_model("./my_sentiment_model")      # recap PyTorch file 14's save_pretrained pattern

from transformers import pipeline
fine_tuned_pipeline = pipeline("sentiment-analysis", model="./my_sentiment_model", tokenizer=tokenizer)
print(fine_tuned_pipeline("This was a genuinely brilliant film"))
```

Genuinely satisfying to see the freshly fine-tuned model plug directly back into the SAME `pipeline()` interface from file 03 — full circle, using the high-level API on a model that was JUST manually trained via the lower-level `Trainer`.

## Manual training loop alternative — for when `Trainer` isn't flexible enough

```python
# Recap PyTorch file 07's raw training loop -- genuinely still fully available
# and sometimes necessary for custom loss functions or unusual training procedures
from torch.utils.data import DataLoader
from torch.optim import AdamW

train_dataloader = DataLoader(tokenized_train, batch_size=8, shuffle=True)
optimizer = AdamW(model.parameters(), lr=2e-5)

model.train()
for epoch in range(3):
    for batch in train_dataloader:
        optimizer.zero_grad()
        outputs = model(**batch)
        loss = outputs.loss
        loss.backward()
        optimizer.step()
```

Genuinely worth knowing this manual option remains fully available — `Trainer` is a convenience layer, not a requirement, precisely the same relationship `pipeline()` has to manual `AutoModel` usage from file 03.

## Try this

```python
# Fine-tune distilbert-base-uncased on a small (1000-2000 example) subset of the imdb
# dataset for 2 epochs, using the Trainer API with accuracy as the eval metric
# Compare its accuracy on the test subset against the ALREADY-fine-tuned
# "distilbert-base-uncased-finetuned-sst-2-english" model's zero-shot performance
# on the SAME test subset (via pipeline(), no training) -- which performs better,
# and does the gap make sense given one was trained specifically on THIS data?
```

Comparing a freshly fine-tuned model against an off-the-shelf pretrained one on the SAME evaluation set is genuinely a good way to build calibrated intuition about when fine-tuning is worth the effort versus when an existing model already performs well enough out of the box.

## What's next
File 06 — Fine-tuning Token Classification (NER), extending the same Trainer-based workflow to a task where EVERY TOKEN needs its own label, rather than one label per whole sequence.
