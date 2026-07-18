# 03 — Advanced Prompting Techniques

Recap file 02's closing note — covering genuinely powerful patterns that go beyond foundational prompting, directly connecting to DSA file 19's problem-solving framework and file 01's honest reasoning-limitation discussion.

## Chain-of-Thought (CoT) prompting — genuinely the single most impactful advanced technique

```python
# Without CoT
prompt = "If a store has 15 apples and sells 40% of them, then receives a shipment of 20 more, how many apples does it have?"

# With CoT
prompt = """
If a store has 15 apples and sells 40% of them, then receives a shipment
of 20 more, how many apples does it have? Think through this step by step.
"""
```

Genuinely worth understanding WHY this works, connecting directly to DSA file 19's problem-solving framework — recap that file's core lesson: jumping straight to an answer skips the actual REASONING process. Chain-of-thought prompting genuinely encourages the model to work through INTERMEDIATE STEPS explicitly (recap Math Foundations file 07's chain rule discussion — a genuinely apt name parallel: breaking a complex computation into smaller, connected steps), which empirically produces significantly more ACCURATE results on multi-step reasoning tasks, since the model's own generated intermediate steps become additional CONTEXT informing the final answer.

## Zero-shot CoT — genuinely the simplest possible version

```python
prompt = "If a store has 15 apples and sells 40% of them, then receives a shipment of 20 more, how many apples does it have? Let's think step by step."
```

Genuinely worth knowing this specific phrase ("Let's think step by step") is a well-documented, empirically effective trigger phrase — a genuinely minimal prompt addition that reliably induces step-by-step reasoning, without needing any few-shot examples (recap file 02's few-shot discussion directly).

## Few-shot CoT — combining file 02's few-shot technique with reasoning

```python
prompt = """
Q: A cafe has 8 tables with 4 chairs each. If 6 chairs are removed, how many remain?
A: 8 tables × 4 chairs = 32 chairs. 32 - 6 removed = 26 chairs remain. The answer is 26.

Q: If a store has 15 apples and sells 40% of them, then receives a shipment of 20 more, how many apples does it have?
A:
"""
```

Genuinely worth recognizing this as the DIRECT combination of file 02's few-shot examples with this file's step-by-step reasoning — the EXAMPLE shows not just the correct final answer, but the correct REASONING PATTERN, genuinely teaching the model (within the prompt, no actual training involved, recap file 02's zero-cost-adaptation point directly) exactly HOW to approach the new problem.

## Self-consistency — genuinely a practical reliability technique

```python
# Recap Hugging Face file 08's temperature/sampling discussion directly
responses = [call_llm(prompt, temperature=0.7) for _ in range(5)]      # genuinely run the SAME
                                                                            # CoT prompt multiple times
final_answer = most_common_answer(responses)      # genuinely take a MAJORITY VOTE across the results
```

Genuinely worth connecting this directly to DSA file 08's "find the most frequent element" pattern, and the general ensemble-methods philosophy from the scikit-learn/XGBoost folders (recap those directly — genuinely the SAME underlying principle: combining multiple independent attempts often produces a more RELIABLE result than trusting any single one, since random errors in individual attempts tend to cancel out rather than agree).

## ReAct (Reasoning + Acting) — genuinely the foundational pattern behind LLM agents

```
Genuinely the core idea, worth understanding clearly before file 09's full
agent treatment: ReAct interleaves REASONING (thinking through what's
needed) with ACTIONS (using a tool, recap file 09's full treatment ahead)
and OBSERVATIONS (the tool's result), in a repeating loop
```

```python
prompt = """
Answer the question using the following format:
Thought: [reasoning about what to do next]
Action: [a tool to use, e.g. search("query") or calculate("expression")]
Observation: [the tool's result]
... (repeat Thought/Action/Observation as needed) ...
Final Answer: [the answer]

Question: What is the population of the capital of France, doubled?
"""
```

Genuinely worth recognizing this as directly connecting to the general iterative problem-solving loop from DSA file 19 and MLOps file 01's full ML lifecycle diagram — a genuinely repeating THINK → ACT → OBSERVE → THINK cycle, continuing until the model determines it has enough information for a final answer. File 09 covers the full technical implementation of this pattern.

## Prompt chaining — genuinely important for complex, multi-stage tasks

```python
# Stage 1: extract key facts
facts_prompt = f"Extract the 5 most important facts from this article: {article_text}"
facts = call_llm(facts_prompt)

# Stage 2: use the output of stage 1 as input to a NEW prompt
summary_prompt = f"Write a concise summary using ONLY these facts: {facts}"
summary = call_llm(summary_prompt)
```

Genuinely worth connecting this directly to Flask file 08's blueprint/separation-of-concerns discussion and MLOps file 08's pipeline-dependency-chain discussion — rather than asking ONE prompt to do everything (extraction AND summarization AND formatting simultaneously, genuinely often producing WORSE results due to the compounded complexity), breaking a complex task into SEPARATE, SEQUENTIAL prompts, each with a genuinely focused, narrow job, often produces more RELIABLE results — directly recapping the general "single responsibility" software design principle.

## Tree of Thoughts — genuinely a more sophisticated extension of CoT

```
Genuinely worth knowing this exists as a more advanced technique: rather
than following ONE linear chain of reasoning (CoT), Tree of Thoughts
explores MULTIPLE possible reasoning paths simultaneously (recap DSA
file 09's BFS/DFS tree-exploration concept directly -- genuinely the
SAME "explore multiple branches, backtrack from dead ends" philosophy
from DSA file 02's backtracking discussion), evaluating and pruning
less-promising paths, similar in spirit to DSA file 02's choose-explore-
un-choose backtracking template
```

## Self-critique / reflection prompting — genuinely a real, practical quality technique

```python
initial_response = call_llm("Write a Python function to check if a number is prime")

critique_prompt = f"""
Review this code for bugs, edge cases, and efficiency issues:
{initial_response}
"""
critique = call_llm(critique_prompt)

final_prompt = f"Improve the code based on this critique: {critique}\n\nOriginal code: {initial_response}"
final_response = call_llm(final_prompt)
```

Genuinely worth connecting this directly to DSA file 19's testing/edge-case-checking discipline and MLOps file 08's model-evaluation-gate discussion — having the model CRITIQUE its own output before finalizing it genuinely catches real errors (recap the general "always verify, don't just trust the first attempt" theme running throughout this whole roadmap), though worth an honest caveat: self-critique is genuinely NOT infallible, and file 11's proper evaluation techniques matter for anything genuinely important.

## The honest, practical decision framework — when to use which technique

```
Simple, well-defined tasks                    -> Basic, specific prompting (file 02) is genuinely sufficient
Multi-step reasoning/math/logic                  -> Chain-of-thought, genuinely essential
High-stakes tasks needing reliability              -> Self-consistency (multiple samples + majority vote)
Tasks requiring external information/computation     -> ReAct (leading into file 09's full agent treatment)
Complex, multi-stage tasks                             -> Prompt chaining
Tasks benefiting from exploring multiple approaches      -> Tree of Thoughts
Quality-critical generation                                -> Self-critique/reflection
```

## Try this

```python
# Take a genuinely multi-step reasoning problem (a word problem, or a
# logic puzzle) and compare a DIRECT prompt against a chain-of-thought
# prompt ("let's think step by step") -- run each 3 times and honestly
# check how often each approach reaches the CORRECT answer
# Then implement basic self-consistency: run the CoT version 5 times at
# a genuinely higher temperature, and take a majority vote -- compare
# this majority-vote accuracy against any single run's accuracy
```

Actually measuring accuracy across multiple runs (not just eyeballing one output) is genuinely the real, honest way to confirm these techniques provide a measurable reliability improvement, rather than assuming it based on how convincing a single example looks.

## What's next
File 04 — Working with LLM APIs, moving from prompt DESIGN into the actual technical implementation: API calls, streaming responses, and function calling — the practical foundation for everything built in the rest of this folder.
