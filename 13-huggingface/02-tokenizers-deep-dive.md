# 02 — Tokenizers Deep Dive

Recap file 01's brief tokenizer introduction and Classical NLP file 12's subword tokenization preview — this file goes properly deep, since correct tokenization is genuinely the foundation every Hugging Face workflow depends on.

## Basic tokenization — seeing subwords in action

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

text = "Tokenization is fascinating"
tokens = tokenizer.tokenize(text)
print(tokens)   # ['token', '##ization', 'is', 'fascinating']
```

Recap Classical NLP file 12 directly — `##ization` is a CONTINUATION subword (the `##` prefix signals "this attaches to the previous token, not a new word"). "Tokenization" got split into "token" + "##ization" because BERT's WordPiece vocabulary contains those pieces but not the whole word "tokenization" as a single unit — genuinely solving the out-of-vocabulary problem flagged in that bridge file.

## From tokens to IDs — what the model actually receives

```python
token_ids = tokenizer.convert_tokens_to_ids(tokens)
print(token_ids)   # [19204, 3989, 2003, 17160]

# Or, more commonly, do both steps at once:
encoded = tokenizer(text)
print(encoded)
# {'input_ids': [101, 19204, 3989, 2003, 17160, 102], 'attention_mask': [1, 1, 1, 1, 1, 1]}
```

Genuinely important detail: `encoded['input_ids']` has TWO extra numbers (101 and 102) compared to the 4 original tokens — these are SPECIAL TOKENS, covered next.

## Special tokens — genuinely essential, easy to overlook

```python
print(tokenizer.cls_token, tokenizer.cls_token_id)   # [CLS] 101
print(tokenizer.sep_token, tokenizer.sep_token_id)     # [SEP] 102
print(tokenizer.pad_token, tokenizer.pad_token_id)       # [PAD] 0
print(tokenizer.unk_token, tokenizer.unk_token_id)         # [UNK] 100
```

- `[CLS]` — added at the START of every input; for classification tasks (file 05), this token's final hidden state is genuinely used as the representation of the WHOLE sequence.
- `[SEP]` — separates sentences/segments (used constantly in Question Answering, file 07, to separate the question from the context passage).
- `[PAD]` — fills shorter sequences up to a common length (covered fully below).
- `[UNK]` — for any token genuinely not representable even at the subword level (rare, given how granular subword vocabularies are).

Genuinely important to know: `[CLS]` and `[SEP]` placement is MODEL-SPECIFIC — GPT-family tokenizers, for instance, use different special tokens entirely (often just an end-of-text token). `AutoTokenizer` handles this correctly automatically, precisely why the AutoClass pattern (file 01) matters — the same surrounding code adapts correctly across different model families.

## Decoding — converting IDs back to readable text

```python
decoded = tokenizer.decode(encoded['input_ids'])
print(decoded)   # "[CLS] tokenization is fascinating [SEP]"

decoded_clean = tokenizer.decode(encoded['input_ids'], skip_special_tokens=True)
print(decoded_clean)   # "tokenization is fascinating"
```

Genuinely useful for debugging — confirming exactly what the model is "seeing" after tokenization, and for converting GENERATED token IDs (file 08's text generation) back into readable output.

## Padding and truncation — handling variable-length text in batches

```python
sentences = ["Short one.", "This is a genuinely much longer sentence with many more words in it."]

encoded_batch = tokenizer(sentences, padding=True, truncation=True, max_length=10, return_tensors="pt")
print(encoded_batch['input_ids'])
print(encoded_batch['attention_mask'])
```

Recap PyTorch file 08's DataLoader/batching discipline — a batch needs UNIFORM tensor shapes (recap PyTorch file 02's shape rules), but sentences naturally have different lengths. `padding=True` pads shorter sequences with `[PAD]` tokens up to the LONGEST sequence in the batch; `truncation=True` cuts off sequences longer than `max_length`. The `attention_mask` (recap the PyTorch file 19 preview) marks which tokens are REAL (1) versus PADDING (0) — critically, this tells the attention mechanism (recap PyTorch file 15) to IGNORE padding tokens when computing attention, so they don't distort the model's understanding of the real content.

```python
# Genuinely important gotcha: WITHOUT return_tensors="pt", tokenizer output is plain
# Python lists, not PyTorch tensors -- return_tensors="pt" gives PyTorch tensors directly,
# ready for model(**encoded_batch) exactly like PyTorch file 19's training loop pattern
```

## Different padding strategies — genuinely practical choices

```python
tokenizer(sentences, padding='max_length', max_length=20)   # pad ALL sequences to a FIXED length
tokenizer(sentences, padding='longest')                        # pad to the LONGEST sequence in THIS batch (the default when padding=True)
tokenizer(sentences, padding=True, pad_to_multiple_of=8)          # pad to a multiple of 8 -- genuinely helps GPU efficiency (recap PyTorch file 13)
```

`pad_to_multiple_of=8` is genuinely a real, practical optimization — GPU tensor operations (recap PyTorch file 13's mixed precision notes) are often more efficient when dimensions are multiples of 8, since it aligns better with how Tensor Cores process data.

## Tokenizing sentence pairs — genuinely essential for tasks like Question Answering (file 07)

```python
question = "What is tokenization?"
context = "Tokenization is the process of splitting text into smaller units."

encoded = tokenizer(question, context, return_tensors="pt")
print(tokenizer.decode(encoded['input_ids'][0]))
# "[CLS] what is tokenization? [SEP] tokenization is the process of splitting text into smaller units. [SEP]"
```

Genuinely worth seeing this pattern now — `[SEP]` divides the two segments, and `token_type_ids` (an additional output, not shown above) marks WHICH segment each token belongs to. File 07 builds directly on this exact pattern for extracting answers from a context passage.

## Fast vs slow tokenizers — a genuinely practical performance note

```python
print(tokenizer.is_fast)   # True for most AutoTokenizer defaults now
```

Recap the `tokenizers` library mention from file 01 — "fast" tokenizers are implemented in Rust (versus a pure-Python "slow" implementation), genuinely significantly faster for large-scale preprocessing. `AutoTokenizer` uses the fast version by default when available — worth knowing this distinction exists if ever debugging unexpectedly slow preprocessing.

## Try this

```python
# Take a sentence with a genuinely rare/unusual word (e.g. a made-up word like "flibbertigibbet"
# or a technical term) and tokenize it with bert-base-uncased -- observe how many subword
# pieces it gets split into
# Then tokenize a BATCH of 3 sentences with very different lengths using padding=True,
# print the attention_mask for each, and confirm the padding positions correctly show 0
# Finally, decode the padded input_ids WITHOUT skip_special_tokens and confirm you can
# see the [PAD] tokens explicitly in the output
```

Seeing the attention_mask's zeros line up exactly with the `[PAD]` tokens in the decoded output is genuinely the clearest way to confirm understanding of how padding and the attention mask work together.

## What's next
File 03 — AutoModel & Pipelines, the two ways to actually RUN a pretrained model — from the highest-level one-liner (`pipeline`) down to the full manual tokenizer+model control shown in this file.
