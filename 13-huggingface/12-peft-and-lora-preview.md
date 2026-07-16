# 12 — Efficient Fine-tuning Preview: PEFT & LoRA

Recap Math Foundations file 03's SVD/low-rank note (flagged explicitly there as a preview of THIS exact topic) and file 11's closing note — a focused look at parameter-efficient fine-tuning, genuinely essential once models get large, and a direct preview of what the Frontier GenAI phase covers in depth.

## The problem — full fine-tuning gets genuinely expensive

```python
# Recap file 05's full fine-tuning -- EVERY parameter in the model gets updated
# For DistilBERT (~66M parameters), genuinely manageable on a single GPU
# For a large modern LLM (billions of parameters), full fine-tuning requires:
# - Enormous GPU memory (storing gradients AND optimizer states for EVERY parameter,
#   recap PyTorch file 08's Adam optimizer state discussion -- Adam genuinely stores
#   TWO additional numbers per parameter, tripling the memory footprint)
# - Genuinely expensive compute, often requiring multiple high-end GPUs
```

Recap PyTorch file 13's GPU/memory concerns directly — this is precisely why full fine-tuning of large models is often impractical for individual practitioners or smaller teams, motivating an entirely different approach.

## The core idea behind LoRA (Low-Rank Adaptation)

Recap Math Foundations file 03's exact preview: *"LoRA fine-tunes huge language models by learning small LOW-RANK matrices instead of updating the full weight matrix... approximating a big matrix using only its most important singular values/vectors."*

```python
# Instead of updating the FULL weight matrix W (potentially huge, e.g. 4096x4096),
# LoRA freezes W entirely and learns TWO much SMALLER matrices, A and B, such that:
# W_new = W_frozen + (A @ B)
# where A is (4096 x r) and B is (r x 4096), with r (the "rank") being GENUINELY SMALL
# (often 4, 8, or 16) compared to 4096
```

Genuinely the key insight, connecting directly back to Math Foundations file 03's rank/SVD theory: the CHANGE needed to adapt a pretrained model to a new task is often well-approximated by a LOW-RANK matrix — meaning most of the "directions" of change genuinely aren't needed, only a small handful of the most important ones. Recap that file's rank example (`[[1,2],[2,4]]` having rank 1 despite being 2x2) — LoRA is betting that the ADAPTATION needed for fine-tuning has a similarly low effective rank, even though the full weight matrix itself doesn't.

## The parameter count savings — genuinely dramatic

```python
# Full fine-tuning of a 4096x4096 weight matrix: 4096 * 4096 = ~16.8 million parameters
# LoRA with rank r=8: (4096*8) + (8*4096) = ~65,000 parameters
# Genuinely over 250x FEWER trainable parameters for this one layer alone
```

Recap PyTorch file 12's transfer learning "count trainable parameters" exercise directly — this is the SAME underlying philosophy (train far fewer parameters than the full model contains), taken to a genuinely more extreme, more clever degree than simple layer-freezing.

## Using PEFT — Hugging Face's library for this exact technique

```python
from peft import LoraConfig, get_peft_model, TaskType

lora_config = LoraConfig(
    task_type=TaskType.SEQ_CLS,          # recap file 05's sequence classification task
    r=8,                                     # the rank -- genuinely the key hyperparameter
    lora_alpha=16,                             # a scaling factor for the LoRA update's magnitude
    lora_dropout=0.1,                            # recap PyTorch file 09's dropout, applied here too
    target_modules=["q_lin", "v_lin"]              # WHICH weight matrices to apply LoRA to
                                                       # (often just the attention Query/Value projections,
                                                       # recap PyTorch file 15's Q/K/V matrices)
)

model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased", num_labels=2)
peft_model = get_peft_model(model, lora_config)

peft_model.print_trainable_parameters()
# trainable params: 739,586 || all params: 67,584,004 || trainable%: 1.09%
```

Genuinely worth actually seeing this printed output — barely 1% of the model's parameters are trainable, yet LoRA fine-tuning genuinely achieves performance close to full fine-tuning on many tasks. `target_modules` specifically targeting the attention Query/Value projection matrices (recap PyTorch file 15) is a genuinely deliberate, common choice — these matrices are where task-specific adaptation tends to matter most.

## Training with PEFT — genuinely the SAME Trainer pattern, unchanged

```python
from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(output_dir="./lora_results", num_train_epochs=3, per_device_train_batch_size=8)

trainer = Trainer(
    model=peft_model,             # the ONLY change -- pass the PEFT-wrapped model instead of the base one
    args=training_args,
    train_dataset=tokenized_train,
    eval_dataset=tokenized_test,
)
trainer.train()
```

Genuinely satisfying to recognize — recap files 05-07's identical `Trainer` usage — literally nothing about the training LOOP changes, only the model object being trained. This is precisely why PEFT/LoRA feels like a natural EXTENSION of everything already learned, not a separate new framework.

## Saving and loading LoRA adapters — genuinely lightweight

```python
peft_model.save_pretrained("./my-lora-adapter")
# Genuinely tiny file size -- only the small A/B matrices get saved, NOT the full base model

from peft import PeftModel
base_model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased", num_labels=2)
loaded_model = PeftModel.from_pretrained(base_model, "./my-lora-adapter")
```

Genuinely a real, practical advantage worth highlighting: since the base model stays FROZEN and unchanged, the same base model can be paired with MANY DIFFERENT LoRA adapters (one per task) — swapping adapters is dramatically cheaper than swapping entire fine-tuned models, since each adapter is genuinely tiny (megabytes, not gigabytes) compared to the full base model.

## QLoRA — a brief, honest mention of the next step up

```python
# QLoRA combines LoRA with QUANTIZATION (recap PyTorch file 18's quantization notes --
# reducing weight precision, e.g. to 4-bit, for even further memory savings)
# Genuinely lets even larger models be fine-tuned on genuinely modest, single-GPU hardware
```

Recap PyTorch file 18's quantization discussion directly — genuinely worth knowing this combination exists as the current practical standard for fine-tuning very large models on limited hardware, though the full depth of QLoRA is genuinely beyond this preview file's scope — the Frontier GenAI phase will cover this properly.

## The honest tradeoff — when full fine-tuning is still worth it

```
Small model (under ~1B parameters), genuinely have the compute            -> Full fine-tuning is fine
Large model, limited compute/GPU memory                                     -> LoRA/PEFT genuinely necessary
Need to serve MANY task-specific variants of the SAME base model              -> LoRA's swappable-adapter advantage
  is genuinely valuable even with sufficient compute for full fine-tuning
Maximum possible performance, compute is genuinely not a constraint             -> Full fine-tuning can still
  slightly outperform LoRA in some cases, though the gap has narrowed considerably
```

## Try this

```python
# Take the text classification fine-tuning setup from file 05, and re-run it using
# PEFT/LoRA instead of full fine-tuning (r=8, targeting the attention q_lin/v_lin modules)
# Print the trainable parameter count/percentage and compare against file 05's full
# fine-tuning parameter count
# Train for the SAME number of epochs on the SAME small dataset, and compare final
# accuracy between the LoRA version and the full fine-tuning version from file 05 --
# how close is the performance, given the dramatically smaller trainable parameter count?
```

Directly comparing LoRA against file 05's full fine-tuning, on the identical dataset, is genuinely the clearest way to build honest, calibrated intuition about the real performance-vs-efficiency tradeoff this technique represents.

## What's next
File 13 — the final Hugging Face → Frontier GenAI Bridge, pulling every concept from files 01-12 together and connecting them explicitly to what's coming next: RAG, vector databases, prompt engineering, and LLM agents.
