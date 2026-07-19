# 11 — Evaluating LLM Applications

Recap file 10's closing note and Hugging Face file 10's evaluation discussion directly — covering genuinely how to MEASURE whether any of this folder's systems (RAG, agents, fine-tuned models) actually work well, rather than just appearing to work on a few manually-checked examples.

## Why evaluating LLM applications is genuinely harder than classical ML evaluation

```
Recap scikit-learn's classification metrics and Hugging Face file 10's
BLEU/ROUGE discussion directly -- classical metrics assume there's ONE
(or a few) CORRECT answer(s) to compare against
Genuinely much of what LLM applications produce (open-ended generation,
agent decisions, RAG answers) has MANY valid phrasings/approaches --
recap Hugging Face file 10's honest "metrics are proxies" framing
directly, now facing this challenge at its MOST acute
```

## RAG-specific evaluation — recap file 06/07's RAG system directly, now measured properly

```python
# Genuinely TWO separate things to evaluate: did RETRIEVAL find the
# right documents, and did GENERATION produce a good answer FROM them
def evaluate_retrieval(test_queries, expected_doc_ids):
    correct = 0
    for query, expected_id in zip(test_queries, expected_doc_ids):
        results = collection.query(query_texts=[query], n_results=3)      # recap file 05/06 directly
        if expected_id in results['ids'][0]:
            correct += 1
    return correct / len(test_queries)      # genuinely a RECALL@3 metric, recap scikit-learn's recall definition
```

Genuinely worth building a TEST SET of (query, expected relevant document) pairs directly recapping the general train/test discipline from scikit-learn onward — this is genuinely essential, real groundwork: WITHOUT a test set, there's no honest way to know if file 07's chunking/re-ranking changes actually IMPROVED retrieval or just felt like they should have.

## RAGAS — genuinely a standard, dedicated RAG evaluation framework

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision

results = evaluate(
    dataset=eval_dataset,      # genuinely (question, retrieved_context, generated_answer, ground_truth) tuples
    metrics=[faithfulness, answer_relevancy, context_precision]
)
```

Genuinely worth understanding what each of these measures, directly recapping file 06's grounding discussion:
- **Faithfulness**: does the generated answer ACTUALLY follow from the retrieved context, or did the model hallucinate beyond it (recap file 01/06's hallucination discussion directly)?
- **Answer relevancy**: does the answer actually ADDRESS the question asked?
- **Context precision**: were the RETRIEVED documents actually relevant, or was irrelevant context included (recap file 07's re-ranking motivation directly)?

Genuinely worth recognizing these as more sophisticated, LLM-based evaluations of the EXACT concerns already discussed conceptually throughout files 05-07 — RAGAS typically uses ANOTHER LLM call to JUDGE these qualities, a technique covered next.

## LLM-as-judge — genuinely a powerful, widely-used evaluation technique

```python
def llm_judge_evaluation(question, generated_answer, reference_answer):
    judge_prompt = f"""
    Compare the generated answer to the reference answer for this question.
    Rate the generated answer's quality from 1-5 on: accuracy, completeness, clarity.

    Question: {question}
    Reference Answer: {reference_answer}
    Generated Answer: {generated_answer}

    Respond in JSON: {{"accuracy": int, "completeness": int, "clarity": int, "reasoning": str}}
    """
    return call_llm(judge_prompt)      # recap file 02's structured-output-specification technique directly
```

Genuinely worth understanding the real, honest tradeoff here — recap file 03's self-critique discussion and Classical NLP file 10's "metrics are proxies" framing directly: using an LLM to JUDGE another LLM's output is genuinely more FLEXIBLE and can capture nuanced quality dimensions that exact-match metrics (recap Hugging Face file 10's BLEU discussion) genuinely can't — but it's also genuinely a real, additional cost (another API call, recap file 01's token-cost discussion), and the judge itself can be WRONG or biased, meaning this technique needs its own validation (does the judge's rating actually correlate with genuine human judgment?) before being fully trusted.

## Human evaluation — recap Hugging Face file 10's "honest gold standard" framing directly

```
Genuinely worth restating this ONE more time, with even higher stakes here
than in Hugging Face file 10's classical NLP context: for GENUINELY
important, high-stakes LLM applications, human evaluation remains the
real, honest ground truth -- automatic metrics (BLEU/ROUGE, RAGAS,
LLM-as-judge) are genuinely useful PROXIES for faster iteration, but a
periodic, real human review of actual outputs is genuinely essential
for catching issues automatic metrics might miss entirely
```

## Evaluating agents — recap file 09's tool-use discussion directly, now measured properly

```python
def evaluate_agent_trajectory(agent_log, expected_tools_used):
    # Recap file 09's logged_agent_step discussion directly -- evaluate
    # WHETHER the agent used the CORRECT tools, not just whether the
    # FINAL answer happened to be right (which could be right by
    # accident, using the WRONG reasoning path)
    tools_used = [step['tool_name'] for step in agent_log if step['type'] == 'tool_use']
    correct_tool_selection = set(tools_used) == set(expected_tools_used)

    final_answer_correct = check_final_answer(agent_log[-1]['output'])

    return {"correct_tools": correct_tool_selection, "correct_answer": final_answer_correct}
```

Genuinely important, real insight worth understanding — recap DSA file 19's problem-solving framework directly: evaluating an agent SOLELY on its final answer misses genuinely important information — an agent that reaches the right answer through the WRONG reasoning/tool-use path is genuinely LESS reliable going forward than one that reasons correctly, even if this specific instance happened to work out. Evaluating the full TRAJECTORY (recap MLOps file 09's full-pipeline monitoring philosophy directly), not just the endpoint, is genuinely important for agent systems specifically.

## A/B testing LLM applications — recap MLOps file 10's A/B testing discussion directly

```python
def ab_test_prompts(user_id, prompt_version_a, prompt_version_b):
    group = "A" if hash(user_id) % 2 == 0 else "B"      # recap MLOps file 10's EXACT assignment function directly
    response = call_llm(prompt_version_a if group == "A" else prompt_version_b)
    log_ab_result(user_id, group, response, user_satisfaction=None)      # collected later
    return response
```

Genuinely worth recognizing this as the IDENTICAL technique from MLOps file 10, applied to comparing DIFFERENT prompt versions or RAG configurations rather than different model VERSIONS — the same statistical framework (Math Foundations file 14's hypothesis testing, recap directly) applies to determine if an observed difference in user satisfaction is genuinely significant or just noise.

## Regression testing for prompts — genuinely important, an LLM-specific CI/CD concern

```python
# Recap MLOps file 08's model-quality-gate discussion directly -- applied
# to PROMPTS instead of trained models
test_cases = [
    {"input": "What's your return policy?", "must_contain": ["30 days"]},
    {"input": "How much is premium?", "must_contain": ["$9.99"]}
]

def test_prompt_regression(prompt_template):
    failures = []
    for case in test_cases:
        response = call_llm(prompt_template.format(question=case["input"]))
        if not all(phrase in response for phrase in case["must_contain"]):
            failures.append(case)
    return failures
```

Genuinely worth recognizing this as directly recapping MLOps file 08's model-evaluation-gate discipline, now applied to PROMPT changes — before deploying a MODIFIED prompt/RAG configuration, running it against a fixed set of test cases (genuinely similar to Flask file 08's pytest suite) catches REGRESSIONS — cases that used to work correctly but broke due to the change, exactly the same "automated safety net before deployment" philosophy from that entire MLOps discussion.

## Try this

```python
# Take the RAG system from file 07's exercise and build a small evaluation
# test set: 10 questions with KNOWN correct answers/expected source documents
# Implement retrieval evaluation (recall@k, recap this file's opening
# example) and an LLM-as-judge evaluation for the GENERATED answers
# (rating accuracy/completeness on a 1-5 scale)
# Make ONE change to your chunking strategy (recap file 07) and re-run
# the FULL evaluation suite -- confirm you can honestly, quantitatively
# state whether the change improved or degraded performance, rather than
# relying on a vague impression from checking a couple of examples manually
```

Being able to honestly quantify whether a change helped or hurt — rather than relying on gut feeling from spot-checking a few examples — is genuinely the entire point of this file, and the real, practical skill that separates rigorous LLM application development from guesswork.

## What's next
File 12 — Responsible AI & LLM Safety, covering the genuinely important safety, bias, and security considerations that apply across every system built throughout this entire folder.
