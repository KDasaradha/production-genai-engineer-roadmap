# RAG Pipeline

## 1. Problem Statement

RAG, or Retrieval-Augmented Generation, solves the problem of making LLMs answer using external knowledge that is private, current, large, or domain-specific.

An LLM by itself only answers from its training patterns and the context you send in the request. It does not automatically know your company's latest policy, your customer's ticket history, your uploaded PDF, or your internal documentation.

Without RAG:

- models may hallucinate facts
- private knowledge is unavailable
- updated documents are not reflected
- answers cannot cite sources
- companies may overuse fine-tuning for knowledge problems

Real-world analogy: RAG is like an open-book exam. The model is the student, retrieval is the act of finding the right pages, and generation is the final answer written from those pages.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | RAG is a pattern where the system retrieves relevant external context and gives it to an LLM before generating an answer. |
| Key terminology | ingestion, chunking, embeddings, vector DB, retriever, reranker, context builder, citations, grounding |
| Simple explanation | Search first, answer second. |
| Mental model | The LLM should answer from selected documents, not from memory alone. |
| Easy example | Ask a question about a company handbook; the system retrieves handbook sections and then answers. |
| Use When | The answer depends on external, private, or frequently changing knowledge. |
| Avoid When | A direct SQL query or simple rule is enough. |
| Advantages | Better grounding, source citations, and easier knowledge updates. |
| Tradeoffs | More components, latency, and failure modes. |
| Limitations | RAG reduces hallucination but does not eliminate it. |
| Production Example | Enterprise assistant retrieves permission-safe document chunks and returns cited answers. |
| Interview Answer | RAG retrieves relevant context from external data and passes it to an LLM so the answer is grounded in source material. |

## 3. Intermediate Explanation

A complete RAG pipeline has two main paths.

Ingestion path:

```text
Documents -> text extraction -> chunking -> embeddings -> vector DB + metadata DB
```

Query path:

```text
User question -> query embedding -> retrieval -> reranking -> context builder -> LLM -> answer with citations
```

Main components:

| Component | Role |
| --- | --- |
| Document loader | Accepts PDFs, Markdown, HTML, docs, or database records |
| Text extractor | Converts source files into clean text |
| Chunker | Splits long content into retrievable units |
| Embedding service | Converts chunks and queries into vectors |
| Vector database | Stores vectors for similarity search |
| Metadata store | Stores documents, chunks, permissions, citations, status |
| Retriever | Finds candidate chunks |
| Reranker | Improves ordering of retrieved chunks |
| Context builder | Creates the final prompt context |
| Generator | LLM that writes the final answer |
| Evaluator | Measures retrieval and answer quality |

Industry usage:

- internal knowledge assistants
- customer support bots
- policy Q&A
- technical documentation assistants
- legal contract analysis
- codebase assistants

## 4. Advanced Explanation

Production RAG is not one prompt. It is a distributed data and retrieval system with an LLM at the end.

Optimization techniques:

- use structure-aware chunking
- store rich metadata
- combine dense and sparse retrieval
- filter by tenant and permissions
- rerank candidates
- compress context
- require citations
- add refusal behavior when context is weak
- evaluate retrieval and answer quality separately

Performance considerations:

- embedding calls add ingestion cost
- retrieval adds query latency
- reranking improves quality but costs time
- long context increases model cost
- document ingestion may need queues and retries

Scaling considerations:

- asynchronous ingestion workers
- vector DB namespaces or tenant filters
- Redis cache for query embeddings
- PostgreSQL for metadata and audit logs
- observability for latency, cost, retrieval quality, and citation coverage

Production challenges:

- bad OCR
- missing documents
- stale embeddings
- permission leaks
- poor chunking
- hallucinated citations
- no-result queries
- model ignoring retrieved context

## 5. Internal Working

```text
Ingestion lifecycle:

File upload
  |
  v
Text extraction
  |
  v
Chunking with metadata
  |
  v
Embedding generation
  |
  v
Vector DB upsert
  |
  v
Metadata stored in PostgreSQL

Query lifecycle:

Question
  |
  v
Permission-aware retrieval
  |
  v
Rerank and filter
  |
  v
Build context with citations
  |
  v
LLM answer
  |
  v
Validate and log
```

Detailed lifecycle:

1. User uploads documents.
2. System extracts text and structure.
3. Text is split into chunks with source metadata.
4. Chunks are embedded.
5. Vectors are stored in a vector DB.
6. Metadata is stored in PostgreSQL.
7. User asks a question.
8. Query is embedded.
9. Retriever finds relevant chunks with permission filters.
10. Reranker selects the best chunks.
11. Context builder creates a grounded prompt.
12. LLM generates an answer.
13. System returns answer with citations.
14. Logs capture latency, cost, retrieved chunks, and feedback.

## 6. When To Use

Use RAG when:

- knowledge changes often
- documents are too large for prompts
- answers need citations
- data is private or tenant-specific
- you need source-grounded answers
- fine-tuning would be the wrong tool for facts

Ideal use cases:

- company knowledge assistant
- documentation assistant
- support assistant
- legal document Q&A
- compliance assistant
- research assistant over controlled data

## 7. When NOT To Use

Avoid RAG when:

- a SQL query gives the exact answer
- the task is pure text generation
- the knowledge base is tiny
- the source data is low quality
- user question requires live actions rather than documents

Better alternatives:

- direct database query
- API/tool calling
- rules engine
- normal search
- fine-tuning for behavior, not facts

## 8. Advantages

- Uses current and private knowledge.
- Reduces unsupported hallucination.
- Supports citations and traceability.
- Avoids retraining for knowledge updates.
- Can enforce document permissions.
- Works well with FastAPI backend systems.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Quality vs complexity | Better grounding requires more components. |
| Latency vs relevance | Retrieval and reranking add time. |
| Cost vs reliability | More validation and reranking cost more. |
| Freshness vs processing load | Frequent updates require re-ingestion and re-embedding. |

## 10. Limitations

- RAG cannot answer from missing documents.
- Bad retrieval causes bad answers.
- Citations can still be wrong.
- Long context can confuse the model.
- It requires operational monitoring.
- It does not replace access control.

## 11. Real-World Examples

Startup example: a SaaS support bot answers customer questions from public help docs.

Enterprise example: an HR assistant answers employee policy questions only from documents the employee can access.

FAANG-style example: an assistant platform uses multiple retrievers, rerankers, model routing, evaluation, and feedback loops.

Production system: a RAG API ingests documents asynchronously, stores metadata in PostgreSQL, stores vectors in Qdrant/Pinecone/pgvector, uses Redis for quotas, and streams cited answers from FastAPI.

## 12. Architecture Diagram

```text
Ingestion:

[Upload API]
    |
    v
[Object Storage] -> [Extractor] -> [Chunker] -> [Embedding Worker]
                                           |              |
                                           v              v
                                    [PostgreSQL]      [Vector DB]

Query:

[User] -> [FastAPI Query API] -> [Retriever] -> [Reranker]
                                      |             |
                                      v             v
                                [Vector DB]   [Context Builder]
                                                    |
                                                    v
                                                  [LLM]
                                                    |
                                                    v
                                           [Cited Answer + Logs]
```

## 13. Python Implementation

RAG prompt builder:

```python
from dataclasses import dataclass

@dataclass
class RetrievedChunk:
    chunk_id: str
    text: str
    source: str
    score: float

def build_rag_prompt(question: str, chunks: list[RetrievedChunk]) -> str:
    if not chunks:
        return "Answer: I do not have enough information in the provided sources."

    context = "\n\n".join(
        f"[source: {chunk.source}, chunk: {chunk.chunk_id}]\n{chunk.text}"
        for chunk in chunks
    )

    return f"""Answer the question using only the context below.
If the context is insufficient, say you do not have enough information.
Cite sources using the source labels.

Context:
{context}

Question:
{question}

Answer:"""
```

Simple context selector:

```python
def select_context(chunks: list[RetrievedChunk], min_score: float, max_chunks: int) -> list[RetrievedChunk]:
    filtered = [chunk for chunk in chunks if chunk.score >= min_score]
    return sorted(filtered, key=lambda chunk: chunk.score, reverse=True)[:max_chunks]
```

RAG service boundary:

```python
class RagService:
    def answer(self, question: str, user_id: str) -> str:
        raise NotImplementedError
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class RagQueryRequest(BaseModel):
    question: str = Field(min_length=1)
    tenant_id: str
    top_k: int = Field(default=5, ge=1, le=20)

class Citation(BaseModel):
    source: str
    chunk_id: str

class RagQueryResponse(BaseModel):
    answer: str
    citations: list[Citation]

@app.post("/rag/query", response_model=RagQueryResponse)
async def rag_query(request: RagQueryRequest) -> RagQueryResponse:
    # In production: retrieve, rerank, build prompt, call LLM, validate citations.
    chunks = [
        RetrievedChunk(
            chunk_id="chunk-1",
            text="Refunds are processed within 7 business days.",
            source="refund-policy.md",
            score=0.91,
        )
    ]

    selected = select_context(chunks, min_score=0.5, max_chunks=request.top_k)
    if not selected:
        raise HTTPException(status_code=404, detail="no relevant context found")

    return RagQueryResponse(
        answer="Refunds are processed within 7 business days. [source: refund-policy.md]",
        citations=[Citation(source=chunk.source, chunk_id=chunk.chunk_id) for chunk in selected],
    )
```

Production-ready structure:

```text
app/
  api/routes/rag.py
  services/rag_service.py
  services/retrieval_service.py
  services/context_builder.py
  services/llm_service.py
  repositories/document_repository.py
  repositories/retrieval_log_repository.py
```

## 15. Database Integration

PostgreSQL:

```text
documents(id, tenant_id, title, source_uri, status, checksum)
chunks(id, document_id, text, page_number, section, token_count)
rag_queries(id, user_id, tenant_id, question, answer, latency_ms, created_at)
rag_citations(id, query_id, chunk_id, source, score, rank)
```

Vector DB:

- chunk ID
- embedding vector
- tenant ID
- source metadata
- permission metadata
- embedding version

Redis:

- rate limits
- query embedding cache
- response cache when safe
- ingestion job status

## 16. Production Considerations

- Enforce auth and permissions before retrieval.
- Log retrieved chunks and scores.
- Return citations.
- Add refusal when context is weak.
- Track token usage and cost.
- Monitor answer feedback.
- Evaluate retrieval and answer quality separately.
- Store prompt, model, embedding, and chunking versions.
- Use queues for ingestion.
- Add retry policies for model and embedding providers.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking RAG is just "put docs in prompt" | Build ingestion, retrieval, and context selection |
| Beginner | Sending full documents to the model | Chunk, retrieve, and send only relevant context |
| Intermediate | No citations | Return source metadata with answers |
| Intermediate | No retrieval logs | Store retrieved chunks and scores |
| Production | No permission filtering | Enforce tenant and document access before generation |
| Production | Assuming RAG eliminates hallucination | Add validation, refusal, evaluation, and monitoring |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is RAG? | Retrieval-Augmented Generation retrieves external context and gives it to an LLM before answering. |
| Basic | Why use RAG? | To answer from current, private, or large knowledge sources without retraining the model. |
| Intermediate | What are the main RAG pipeline stages? | Ingestion, chunking, embeddings, storage, retrieval, reranking, context building, generation, citations, evaluation. |
| Advanced | Why can RAG still hallucinate? | Retrieval can miss sources, context can be weak, citations can be wrong, or the model can ignore context. |
| Scenario | RAG answer is wrong. | Inspect retrieved chunks, scores, filters, chunking, reranking, prompt, model output, and citation validation. |

## 19. System Design Discussion

RAG fits into large systems as a knowledge access layer. It connects document ingestion, search, access control, generation, and monitoring.

Design decisions:

- chunk size and overlap
- dense, sparse, or hybrid retrieval
- metadata filters
- reranking
- context budget
- citation format
- no-context fallback
- streaming vs non-streaming response
- sync vs async ingestion

## 20. Hands-On Assignment

- Easy: Build a RAG prompt from three chunks.
- Medium: Add source citations to each chunk.
- Hard: Build a retrieval log object that stores question, chunk IDs, scores, and answer.

## 21. Mini Project

Build a Documentation Assistant.

Requirements:

- Load Markdown documents.
- Split them into chunks.
- Mock or generate embeddings.
- Retrieve relevant chunks.
- Build a grounded prompt.
- Return answer with citations.

Folder structure:

```text
docs-assistant/
  app/
    main.py
    chunker.py
    retriever.py
    context_builder.py
    schemas.py
  tests/
    test_context_builder.py
```

## 22. Production-Level Project

Build an Enterprise RAG Platform.

Real-world problem:

- Employees need reliable answers from internal documents with permissions and citations.

Architecture:

```text
[Upload API] -> [Object Storage] -> [Ingestion Queue]
                                  -> [Extractor]
                                  -> [Chunker]
                                  -> [Embedding Worker]
                                  -> [Vector DB]
                                  -> [PostgreSQL]

[Query API] -> [Auth] -> [Retriever] -> [Reranker] -> [LLM] -> [Cited Answer]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- Vector database
- Object storage
- Background workers
- LLM API

Scaling strategy:

- asynchronous ingestion
- batch embeddings
- tenant-aware indexes
- query caching
- model streaming
- observability dashboards
- retrieval and answer evaluation

## Quiz

1. What does RAG stand for?
2. What problem does RAG solve?
3. What are the ingestion stages?
4. What are the query stages?
5. Why is chunking important?
6. Why use embeddings?
7. Why are citations important?
8. Why can RAG still hallucinate?
9. What should be logged for a RAG query?
10. How would you design permission-safe RAG?

## Knowledge Check

You should be able to draw a full RAG pipeline, explain each component, and debug the difference between retrieval failure and generation failure.

Are you ready for the next section?
