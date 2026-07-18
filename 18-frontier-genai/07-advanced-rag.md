# 07 — Advanced RAG

Recap file 06's closing note directly — covering the genuinely important refinements flagged in that file's honest limitations section: chunking strategies, re-ranking, and hybrid search, moving from "a working RAG system" to "a genuinely production-quality one."

## Chunking strategies — recap file 05's simple preview, now properly treated

```python
# Recap file 05's naive fixed-size chunking directly:
def simple_chunk(text, chunk_size=500, overlap=50):
    return [text[i:i+chunk_size] for i in range(0, len(text), chunk_size-overlap)]
```

Genuinely worth understanding the REAL problem with this naive approach — recap DSA file 13's sliding window discussion directly: a FIXED character count genuinely doesn't respect natural boundaries (sentences, paragraphs) — it can genuinely cut a sentence in half, splitting a coherent thought across two separate, less-meaningful chunks.

```python
# Recursive/semantic chunking -- genuinely respects natural text boundaries
def semantic_chunk(text, chunk_size=500):
    paragraphs = text.split('\n\n')      # genuinely try paragraph boundaries FIRST
    chunks = []
    current_chunk = ""

    for para in paragraphs:
        if len(current_chunk) + len(para) < chunk_size:
            current_chunk += para + "\n\n"
        else:
            if current_chunk:
                chunks.append(current_chunk.strip())
            current_chunk = para + "\n\n"
    if current_chunk:
        chunks.append(current_chunk.strip())
    return chunks
```

Genuinely worth recognizing this as a real, meaningful quality improvement — respecting PARAGRAPH boundaries (falling back to sentence boundaries if a single paragraph is too long) genuinely keeps each chunk as a coherent, self-contained unit of meaning, directly improving both embedding quality (recap file 05 directly — a coherent chunk produces a more meaningful embedding vector) and generation quality (a complete, uncut thought is more useful as retrieved context).

## Chunk size tradeoffs — genuinely an important, real design decision

```
Genuinely SMALL chunks: more PRECISE retrieval (a smaller chunk is more
  narrowly focused, recap file 05's similarity-search precision directly),
  but genuinely LESS context per chunk -- might miss surrounding
  information needed for a complete answer

Genuinely LARGE chunks: more CONTEXT per chunk, but genuinely LESS
  precise retrieval (recap file 01's context-window/token-cost discussion
  directly -- also genuinely more expensive per retrieved chunk)
```

Genuinely worth recognizing this as directly analogous to Math Foundations file 12's bias-variance tradeoff — genuinely no single "correct" chunk size; the right choice depends on the SPECIFIC knowledge base and query patterns, and is genuinely something to empirically TUNE (recap XGBoost file 08's hyperparameter-tuning discipline directly) rather than guess once and never revisit.

## Overlapping chunks — recap file 05's preview, now with clear reasoning

```python
# Genuinely important reasoning: overlap ensures information NEAR a chunk
# boundary doesn't get "lost" by being split awkwardly between two
# non-overlapping chunks -- a piece of context appearing at the very END
# of one chunk ALSO appears at the START of the next
```

## Re-ranking — genuinely important, a two-stage retrieval refinement

```python
def rag_with_reranking(user_question, initial_k=20, final_k=3):
    # STAGE 1: cast a WIDE net -- retrieve MANY candidates cheaply (recap file 05's vector search)
    results = collection.query(query_texts=[user_question], n_results=initial_k)

    # STAGE 2: RE-RANK using a more expensive, more accurate model
    from sentence_transformers import CrossEncoder      # genuinely a different model TYPE than the embedding model
    reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')

    pairs = [[user_question, doc] for doc in results['documents'][0]]
    scores = reranker.predict(pairs)      # genuinely scores EACH (query, document) pair directly, jointly

    ranked_indices = np.argsort(scores)[::-1][:final_k]      # recap DSA file 11's argsort-based
                                                                  # top-k pattern directly
    top_documents = [results['documents'][0][i] for i in ranked_indices]
    return top_documents
```

Genuinely worth understanding WHY this two-stage approach matters, connecting directly to Cloud file 10's cost-optimization discipline and DSA file 08's Top-K pattern — recap file 05's embedding-based similarity search directly: it's genuinely FAST (comparing pre-computed vectors, recap DSA file 12's binary-search-speed discussion) but LESS ACCURATE, since it scores query and document INDEPENDENTLY. A cross-encoder re-ranker scores the query and document TOGETHER, jointly, genuinely capturing more nuanced relevance signals — but is significantly SLOWER, making it impractical to run against an ENTIRE knowledge base. The two-stage pattern gets the best of both: FAST initial retrieval narrows down to a reasonable candidate set (`initial_k`), then the SLOWER, more ACCURATE re-ranker only needs to evaluate that smaller set.

## Hybrid search — combining semantic and keyword matching

```python
def hybrid_search(query, alpha=0.5):
    # Semantic search (recap file 05 directly)
    semantic_results = collection.query(query_texts=[query], n_results=10)

    # Keyword search (recap Classical NLP file 04's TF-IDF/BM25-style matching directly)
    keyword_results = bm25_search(query, all_documents)      # BM25, genuinely a refined, standard
                                                                  # evolution of TF-IDF's core idea

    # Combine scores -- genuinely a weighted blend
    combined_scores = alpha * semantic_scores + (1 - alpha) * keyword_scores
    return top_results_from(combined_scores)
```

Genuinely worth understanding WHY hybrid search matters, directly recapping Classical NLP file 12's bridge discussion — semantic/embedding search (file 05) is genuinely excellent at capturing MEANING even without exact word overlap, but can genuinely UNDERPERFORM on queries with SPECIFIC, EXACT terms (product codes, proper nouns, technical jargon) that keyword-based search (recap Classical NLP file 04's TF-IDF, or BM25 as its refined successor) handles naturally. Combining both, weighted by `alpha`, genuinely captures the strengths of each approach — a real, practical technique used widely in production RAG systems.

## Query transformation — improving the QUERY before retrieval

```python
def transform_query(original_query):
    # Genuinely useful: have the LLM itself REWRITE a vague query into a
    # more specific, retrieval-friendly one BEFORE searching
    prompt = f"Rewrite this question to be more specific and searchable: {original_query}"
    return call_llm(prompt)
```

Genuinely worth recognizing this as a real, practical application of file 02's prompt-engineering discipline, applied to IMPROVE the retrieval step itself, not just the final generation — a genuinely common real technique for handling vague or conversational user queries that might not retrieve well in their original phrasing.

## HyDE (Hypothetical Document Embeddings) — genuinely a clever, advanced technique

```python
def hyde_retrieval(user_question):
    # Genuinely clever idea: have the LLM GENERATE a hypothetical answer
    # FIRST, then embed THAT (rather than the original question) for retrieval
    hypothetical_answer = call_llm(f"Write a short, plausible answer to: {user_question}")
    results = collection.query(query_texts=[hypothetical_answer], n_results=5)
    return results
```

Genuinely worth understanding the clever reasoning here — recap file 05's embedding-similarity discussion directly: a QUESTION and its ANSWER are often semantically quite DIFFERENT in phrasing/structure, even though they're clearly related. A hypothetical, LLM-generated answer (even if not perfectly accurate) tends to be MORE similar in phrasing/structure to the REAL answer documents in the knowledge base than the original question was, genuinely often improving retrieval quality, at the cost of one extra LLM call.

## Try this

```python
# Take the RAG system from file 06's exercise and upgrade it with:
# 1. Semantic (paragraph-respecting) chunking instead of fixed-size chunking
# 2. A two-stage retrieve-then-rerank pipeline using a CrossEncoder
# 3. A simple hybrid search combining your vector search with a basic
#    keyword-overlap score
# Compare retrieval quality on a genuinely tricky query containing a
# SPECIFIC term (e.g. a product name or code) against the file 06 baseline
# -- confirm the hybrid approach retrieves it more reliably than semantic
# search alone
```

Testing specifically with a query containing an exact, specific term is genuinely the clearest way to see hybrid search's real advantage over pure semantic search — a case where keyword matching genuinely adds value that embedding similarity alone might miss.

## What's next
File 08 — LoRA/PEFT/QLoRA Deep Dive, moving from RETRIEVAL-based grounding (RAG) into WEIGHT-based customization — the full, proper treatment of efficient fine-tuning previewed in Hugging Face file 12.
