# Vector Databases

## 1. Problem Statement

Vector databases solve the problem of storing, indexing, filtering, and searching large numbers of embeddings efficiently.

A small demo can compare vectors in a Python list. A production system may have millions or billions of chunks across tenants, documents, permissions, and model versions. Brute-force comparison becomes too slow and hard to operate.

Without vector databases:

- semantic search does not scale
- filtering by metadata becomes difficult
- retrieval latency grows with corpus size
- updates and deletes are harder to manage
- RAG systems become slow and unreliable

Real-world analogy: a vector database is like a specialized library catalog for meaning. Instead of searching only titles or keywords, it finds nearby concepts.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | A vector database stores embeddings and retrieves similar vectors using nearest-neighbor search. |
| Key terminology | vector index, ANN, top-k, metadata filter, namespace, collection, dimension, similarity metric |
| Simple explanation | It stores vectors and quickly finds the most similar ones for a query. |
| Mental model | A search engine for meaning-space. |
| Easy example | Store document chunks and retrieve the chunks closest to a user's question. |
| Use When | You need semantic search over many embeddings. |
| Avoid When | Your dataset is tiny or exact SQL search is enough. |
| Advantages | Fast similarity search, metadata filtering, and scalable retrieval. |
| Tradeoffs | Adds infrastructure, cost, and migration complexity. |
| Limitations | Vector DBs do not fix bad embeddings, bad chunking, or missing data. |
| Production Example | Enterprise RAG stores chunk vectors with tenant and permission metadata. |
| Interview Answer | Vector databases index embeddings for efficient similarity search, usually with metadata filtering for production retrieval. |

## 3. Intermediate Explanation

Common options:

| Tool | Type | Best For | Notes |
| --- | --- | --- | --- |
| FAISS | library | local experiments, custom systems | powerful but you operate persistence/API |
| ChromaDB | local/open-source DB | prototypes and small apps | simple developer experience |
| Qdrant | vector DB | production APIs and filtering | strong metadata filtering |
| Weaviate | vector DB | semantic search platforms | rich features |
| Pinecone | managed vector DB | managed production search | less ops burden |
| PostgreSQL + pgvector | relational extension | simple stacks and metadata joins | can be enough for moderate scale |

Core concepts:

- Collection: group of vectors.
- Namespace: logical isolation, often tenant or environment.
- Dimension: vector length expected by the index.
- Similarity metric: cosine, dot product, Euclidean.
- ANN index: approximate nearest neighbor index for speed.
- Metadata filter: structured constraints during search.

Data flow:

```text
Chunk text -> embedding vector -> vector DB upsert -> query vector -> similarity search -> top-k chunks
```

## 4. Advanced Explanation

Production vector DB design is about retrieval quality, latency, isolation, metadata, and migration.

Optimization techniques:

- Choose index type based on latency and recall.
- Use metadata filters to narrow search.
- Store enough metadata for citations and permissions.
- Use namespaces for tenants or environments.
- Batch upserts during ingestion.
- Cache query embeddings.
- Use reranking after vector search.
- Track embedding model version and dimension.

Performance considerations:

- Larger dimensions require more memory.
- Higher top-k increases latency.
- Heavy metadata filters can affect performance.
- Approximate search may miss exact nearest vectors.
- Frequent updates can affect index performance.

Scaling considerations:

- Partition by tenant or corpus.
- Plan index rebuilds for embedding migrations.
- Use background ingestion workers.
- Monitor query latency and index size.
- Set retention and delete strategies.

Production challenges:

- deleting vectors when documents are removed
- avoiding cross-tenant leaks
- migrating embedding models
- filtering performance
- stale vectors after document updates
- cost growth from duplicate chunks and overlap

## 5. Internal Working

```text
Ingestion path:
Document chunk -> embedding -> vector record -> index upsert

Query path:
User question -> query embedding -> vector search -> metadata filter -> top-k results
```

Detailed lifecycle:

1. Document is chunked.
2. Each chunk is embedded.
3. Vector record is created with ID and metadata.
4. Record is inserted or updated in vector DB.
5. User query is embedded.
6. Vector DB searches nearest vectors.
7. Metadata filters restrict results.
8. Top-k chunks are returned with scores.
9. App fetches full metadata or source text if needed.

## 6. When To Use

Use vector databases for:

- RAG
- semantic search
- recommendation systems
- duplicate detection
- image search
- code search
- clustering workflows
- large-scale embedding retrieval

Ideal use cases:

- more than a few thousand embeddings
- need low-latency top-k search
- need metadata filtering
- need update/delete operations
- need production APIs and monitoring

## 7. When NOT To Use

Avoid vector databases when:

- your dataset is tiny
- exact SQL queries solve the task
- keyword search is the core requirement
- you do not need semantic similarity
- infrastructure cost is not justified

Better alternatives:

- Python list for tiny demos
- PostgreSQL full-text search
- Elasticsearch/OpenSearch
- PostgreSQL with pgvector for simpler systems
- direct API/database lookup

## 8. Advantages

- Fast vector similarity search.
- Metadata filtering.
- Scales beyond in-memory demos.
- Supports RAG pipelines.
- Enables semantic search APIs.
- Often supports batch upserts and deletes.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Speed vs recall | Approximate search is fast but may miss some best matches. |
| Managed vs self-hosted | Managed reduces ops but costs more and can add lock-in. |
| Separate DB vs pgvector | Dedicated DB may scale better; pgvector simplifies stack. |
| More metadata vs performance | Filters help relevance but can affect query speed. |

## 10. Limitations

- Does not create good embeddings.
- Does not choose good chunks.
- Does not guarantee correct answers.
- Can be expensive at scale.
- Migration between vector DBs can be painful.
- Requires careful tenant isolation and deletion handling.

## 11. Real-World Examples

Startup example: ChromaDB or pgvector powers a small documentation assistant.

Enterprise example: Qdrant or Pinecone stores millions of policy and support chunks with tenant filters.

FAANG-style example: large retrieval platforms use multiple indexes, rerankers, caches, and evaluation pipelines.

Production system: a RAG API stores vectors in a vector DB, document metadata in PostgreSQL, and query embeddings in Redis cache.

## 12. Architecture Diagram

```text
Ingestion:
[Document] -> [Chunker] -> [Embedding Service] -> [Vector DB]
                                             \-> [PostgreSQL Metadata]

Query:
[User Query] -> [Embedding Service] -> [Vector Search]
                                      -> [Top-k Chunks]
                                      -> [RAG Context Builder]
```

Multi-tenant layout:

```text
[Tenant A Namespace] -> vectors and filters for Tenant A
[Tenant B Namespace] -> vectors and filters for Tenant B
```

## 13. Python Implementation

Vector record:

```python
from dataclasses import dataclass

@dataclass
class VectorRecord:
    id: str
    vector: list[float]
    text: str
    metadata: dict[str, str]
```

In-memory toy vector store:

```python
import math

def cosine(a: list[float], b: list[float]) -> float:
    dot = sum(x * y for x, y in zip(a, b))
    a_mag = math.sqrt(sum(x * x for x in a))
    b_mag = math.sqrt(sum(y * y for y in b))
    if a_mag == 0 or b_mag == 0:
        return 0.0
    return dot / (a_mag * b_mag)

class InMemoryVectorStore:
    def __init__(self) -> None:
        self.records: list[VectorRecord] = []

    def upsert(self, record: VectorRecord) -> None:
        self.records = [item for item in self.records if item.id != record.id]
        self.records.append(record)

    def search(self, query_vector: list[float], top_k: int, tenant_id: str) -> list[VectorRecord]:
        allowed = [
            record for record in self.records
            if record.metadata.get("tenant_id") == tenant_id
        ]
        return sorted(allowed, key=lambda item: cosine(query_vector, item.vector), reverse=True)[:top_k]
```

Repository boundary:

```python
class VectorRepository:
    def upsert_many(self, records: list[VectorRecord]) -> None:
        raise NotImplementedError

    def search(self, query_vector: list[float], filters: dict[str, str], top_k: int) -> list[VectorRecord]:
        raise NotImplementedError
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()
store = InMemoryVectorStore()

class VectorUpsertRequest(BaseModel):
    id: str
    vector: list[float]
    text: str
    tenant_id: str

class VectorSearchRequest(BaseModel):
    query_vector: list[float]
    tenant_id: str
    top_k: int = Field(default=5, ge=1, le=20)

@app.post("/vectors/upsert")
async def upsert_vector(request: VectorUpsertRequest) -> dict[str, str]:
    store.upsert(
        VectorRecord(
            id=request.id,
            vector=request.vector,
            text=request.text,
            metadata={"tenant_id": request.tenant_id},
        )
    )
    return {"status": "stored"}

@app.post("/vectors/search")
async def search_vectors(request: VectorSearchRequest) -> list[dict[str, str]]:
    results = store.search(request.query_vector, request.top_k, request.tenant_id)
    return [{"id": item.id, "text": item.text} for item in results]
```

Production-ready structure:

```text
app/
  api/routes/vector_search.py
  services/vector_search_service.py
  services/embedding_service.py
  repositories/vector_repository.py
  repositories/document_repository.py
  settings.py
```

## 15. Database Integration

PostgreSQL:

```text
documents(id, tenant_id, title, source_uri, status, checksum)
chunks(id, document_id, chunk_index, text, token_count)
vector_index_records(id, chunk_id, embedding_model, vector_db_id, status)
```

Vector DB:

- vector ID
- chunk ID
- vector
- tenant ID
- permissions
- document ID
- page
- embedding model version

Redis:

- cache query vectors
- cache frequent search result IDs
- ingestion job progress

Data consistency pattern:

```text
PostgreSQL is source of truth.
Vector DB is retrieval index.
If document changes, update metadata and reindex vectors.
```

## 16. Production Considerations

- Choose vector DB based on scale, filters, ops, and cost.
- Store embedding model and dimension.
- Enforce tenant filters on every query.
- Plan deletes and updates.
- Monitor query latency.
- Monitor index size.
- Track retrieval quality.
- Back up metadata and know how to rebuild vector index.
- Use batch upserts.
- Avoid mixing embeddings from incompatible models in one index.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking vector DB fixes bad search | Improve embeddings, chunking, and metadata |
| Beginner | Using vector DB when Python list is enough | Keep demos simple |
| Intermediate | No metadata filters | Store tenant, source, permissions, and type |
| Intermediate | Mixing vector dimensions | Keep one compatible model per collection/index |
| Production | No delete/reindex strategy | Track document lifecycle and index status |
| Production | No tenant isolation | Enforce filters and namespaces |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is a vector database? | A database optimized for storing embeddings and retrieving similar vectors. |
| Basic | What is top-k vector search? | Returning the k most similar vectors to a query vector. |
| Intermediate | Why not use PostgreSQL only? | PostgreSQL can work for some cases with extensions, but dedicated vector DBs often provide stronger vector indexing, filtering, and scaling. |
| Advanced | How choose a vector DB? | Compare scale, metadata filters, latency, hybrid support, ops burden, cost, persistence, and team familiarity. |
| Scenario | Vector search is slow. | Check index type, top-k, filters, vector dimension, hardware, namespace size, and query patterns. |

## 19. System Design Discussion

Vector databases are retrieval indexes, not the whole RAG system. They should work with:

- document ingestion
- chunking
- embedding services
- metadata storage
- permissions
- reranking
- monitoring
- evaluation

Design decisions:

- managed vs self-hosted
- pgvector vs dedicated vector DB
- namespace per tenant vs metadata filter
- index type
- delete and reindex strategy
- embedding migration plan

## 20. Hands-On Assignment

- Easy: Build an in-memory vector store.
- Medium: Add tenant filtering.
- Hard: Add vector record versioning and delete behavior.

## 21. Mini Project

Build a Local Vector Search API.

Requirements:

- Upsert vector records.
- Store metadata.
- Search by query vector.
- Filter by tenant.
- Return top-k results.
- Log query latency.

Folder structure:

```text
local-vector-search/
  app/
    main.py
    vector_store.py
    schemas.py
  tests/
    test_vector_store.py
```

## 22. Production-Level Project

Build a Vector Search Service for Enterprise RAG.

Real-world problem:

- A company needs fast, permission-safe semantic search across thousands of documents.

Architecture:

```text
[Document Ingestion] -> [Embedding Worker] -> [Vector DB]
                       -> [PostgreSQL Metadata]

[Query API] -> [Embedding Service] -> [Vector DB Search]
              -> [Permission Filter]
              -> [Search Logs]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- Qdrant, Pinecone, Weaviate, ChromaDB, FAISS, or pgvector
- Background workers

Scaling strategy:

- Batch upserts.
- Namespace by tenant.
- Cache query embeddings.
- Rebuild indexes asynchronously.
- Track latency and retrieval quality.
- Keep metadata source of truth in PostgreSQL.

## Quiz

1. What is a vector database?
2. What is nearest-neighbor search?
3. What is approximate nearest-neighbor search?
4. What is top-k?
5. Why use metadata filters?
6. Why store vector dimension?
7. Why store embedding model version?
8. What is the difference between PostgreSQL metadata and vector DB index?
9. How do you handle document deletion?
10. How would you choose between pgvector and a dedicated vector DB?

## Knowledge Check

You should be able to explain what vector databases do, where they fit in RAG, how to choose one, and how to operate them safely in production.

Are you ready for the next section?
