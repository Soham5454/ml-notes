# 06 — Building RAG Systems from Scratch

Recap file 05's closing note and Hugging Face file 13's original RAG preview directly — combining file 05's retrieval infrastructure with file 04's LLM API calls into a complete, working system. Genuinely the payoff moment this entire folder's first half has been building toward.

## RAG, precisely defined — recap Hugging Face file 13's exact framing directly

```
Retrieval-Augmented Generation genuinely combines TWO steps:
1. RETRIEVAL: given a user's query, find relevant documents/chunks
   (recap file 05's vector similarity search directly)
2. GENERATION: feed those retrieved documents to an LLM (file 04) AS
   CONTEXT, so it generates an answer GROUNDED in real, verifiable text
   rather than purely from its own training data (recap file 01's
   hallucination discussion directly)
```

## A complete, minimal RAG pipeline

```python
import chromadb      # recap file 05 directly
import anthropic      # recap file 04 directly

# STEP 1: Set up the knowledge base (recap file 05's collection setup)
client = chromadb.Client()
collection = client.create_collection(name="company_docs")
collection.add(
    documents=["Our return policy allows returns within 30 days of purchase with a receipt.",
               "Shipping typically takes 3-5 business days within the continental US.",
               "Our premium subscription costs $9.99/month and includes priority support."],
    ids=["doc1", "doc2", "doc3"]
)

llm_client = anthropic.Anthropic()

def rag_query(user_question):
    # STEP 2: RETRIEVE relevant documents (recap file 05's query() directly)
    results = collection.query(query_texts=[user_question], n_results=2)
    retrieved_context = "\n".join(results['documents'][0])

    # STEP 3: GENERATE an answer, grounded in the retrieved context
    prompt = f"""
    Answer the question using ONLY the information provided below.
    If the answer isn't in the provided information, say so honestly.

    Context:
    {retrieved_context}

    Question: {user_question}
    """

    response = llm_client.messages.create(
        model="claude-sonnet-4-5", max_tokens=500,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text

print(rag_query("How long do I have to return an item?"))
```

Genuinely worth recognizing this as directly combining file 04's `messages.create()` call and file 05's `collection.query()` call, with ONE additional, genuinely critical instruction: "answer using ONLY the provided information... if not present, say so honestly" — this single instruction is precisely what shifts the model from potentially HALLUCINATING (recap file 01's limitation directly) toward being genuinely GROUNDED in verifiable, retrieved text.

## Why "say so honestly" matters — recap file 01's hallucination discussion directly

```python
# WITHOUT this instruction, a genuinely common failure mode:
# user asks something NOT covered in the retrieved docs, and the model
# confidently FABRICATES a plausible-sounding but genuinely WRONG answer,
# drawing on its general training knowledge instead of the actual context
```

Genuinely worth internalizing this as one of the most important, practical RAG-prompt-design lessons — explicitly instructing the model to ACKNOWLEDGE when the context doesn't contain an answer is a real, meaningful defense against hallucination, though genuinely not a perfect, guaranteed one (file 11 covers proper evaluation of how well this actually works).

## Including source citations — genuinely important for trustworthiness

```python
def rag_query_with_citations(user_question):
    results = collection.query(query_texts=[user_question], n_results=2)
    context_with_ids = "\n".join(
        f"[Source {id}]: {doc}" for id, doc in zip(results['ids'][0], results['documents'][0])
    )

    prompt = f"""
    Answer using ONLY the context below. Cite the source ID(s) you used.

    Context:
    {context_with_ids}

    Question: {user_question}
    """
    response = llm_client.messages.create(model="claude-sonnet-4-5", max_tokens=500,
                                             messages=[{"role": "user", "content": prompt}])
    return response.content[0].text
```

Genuinely worth connecting this to the general "trust but verify" theme from throughout this roadmap — recap Classical NLP file 10's honest "metrics are proxies, not ground truth" framing directly: citations let a human user (or an automated evaluation, file 11) actually VERIFY the answer traces back to real, specific source material, rather than trusting the LLM's claim blindly.

## Handling "no relevant documents found" — genuinely an important edge case

```python
def rag_query_robust(user_question, similarity_threshold=0.3):
    results = collection.query(query_texts=[user_question], n_results=2, include=["distances"])

    if results['distances'][0][0] > similarity_threshold:      # genuinely important -- distance,
                                                                     # not similarity -- SMALLER means closer
        return "I don't have information relevant to that question in my knowledge base."

    # ... proceed with normal RAG generation ...
```

Genuinely important, real practical detail worth flagging directly: even the BEST-matching retrieved document might genuinely NOT be relevant enough to actually answer the question — recap file 05's cosine similarity discussion directly, most vector databases return a DISTANCE metric (where smaller is more similar, the inverse of similarity) — checking this against a threshold BEFORE even attempting generation is a real, practical safeguard against confidently answering from irrelevant retrieved context.

## Multi-document synthesis — genuinely a more advanced, realistic use case

```python
def rag_query_multi_doc(user_question, n_results=5):
    results = collection.query(query_texts=[user_question], n_results=n_results)      # genuinely retrieve MORE documents
    context = "\n\n".join(f"Document {i+1}: {doc}" for i, doc in enumerate(results['documents'][0]))

    prompt = f"""
    Synthesize an answer using information from the following documents.
    Note any CONTRADICTIONS between documents if they exist.

    {context}

    Question: {user_question}
    """
    # ... same generation call ...
```

Genuinely worth knowing this represents a REAL, common challenge beyond the simple single-best-match retrieval shown earlier — real knowledge bases genuinely contain overlapping, sometimes CONTRADICTORY information across multiple documents (recap the general "data quality" concerns from MLOps file 08's data-validation discussion directly, now applied to knowledge-base content), and a genuinely robust RAG system needs to handle synthesizing across MULTIPLE sources, not just picking the single closest match.

## A complete RAG system connected to Flask — recap Flask folder directly

```python
from flask import Flask, request, jsonify      # recap Flask file 07's REST API discipline directly

app = Flask(__name__)

@app.route('/api/v1/ask', methods=['POST'])
def ask():
    data = request.get_json()
    question = data.get('question')

    if not question:      # recap Flask file 07's input-validation discipline directly
        return jsonify({"error": "Missing 'question' field"}), 400

    answer = rag_query_with_citations(question)
    return jsonify({"answer": answer}), 200
```

Genuinely satisfying to see this — a full RAG system, wrapped in the EXACT SAME Flask REST API pattern from that entire folder, directly connecting this brand-new Frontier GenAI capability to infrastructure already deeply understood.

## The honest, real limitations of this basic approach — a preview of file 07

```
Genuinely worth an honest note directly: this file's approach is a GENUINE,
working RAG system, but real production systems need MORE sophistication:
better CHUNKING strategies (file 07), RE-RANKING retrieved results before
generation (file 07), and HYBRID search combining semantic + keyword
matching (file 07) -- this file covers the CORE pattern; file 07 covers
the refinements that matter for genuinely production-quality results
```

## Try this

```python
# Build a complete RAG system over a small, genuinely real knowledge base
# (e.g. 15-20 chunks from a document you actually have -- a project README,
# a set of FAQs, or notes from earlier in this repo)
# Implement: retrieval with a similarity threshold check, citation
# inclusion, and honest "I don't know" fallback for out-of-scope questions
# Test it with 3 genuinely IN-SCOPE questions, 2 genuinely OUT-OF-SCOPE
# questions (should trigger the "don't know" response), and 1 question
# that's PARTIALLY answerable (some relevant info exists, but not complete)
# -- honestly evaluate how the system handles each case
```

Testing the out-of-scope and partially-answerable cases explicitly (not just easy, well-covered questions) is genuinely the real test of whether this RAG system is actually grounded and honest, rather than just working on the easiest possible examples.

## What's next
File 07 — Advanced RAG, covering the genuinely important refinements flagged in this file's honest limitations section: chunking strategies, re-ranking, and hybrid search.
