# Retrieval Strategies

## 1. Problem Statement

Retrieval strategies solve the problem of finding the right information before an LLM generates an answer.

LLMs are powerful, but they do not automatically know your private documents, latest policies, user-specific permissions, or changing business data. Retrieval brings relevant external information into the workflow.

Without good retrieval:

- RAG answers hallucinate or cite wrong sources.
- Users see irrelevant chunks.
- Private documents may leak across tenants.
- The LLM receives noisy or missing context.
- System quality depends too much on the generation prompt.

Real-world analogy: before answering a legal question, a lawyer searches the right laws, cases, and contract clauses. Retrieval is that search step for AI systems.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Retrieval is the process of finding relevant information for a query before generation or decision-making. |
| Key terminology | dense retrieval, sparse retrieval, hybrid retrieval, metadata filter, top-k, threshold, reranking, recall, precision |
| Simple explanation | The system searches your knowledge base and returns the best candidate chunks. |
| Mental model | Retrieval is the model's open-book search step. |
| Easy example | Search company policy documents before answering an HR question. |
| Use When | The answer depends on external, private, or changing knowledge. |
| Avoid When | A direct database query or simple rule answers the question. |
| Advantages | Improves grounding, freshness, and source traceability. |
| Tradeoffs | Adds latency, indexes, and retrieval failure modes. |
| Limitations | Retrieval cannot find information that was never indexed or is poorly chunked. |
| Production Example | Customer support bot retrieves active policy chunks filtered by tenant and region. |
| Interview Answer | Retrieval finds relevant external context, usually using semantic, lexical, or hybrid search, before an LLM generates an answer. |

## 3. Intermediate Explanation

Common retrieval strategies:

| Strategy | How It Works | Best For | Weakness |
| --- | --- | --- | --- |
| Dense retrieval | query vector vs document vectors | semantic search | exact terms can be missed |
| Sparse retrieval | keyword or lexical scoring | IDs, legal terms, error codes | weaker synonym handling |
| Hybrid retrieval | combines dense and sparse | enterprise search | score tuning required |
| Metadata filtering | filters by structured fields | tenant, permission, date, type | requires good metadata |
| Multi-query retrieval | generates several query variants | broad recall | extra cost |
| Query rewriting | rewrites unclear query | conversational search | can change intent |
| Parent-child retrieval | retrieves small chunks but returns larger parent context | docs with sections | more storage logic |
| Reranking | reorders candidates after retrieval | precision improvement | extra latency |

Core metrics:

- Recall: did we retrieve the correct source?
- Precision: are retrieved results mostly relevant?
- MRR: how high is the first correct result?
- Latency: how long retrieval takes.
- Coverage: how often retrieval finds usable context.

Data flow:

```text
User query -> query analysis -> retrieval method -> filters -> candidates -> ranking -> selected context
```

## 4. Advanced Explanation

Production retrieval should be evaluated independently from LLM generation. If retrieval fails, even the best answer prompt may produce weak or hallucinated answers.

Optimization techniques:

- Use metadata filters before or during vector search.
- Use hybrid retrieval for enterprise docs.
- Use query rewriting for conversational follow-ups.
- Use multi-query retrieval when one query misses needed context.
- Use score thresholds to avoid low-confidence context.
- Use reranking when recall is okay but precision is weak.
- Use parent-child retrieval when chunks lack surrounding context.

Performance considerations:

- Higher top-k increases recall but adds noise.
- Reranking increases latency.
- Multi-query increases model and retrieval calls.
- Filters can improve relevance but may reduce recall if metadata is wrong.
- Approximate vector search trades perfect recall for speed.

Scaling considerations:

- Namespace or partition by tenant.
- Precompute embeddings.
- Cache repeated queries.
- Monitor index size and query latency.
- Use asynchronous ingestion.

Production challenges:

- missing or stale documents
- bad chunking
- weak metadata
- permission leaks
- query ambiguity
- retrieval drift after data changes

## 5. Internal Working

```text
Input question
  |
  v
Normalize and analyze query
  |
  v
Apply tenant and permission filters
  |
  v
Run dense, sparse, or hybrid retrieval
  |
  v
Merge and rank candidates
  |
  v
Apply thresholds and reranking if needed
  |
  v
Return top chunks with metadata and scores
```

Detailed lifecycle:

1. User sends query.
2. Backend identifies tenant, user, permissions, and retrieval mode.
3. Query is embedded and/or keyword encoded.
4. Search runs against permitted indexes or metadata filters.
5. Candidate chunks are scored.
6. Scores are merged or reranked.
7. Top chunks are selected.
8. Retrieval trace is logged.
9. Context is passed to the LLM or returned to the user.

## 6. When To Use

Use retrieval when:

- answers need private company data
- documents change often
- source citations matter
- the model must answer from a knowledge base
- user-specific access control matters
- you need semantic search
- you are building RAG

Industry examples:

- HR policy assistant
- legal document Q&A
- customer support bot
- technical docs assistant
- code search assistant
- compliance knowledge platform

## 7. When NOT To Use

Avoid retrieval when:

- SQL can directly answer the question
- data is fully structured
- the dataset is too small to justify indexes
- the task is pure creative generation
- the answer should come from live APIs instead of documents

Better alternatives:

- direct PostgreSQL query
- business rules
- live tool/API call
- cached lookup
- deterministic parser

## 8. Advantages

- Keeps knowledge updatable.
- Reduces unsupported model guesses.
- Enables citations.
- Works with private data.
- Can enforce permissions.
- Improves answer relevance when tuned well.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Recall vs precision | More candidates find more answers but add noise. |
| Latency vs quality | Reranking and multi-query improve quality but slow response. |
| Filters vs coverage | Strict filters protect security but may hide relevant results. |
| Simplicity vs robustness | Basic vector search is simple; hybrid retrieval is stronger but complex. |

## 10. Limitations

- Retrieval cannot find missing data.
- Similarity scores are not truth scores.
- Metadata errors can block or leak results.
- Query rewriting can change user intent.
- Top-k retrieval may miss long-range relationships.
- Retrieval quality depends heavily on chunking.

## 11. Real-World Examples

Startup example: a support chatbot retrieves top FAQ entries before answering.

Enterprise example: a multi-tenant RAG platform retrieves only documents the current employee can access.

FAANG-style example: an assistant system combines dense retrieval, lexical retrieval, query understanding, reranking, and feedback loops.

Production system: a documentation assistant logs query, filters, retrieved chunk IDs, scores, final citations, and user feedback.

## 12. Architecture Diagram

```text
[User Query]
    |
    v
[Query Analyzer]
    |
    +--> [Dense Retriever]
    +--> [Sparse Retriever]
    +--> [Metadata Filters]
    |
    v
[Candidate Merger]
    |
    v
[Reranker / Threshold]
    |
    v
[Selected Context]
    |
    v
[LLM or Search Results]
```

## 13. Python Implementation

Top-k ranking:

```python
from dataclasses import dataclass

@dataclass
class Candidate:
    chunk_id: str
    text: str
    score: float
    metadata: dict[str, str]

def top_k(candidates: list[Candidate], k: int) -> list[Candidate]:
    return sorted(candidates, key=lambda item: item.score, reverse=True)[:k]
```

Score threshold:

```python
def filter_by_threshold(candidates: list[Candidate], min_score: float) -> list[Candidate]:
    return [candidate for candidate in candidates if candidate.score >= min_score]
```

Metadata filter:

```python
def allowed_for_user(candidate: Candidate, tenant_id: str, role: str) -> bool:
    return (
        candidate.metadata.get("tenant_id") == tenant_id
        and role in candidate.metadata.get("allowed_roles", "").split(",")
    )
```

Hybrid merge:

```python
def reciprocal_rank_fusion(result_lists: list[list[str]], k: int = 60) -> dict[str, float]:
    scores: dict[str, float] = {}
    for results in result_lists:
        for rank, chunk_id in enumerate(results):
            scores[chunk_id] = scores.get(chunk_id, 0.0) + 1.0 / (k + rank + 1)
    return scores
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class SearchRequest(BaseModel):
    query: str = Field(min_length=1)
    tenant_id: str
    top_k: int = Field(default=5, ge=1, le=50)
    retrieval_mode: str = "hybrid"

class SearchResultResponse(BaseModel):
    chunk_id: str
    text: str
    score: float
    source: str | None = None

@app.post("/search", response_model=list[SearchResultResponse])
async def search(request: SearchRequest) -> list[SearchResultResponse]:
    # In production, call dense/sparse retrievers and apply permissions.
    candidates = [
        Candidate("chunk-1", "Refunds are processed within 7 days.", 0.91, {"tenant_id": request.tenant_id, "source": "policy.md"}),
        Candidate("chunk-2", "Password reset steps.", 0.44, {"tenant_id": request.tenant_id, "source": "help.md"}),
    ]
    selected = top_k(filter_by_threshold(candidates, min_score=0.5), request.top_k)
    return [
        SearchResultResponse(
            chunk_id=item.chunk_id,
            text=item.text,
            score=item.score,
            source=item.metadata.get("source"),
        )
        for item in selected
    ]
```

Production-ready structure:

```text
app/
  api/routes/search.py
  services/query_analyzer.py
  services/retrieval_service.py
  services/reranker.py
  repositories/vector_repository.py
  repositories/search_log_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
documents(id, tenant_id, title, source_uri, status)
chunks(id, document_id, text, token_count, page_number)
retrieval_logs(id, query, tenant_id, retrieval_mode, top_k, latency_ms)
retrieval_results(id, retrieval_log_id, chunk_id, score, rank)
```

Vector database:

- stores chunk vectors
- supports similarity search
- stores metadata for filtering

Redis:

- cache query embeddings
- cache frequent search results
- rate-limit high-cost retrieval modes

## 16. Production Considerations

- Enforce tenant and permission filters before results reach the LLM.
- Log query, retrieval mode, filters, scores, and chunk IDs.
- Use evaluation datasets with expected sources.
- Track no-result rate.
- Track low-score result rate.
- Monitor retrieval latency.
- Add fallback when retrieval returns no strong context.
- Separate retrieval evaluation from answer evaluation.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Sending the whole document to the LLM | Retrieve only relevant chunks |
| Beginner | Thinking vector search equals RAG | Retrieval is only one part of RAG |
| Intermediate | No metadata filters | Store and apply tenant, role, source, and date filters |
| Intermediate | Top-k too high or too low | Tune with evaluation |
| Production | Permission checks after generation | Filter before retrieval output is used |
| Production | No retrieval logs | Log chunk IDs, scores, filters, and query |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is retrieval? | Finding relevant external information for a query before generating an answer. |
| Basic | What is top-k? | The number of highest-ranked results returned by retrieval. |
| Intermediate | What is hybrid retrieval? | Combining semantic dense retrieval and lexical sparse retrieval. |
| Advanced | How do you evaluate retrieval? | Use labeled queries with expected documents, measure recall, precision, MRR, latency, and downstream answer quality. |
| Scenario | A RAG system gives wrong answers. | Check whether the right chunks were retrieved, inspect filters, chunking, scores, reranking, and prompt context. |

## 19. System Design Discussion

Retrieval is one of the most important quality layers in GenAI systems. The LLM can only answer from the context it receives. If retrieval returns irrelevant or unauthorized chunks, the final answer will be unreliable or unsafe.

Design decisions:

- dense vs sparse vs hybrid
- top-k value
- score threshold
- metadata filters
- reranking
- query rewriting
- no-result fallback
- retrieval logging

## 20. Hands-On Assignment

- Easy: Implement top-k ranking for scored documents.
- Medium: Add metadata filters by tenant and document type.
- Hard: Build hybrid rank fusion for two result lists.

## 21. Mini Project

Build a Document Search API.

Requirements:

- Accept query and tenant ID.
- Search mock dense and sparse results.
- Apply metadata filters.
- Merge results.
- Return top-k chunks with scores.

Folder structure:

```text
document-search/
  app/
    main.py
    schemas.py
    retrievers.py
    ranker.py
  tests/
    test_ranker.py
```

## 22. Production-Level Project

Build a Retrieval Service for Enterprise RAG.

Real-world problem:

- Internal AI assistant must retrieve accurate, permission-safe context from thousands of documents.

Architecture:

```text
[Query API] -> [Query Analyzer]
              -> [Dense Retriever]
              -> [Sparse Retriever]
              -> [Permission Filter]
              -> [Reranker]
              -> [Retrieval Logger]
              -> [Context Builder]
```

Tech stack:

- FastAPI
- PostgreSQL
- Vector database
- Redis
- Background ingestion worker

Scaling strategy:

- Namespace indexes by tenant.
- Cache query embeddings.
- Batch retrieval logs.
- Monitor latency and recall.
- Reindex asynchronously.

## Quiz

1. What is retrieval?
2. Why is retrieval important for RAG?
3. What is top-k?
4. What is a score threshold?
5. What is metadata filtering?
6. What is hybrid retrieval?
7. What is query rewriting?
8. Why can query rewriting be risky?
9. What retrieval metrics matter?
10. How would you debug irrelevant retrieved chunks?

## Knowledge Check

You should be able to design a retrieval pipeline, explain dense/sparse/hybrid strategies, and debug common retrieval failures.

Are you ready for the next section?
