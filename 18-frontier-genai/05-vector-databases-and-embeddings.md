# 05 — Vector Databases & Embeddings Deep Dive

Recap file 04's closing note and Cloud file 12's vector-database preview directly — covering the retrieval infrastructure that directly addresses file 01's hallucination/knowledge-cutoff limitations, genuinely the foundation for file 06's RAG systems.

## Embeddings, recapped fully — recap Hugging Face file 06 and Classical NLP file 04/06 directly

```python
# Recap Hugging Face file 13's bridge discussion DIRECTLY: contextual
# embeddings (unlike Classical NLP file 06's fixed Word2Vec vectors) capture
# GENUINE semantic meaning, dynamically shaped by context (recap PyTorch
# file 15's attention mechanism directly)
from openai import OpenAI      # genuinely any embedding provider works structurally similarly

client = OpenAI()
response = client.embeddings.create(model="text-embedding-3-small", input="The cat sat on the mat")
embedding_vector = response.data[0].embedding
print(len(embedding_vector))   # genuinely a fixed-size dense vector, e.g. 1536 dimensions
```

Genuinely worth recognizing this as PRECISELY Hugging Face file 06's contextual embedding concept, now via a dedicated EMBEDDING model API (rather than extracting hidden states from a general-purpose language model directly) — these specialized embedding models are genuinely optimized SPECIFICALLY for producing vectors good at SIMILARITY comparison, the exact use case this whole file builds on.

## Cosine similarity, one more time — recap Math Foundations file 01 and Classical NLP file 04 directly

```python
import numpy as np

def cosine_similarity(a, b):      # recap Math Foundations file 01's EXACT formula, genuinely unchanged
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

query_embedding = get_embedding("What is a neural network?")
doc_embedding = get_embedding("Neural networks are computing systems inspired by biological brains")

similarity = cosine_similarity(query_embedding, doc_embedding)
```

Genuinely worth recognizing this as the SAME formula that has now appeared in Math Foundations file 01 (the original derivation), Classical NLP file 04 (TF-IDF document similarity), Hugging Face file 06 (word embedding similarity), and OpenCV file 08 (feature descriptor matching) — a genuinely remarkable, real demonstration of one foundational concept underlying an enormous range of applications across this entire roadmap.

## Why a SPECIALIZED vector database, not just NumPy arrays — genuinely a real, practical scale problem

```python
# Recap DSA file 01's Big O discussion directly -- a NAIVE approach:
similarities = [cosine_similarity(query_embedding, doc) for doc in all_document_embeddings]
best_match = np.argmax(similarities)
# Genuinely O(n) per query, where n = number of stored documents
# For a FEW hundred documents, genuinely fine
# For MILLIONS of documents, genuinely far too slow for real-time queries
```

Genuinely the core, real motivation for vector databases — recap DSA file 12's binary search and file 08's heap-based efficient-search discussion directly: vector databases use specialized indexing structures (Approximate Nearest Neighbor algorithms like HNSW — Hierarchical Navigable Small World graphs, genuinely related in spirit to DSA file 07's tree-based search efficiency, though a more complex, specialized structure) to find APPROXIMATE nearest neighbors in sub-linear time, trading a small amount of ACCURACY (genuinely might miss the absolute best match occasionally) for a MASSIVE speed improvement at real scale.

## Using a vector database — recap SQL folder's CRUD operations directly, now for vectors

```python
import chromadb      # genuinely a popular, simple-to-start vector database

client = chromadb.Client()
collection = client.create_collection(name="documents")      # recap SQL file 09's CREATE TABLE directly

collection.add(
    documents=["Neural networks are inspired by biological brains", "Attention mechanisms let models focus on relevant parts of input"],
    ids=["doc1", "doc2"]
)      # recap SQL file 10's INSERT directly -- genuinely handles embedding generation internally in this case

results = collection.query(query_texts=["How do neural networks work?"], n_results=2)      # recap SQL
                                                                                                # file 02's SELECT/WHERE
                                                                                                # and file 12's ORDER BY
                                                                                                # LIMIT combined, conceptually
print(results['documents'])
```

Genuinely worth recognizing this API structure as directly parallel to the SQL folder's CRUD discipline (recap that whole folder) — `add()` is genuinely INSERT, `query()` is genuinely SELECT ... ORDER BY similarity LIMIT n, just with SIMILARITY as the sorting criterion instead of a SQL column.

## Metadata filtering — recap SQL file 02's WHERE clause, combined with vector search

```python
collection.add(
    documents=["Neural network basics", "Advanced transformer architectures"],
    metadatas=[{"difficulty": "beginner"}, {"difficulty": "advanced"}],
    ids=["doc1", "doc2"]
)

results = collection.query(
    query_texts=["explain attention"],
    n_results=5,
    where={"difficulty": "beginner"}      # recap SQL file 02's WHERE directly -- combines EXACT filtering
                                              # with APPROXIMATE similarity search
)
```

Genuinely important, real practical technique worth understanding — recap SQL file 02's `WHERE` clause directly: real RAG systems often need BOTH exact filtering (only documents from a specific category, date range, or permission level) AND similarity ranking simultaneously — this is genuinely called "hybrid" or "filtered" vector search, and virtually every production vector database supports combining both.

## Choosing a vector database — genuinely the honest, practical landscape

```
Chroma -- genuinely simple, good for learning/prototyping, can run
  fully locally
Pinecone -- genuinely a popular managed/hosted option, recap Cloud file 12's
  managed-vs-self-managed framework directly
Weaviate -- genuinely open-source, self-hostable, feature-rich
pgvector -- genuinely a PostgreSQL EXTENSION adding vector search directly
  to RDS/Cloud SQL (recap Cloud files 05/07 directly) -- worth knowing
  this exists specifically because it lets vector search live ALONGSIDE
  regular relational data (recap the ENTIRE SQL folder) in the SAME database,
  avoiding the need for a genuinely separate system
FAISS -- genuinely Facebook's library, not a full "database" but a
  powerful, efficient similarity-search LIBRARY, good for embedding
  directly into an application without a separate service
```

Genuinely worth recognizing `pgvector` as a particularly satisfying, direct connection back to the SQL folder — it's genuinely possible to add vector similarity search as JUST ANOTHER COLUMN TYPE in a normal PostgreSQL table, combining SQL file 05's joins, file 04's aggregations, and this file's similarity search all in ONE query, rather than needing a completely separate system.

## Chunking — a genuinely critical preprocessing step, full treatment in file 07

```python
# Genuinely a real, important problem: documents are often LONGER than
# what fits well in one embedding, and RAG (file 06) needs to retrieve
# RELEVANT PIECES, not entire documents
def simple_chunk(text, chunk_size=500, overlap=50):
    chunks = []
    for i in range(0, len(text), chunk_size - overlap):      # recap DSA file 13's sliding window technique directly
        chunks.append(text[i:i + chunk_size])
    return chunks
```

Genuinely worth recognizing this preview as directly recapping DSA file 13's sliding window pattern — splitting a long document into overlapping CHUNKS, each embedded and stored separately, is genuinely necessary for effective retrieval; file 07 covers the full, more sophisticated chunking strategies properly.

## Embedding model choice — genuinely matters, a real practical decision

```
Genuinely worth knowing different embedding models exist with different
tradeoffs: dimensionality (larger genuinely captures more nuance, but
costs more storage/compute for similarity search, recap DSA file 01's Big
O discussion), domain specialization (some models are genuinely better
for code, others for general text), and genuinely real cost differences
between providers (recap Cloud file 10's cost-management discipline directly)
```

## Try this

```python
# Using Chroma (or any simple vector database), create a small collection
# of 10-15 short documents on a topic of your choice
# Query it with several DIFFERENT phrasings of the SAME underlying question
# and confirm the retrieval finds genuinely relevant documents even when
# the query doesn't share exact words with the documents (this is genuinely
# the whole point -- semantic similarity, not keyword matching, recap
# Classical NLP file 04's TF-IDF limitation this directly overcomes)
# Add metadata to your documents (e.g. a category field) and combine
# metadata filtering with similarity search in one query
```

Confirming retrieval works even WITHOUT exact word overlap between query and documents is genuinely the clearest proof that semantic embedding search is doing something meaningfully different from Classical NLP file 03/04's keyword-based Bag of Words/TF-IDF approaches.

## What's next
File 06 — Building RAG Systems from Scratch, combining this file's retrieval infrastructure with file 04's LLM API calls into a complete, working Retrieval-Augmented Generation system.
