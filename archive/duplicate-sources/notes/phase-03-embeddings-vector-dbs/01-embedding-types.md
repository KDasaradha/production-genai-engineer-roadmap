# Embedding Types

## 1. Problem Statement

Embedding types solve the problem of representing meaning, keywords, images, code, and domain-specific content in a way machines can compare.

Not every retrieval problem is the same. A user searching "refund policy" may need semantic matching. A user searching "ERR_AUTH_403" needs exact matching. A user searching product images needs visual similarity. Different embedding and retrieval types exist because different search problems need different representations.

Without understanding embedding types, engineers often use dense semantic vectors for everything and then get surprised when product IDs, legal terms, error codes, or exact phrases are missed.

Real-world analogy: a city map, subway map, and weather map all describe the same city, but each is useful for a different task. Embeddings are similar: the representation must match the job.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Embedding types are different ways of representing data as searchable numeric features, such as dense vectors, sparse vectors, or multimodal embeddings. |
| Key terminology | dense vector, sparse vector, hybrid search, dimension, embedding model, semantic similarity, lexical match |
| Simple explanation | Different embeddings capture different kinds of similarity. |
| Mental model | Choose the right "meaning map" for your search problem. |
| Easy example | Dense embeddings match "refund policy" with "return rules"; sparse retrieval matches exact phrase "invoice_id". |
| Use When | Building semantic search, RAG, recommendations, clustering, or multimodal search. |
| Avoid When | A normal SQL filter or exact lookup is enough. |
| Advantages | Enables search beyond exact word matching. |
| Tradeoffs | Requires model choice, evaluation, storage, and versioning. |
| Limitations | No embedding type works perfectly for every use case. |
| Production Example | Enterprise RAG uses dense vectors plus keyword search plus permission metadata. |
| Interview Answer | Dense embeddings capture semantic meaning, sparse vectors capture lexical matching, and hybrid retrieval combines both for stronger search. |

## 3. Intermediate Explanation

Common embedding and retrieval representations:

| Type | What It Captures | Best For | Weakness |
| --- | --- | --- | --- |
| Dense embeddings | semantic meaning | natural language questions, RAG, recommendations | can miss exact IDs and rare terms |
| Sparse vectors | keyword/lexical importance | exact terms, product codes, error IDs | weaker semantic understanding |
| Hybrid retrieval | dense plus sparse | enterprise search, support docs, legal docs | more complex tuning |
| Multimodal embeddings | text/image/audio relationships | image search, visual QA, product catalogues | model and storage complexity |
| Code embeddings | code semantics | code search, coding assistants | language/framework variance |
| Domain embeddings | specialized meaning | medical, legal, finance | dataset and model dependency |

Components:

- embedding model
- tokenizer
- vector dimension
- vector store or search index
- similarity metric
- metadata filters
- evaluation dataset

Data flow:

```text
Input data -> embedding/encoding model -> vector representation -> index -> similarity search
```

Practical examples:

- Customer asks "How do I get my money back?" Dense retrieval finds refund policy.
- Engineer searches "TimeoutError in asyncpg." Hybrid search finds exact error and related docs.
- User searches for a product using an image. Multimodal embeddings find similar products.
- Developer searches "function that validates JWT." Code embeddings find relevant code.

## 4. Advanced Explanation

Embedding type selection should be driven by retrieval failures, not hype.

Optimization techniques:

- Use dense vectors for semantic recall.
- Use sparse retrieval for exact terms.
- Use hybrid search when both meaning and exact terms matter.
- Add metadata filters for tenant, permission, source, date, and document type.
- Use domain-specific models only if general models fail evaluation.
- Store embedding model version with every vector.
- Use reranking after broad retrieval.

Performance considerations:

- Dense vector dimension affects memory and latency.
- Sparse indexes can be efficient for keyword-heavy workloads.
- Hybrid search requires score normalization or rank fusion.
- Multimodal retrieval increases storage and model complexity.
- Re-embedding is expensive after model changes.

Scaling considerations:

- Batch embedding generation.
- Queue ingestion jobs.
- Partition or namespace by tenant.
- Monitor vector index size and query latency.
- Keep source metadata in PostgreSQL and vectors in vector DB.

Production challenges:

- semantic matches that are related but wrong
- exact IDs missed by dense retrieval
- stale embeddings after document changes
- model migrations requiring re-embedding
- inconsistent retrieval quality across domains

## 5. Internal Working

```text
Raw input
  |
  v
Preprocessing and normalization
  |
  v
Embedding or sparse encoding model
  |
  v
Vector/features plus metadata
  |
  v
Index storage
  |
  v
Query encoded the same way
  |
  v
Similarity or lexical matching
  |
  v
Ranked results
```

Detailed lifecycle:

1. Choose representation based on search problem.
2. Prepare source data.
3. Generate dense, sparse, or multimodal representations.
4. Store representation with metadata and model version.
5. Encode user query.
6. Search index.
7. Filter and rank results.
8. Evaluate whether the right sources appear in top-k.

## 6. When To Use

Use dense embeddings when:

- users ask natural-language questions
- semantic similarity matters
- wording differs between query and document

Use sparse retrieval when:

- exact terms matter
- error codes, IDs, names, or product SKUs matter
- legal or technical phrasing matters

Use hybrid retrieval when:

- both semantic and exact matching matter
- search quality from dense-only retrieval is inconsistent
- enterprise users expect search-like precision

Use multimodal embeddings when:

- inputs include images, audio, video, or screenshots

## 7. When NOT To Use

Avoid embeddings when:

- primary need is exact lookup by ID
- SQL filters answer the question
- the dataset is tiny and simple keyword search is enough
- strong auditability requires deterministic rule matching

Better alternatives:

- PostgreSQL queries
- full-text search
- keyword search
- rules engines
- deterministic parsers

## 8. Advantages

- Enables semantic search.
- Powers RAG systems.
- Supports recommendations and clustering.
- Handles user wording variation.
- Can combine with metadata filters for production control.
- Hybrid approaches improve recall and precision.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Semantic recall vs exact precision | Dense vectors find related meaning but may miss exact strings. |
| Quality vs cost | Better embedding models may cost more. |
| Simplicity vs hybrid quality | Hybrid retrieval improves results but adds tuning. |
| Model upgrades vs migration cost | New models may improve quality but require re-embedding. |

## 10. Limitations

- Embeddings do not guarantee truth.
- Similarity does not equal answer correctness.
- Dense vectors may miss rare terms.
- Sparse retrieval may miss synonyms.
- Hybrid scoring can be hard to tune.
- Domain-specific language may require evaluation or custom models.

## 11. Real-World Examples

Startup example: a help-center chatbot uses dense embeddings for support article search.

Enterprise example: a policy assistant uses hybrid retrieval because employee questions include natural language and exact policy IDs.

FAANG-style example: a large assistant platform routes queries to dense, sparse, code, or multimodal retrievers based on content type.

Production system: a documentation assistant stores dense embeddings, keyword indexes, metadata filters, and reranker scores for each search result.

## 12. Architecture Diagram

```text
                 Ingestion
[Documents] -> [Chunker] -> [Dense Embedder] -> [Vector Index]
                       \-> [Sparse Encoder] -> [Lexical Index]
                       \-> [Metadata Store] -> [PostgreSQL]

                   Query
[User Query] -> [Query Router] -> [Dense Search]
                              -> [Sparse Search]
                              -> [Merge + Filter]
                              -> [Ranked Results]
```

## 13. Python Implementation

Dense vector record:

```python
from dataclasses import dataclass

@dataclass
class DenseVectorRecord:
    id: str
    text: str
    vector: list[float]
    model_version: str
    metadata: dict[str, str]
```

Sparse representation:

```python
def sparse_terms(text: str) -> dict[str, float]:
    terms: dict[str, float] = {}
    for word in text.lower().split():
        terms[word] = terms.get(word, 0.0) + 1.0
    return terms
```

Hybrid result model:

```python
@dataclass
class SearchResult:
    chunk_id: str
    dense_score: float
    sparse_score: float
    final_score: float
    text: str

def combine_scores(dense_score: float, sparse_score: float, alpha: float = 0.7) -> float:
    return alpha * dense_score + (1 - alpha) * sparse_score
```

Routing decision:

```python
def needs_sparse_retrieval(query: str) -> bool:
    has_digits = any(char.isdigit() for char in query)
    has_error_code = "error" in query.lower() or "_" in query
    return has_digits or has_error_code
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class EmbedTypeRequest(BaseModel):
    text: str = Field(min_length=1)
    retrieval_mode: str = "hybrid"

class EmbedTypeResponse(BaseModel):
    retrieval_mode: str
    use_dense: bool
    use_sparse: bool
    reason: str

@app.post("/retrieval/mode", response_model=EmbedTypeResponse)
async def choose_retrieval_mode(request: EmbedTypeRequest) -> EmbedTypeResponse:
    if request.retrieval_mode == "dense":
        return EmbedTypeResponse(retrieval_mode="dense", use_dense=True, use_sparse=False, reason="semantic search only")

    if request.retrieval_mode == "sparse":
        return EmbedTypeResponse(retrieval_mode="sparse", use_dense=False, use_sparse=True, reason="keyword search only")

    use_sparse = needs_sparse_retrieval(request.text)
    return EmbedTypeResponse(
        retrieval_mode="hybrid",
        use_dense=True,
        use_sparse=use_sparse,
        reason="hybrid mode uses sparse search when exact terms appear",
    )
```

Production-ready structure:

```text
app/
  api/routes/search.py
  services/embedding_service.py
  services/retrieval_router.py
  services/hybrid_ranker.py
  repositories/vector_repository.py
  repositories/document_repository.py
```

## 15. Database Integration

PostgreSQL stores source-of-truth metadata:

```text
documents(id, tenant_id, title, source_uri, status, created_at, updated_at)
chunks(id, document_id, chunk_index, text, token_count, metadata_json)
embedding_versions(id, provider, model_name, dimensions, created_at)
```

Vector DB stores:

- chunk ID
- dense vector
- sparse representation if supported
- tenant ID
- permission fields
- document metadata
- embedding model version

Redis use:

- cache repeated query embeddings
- rate-limit expensive embedding generation
- cache frequent search results when safe

## 16. Production Considerations

- Store model version with every vector.
- Evaluate retrieval separately for dense, sparse, and hybrid.
- Add metadata filters before returning results.
- Re-embed documents when embedding model or chunking strategy changes.
- Track retrieval latency and top-k quality.
- Monitor no-result and low-score queries.
- Avoid embedding sensitive content unless policy allows it.
- Protect tenant boundaries in both metadata and vector indexes.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking all vectors are the same | Learn dense, sparse, hybrid, and multimodal use cases |
| Beginner | Using embeddings when SQL is enough | Use the simplest correct retrieval method |
| Intermediate | Dense-only search for exact IDs | Add sparse search or metadata filters |
| Intermediate | No embedding version stored | Store provider, model, and dimension |
| Production | Changing embedding model without migration | Plan re-embedding and index rebuilds |
| Production | No retrieval evaluation | Measure top-k recall and user success |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is a dense embedding? | A compact vector that captures semantic meaning of text or data. |
| Basic | What is sparse retrieval? | A retrieval method based on lexical or keyword-like features. |
| Intermediate | Why use hybrid retrieval? | It combines semantic matching from dense vectors with exact matching from sparse retrieval. |
| Advanced | How choose an embedding type in production? | Start from user queries and failure modes, evaluate dense, sparse, and hybrid methods, then choose based on recall, precision, latency, and cost. |
| Scenario | Users search error codes but dense retrieval misses them. | Add sparse retrieval, exact filters, or hybrid search and evaluate improvement. |

## 19. System Design Discussion

Embedding type is a retrieval architecture decision. It affects accuracy, cost, latency, storage, explainability, and migration strategy.

Design decisions:

- Use dense search for meaning.
- Use sparse search for exact language.
- Use hybrid retrieval for enterprise systems.
- Store metadata for filtering and security.
- Keep retrieval evaluation independent from generation quality.

## 20. Hands-On Assignment

- Easy: Write examples where dense retrieval is better than keyword search.
- Medium: Write examples where sparse retrieval is better than dense retrieval.
- Hard: Design a hybrid scoring strategy for support documents.

## 21. Mini Project

Build a Dense vs Sparse Search Comparator.

Requirements:

- Store 10 small documents.
- Implement simple keyword scoring.
- Mock dense scores manually.
- Compare results for natural language queries and exact-code queries.
- Document when each retrieval type wins.

Folder structure:

```text
retrieval-comparator/
  app/
    main.py
    sparse.py
    dense_mock.py
    hybrid.py
  tests/
    test_hybrid_scores.py
```

## 22. Production-Level Project

Build a Hybrid Retrieval Service.

Real-world problem:

- Enterprise users search internal docs using both natural language and exact technical terms.

Architecture:

```text
[User Query] -> [Query Analyzer]
                  |
                  +-> [Dense Retriever]
                  +-> [Sparse Retriever]
                  +-> [Metadata Filter]
                  v
              [Rank Fusion]
                  |
                  v
            [Top Results API]
```

Tech stack:

- FastAPI
- PostgreSQL
- Vector database
- Redis cache
- Background embedding worker

Scaling strategy:

- Batch embeddings.
- Cache query embeddings.
- Namespace by tenant.
- Track retrieval scores and failures.
- Rebuild indexes asynchronously during migrations.

## Quiz

1. What is a dense embedding?
2. What is a sparse vector?
3. What is hybrid retrieval?
4. Why can dense retrieval miss exact IDs?
5. When is sparse retrieval better?
6. What metadata should be stored with vectors?
7. Why store embedding model version?
8. What is multimodal embedding useful for?
9. How would you evaluate dense vs hybrid retrieval?
10. When is SQL better than embeddings?

## Knowledge Check

You should be able to compare dense, sparse, hybrid, multimodal, code, and domain embeddings, then choose the right representation for a production search use case.

Are you ready for the next section?
