# Advanced Retrieval

## 1. Problem Statement

Advanced retrieval solves the problem of basic vector search returning incomplete, noisy, stale, or unauthorized context.

In real RAG systems, users ask messy questions. Documents contain exact terms, synonyms, dates, versions, permissions, and domain-specific language. Dense vector search alone is often not enough.

Without advanced retrieval:

- exact terms like error codes are missed
- wrong tenant data may appear
- old documents outrank current ones
- ambiguous questions retrieve weak context
- relevant sources are buried below irrelevant chunks

Real-world analogy: a professional researcher does not use only one search method. They combine keywords, filters, source quality, dates, and follow-up searches.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Advanced retrieval combines techniques such as metadata filtering, hybrid search, query rewriting, multi-query retrieval, and reranking to improve retrieved context. |
| Key terminology | metadata filter, hybrid search, query rewrite, multi-query, recency filter, permission filter, top-k, threshold |
| Simple explanation | Basic search finds candidates; advanced retrieval makes search safer and more accurate. |
| Mental model | Retrieval is a search pipeline, not one function call. |
| Easy example | Search only documents from the user's tenant and current product version. |
| Use When | Basic retrieval returns irrelevant or incomplete chunks. |
| Avoid When | Simple top-k retrieval already works well and meets latency goals. |
| Advantages | Better relevance, safety, and business control. |
| Tradeoffs | More latency, evaluation, and system complexity. |
| Limitations | It cannot retrieve data that was never indexed or extracted. |
| Production Example | Support RAG uses hybrid retrieval plus filters for product, region, tenant, and active policy status. |
| Interview Answer | Advanced retrieval improves candidate selection using filters, hybrid search, query transformations, and ranking before the LLM sees context. |

## 3. Intermediate Explanation

Common advanced retrieval techniques:

| Technique | Purpose | Example |
| --- | --- | --- |
| Metadata filtering | restrict by structured fields | tenant, role, region, product version |
| Hybrid search | combine semantic and keyword retrieval | dense plus sparse |
| Query rewriting | improve unclear user query | "it" -> previous topic |
| Multi-query retrieval | search several query variants | broad research questions |
| Recency filtering | prefer fresh sources | latest policy docs |
| Score thresholds | avoid weak context | reject results below 0.45 |
| Parent-child retrieval | retrieve precise child, return larger parent | legal sections |
| Reranking | reorder candidates by relevance | top 50 -> top 5 |

Data flow:

```text
Question -> query analysis -> filters -> dense/sparse retrieval -> merge -> threshold -> rerank -> context
```

Practical examples:

- Query: "How do I fix ERR_AUTH_403?" Hybrid search catches exact error code.
- Query: "What is our refund rule in Germany?" Metadata filters region and policy type.
- Query: "Does this apply to version 2.1?" Version filter prevents old docs.
- Query: "What about that?" Query rewriting uses chat context.

## 4. Advanced Explanation

Advanced retrieval should be designed around observed failure patterns.

Optimization techniques:

- Use query classification to choose retrieval mode.
- Add metadata filters from user context and document metadata.
- Use hybrid search for enterprise docs.
- Retrieve more candidates than final context needs.
- Rerank candidates for precision.
- Use recency boosts when freshness matters.
- Add fallback search if first retrieval is weak.
- Log query transformations for debugging.

Performance considerations:

- multi-query retrieval increases calls and latency
- heavy filters can reduce recall
- hybrid retrieval requires score merging
- reranking adds compute
- query rewriting can alter user intent

Scaling considerations:

- cache query rewrites and embeddings
- precompute sparse indexes
- partition by tenant
- tune top-k per corpus
- monitor no-result rate by retrieval mode

Production challenges:

- filter metadata missing or wrong
- query rewrite creates a different question
- old documents still indexed
- score thresholds reject valid low-score results
- multi-query increases cost too much

## 5. Internal Working

```text
User question
  |
  v
Query analyzer extracts:
  - tenant
  - permissions
  - product
  - date/version
  - exact terms
  |
  v
Retriever runs dense, sparse, or hybrid search
  |
  v
Candidate chunks are filtered and merged
  |
  v
Weak candidates are removed
  |
  v
Best candidates are sent to reranker/context builder
```

Detailed lifecycle:

1. Authenticate user.
2. Identify tenant and permissions.
3. Analyze query for exact terms, topic, and filters.
4. Rewrite query only if needed.
5. Run dense and/or sparse retrieval.
6. Apply metadata filters.
7. Merge candidates.
8. Apply thresholds.
9. Rerank if enabled.
10. Log all retrieval decisions.

## 6. When To Use

Use advanced retrieval when:

- dense-only search misses exact terms
- users need permission-safe answers
- documents have product versions or dates
- multiple document types exist
- simple top-k produces noisy context
- RAG answers often cite weak sources
- enterprise search quality matters

Ideal use cases:

- legal RAG
- customer support RAG
- internal knowledge assistants
- technical documentation
- codebase search
- compliance assistants

## 7. When NOT To Use

Avoid advanced retrieval when:

- dataset is small
- simple retrieval performs well
- latency budget is very tight
- metadata quality is poor
- the problem should be solved with SQL or tools

Better alternatives:

- improve chunking first
- improve metadata extraction
- use direct database queries
- use simpler keyword search
- use a live tool call

## 8. Advantages

- Improves relevance.
- Enforces business rules.
- Reduces unauthorized context risk.
- Handles exact terms better.
- Supports freshness and version control.
- Makes retrieval more debuggable.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Quality vs latency | More retrieval steps slow down answers. |
| Recall vs filters | Strict filters protect scope but may hide useful docs. |
| Query rewrite vs intent | Rewriting can help or distort the question. |
| Hybrid quality vs tuning | Hybrid search is stronger but needs score strategy. |

## 10. Limitations

- Cannot recover missing indexed data.
- Metadata filters depend on metadata quality.
- Query rewriting may introduce errors.
- Hybrid score tuning is corpus-specific.
- More stages make debugging harder unless traces are logged.

## 11. Real-World Examples

Startup example: a support assistant adds keyword search after dense retrieval misses product error codes.

Enterprise example: HR RAG filters by country, employment type, and policy effective date.

FAANG-style example: a retrieval platform routes queries through specialized retrievers and rerankers based on intent and corpus.

Production system: a RAG API logs raw query, rewritten query, filters, retrieval modes, candidate IDs, scores, and final selected chunks.

## 12. Architecture Diagram

```text
[Question]
    |
    v
[Query Analyzer]
    |
    +--> [Metadata Filters]
    +--> [Dense Retriever]
    +--> [Sparse Retriever]
    +--> [Query Rewrite / Multi-Query]
    |
    v
[Candidate Merge]
    |
    v
[Threshold + Rerank]
    |
    v
[Context Builder]
```

## 13. Python Implementation

Filter builder:

```python
def build_filters(tenant_id: str, role: str, product: str | None = None) -> dict[str, str]:
    filters = {"tenant_id": tenant_id, "allowed_role": role}
    if product:
        filters["product"] = product
    return filters
```

Query mode selection:

```python
def choose_retrieval_mode(query: str) -> str:
    has_exact_token = any(char.isdigit() for char in query) or "_" in query
    if has_exact_token:
        return "hybrid"
    return "dense"
```

Candidate merge:

```python
from dataclasses import dataclass

@dataclass
class Candidate:
    chunk_id: str
    text: str
    score: float
    source: str

def merge_candidates(*candidate_lists: list[Candidate]) -> list[Candidate]:
    by_id: dict[str, Candidate] = {}
    for candidates in candidate_lists:
        for candidate in candidates:
            current = by_id.get(candidate.chunk_id)
            if current is None or candidate.score > current.score:
                by_id[candidate.chunk_id] = candidate
    return list(by_id.values())
```

Score threshold:

```python
def apply_threshold(candidates: list[Candidate], min_score: float) -> list[Candidate]:
    return [candidate for candidate in candidates if candidate.score >= min_score]
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class AdvancedSearchRequest(BaseModel):
    query: str = Field(min_length=1)
    tenant_id: str
    role: str
    product: str | None = None
    top_k: int = Field(default=5, ge=1, le=20)

class AdvancedSearchResponse(BaseModel):
    retrieval_mode: str
    filters: dict[str, str]
    results: list[str]

@app.post("/retrieval/advanced", response_model=AdvancedSearchResponse)
async def advanced_search(request: AdvancedSearchRequest) -> AdvancedSearchResponse:
    filters = build_filters(request.tenant_id, request.role, request.product)
    mode = choose_retrieval_mode(request.query)

    # In production: call dense/sparse retrievers, merge, threshold, rerank.
    results = ["chunk-1", "chunk-2"][:request.top_k]

    return AdvancedSearchResponse(
        retrieval_mode=mode,
        filters=filters,
        results=results,
    )
```

Production-ready structure:

```text
app/
  services/query_analyzer.py
  services/filter_builder.py
  services/dense_retriever.py
  services/sparse_retriever.py
  services/candidate_merger.py
  services/retrieval_tracer.py
```

## 15. Database Integration

PostgreSQL:

```text
documents(id, tenant_id, product, region, effective_date, status)
chunks(id, document_id, text, section, page_number)
retrieval_traces(id, query, rewritten_query, retrieval_mode, filters_json, latency_ms)
retrieval_candidates(id, trace_id, chunk_id, score, rank, stage)
```

Vector DB metadata:

- tenant ID
- roles or ACL tags
- product
- region
- document type
- effective date
- status

Redis:

- cache query rewrites
- cache query embeddings
- rate-limit multi-query retrieval

## 16. Production Considerations

- Always enforce permission filters.
- Log original and rewritten queries.
- Log filters and retrieval mode.
- Compare dense-only vs hybrid retrieval quality.
- Keep metadata complete and validated.
- Add no-result fallback.
- Tune score thresholds with eval data.
- Use recency carefully when old documents are still relevant.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Using dense vector search for everything | Add sparse or hybrid for exact terms |
| Beginner | Ignoring metadata | Store and filter by business fields |
| Intermediate | Query rewriting every question | Rewrite only when useful and log it |
| Intermediate | No threshold | Avoid sending weak context |
| Production | Applying permissions after LLM generation | Filter before context reaches the model |
| Production | No retrieval trace | Log every retrieval stage |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is advanced retrieval? | Retrieval techniques beyond basic top-k vector search, such as filtering, hybrid search, query rewriting, and reranking. |
| Intermediate | Why use metadata filters? | To enforce scope, permissions, freshness, product, region, or document type constraints. |
| Intermediate | Why use hybrid search? | Dense retrieval handles meaning, while sparse retrieval catches exact terms. |
| Advanced | How do you tune retrieval? | Use evaluation queries, expected sources, recall, precision, latency, thresholds, and answer quality metrics. |
| Scenario | Users see outdated policy answers. | Add effective-date metadata, status filters, recency handling, and reindex stale documents. |

## 19. System Design Discussion

Advanced retrieval makes RAG enterprise-ready. It turns retrieval from "nearest vector wins" into a controlled search pipeline with business rules, security, and evaluation.

Design decisions:

- dense vs sparse vs hybrid
- query rewriting policy
- metadata schema
- score threshold
- reranking stage
- fallback strategy
- trace logging

## 20. Hands-On Assignment

- Easy: Build filters from tenant, role, and product.
- Medium: Route queries with error codes to hybrid retrieval.
- Hard: Build a trace object that records retrieval mode, filters, candidates, and selected chunks.

## 21. Mini Project

Build an Advanced Retrieval Simulator.

Requirements:

- Accept query and metadata filters.
- Choose retrieval mode.
- Merge mock dense and sparse results.
- Apply thresholds.
- Return a retrieval trace.

Folder structure:

```text
advanced-retrieval/
  app/
    main.py
    query_analyzer.py
    filters.py
    merger.py
    trace.py
  tests/
    test_query_analyzer.py
```

## 22. Production-Level Project

Build a Permission-Safe Hybrid Retriever.

Real-world problem:

- Enterprise RAG must retrieve accurate context without leaking documents across users or tenants.

Architecture:

```text
[Query API] -> [Auth Context]
              -> [Query Analyzer]
              -> [Filter Builder]
              -> [Dense + Sparse Retrieval]
              -> [Candidate Merge]
              -> [Rerank]
              -> [Trace Logs]
```

Tech stack:

- FastAPI
- PostgreSQL
- Vector DB
- Redis
- OpenSearch or sparse index if needed

Scaling strategy:

- cache embeddings and rewrites
- tune top-k by corpus
- record retrieval metrics
- evaluate by tenant and document type
- rate-limit expensive multi-query search

## Quiz

1. What is advanced retrieval?
2. Why is dense-only retrieval sometimes weak?
3. What are metadata filters?
4. What is hybrid retrieval?
5. What is query rewriting?
6. Why can query rewriting be dangerous?
7. What is multi-query retrieval?
8. Why use thresholds?
9. Why log retrieval traces?
10. How do you prevent permission leaks in retrieval?

## Knowledge Check

You should be able to design an advanced retrieval pipeline with filters, hybrid search, trace logging, and production-safe permissions.

Are you ready for the next section?
