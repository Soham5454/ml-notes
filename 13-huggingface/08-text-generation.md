# 08 — Text Generation

Recap file 07's closing note — shifting into GENERATIVE tasks. Recap Classical NLP file 05's n-gram language model directly: this is the SAME core task (predict the next token) at a vastly more capable scale, using GPT-style **causal language models**.

## Causal vs Masked language modeling — the genuinely important architectural distinction

```python
# BERT (used in files 05-07): MASKED language modeling
# "The [MASK] sat on the mat" -> predict "cat" -- can see BOTH left AND right context

# GPT (this file): CAUSAL language modeling
# "The cat sat on the" -> predict "mat" -- can ONLY see LEFT context (everything before)
```

Genuinely the key reason BERT-family models (files 05-07) are used for classification/extraction tasks, while GPT-family models are used for generation: BERT's bidirectional attention (recap PyTorch file 15) sees the WHOLE input at once, genuinely great for UNDERSTANDING a complete piece of text, but architecturally can't generate text one token at a time in a genuinely causal way. GPT's attention is MASKED to only look backward (recap PyTorch file 15's mention of masked attention for decoder-only transformers) — precisely what makes autoregressive generation (predicting one token, then using it as input to predict the next) work correctly.

## Basic generation with a pipeline

```python
from transformers import pipeline

generator = pipeline("text-generation", model="gpt2")
result = generator("The future of artificial intelligence is", max_length=50, num_return_sequences=1)
print(result[0]['generated_text'])
```

## Manual generation — seeing the autoregressive loop directly

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

tokenizer = AutoTokenizer.from_pretrained("gpt2")
model = AutoModelForCausalLM.from_pretrained("gpt2")

input_text = "The future of artificial intelligence is"
input_ids = tokenizer.encode(input_text, return_tensors="pt")

with torch.no_grad():
    output = model.generate(input_ids, max_length=50)

print(tokenizer.decode(output[0], skip_special_tokens=True))
```

## Building the autoregressive loop BY HAND — genuinely worth doing once

```python
generated = input_ids
for _ in range(20):                                     # generate 20 new tokens, one at a time
    with torch.no_grad():
        outputs = model(generated)
    next_token_logits = outputs.logits[0, -1, :]           # logits for the NEXT token, from the LAST position
    next_token_id = torch.argmax(next_token_logits).unsqueeze(0).unsqueeze(0)
    generated = torch.cat([generated, next_token_id], dim=1)   # append the new token, feed the WHOLE thing back in

print(tokenizer.decode(generated[0], skip_special_tokens=True))
```

Genuinely this hand-rolled loop IS exactly what `.generate()` does internally (in its simplest form) — recap Classical NLP file 05's `generate_text()` function for the n-gram model; this is STRUCTURALLY the identical loop (predict next token, append, repeat), just using a vastly more powerful model to do the predicting. Recognizing this parallel is genuinely the clearest way to see how directly this connects back to that earlier, simpler file.

## Decoding strategies — genuinely important, easy-to-underestimate choices

```python
# Greedy decoding (what the manual loop above does) -- ALWAYS pick the single most likely next token
output = model.generate(input_ids, max_length=50, do_sample=False)

# Genuinely a real problem: greedy decoding tends to produce REPETITIVE, boring text,
# since it always takes the "safest" choice at every single step
```

```python
# Sampling -- introduce genuine randomness, weighted by predicted probability
output = model.generate(input_ids, max_length=50, do_sample=True, temperature=0.7)
```

`temperature` genuinely controls RANDOMNESS: recap Math Foundations file 10's softmax/probability notes — temperature rescales the logits BEFORE softmax. Low temperature (e.g. 0.3) makes the distribution SHARPER (closer to greedy, more predictable/safe); high temperature (e.g. 1.5) makes it FLATTER (more random, more creative but riskier/less coherent). Genuinely one of the most important practical knobs when working with any generative language model, including the LLM APIs used in the upcoming Frontier GenAI phase.

```python
# Top-k sampling -- only consider the K most likely next tokens, then sample among those
output = model.generate(input_ids, max_length=50, do_sample=True, top_k=50)

# Top-p (nucleus) sampling -- consider the SMALLEST set of tokens whose CUMULATIVE
# probability exceeds p -- genuinely more adaptive than top-k's fixed count
output = model.generate(input_ids, max_length=50, do_sample=True, top_p=0.9)
```

Genuinely the honest comparison: top-k always considers exactly K options, regardless of how confident the model actually is — sometimes too restrictive (when the model is genuinely uncertain among many options), sometimes too permissive (when the model is very confident about just one or two). Top-p adapts the candidate pool SIZE based on the model's actual confidence at each step — genuinely the more commonly preferred approach in modern generation setups for this reason.

## Beam search — a genuinely different strategy, optimizing for OVERALL sequence quality

```python
output = model.generate(input_ids, max_length=50, num_beams=5, early_stopping=True)
```

Recap DSA file 08's heap-based "track the best K candidates" pattern — genuinely a related idea: instead of committing to the single best token at EVERY step (greedy) or sampling randomly, beam search keeps track of the TOP `num_beams` most promising PARTIAL sequences simultaneously, only committing to a final choice once the full sequence is complete. This can find a genuinely better OVERALL sequence than pure greedy decoding, at the cost of more computation (`num_beams` times the work per step).

## Repetition penalties — a genuinely practical fix for a common failure mode

```python
output = model.generate(
    input_ids, max_length=100,
    repetition_penalty=1.2,          # discourages repeating already-generated tokens
    no_repeat_ngram_size=3             # HARD-blocks any 3-gram (recap Classical NLP file 05) from repeating
)
```

Genuinely a real, common problem worth knowing how to fix directly — language models (especially with greedy/low-temperature decoding) can get stuck in repetitive loops ("the the the" or repeating whole phrases). These two parameters are the standard, practical tools for addressing it.

## Prompt engineering — a genuinely important preview for the Frontier GenAI phase

```python
# The SAME model can produce very different quality output depending on how the prompt is framed
basic_prompt = "Write about dogs"
better_prompt = "Write a short, engaging paragraph about why dogs make loyal companions, focusing on their emotional intelligence"
```

Genuinely worth flagging as a preview — prompt engineering (structuring input to get better generation output) becomes a MUCH bigger, more formalized topic once the roadmap reaches large instruction-tuned/chat models in the Frontier GenAI phase. Even with a base model like GPT-2 here, the general principle — more specific, well-structured prompts tend to produce more useful output — genuinely already applies.

## Try this

```python
# Using GPT-2 via the manual .generate() approach, generate text from the SAME prompt
# using: (1) greedy decoding, (2) sampling with temperature=1.5, (3) top-p sampling
# with top_p=0.9, and (4) beam search with num_beams=5
# Compare all 4 outputs side by side -- which feels most repetitive? Which feels most
# incoherent/random? Which feels like the best balance?
# Then deliberately trigger a repetitive loop using greedy decoding with a longer
# max_length, and confirm that adding no_repeat_ngram_size=3 fixes it
```

Directly comparing all four decoding strategies on the identical prompt is genuinely the clearest way to build real, practical intuition about these tradeoffs — reading about them is far less convincing than seeing greedy decoding visibly get stuck in a loop firsthand.

## What's next
File 09 — Summarization & Translation, extending generation into ENCODER-DECODER (Seq2Seq) architectures, which combine BERT-style understanding with GPT-style generation in a single model.
