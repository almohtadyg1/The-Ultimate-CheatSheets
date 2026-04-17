# RAG & Vector Databases: A Complete Progressive Tutorial

---

## 1. What & Why

Retrieval-Augmented Generation (RAG) is an architecture that combines search over your private data with an LLM's generative capability to produce answers grounded in specific knowledge rather than training data alone.

The core problem: LLMs are trained on public data up to a cutoff date. They cannot answer questions about your internal documentation, your company's policies, your codebase, or yesterday's news. You could fine-tune a model on your data, but fine-tuning is expensive, slow to update, and bakes facts into weights that you can't cite. You could stuff documents directly into the context window, but that gets expensive and hits length limits quickly.

RAG solves this cleanly: search for the relevant parts of your knowledge base, inject them into the prompt, and let the model synthesize an answer. When the data changes, you reindex. The model stays general; your data stays current.

Where RAG excels: question answering over internal documents, knowledge bases, support ticket systems, codebases, research corpora. Where it doesn't: tasks requiring deep domain knowledge baked into the model's behavior (use fine-tuning), or when the dataset is tiny enough to fit in the prompt (just put it in the system prompt).

---

## 2. Mental Model

```
INDEXING (run once per document update):

  Document
  "Our refund policy allows returns within 30 days..."
       │
       ▼
  Chunking: split into overlapping segments
  ["Our refund policy allows returns", "returns within 30 days of purchase", ...]
       │
       ▼
  Embedding model: text → dense vector (e.g., 1536 dimensions)
  [0.023, -0.145, 0.872, ..., -0.031]   ← captures semantic meaning
       │
       ▼
  Vector database: stores (vector, original_text, metadata)

QUERYING (runs per user request):

  User: "Can I return something after 3 weeks?"
       │
       ▼
  Same embedding model → query vector
       │
       ▼
  Vector similarity search: find the k vectors closest to the query
  (cosine similarity or dot product distance)
       │
       ▼
  Top-k chunks retrieved: "returns within 30 days of purchase..."
       │
       ▼
  LLM prompt: [system] + [retrieved chunks as context] + [user question]
       │
       ▼
  Answer: "Yes, since 3 weeks = 21 days which is within the 30-day window..."
```

The quality of a RAG system depends on: chunk quality, embedding model quality, retrieval precision, and how well the prompt uses the retrieved context.

---

## 3. Progressive Examples

### Level 1: Vector Embeddings Explained

```python
from openai import OpenAI
import numpy as np

client = OpenAI()

def embed(text: str) -> list[float]:
    """Convert text to a dense vector using OpenAI's embedding model."""
    response = client.embeddings.create(
        model="text-embedding-3-small",   # 1536 dimensions, fast, cheap
        input=text,
    )
    return response.data[0].embedding

def cosine_similarity(a: list[float], b: list[float]) -> float:
    """Measure how similar two vectors are. 1.0 = identical, 0.0 = unrelated."""
    a, b = np.array(a), np.array(b)
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

# Embeddings capture meaning, not just keywords
texts = [
    "Returns are accepted within 30 days of purchase.",     # semantically relevant
    "You may exchange items up to one month after buying.", # same meaning, different words
    "Our office hours are Monday through Friday 9-5.",     # different topic
]

query = "What is the return window?"
query_vec = embed(query)

for text in texts:
    sim = cosine_similarity(query_vec, embed(text))
    print(f"Similarity: {sim:.3f} | {text[:60]}")

# Output (approximate):
# Similarity: 0.891 | Returns are accepted within 30 days of purchase.
# Similarity: 0.873 | You may exchange items up to one month after buying.
# Similarity: 0.312 | Our office hours are Monday through Friday 9-5.
```

### Level 2: Chunking Strategies

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
import re

# Chunking: split documents into pieces small enough to embed and retrieve,
# but large enough to contain meaningful context.

# Key parameters:
# chunk_size:    target size in characters (typically 200-1000)
# chunk_overlap: overlap between consecutive chunks (prevents cutting context)

def naive_chunking(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    """Simple fixed-size chunking — fast but may cut mid-sentence."""
    chunks = []
    start = 0
    while start < len(text):
        end = min(start + chunk_size, len(text))
        chunks.append(text[start:end])
        start += chunk_size - overlap
    return chunks

def sentence_chunking(text: str, max_chunk_size: int = 500) -> list[str]:
    """Split at sentence boundaries — better context preservation."""
    sentences = re.split(r'(?<=[.!?])\s+', text)
    chunks = []
    current = ""
    for sentence in sentences:
        if len(current) + len(sentence) < max_chunk_size:
            current += sentence + " "
        else:
            if current:
                chunks.append(current.strip())
            current = sentence + " "
    if current:
        chunks.append(current.strip())
    return chunks

# LangChain's recursive splitter — the best default choice
def smart_chunking(text: str) -> list[str]:
    """Tries to split on paragraphs, then sentences, then words, then chars."""
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=800,
        chunk_overlap=100,
        separators=["\n\n", "\n", ". ", " ", ""],  # try these in order
    )
    return splitter.split_text(text)

# Metadata attachment — critical for filtering and citation
def chunk_with_metadata(document: dict) -> list[dict]:
    """Chunk a document and attach metadata to each chunk."""
    chunks = smart_chunking(document["content"])
    return [
        {
            "text": chunk,
            "doc_id": document["id"],
            "source": document["filename"],
            "page": document.get("page"),
            "section": document.get("section"),
        }
        for chunk in chunks
    ]
```

### Level 3: Building a Simple Vector Store

```python
import json
import numpy as np
from openai import OpenAI

client = OpenAI()

class SimpleVectorStore:
    """
    In-memory vector store with cosine similarity search.
    Good for datasets up to ~100K documents.
    For larger datasets, use ChromaDB, Pinecone, Weaviate, or pgvector.
    """

    def __init__(self, embedding_model: str = "text-embedding-3-small"):
        self.model = embedding_model
        self.vectors: list[np.ndarray] = []
        self.documents: list[dict] = []

    def _embed(self, texts: list[str]) -> list[np.ndarray]:
        """Batch embed multiple texts in one API call."""
        response = client.embeddings.create(model=self.model, input=texts)
        return [np.array(item.embedding) for item in response.data]

    def add_documents(self, documents: list[dict]):
        """Add documents to the store. Each doc must have a 'text' field."""
        texts = [doc["text"] for doc in documents]
        vectors = self._embed(texts)
        for doc, vec in zip(documents, vectors):
            self.documents.append(doc)
            self.vectors.append(vec)
        print(f"Added {len(documents)} documents. Total: {len(self.documents)}")

    def search(self, query: str, top_k: int = 5) -> list[dict]:
        """Find the top_k most relevant documents for a query."""
        if not self.vectors:
            return []

        query_vec = self._embed([query])[0]
        matrix = np.stack(self.vectors)

        # Cosine similarity: dot product of unit vectors
        norms = np.linalg.norm(matrix, axis=1, keepdims=True)
        normalized = matrix / norms
        query_norm = query_vec / np.linalg.norm(query_vec)
        similarities = normalized @ query_norm

        top_indices = np.argsort(similarities)[::-1][:top_k]
        return [
            {**self.documents[i], "similarity": float(similarities[i])}
            for i in top_indices
            if similarities[i] > 0.3   # filter out very weak matches
        ]

# Usage
store = SimpleVectorStore()

documents = [
    {"text": "Returns are accepted within 30 days of purchase. Items must be unused.", "source": "refund_policy.txt"},
    {"text": "Shipping takes 3-5 business days for standard delivery.", "source": "shipping.txt"},
    {"text": "Premium members receive free shipping on all orders.", "source": "membership.txt"},
    {"text": "To initiate a return, contact support@company.com with your order number.", "source": "refund_policy.txt"},
]

store.add_documents(documents)
results = store.search("How do I return something?", top_k=2)
for r in results:
    print(f"[{r['similarity']:.3f}] ({r['source']}) {r['text'][:80]}...")
```

### Level 4: Production-Quality RAG Pipeline

```python
import anthropic
from typing import Optional

anthropic_client = anthropic.Anthropic()

def build_rag_prompt(query: str, retrieved_chunks: list[dict]) -> str:
    """
    Build a grounded prompt that anchors the model to retrieved facts.
    The model should only use information from the provided context.
    """
    if not retrieved_chunks:
        context = "No relevant documents found."
    else:
        context_parts = []
        for i, chunk in enumerate(retrieved_chunks, 1):
            source = chunk.get("source", "Unknown")
            context_parts.append(f"[{i}] Source: {source}\n{chunk['text']}")
        context = "\n\n".join(context_parts)

    return f"""You are an assistant that answers questions based strictly on the provided context.

INSTRUCTIONS:
- Answer ONLY using information from the context below
- If the answer is not in the context, say: "I don't have information about that in the available documents."
- Cite the source number [1], [2], etc. when you use information
- Do not use outside knowledge or make assumptions

CONTEXT:
{context}

QUESTION: {query}"""

def answer_with_rag(
    query: str,
    store: SimpleVectorStore,
    top_k: int = 3,
    model: str = "claude-sonnet-4-20250514",
) -> dict:
    """Full RAG pipeline: retrieve → augment → generate."""

    # Step 1: Retrieve relevant chunks
    retrieved = store.search(query, top_k=top_k)

    if not retrieved:
        return {
            "answer": "I couldn't find relevant information to answer your question.",
            "sources": [],
            "retrieved_chunks": [],
        }

    # Step 2: Build grounded prompt
    prompt = build_rag_prompt(query, retrieved)

    # Step 3: Generate answer
    message = anthropic_client.messages.create(
        model=model,
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
        temperature=0,   # deterministic for factual Q&A
    )

    return {
        "answer": message.content[0].text,
        "sources": list(set(c.get("source", "") for c in retrieved)),
        "retrieved_chunks": retrieved,
        "tokens_used": {
            "input": message.usage.input_tokens,
            "output": message.usage.output_tokens,
        }
    }

# Usage
result = answer_with_rag("Can I return a product after 45 days?", store)
print(result["answer"])
print(f"Sources: {result['sources']}")
```

### Level 5: ChromaDB — Production Vector Database

```python
# ChromaDB: open-source vector database that runs locally or as a server
# pip install chromadb

import chromadb
from chromadb.utils import embedding_functions

# Local persistent storage
chroma_client = chromadb.PersistentClient(path="./chroma_db")

# Use OpenAI embeddings (or any embedding function)
openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key=os.environ["OPENAI_API_KEY"],
    model_name="text-embedding-3-small"
)

# Collection = a named group of documents (like a table in SQL)
collection = chroma_client.get_or_create_collection(
    name="company_docs",
    embedding_function=openai_ef,
    metadata={"hnsw:space": "cosine"}   # use cosine similarity
)

# Add documents (ChromaDB handles embedding automatically)
documents = [
    "Returns are accepted within 30 days of purchase.",
    "Shipping takes 3-5 business days for standard delivery.",
    "Premium members receive free shipping on all orders.",
]
metadatas = [
    {"source": "refund_policy.txt", "section": "returns"},
    {"source": "shipping.txt", "section": "delivery"},
    {"source": "membership.txt", "section": "benefits"},
]
ids = ["doc_0", "doc_1", "doc_2"]

collection.add(documents=documents, metadatas=metadatas, ids=ids)
print(f"Collection size: {collection.count()}")

# Query with filters
results = collection.query(
    query_texts=["How do I return a product?"],
    n_results=2,
    where={"source": "refund_policy.txt"},   # metadata filter
)

for doc, meta, distance in zip(
    results["documents"][0],
    results["metadatas"][0],
    results["distances"][0]
):
    print(f"Distance: {distance:.3f} | Source: {meta['source']}")
    print(f"  {doc[:80]}")

# Update and delete
collection.update(ids=["doc_0"], documents=["Updated refund policy..."])
collection.delete(ids=["doc_2"])
```

### Level 6: Advanced RAG — Hybrid Search and Reranking

```python
# Hybrid search: combine vector similarity (semantic) with keyword search (BM25)
# Vector alone: finds semantically similar text but may miss exact keywords
# Keyword alone: finds exact matches but misses paraphrases
# Combined: best of both worlds

from rank_bm25 import BM25Okapi   # pip install rank-bm25
import numpy as np

class HybridSearchStore:
    def __init__(self):
        self.documents: list[dict] = []
        self.vectors: list[np.ndarray] = []
        self.bm25 = None
        self._tokenized_corpus = []

    def add_documents(self, documents: list[dict]):
        texts = [doc["text"] for doc in documents]
        self.documents.extend(documents)
        self.vectors.extend(self._embed(texts))

        # Build BM25 index
        self._tokenized_corpus.extend([t.lower().split() for t in texts])
        self.bm25 = BM25Okapi(self._tokenized_corpus)

    def _embed(self, texts):
        response = client.embeddings.create(model="text-embedding-3-small", input=texts)
        return [np.array(item.embedding) for item in response.data]

    def search(self, query: str, top_k: int = 5, alpha: float = 0.7) -> list[dict]:
        """
        Hybrid search: alpha controls the blend.
        alpha=1.0: pure vector search
        alpha=0.0: pure BM25 keyword search
        alpha=0.7: recommended default (favor semantic)
        """
        # Vector scores
        query_vec = self._embed([query])[0]
        matrix = np.stack(self.vectors)
        norms = np.linalg.norm(matrix, axis=1, keepdims=True)
        vector_scores = (matrix / norms) @ (query_vec / np.linalg.norm(query_vec))

        # BM25 keyword scores
        tokenized_query = query.lower().split()
        bm25_scores = np.array(self.bm25.get_scores(tokenized_query))

        # Normalize both to [0, 1]
        def normalize(arr):
            if arr.max() == arr.min():
                return arr
            return (arr - arr.min()) / (arr.max() - arr.min())

        combined = alpha * normalize(vector_scores) + (1 - alpha) * normalize(bm25_scores)
        top_indices = np.argsort(combined)[::-1][:top_k]

        return [
            {**self.documents[i], "score": float(combined[i])}
            for i in top_indices
        ]

# Cross-encoder reranking: more expensive but more accurate
# First retrieve many candidates (top-20), then rerank to top-5
def rerank_with_cross_encoder(query: str, candidates: list[dict], top_k: int = 5) -> list[dict]:
    """
    Use an LLM to score relevance of each candidate to the query.
    More accurate than vector similarity for the final ranking.
    """
    scored = []
    for candidate in candidates:
        response = client.chat.completions.create(
            model="gpt-4o-mini",   # cheap model for scoring
            temperature=0,
            messages=[{
                "role": "user",
                "content": f"""Rate the relevance of this passage to the query on a scale of 1-10.
Query: {query}
Passage: {candidate['text'][:500]}
Respond with only a number 1-10."""
            }]
        )
        try:
            score = int(response.choices[0].message.content.strip())
        except ValueError:
            score = 5
        scored.append({**candidate, "rerank_score": score})

    return sorted(scored, key=lambda x: x["rerank_score"], reverse=True)[:top_k]
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Chunks too small or too large**

```python
# TOO SMALL (50 chars): chunks lack context — "returns within 30 days" is useless
# without knowing what it's talking about.

# TOO LARGE (5000 chars): embeddings become diluted — the dense vector must represent
# too many different topics, so similarity search becomes inaccurate.

# SWEET SPOT: 300-800 characters with 10-20% overlap
# - Large enough to contain meaningful context
# - Small enough for focused embeddings
# - Overlap prevents important context from being split across chunks
```

**Mistake 2: Not filtering by metadata before vector search**

```python
# WRONG: searching all documents when you have product-specific content
results = collection.query(query_texts=["billing issue"], n_results=5)
# May return relevant results from a completely different product line!

# CORRECT: filter by relevant metadata first
results = collection.query(
    query_texts=["billing issue"],
    n_results=5,
    where={"product": "enterprise", "language": "en"}
)
```

**Mistake 3: Hallucination despite RAG**

```
The model can still hallucinate by:
1. Ignoring retrieved context and using training knowledge
2. Misreading or misinterpreting the context
3. Combining context with training knowledge incorrectly

Prevention:
- Include "ONLY use information from the context" in the prompt
- Ask the model to cite specific chunk numbers [1], [2]
- Set temperature=0 for factual Q&A
- Use grounding checks: ask the model to quote the relevant passage before answering
```

**Mistake 4: Embedding the wrong things**

```python
# WRONG: embedding raw HTML or PDF text with lots of formatting characters
text = "**Return Policy**\n\n- Items must be returned within 30 days\n- Original packaging required"
# The embedding will partly represent markdown syntax, not just meaning

# CORRECT: clean text before embedding
import re
def clean_text(text: str) -> str:
    text = re.sub(r'\*+', '', text)           # remove markdown bold/italic
    text = re.sub(r'#+\s+', '', text)          # remove headers
    text = re.sub(r'\s+', ' ', text).strip()   # normalize whitespace
    return text
```

---

## 5. The "Why Does This Work" Layer

### Why Cosine Similarity Works for Semantic Search

An embedding model maps text to a point in high-dimensional space (1536 dimensions for `text-embedding-3-small`). Texts with similar meanings are mapped to nearby points. The embedding model is trained so that "30-day return policy" and "you can return items within a month" end up close together, while "company office hours" ends up far away.

Cosine similarity measures the angle between two vectors rather than the distance. This makes it robust to text length: a short sentence and a long paragraph about the same topic will have similar directional orientation even if their magnitudes differ. Dot product similarity is faster (no normalization needed) and equivalent to cosine when vectors are pre-normalized.

### Why Chunk Overlap Prevents Broken Context

When a document is split into chunks, important information may fall at the boundary of two adjacent chunks:

```
Chunk 1: "...Products can be returned within 30 days of purchase. The item
Chunk 2: must be in its original condition and packaging. Proof of purchase..."
```

Without overlap, a query about "return conditions" might retrieve Chunk 2 — which lacks the critical "within 30 days" detail. With 100-character overlap, both chunks include this bridging sentence, and whichever is retrieved gives the full context.

---

## 6. Quick Reference

### RAG vs Alternatives

| Approach | Best For | Update | Cost |
|----------|---------|--------|------|
| **RAG** | Private facts, frequently updated | Reindex | Per query |
| **Fine-tuning** | Behavior, style, domain language | Re-train | High upfront |
| **Long context** | Tiny datasets, one-off tasks | Always fresh | Per token |
| **System prompt** | <20 items, static FAQ | Manual | Per token |

### Chunking Guidelines

| Parameter | Typical Value | Notes |
|-----------|--------------|-------|
| `chunk_size` | 400-800 chars | Smaller for precise retrieval |
| `chunk_overlap` | 50-150 chars | 10-20% of chunk_size |
| Separator priority | `\n\n` → `\n` → `. ` → ` ` | Try paragraph breaks first |

### Vector Database Comparison

| Database | Best For | Deployment |
|----------|---------|-----------|
| ChromaDB | Local dev, small-medium datasets | Local or server |
| Pinecone | Managed, high scale, minimal ops | Cloud SaaS |
| Weaviate | Multi-modal, hybrid search | Self-hosted or cloud |
| pgvector | Already using PostgreSQL | PostgreSQL extension |
| Qdrant | High performance, Rust-based | Self-hosted or cloud |
| FAISS | Research, pure vector speed | Library (no server) |

### Retrieval Quality Checklist

```
□ Chunks are 300-800 characters with 50-150 char overlap
□ Text is cleaned before chunking (remove HTML, markdown artifacts)
□ Each chunk has metadata: source, section, date, document_id
□ Using hybrid search (vector + BM25) for production
□ Reranking top-20 results to top-5 for critical applications
□ Filtering by metadata before vector search where possible
□ Prompt explicitly says "only use the provided context"
□ Model asked to cite chunk numbers [1], [2]
□ Evaluating retrieval precision separately from generation quality
```
