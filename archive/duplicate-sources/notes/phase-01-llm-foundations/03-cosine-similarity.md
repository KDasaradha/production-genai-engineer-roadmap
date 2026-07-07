# Cosine Similarity

## 1. Problem Statement

Once text is converted into vectors, the system needs a way to compare them. Cosine similarity solves this by measuring how similar two vectors are in direction.

Without similarity metrics, embeddings are just lists of numbers with no useful comparison.

Analogy: two arrows pointing in the same direction represent similar meaning, even if one arrow is longer.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Cosine similarity measures the angle between two vectors. |
| Key terminology | vector, dot product, magnitude, normalization, similarity score |
| Simple explanation | Similar vectors point in similar directions. |
| Mental model | Compare direction, not size. |
| Easy example | A query vector is compared with document vectors to find the nearest meaning. |
| Use When | Ranking embeddings by semantic similarity. |
| Avoid When | Magnitude carries important meaning. |
| Advantages | Simple, common, and effective for normalized embeddings. |
| Tradeoffs | High score does not guarantee factual correctness. |
| Limitations | Depends on embedding quality. |
| Production Example | Rank top document chunks for a RAG answer. |
| Interview Answer | Cosine similarity compares vector direction and is commonly used to rank embedding similarity. |

## 3. Intermediate Explanation

Formula:

```text
cosine_similarity(A, B) = dot(A, B) / (magnitude(A) * magnitude(B))
```

Scores are often between -1 and 1:

- 1 means same direction.
- 0 means unrelated direction.
- -1 means opposite direction.

In many embedding systems, practical scores are often positive because of model behavior.

## 4. Advanced Explanation

Production systems may use cosine similarity, dot product, or Euclidean distance depending on the embedding model and vector database index.

Optimization techniques:

- Normalize vectors.
- Precompute document embeddings.
- Use approximate nearest neighbor indexes.
- Combine similarity with metadata filters and reranking.

Performance considerations:

- Comparing one query to millions of vectors requires indexes.
- Exact search is expensive at scale.
- Approximate search trades accuracy for speed.

## 5. Internal Working

```text
Query text -> query embedding
Document chunks -> stored embeddings
Query vector compared with each stored vector
Scores sorted descending
Top-k chunks returned
```

## 6. When To Use

Use cosine similarity for:

- semantic search
- RAG retrieval
- duplicate detection
- clustering support
- recommendations

## 7. When NOT To Use

Avoid relying only on cosine similarity when:

- exact identifiers matter
- permissions matter
- temporal freshness matters
- legal wording must match exactly

Use filters, keyword search, or rerankers as needed.

## 8. Advantages

- Easy to understand.
- Works well with embeddings.
- Commonly supported by vector databases.
- Good baseline for semantic retrieval.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Speed vs accuracy | Approximate search is faster but may miss best matches. |
| Similarity vs correctness | Similar text may not answer the question. |
| Simplicity vs control | Pure cosine ignores metadata and business rules. |

## 10. Limitations

- Does not understand permissions.
- Does not ensure factual grounding.
- Cannot fix bad embeddings.
- Needs indexing at scale.

## 11. Real-World Examples

Startup example: match a user question to FAQ entries.

Enterprise example: find policy chunks relevant to an HR question.

FAANG-style example: retrieve candidate passages from a huge knowledge corpus before reranking.

Production system: vector search returns top 20 chunks, then a reranker chooses top 5 for the LLM.

## 12. Architecture Diagram

```text
[Query Vector]
      |
      v
[Similarity Search] <-> [Stored Document Vectors]
      |
      v
[Top-k Results]
```

## 13. Python Implementation

Beginner implementation:

```python
import math

def dot(a: list[float], b: list[float]) -> float:
    return sum(x * y for x, y in zip(a, b))

def magnitude(vector: list[float]) -> float:
    return math.sqrt(sum(x * x for x in vector))

def cosine_similarity(a: list[float], b: list[float]) -> float:
    return dot(a, b) / (magnitude(a) * magnitude(b))

print(cosine_similarity([1, 1], [1, 0.9]))
```

Intermediate ranking:

```python
documents = {
    "refund policy": [1.0, 0.9, 0.0],
    "login help": [0.0, 0.1, 1.0],
    "pricing page": [0.2, 1.0, 0.1],
}

query = [1.0, 0.8, 0.0]

ranked = sorted(
    documents.items(),
    key=lambda item: cosine_similarity(query, item[1]),
    reverse=True,
)

print(ranked[0])
```

Advanced guard:

```python
def safe_cosine_similarity(a: list[float], b: list[float]) -> float:
    a_mag = magnitude(a)
    b_mag = magnitude(b)
    if a_mag == 0 or b_mag == 0:
        return 0.0
    return dot(a, b) / (a_mag * b_mag)
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class SimilarityRequest(BaseModel):
    query_vector: list[float]
    document_vectors: dict[str, list[float]]

class SimilarityResult(BaseModel):
    document_id: str
    score: float

@app.post("/similarity", response_model=list[SimilarityResult])
async def similarity(request: SimilarityRequest) -> list[SimilarityResult]:
    results = [
        SimilarityResult(document_id=doc_id, score=cosine_similarity(request.query_vector, vector))
        for doc_id, vector in request.document_vectors.items()
    ]
    return sorted(results, key=lambda item: item.score, reverse=True)
```

## 15. Database Integration

Vector databases use similarity search indexes. PostgreSQL can store metadata and sometimes vectors through extensions, while dedicated vector databases focus on efficient vector retrieval.

## 16. Production Considerations

- Use top-k carefully.
- Add score thresholds.
- Filter by tenant and permissions before or during retrieval.
- Evaluate retrieval quality using labeled questions.
- Log query, retrieved IDs, scores, and final answer quality.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking highest score always means correct answer | Check relevance and context |
| Intermediate | No threshold | Reject weak matches |
| Production | Similarity without access control | Always filter by permissions |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is cosine similarity? | A measure of how close two vectors are in direction. |
| Intermediate | Why use it with embeddings? | It ranks text by semantic similarity. |
| Advanced | Why use reranking after vector search? | Initial similarity search is broad and may retrieve related but less useful chunks. |
| Scenario | Top result is irrelevant. | Improve chunking, add filters, use thresholds, hybrid search, or reranking. |

## 19. System Design Discussion

Cosine similarity is usually the first retrieval step, not the final truth check. Production RAG systems often combine vector search with filters, reranking, citations, and answer validation.

## 20. Hands-On Assignment

- Easy: Implement dot product.
- Medium: Implement cosine similarity.
- Hard: Build top-k document search.

## 21. Mini Project

Build a mini semantic search engine using hand-written vectors.

## 22. Production-Level Project

Implement retrieval evaluation for a RAG system by recording expected documents and comparing top-k results.

## Quiz

1. What does cosine similarity measure?
2. What is a dot product?
3. What is vector magnitude?
4. Why compare direction instead of length?
5. What does top-k mean?
6. Why use a score threshold?
7. Why can similar text be wrong?
8. How do metadata filters help?
9. Why do vector databases use indexes?
10. What does reranking improve?

## Knowledge Check

You should be able to implement cosine similarity and explain how it powers semantic retrieval.

Are you ready for the next section?