# Reranking and Context Compression

## 1. Problem Statement

Reranking and context compression solve the problem of retrieval returning too much context, noisy context, or context that does not fit inside the prompt budget.

Basic retrieval often optimizes for recall: "find anything that might be relevant." But the LLM needs precise, compact, high-quality context. Reranking improves candidate order. Context compression reduces the amount of text while preserving useful evidence.

Without these techniques:

- irrelevant chunks enter the prompt
- token cost increases
- answer quality drops
- important chunks may be buried
- context windows overflow
- citations become noisy

Real-world analogy: retrieval gives you a pile of books. Reranking chooses the best pages. Compression highlights the important paragraphs.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Reranking reorders retrieved candidates by relevance; context compression reduces selected context to the most useful information. |
| Key terminology | candidate, top-k, reranker, cross-encoder, context budget, compression, summarization, precision |
| Simple explanation | First retrieve broadly, then select and shrink the best evidence. |
| Mental model | Find many, choose few, send only what matters. |
| Easy example | Retrieve 30 chunks, rerank them, send the best 5 to the LLM. |
| Use When | Retrieval returns noisy or too many chunks. |
| Avoid When | Simple retrieval already returns precise context within budget. |
| Advantages | Better relevance, lower prompt noise, improved citations. |
| Tradeoffs | Extra latency, cost, and possible information loss. |
| Limitations | Reranking cannot fix missing candidates; compression can remove key details. |
| Production Example | Legal RAG retrieves many clauses, reranks them, then sends only relevant clause excerpts. |
| Interview Answer | Reranking improves the order of retrieved candidates, while context compression fits the most useful evidence into the LLM context window. |

## 3. Intermediate Explanation

Reranking approaches:

| Approach | How It Works | Best For | Risk |
| --- | --- | --- | --- |
| Score-based | sort by retrieval score | simple pipelines | retrieval score may be weak |
| Heuristic | boost source, recency, exact matches | business rules | manual tuning |
| Cross-encoder reranker | model compares query and candidate together | high precision | extra latency |
| LLM reranking | LLM judges relevance | complex relevance | cost and inconsistency |
| Rank fusion | merge dense and sparse rankings | hybrid retrieval | scoring complexity |

Compression approaches:

| Approach | Meaning | Risk |
| --- | --- | --- |
| Trimming | cut chunks by token budget | may remove evidence |
| Extractive compression | keep exact relevant sentences | safer for citations |
| Summarization | rewrite context shorter | can distort meaning |
| Metadata-aware selection | keep source, page, section | needs good metadata |

Data flow:

```text
Retrieve top 50 -> rerank -> keep top 5 -> compress -> build final prompt
```

## 4. Advanced Explanation

Reranking works best when the retriever has good recall but weak precision. If the correct document is not in the candidate set, reranking cannot recover it.

Optimization techniques:

- retrieve more candidates than final prompt needs
- rerank with a stronger relevance model
- use thresholds after reranking
- preserve citations during compression
- prefer extractive compression for high-risk domains
- allocate context budget by importance
- compare answer quality with and without reranking

Performance considerations:

- reranking top 100 is slower than top 20
- LLM reranking can be expensive
- compression can add another model call
- long compressed context can still be costly

Scaling considerations:

- use reranking only for high-value queries
- cache reranker results for repeated queries
- batch reranking when supported
- tune candidate count per corpus
- monitor latency by stage

Production challenges:

- reranker quality differs by domain
- compressed summaries may lose legal nuance
- citations may no longer map to exact text
- extra stages complicate debugging
- users may need original source inspection

## 5. Internal Working

```text
Question
  |
  v
Broad retrieval returns many candidates
  |
  v
Reranker scores each candidate against the question
  |
  v
Top candidates are selected
  |
  v
Context compressor extracts or summarizes relevant parts
  |
  v
Context builder creates final prompt with citations
```

Detailed lifecycle:

1. Retriever returns candidate chunks.
2. System logs original retrieval scores.
3. Reranker computes improved relevance scores.
4. Candidates are sorted by reranker score.
5. Top candidates are selected.
6. Context compression removes irrelevant text.
7. Citation metadata is preserved.
8. Prompt is built within token budget.
9. Answer quality and citations are evaluated.

## 6. When To Use

Use reranking when:

- retrieval returns relevant docs but not in top positions
- top-k contains too much noise
- answer quality improves when better chunks are selected
- legal, support, or technical precision matters

Use compression when:

- chunks are long
- context budget is tight
- retrieved chunks contain repeated boilerplate
- only specific sentences answer the question

## 7. When NOT To Use

Avoid reranking or compression when:

- retrieval already returns precise chunks
- latency budget is strict
- candidate count is tiny
- compression risks losing required exact language

Better alternatives:

- improve chunking
- improve metadata filters
- reduce top-k
- use parent-child retrieval
- use larger context only when justified

## 8. Advantages

- Improves precision.
- Reduces prompt noise.
- Helps fit context windows.
- Can lower generation cost.
- Improves citation quality.
- Supports enterprise-grade RAG answers.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Quality vs latency | Reranking improves relevance but adds time. |
| Compression vs fidelity | Shorter context may lose important nuance. |
| Cost vs precision | Better rerankers may cost more. |
| Automation vs auditability | Summarized context is harder to audit than exact excerpts. |

## 10. Limitations

- Reranking cannot rank missing candidates.
- Compression can remove evidence.
- LLM-based reranking can be inconsistent.
- Reranker scores are not truth scores.
- Citation mapping must be preserved carefully.

## 11. Real-World Examples

Startup example: support bot reranks FAQ candidates to improve answer precision.

Enterprise example: contract assistant compresses retrieved clauses into exact relevant excerpts.

FAANG-style example: a retrieval stack uses fast candidate generation, neural reranking, business-rule boosts, and answer evaluation.

Production system: a RAG pipeline retrieves 40 chunks, reranks to 8, compresses to 4 excerpts, then sends the final context to the LLM.

## 12. Architecture Diagram

```text
[Query]
   |
   v
[Retriever] -> top 50 candidates
   |
   v
[Reranker] -> top 8 candidates
   |
   v
[Context Compressor] -> compact excerpts
   |
   v
[Context Builder] -> prompt with citations
   |
   v
[LLM Answer]
```

## 13. Python Implementation

Candidate model:

```python
from dataclasses import dataclass

@dataclass
class CandidateChunk:
    chunk_id: str
    text: str
    retrieval_score: float
    rerank_score: float | None
    source: str
```

Simple rerank:

```python
def rerank_by_keyword_overlap(query: str, candidates: list[CandidateChunk]) -> list[CandidateChunk]:
    query_terms = set(query.lower().split())

    def score(candidate: CandidateChunk) -> int:
        candidate_terms = set(candidate.text.lower().split())
        return len(query_terms & candidate_terms)

    return sorted(candidates, key=score, reverse=True)
```

Extractive compression:

```python
def extract_relevant_sentences(query: str, text: str, max_sentences: int = 3) -> str:
    query_terms = set(query.lower().split())
    sentences = [sentence.strip() for sentence in text.split(".") if sentence.strip()]

    ranked = sorted(
        sentences,
        key=lambda sentence: len(query_terms & set(sentence.lower().split())),
        reverse=True,
    )
    return ". ".join(ranked[:max_sentences])
```

Context budget:

```python
def estimate_tokens(text: str) -> int:
    return max(1, len(text) // 4)

def fit_context(chunks: list[str], max_tokens: int) -> list[str]:
    selected: list[str] = []
    used = 0
    for chunk in chunks:
        cost = estimate_tokens(chunk)
        if used + cost > max_tokens:
            break
        selected.append(chunk)
        used += cost
    return selected
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class RerankRequest(BaseModel):
    query: str = Field(min_length=1)
    candidates: list[str]
    max_context_tokens: int = Field(default=3000, ge=500, le=12000)

class RerankResponse(BaseModel):
    selected_context: list[str]

@app.post("/retrieval/rerank-compress", response_model=RerankResponse)
async def rerank_compress(request: RerankRequest) -> RerankResponse:
    candidate_objects = [
        CandidateChunk(
            chunk_id=f"chunk-{index}",
            text=text,
            retrieval_score=1.0 / (index + 1),
            rerank_score=None,
            source="demo",
        )
        for index, text in enumerate(request.candidates)
    ]
    reranked = rerank_by_keyword_overlap(request.query, candidate_objects)
    compressed = [
        extract_relevant_sentences(request.query, candidate.text)
        for candidate in reranked
    ]
    return RerankResponse(selected_context=fit_context(compressed, request.max_context_tokens))
```

Production-ready structure:

```text
app/
  services/reranker.py
  services/context_compressor.py
  services/context_budget.py
  services/citation_mapper.py
  repositories/retrieval_log_repository.py
```

## 15. Database Integration

Store:

```text
retrieval_candidates(id, query_id, chunk_id, retrieval_score, rank)
rerank_results(id, query_id, chunk_id, rerank_score, rank)
compressed_context(id, query_id, chunk_id, compressed_text, token_count)
```

PostgreSQL:

- retrieval traces
- reranker scores
- selected chunks
- citation mappings

Redis:

- cache reranker outputs for repeated queries
- cache compressed context for common questions when safe

Vector DB:

- still performs initial candidate generation

## 16. Production Considerations

- Measure quality before and after reranking.
- Preserve citation source and page.
- Prefer extractive compression for legal/compliance use cases.
- Log compression output for debugging.
- Monitor latency added by reranking.
- Use timeout budgets.
- Use fallback if reranker fails.
- Do not compress away evidence required for citations.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Passing too many chunks to the LLM | Select and budget context |
| Beginner | Thinking top retrieval score is always best | Evaluate relevance |
| Intermediate | Reranking without measuring improvement | Compare against baseline |
| Intermediate | Summarizing context and losing citations | Preserve source mappings |
| Production | No latency budget | Set timeouts and stage metrics |
| Production | Compressing high-risk legal text incorrectly | Use extractive excerpts or human review |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is reranking? | Reordering retrieved candidates based on relevance to the query. |
| Basic | What is context compression? | Reducing retrieved text so useful evidence fits into the prompt budget. |
| Intermediate | When does reranking help? | When retrieval has good recall but weak precision or ordering. |
| Advanced | What are compression risks? | It can remove important evidence, distort meaning, or break citation traceability. |
| Scenario | RAG prompt is too large. | Reduce top-k, rerank, compress, improve chunking, or use parent-child retrieval. |

## 19. System Design Discussion

Reranking and compression sit between retrieval and generation. They are quality-control stages.

Design decisions:

- how many candidates to retrieve
- how many to rerank
- reranker model or heuristic
- compression type
- context token budget
- citation preservation
- fallback if reranking fails

## 20. Hands-On Assignment

- Easy: Sort candidate chunks by a rerank score.
- Medium: Implement extractive sentence compression.
- Hard: Build a context budgeter that keeps citations and fits a max token budget.

## 21. Mini Project

Build a Reranking Simulator.

Requirements:

- Accept query and candidate chunks.
- Score candidates using keyword overlap.
- Select top candidates.
- Compress selected chunks.
- Show before/after context length.

Folder structure:

```text
reranking-simulator/
  app/
    main.py
    reranker.py
    compressor.py
    budget.py
  tests/
    test_compressor.py
```

## 22. Production-Level Project

Add Reranking and Compression to Enterprise RAG.

Real-world problem:

- Retrieval finds too many chunks and LLM answers are noisy.

Architecture:

```text
[Retriever] -> [Candidate Pool] -> [Reranker] -> [Compressor] -> [Context Builder] -> [LLM]
```

Tech stack:

- FastAPI
- Vector DB
- PostgreSQL trace logs
- optional reranking model
- Redis cache

Scaling strategy:

- rerank only top N candidates
- cache repeated rerank results
- set per-stage timeout
- compare quality and latency dashboards
- preserve citations during compression

## Quiz

1. What is reranking?
2. What is context compression?
3. Why retrieve more candidates than final context needs?
4. What is recall?
5. What is precision?
6. Why can compression be risky?
7. What is extractive compression?
8. Why preserve citation mapping?
9. How do you measure reranking value?
10. What fallback would you use if reranking fails?

## Knowledge Check

You should be able to explain how reranking and compression improve RAG quality, and what they cost in latency, complexity, and risk.

Are you ready for the next section?
