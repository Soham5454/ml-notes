# 08 — LoRA/PEFT/QLoRA Deep Dive

Recap file 07's closing note and Hugging Face file 12's preview directly — moving from RETRIEVAL-based grounding (RAG, files 06-07) into WEIGHT-based customization, the full, proper treatment of efficient fine-tuning, and Math Foundations file 03's original SVD/low-rank preview finally getting its complete, applied payoff.

## Recap the full LoRA math — Math Foundations file 03 and Hugging Face file 12, now completely tied together

```python
# Recap Math Foundations file 03's EXACT words: "LoRA fine-tunes huge language
# models by learning small LOW-RANK matrices instead of updating the full
# weight matrix... approximating a big matrix using only its most important
# singular values/vectors"
# Recap Hugging Face file 12's implementation directly:
# W_new = W_frozen + (A @ B), where A is (d x r), B is (r x d), r << d
```

Genuinely worth revisiting this formula one final time with COMPLETE understanding now — recap Math Foundations file 03's rank/SVD derivation directly: the bet LoRA makes is that the TASK-SPECIFIC adaptation needed (the difference between a general base model's behavior and the desired fine-tuned behavior) lives in a genuinely LOW-DIMENSIONAL subspace, even though the full weight matrix itself is high-dimensional — precisely the same "most information lives in a few dominant directions" insight from that file's PCA discussion, now applied to WEIGHT UPDATES instead of DATA.

## When fine-tuning is genuinely still the right choice — recap Hugging Face file 13's bridge framework directly

```
Recap Hugging Face file 13's decision framework directly:
- Need CONSISTENT behavior/style/format across many requests -> fine-tuning
  (or careful prompting, file 02) -- genuinely more reliable than hoping
  a prompt instruction is followed every single time
- Need the model to internalize DOMAIN-SPECIFIC knowledge/vocabulary
  (recap the general "specialized terminology" theme) that's genuinely
  hard to convey via a prompt/RAG alone
- Need a SMALLER, cheaper, faster specialized model rather than always
  calling a large general-purpose one (recap Cloud file 10's cost-
  optimization discipline directly)
```

Genuinely worth being honest and direct — recap file 06/07's RAG discussion: if the need is genuinely "give the model access to specific FACTS/DOCUMENTS," RAG is usually the better, more maintainable choice (facts can be updated without retraining). Fine-tuning (this file) is genuinely better suited to changing HOW the model behaves/responds, not WHAT it knows.

## QLoRA — recap Hugging Face file 12's brief mention, now fully explained

```python
from transformers import BitsAndBytesConfig
import torch

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,                              # recap PyTorch file 18's quantization discussion directly
    bnb_4bit_quant_type="nf4",                         # genuinely a specialized 4-bit format, optimized
                                                            # for the STATISTICAL distribution neural network
                                                            # weights typically follow (recap Math Foundations
                                                            # file 10's Normal distribution discussion directly)
    bnb_4bit_compute_dtype=torch.bfloat16
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=bnb_config,
    device_map="auto"
)
```

Genuinely worth understanding the COMBINATION precisely — recap PyTorch file 18's quantization discussion (shrinking weights from 32-bit to lower precision) directly: QLoRA combines (1) loading the FROZEN base model weights in 4-BIT precision (genuinely quartering memory usage compared to standard 16-bit), with (2) LoRA's low-rank adapter matrices (this file, trained in higher precision for numerical stability) on top. This combination genuinely enables fine-tuning models with BILLIONS of parameters on a SINGLE consumer GPU — recap PyTorch file 13's GPU memory constraints directly, this is precisely the technique that made that constraint dramatically less limiting.

## A complete PEFT/LoRA fine-tuning walkthrough — recap Hugging Face files 05 and 12 combined

```python
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments, Trainer

model_name = "meta-llama/Llama-2-7b-hf"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, quantization_config=bnb_config, device_map="auto")

model = prepare_model_for_kbit_training(model)      # genuinely prepares a quantized model for LoRA training

lora_config = LoraConfig(
    r=16, lora_alpha=32, lora_dropout=0.05,
    target_modules=["q_proj", "v_proj"],      # recap PyTorch file 15's Q/K/V attention matrices directly --
                                                    # genuinely the SAME target choice as Hugging Face file 12
    task_type="CAUSAL_LM"      # recap Hugging Face file 08's causal LM task directly, NOT sequence
                                    # classification like file 12's earlier example
)

peft_model = get_peft_model(model, lora_config)
peft_model.print_trainable_parameters()      # recap Hugging Face file 12's "barely 1% trainable" observation directly
```

Genuinely worth recognizing this as a DIRECT extension of Hugging Face file 12's exact pattern, now applied to a CAUSAL language model (recap Hugging Face file 08's generation-task discussion) rather than that file's classification example — the `task_type` changes, but every other concept (LoraConfig, target_modules, print_trainable_parameters) is genuinely identical.

## Training on instruction-following data — recap file 01's instruction-tuning discussion directly

```python
def format_instruction(example):
    return f"### Instruction:\n{example['instruction']}\n\n### Response:\n{example['response']}"

# Recap Hugging Face file 04's Datasets library and .map() directly
dataset = dataset.map(lambda x: tokenizer(format_instruction(x), truncation=True, max_length=512))

training_args = TrainingArguments(
    output_dir="./lora-instruction-tuned",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    learning_rate=2e-4,      # genuinely often HIGHER than full fine-tuning's typical rate (recap Hugging Face
                                  # file 05's 2e-5 example) -- LoRA's small parameter count can genuinely tolerate,
                                  # and often benefits from, a larger learning rate
    fp16=True      # recap PyTorch file 13's mixed precision discussion directly
)

trainer = Trainer(model=peft_model, args=training_args, train_dataset=dataset)
trainer.train()
```

Genuinely worth flagging the learning rate difference directly — recap Math Foundations file 08's optimization landscape discussion: LoRA is updating a much SMALLER, lower-dimensional set of parameters, and empirically tends to tolerate/benefit from a higher learning rate than full fine-tuning would, a genuinely real, practical tuning consideration worth knowing.

## Merging LoRA weights — genuinely important for deployment simplicity

```python
merged_model = peft_model.merge_and_unload()      # genuinely combines W_frozen + (A@B) into ONE regular weight matrix
merged_model.save_pretrained("./merged-model")      # recap PyTorch file 14's saving discussion directly
```

Genuinely worth understanding WHY this matters for deployment — recap Hugging Face file 12's "swappable adapter" advantage directly: KEEPING the adapter separate (not merged) is genuinely useful when serving MULTIPLE different fine-tuned variants from the SAME base model efficiently. But for a SINGLE, final deployed model, MERGING the LoRA weights back into the base model produces a normal, standard model with NO extra inference overhead from the adapter computation — genuinely the right choice when only one fine-tuned variant will ever be served.

## Multi-adapter serving — genuinely a powerful, real production pattern

```python
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained(model_name)
model_with_adapter_a = PeftModel.from_pretrained(base_model, "./adapter-customer-support")
model_with_adapter_b = PeftModel.from_pretrained(base_model, "./adapter-code-review")
```

Genuinely worth recognizing this as the real, practical payoff of Hugging Face file 12's "swappable adapter" advantage — recap Cloud file 10's cost-optimization discipline directly: ONE base model's memory footprint, shared across MULTIPLE task-specific adapters (each genuinely tiny, megabytes not gigabytes), is dramatically more resource-efficient than deploying multiple FULLY fine-tuned models separately.

## The honest, complete efficient fine-tuning decision framework

```
Small model (< 1B params), plenty of compute      -> Full fine-tuning genuinely fine
Large model, limited GPU memory                     -> LoRA
Large model, VERY limited GPU memory (consumer GPU)   -> QLoRA
Need MULTIPLE task-specific variants of ONE base model  -> LoRA/PEFT with multi-adapter serving
Need the ABSOLUTE best possible performance,              -> Full fine-tuning MIGHT still edge out
  genuinely unlimited compute budget                          LoRA slightly, though the gap has
                                                                    narrowed considerably (recap
                                                                    Hugging Face file 12's honest note)
```

## Try this

```python
# Take a small open-weight model (recap Hugging Face folder's model-loading
# discussion, genuinely something small enough to run comfortably, e.g. a
# GPT-2 variant or a small Llama/Mistral model if resources allow)
# Fine-tune it using QLoRA on a small instruction-following dataset
# (format_instruction pattern above), targeting the attention Q/V projections
# Print the trainable-parameter percentage (recap Hugging Face file 12's
# exercise directly) and compare it against what full fine-tuning would
# require
# After training, merge the LoRA weights and confirm the merged model
# produces IDENTICAL outputs to the unmerged peft_model on a test prompt --
# directly verifying merge_and_unload() preserves behavior correctly
```

Confirming merged and unmerged models produce identical outputs is genuinely a good, concrete verification that the mathematical equivalence (`W_frozen + A@B` merged into one matrix) actually holds in practice, not just in theory.

## What's next
File 09 — LLM Agents & Tool Use, the full technical treatment of file 03's ReAct preview and file 04's function-calling foundation — building systems that genuinely take actions, not just generate text.
