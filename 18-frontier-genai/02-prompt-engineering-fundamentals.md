# 02 — Prompt Engineering Fundamentals

Recap file 01's closing note — the genuinely foundational skill for working effectively with any LLM, directly connecting to Hugging Face file 13's bridge discussion of zero-shot classification's evolution into full prompting.

## Why prompt engineering is genuinely a real skill, not just "asking nicely"

```
Recap Hugging Face file 03's zero-shot classification discussion directly --
an LLM's output GENUINELY depends heavily on exactly HOW a request is
phrased, structured, and contextualized -- the SAME underlying task can
produce dramatically different quality results depending on prompt design
```

## Being specific and clear — genuinely the most impactful, simplest technique

```
VAGUE: "Write about dogs"
SPECIFIC: "Write a 150-word paragraph explaining why dogs make good
  companions for elderly people living alone, focusing on emotional
  support and daily routine benefits"
```

Genuinely worth recognizing this as directly analogous to file 04's REST API input validation discipline (recap Flask folder) — a vague, underspecified prompt genuinely gives the model more "room" to produce something not quite matching what's actually wanted, exactly as an underspecified API request leaves room for ambiguous interpretation.

## System prompts vs user prompts — genuinely important structural distinction

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant specializing in Python debugging. Always include code examples in your responses."},
    {"role": "user", "content": "Why is my for loop not working?"}
]
```

Genuinely worth understanding this distinction clearly — recap Flask file 08's config-vs-request-logic separation directly: the SYSTEM prompt sets PERSISTENT behavior/context for the whole conversation (genuinely similar to a Flask app's configuration, set once), while the USER prompt is the SPECIFIC request for this turn — separating these lets an application maintain CONSISTENT behavior across many different user requests, exactly the way Flask file 08's config classes kept behavior consistent across many different routes.

## Few-shot prompting — genuinely powerful, directly recapping Classical NLP

```
ZERO-SHOT (recap Hugging Face file 03 directly): "Classify this review's
  sentiment: 'The food was cold and the service was slow.'"

FEW-SHOT: "Classify sentiment as positive/negative.
  Review: 'Amazing experience!' -> Positive
  Review: 'Never coming back.' -> Negative
  Review: 'The food was cold and the service was slow.' -> ?"
```

Genuinely worth recognizing few-shot prompting as directly analogous to Math Foundations file 15's MLE discussion and the general "learning from examples" theme running throughout this ENTIRE roadmap — providing a few labeled EXAMPLES within the prompt itself lets the model infer the EXACT desired format/reasoning pattern, genuinely often improving reliability significantly over zero-shot alone, without any actual model retraining (recap Hugging Face file 05's fine-tuning discussion directly — few-shot prompting achieves SOME of fine-tuning's benefit, with zero training cost, though genuinely less reliably for complex, specialized tasks).

## Output format specification — genuinely important for building real applications

```python
prompt = """
Extract the following information from this customer review and return
ONLY valid JSON, no other text:
{
  "sentiment": "positive" | "negative" | "neutral",
  "key_issues": [list of strings],
  "rating_estimate": integer from 1-5
}

Review: "The delivery was late but the product quality was excellent."
"""
```

Genuinely important, real practical technique — recap Flask file 07's REST API JSON-response discipline directly: when an LLM's output needs to be PARSED programmatically (rather than just displayed to a human), explicitly specifying the exact output format (genuinely often JSON) is essential — an LLM's default conversational, prose-heavy output is genuinely hard to parse reliably without this explicit structure request.

## Role-playing / persona prompts — genuinely useful, worth using thoughtfully

```python
system_prompt = "You are an experienced senior software engineer conducting a code review. Be direct and specific about issues, but constructive."
```

Genuinely worth understanding WHY this works — recap the general "training data contains many examples of different writing styles/expertise levels" observation from file 01's pretraining discussion: a persona prompt genuinely steers the model toward patterns learned from SIMILAR contexts in its training data, often producing more consistent, appropriately-toned output for a specific use case.

## Delimiters — genuinely important for separating instructions from content

```python
prompt = f"""
Summarize the text between the triple backticks in 2 sentences.

```
{user_provided_text}
```
"""
```

Genuinely important, real security-adjacent practice worth flagging directly — recap Flask file 04's input-sanitization discipline and Cloud file 12's prompt-injection preview directly: clearly DELIMITING where user-provided content begins and ends helps the model distinguish between the ACTUAL INSTRUCTIONS and content that should merely be PROCESSED, genuinely reducing (though not eliminating) the risk of a malicious user's input being misinterpreted as a new instruction — full treatment of this security concern in file 12.

## Iterative refinement — genuinely the realistic, practical workflow

```
Genuinely worth accepting directly: prompt engineering is GENUINELY
iterative -- write a prompt, evaluate the output (recap file 11's full
evaluation treatment ahead), identify what's wrong or missing, REFINE
the prompt, repeat
Directly parallels the general "brute force first, then optimize" problem-
solving framework from DSA file 19 -- start with a reasonable first attempt,
then improve based on OBSERVED results, not just theoretical reasoning alone
```

## Common prompt engineering mistakes — genuinely worth knowing explicitly

```
1. Being TOO vague (covered above) -- genuinely the most common issue
2. Asking for MULTIPLE distinct things in one prompt without clear structure
   -- genuinely better to break into separate, sequential prompts or
   clearly numbered sub-requests
3. NOT specifying constraints (length, format, tone) -- recap this file's
   output-format discussion directly
4. Assuming the model KNOWS something it genuinely doesn't (recap file 01's
   knowledge-cutoff limitation directly) -- providing necessary context
   explicitly, rather than assuming it, is genuinely essential
5. NOT testing with EDGE CASES -- recap Flask file 04's "never trust happy-
   path-only testing" discipline directly, now applied to prompts: test
   with unusual, ambiguous, or adversarial inputs, not just clean examples
```

## Try this

```python
# Take a genuinely real task (e.g. "summarize this article," "extract
# action items from this meeting transcript") and write THREE versions
# of a prompt for it: a genuinely vague one, a SPECIFIC one with clear
# constraints, and a FEW-SHOT version with 2-3 examples
# Run all three against the same input and honestly compare the outputs --
# write down specific, concrete differences in quality/reliability/format
# consistency, not just a vague "the better one felt better" impression
```

Writing down SPECIFIC, concrete differences (not vague impressions) between prompt versions is genuinely the discipline that turns prompt engineering into a real, improvable skill rather than trial-and-error guessing.

## What's next
File 03 — Advanced Prompting Techniques, covering chain-of-thought reasoning, ReAct, and other genuinely powerful patterns that go beyond this file's foundational techniques.
