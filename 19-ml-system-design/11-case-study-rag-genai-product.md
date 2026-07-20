# 11 — Case Study: RAG/GenAI Product Design

Recap file 10's closing note — applying the framework to a genuinely current, cutting-edge system type, pulling together the ENTIRE Frontier GenAI folder into one complete case study. Genuinely the most directly relevant case study given this roadmap's most recent, cutting-edge phase.

## Step 1: Clarify requirements — recap file 02's framework, now for a GenAI product

```
Prompt: "Design an AI-powered customer support assistant for a SaaS company"

Genuinely important clarifying questions, recap file 02 directly:
- "What's the knowledge base -- product docs, past support tickets,
  both?" (recap Frontier GenAI file 05's vector-database discussion directly)
- Latency: "Users expect a response within a few seconds" (recap
  Frontier GenAI file 04's streaming discussion directly as a genuinely
  important UX mitigation)
- "Should it be ABLE to take actions (e.g. issue a refund), or purely
  answer questions?" -> determines whether AGENTS (Frontier GenAI file
  09) are genuinely needed, or RAG alone (files 06-07) suffices
- "What's the acceptable HALLUCINATION rate?" (recap Frontier GenAI file
  01's honest limitation discussion directly -- genuinely worth
  establishing this as an explicit, measurable requirement, not left
  implicit)
```

## Step 2: High-level architecture — recap Frontier GenAI file 13's complete system list directly

```
Recap Frontier GenAI file 13's bridge-file system diagram DIRECTLY, now
applied concretely:
User query -> Query understanding (recap Frontier GenAI file 07's query
  transformation discussion) -> Retrieval from knowledge base (recap
  Frontier GenAI files 05-07) -> [If action needed: Agent/tool-use, recap
  Frontier GenAI file 09] -> Generation (recap Frontier GenAI file 04) ->
  Response (streamed, recap Frontier GenAI file 04's streaming discussion)
```

## Step 3: Knowledge base design — recap Frontier GenAI files 05-07 directly, now concretely applied

```
Genuinely worth proposing CONCRETE decisions, recap Frontier GenAI file
07's chunking discussion directly: semantic chunking of product docs and
past support tickets, with METADATA (recap Frontier GenAI file 05's
metadata-filtering discussion directly) tagging each chunk by product
area, so retrieval can be FILTERED by the user's specific product/plan,
combined with hybrid search (recap Frontier GenAI file 07 directly) for
handling exact error codes/product names alongside conceptual queries
```

## Step 4: RAG vs Fine-tuning vs Agents — recap Frontier GenAI files 08/13's decision framework directly

```
Genuinely worth an honest, explicit justification: "The knowledge base
(product docs, ticket history) CHANGES frequently as the product
evolves, so RAG (recap Frontier GenAI files 06-07 directly) is the right
foundation, rather than fine-tuning facts INTO the model (recap Frontier
GenAI file 08's honest 'RAG for facts, fine-tuning for behavior'
distinction directly). If the assistant needs to take ACTIONS (issue
refunds, update account settings), an AGENT layer (recap Frontier GenAI
file 09) sits on top, using RAG's retrieved context to INFORM its
decisions but genuinely EXECUTING real actions through defined tools"
```

## Step 5: Handling the honest "I don't know" case — recap Frontier GenAI file 06's core lesson directly

```
Genuinely worth recapping Frontier GenAI file 06's EXACT grounding
discipline here as a CORE design requirement, not an afterthought:
explicit prompt instructions to acknowledge when retrieved context is
insufficient (recap that file's "say so honestly" example directly), a
similarity-threshold check (recap that file's robust-retrieval example
directly) BEFORE attempting generation, and a clear ESCALATION path to
a human support agent when the system genuinely can't help
```

## Step 6: Model choice — recap Frontier GenAI file 01's landscape discussion directly

```
Genuinely worth an honest, reasoned choice, recap Frontier GenAI file
01's closed-vs-open-weight discussion directly: "Given this handles
potentially sensitive customer data (recap Frontier GenAI file 12's
privacy discussion directly), I'd evaluate whether a CLOSED-weight API
model's data-handling policies meet compliance needs, or whether an
OPEN-weight, SELF-HOSTED model (recap Cloud file 06's SageMaker/Cloud
file 09's private-networking discussion directly) is genuinely necessary
to keep data within our own infrastructure"
```

## Step 7: Evaluation — recap Frontier GenAI file 11's ENTIRE evaluation discussion directly

```
Genuinely worth recapping this in full: retrieval evaluation (recall@k,
recap Frontier GenAI file 11's opening example), RAGAS-style faithfulness/
relevancy scoring (recap that file's RAGAS discussion), and human
evaluation for a genuinely representative SAMPLE of real conversations
(recap that file's "human eval is the honest gold standard" framing directly)
ONLINE: recap MLOps file 10's A/B testing discussion directly --
  deflection rate (conversations resolved WITHOUT human escalation) as a
  genuinely concrete, measurable business-relevant online metric
```

## Step 8: Safety and monitoring — recap Frontier GenAI file 12's ENTIRE checklist directly

```
Genuinely worth recapping Frontier GenAI file 12's complete responsible-
AI checklist here, applied concretely: prompt injection defenses (recap
that file's delimiter/explicit-instruction discussion, genuinely
important since customer input is UNTRUSTED, recap Flask file 04's
"never trust client input" principle directly, now for LLM prompts),
human confirmation before any agent action with real consequences (recap
Frontier GenAI file 09's confirmation discussion directly, applied to
"issue a refund" specifically), and ongoing hallucination-rate monitoring
(recap Frontier GenAI file 11's faithfulness-evaluation discussion,
tracked continuously in production, not just at launch)
```

## Step 9: Cost considerations — recap Cloud file 10 and Frontier GenAI file 01's token-cost discussion directly

```
Genuinely worth an explicit, honest cost discussion: LLM API costs scale
with USAGE (recap Frontier GenAI file 01/Cloud file 12's LLM-cost
discussion directly) in a way traditional infrastructure costs (recap
Cloud file 10's compute-cost discussion) don't quite mirror -- worth
proposing PROMPT-length optimization (recap Frontier GenAI file 02's
concise-prompting discipline, retrieving only genuinely necessary
context rather than maximal context) as a real, direct cost lever
```

## The honest, complete tradeoffs to articulate

```
- RAG freshness vs fine-tuning behavioral consistency (recap Frontier
  GenAI file 08/13's decision framework directly)
- Closed-weight capability vs open-weight control/privacy (recap Step 6 directly)
- Full automation vs human-in-the-loop safety (recap Step 8 directly)
- Retrieval context size vs cost/latency (recap Step 9 and Frontier
  GenAI file 01's context-window discussion directly)
```

## Try this

```
# Genuinely worth practicing this complete case study, then explicitly
# articulating how MANY distinct folders from this ENTIRE roadmap it
# draws on directly -- Frontier GenAI (obviously), but also Flask
# (security/input-validation), MLOps (A/B testing/monitoring), Cloud
# (privacy/hosting decisions), Math Foundations (the cosine-similarity
# foundation underlying retrieval) -- write down the FULL list, honestly
```

Explicitly cataloging how many folders this ONE case study genuinely draws on is a powerful, concrete demonstration of just how INTEGRATED this entire roadmap's knowledge base has become — genuinely worth doing as a confidence-building exercise before interviews.

## What's next
File 12 — Scaling, Reliability & Trade-off Communication, covering the cross-cutting concerns that apply across EVERY case study in this folder — genuinely essential for handling an interviewer's "but what if..." follow-up questions.
