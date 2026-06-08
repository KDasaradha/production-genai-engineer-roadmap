# RAG Frameworks

## 1. Problem Statement

RAG frameworks solve the problem of repeatedly wiring loaders, splitters, retrievers, prompts, LLM calls, tools, and workflows from scratch.

Frameworks can help you move faster, but they can also hide important production behavior. A strong engineer understands the RAG pipeline first, then uses frameworks selectively.

Without frameworks:

- prototypes may take longer
- integrations require more custom code
- common patterns are reimplemented repeatedly

With frameworks used poorly:

- debugging becomes harder
- business logic hides inside chains
- dependency changes break behavior
- production observability is weak

Real-world analogy: a framework is scaffolding. It helps you build faster, but the building still needs proper architecture.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | RAG frameworks provide reusable components for document loading, indexing, retrieval, generation, and workflow orchestration. |
| Key terminology | loader, splitter, retriever, chain, index, workflow, graph, callback, trace |
| Simple explanation | Frameworks give prebuilt pieces for RAG apps. |
| Mental model | Use frameworks as tools, not magic. |
| Easy example | Use a framework loader to read PDFs and a retriever to search chunks. |
| Use When | You need faster prototyping or standard integrations. |
| Avoid When | Framework abstractions make a simple system harder to debug. |
| Advantages | Faster development, integrations, reusable patterns. |
| Tradeoffs | Lock-in, hidden behavior, fast-changing APIs. |
| Limitations | Frameworks do not replace evaluation, security, or system design. |
| Production Example | A RAG service wraps LlamaIndex ingestion behind internal service interfaces. |
| Interview Answer | RAG frameworks speed up development, but production systems should wrap them behind clear interfaces and still control data flow, observability, and evaluation. |

## 3. Intermediate Explanation

Common frameworks:

| Framework | Strength | Use Case |
| --- | --- | --- |
| LangChain | chains, tools, retrievers, integrations | prototypes and app workflows |
| LlamaIndex | document ingestion and indexing | RAG over documents |
| LangGraph | stateful workflows and agents | agentic RAG and controlled graphs |
| Haystack | search/RAG pipelines | production-oriented retrieval pipelines |
| Semantic Kernel | orchestration and enterprise integration | Microsoft ecosystem workflows |

Framework components:

- document loaders
- text splitters
- embedding wrappers
- vector store adapters
- retrievers
- prompt templates
- output parsers
- workflow graphs
- tracing callbacks

Data flow:

```text
Your app -> framework wrapper -> retriever/LLM/vector DB -> framework output -> your response contract
```

## 4. Advanced Explanation

Production framework usage should protect your architecture from framework churn.

Optimization techniques:

- wrap framework code in service classes
- keep route handlers framework-free
- define your own request/response schemas
- log inputs, outputs, latency, and errors around framework calls
- test each boundary
- pin framework versions
- avoid mixing business rules deeply into chains
- keep metadata and permissions in your own data model

Performance considerations:

- framework defaults may be inefficient
- callbacks and tracing add overhead
- hidden prompt templates may increase token cost
- default chunking may not fit your corpus

Scaling considerations:

- decouple ingestion from query path
- use background workers
- persist framework outputs in your databases
- support replacement of framework components
- monitor dependency updates

Production challenges:

- breaking API changes
- debugging nested chains
- unclear retry behavior
- difficult tracing
- hidden prompt or retriever settings
- framework lock-in

## 5. Internal Working

```text
Application request
  |
  v
Your service interface
  |
  v
Framework pipeline:
  - loader
  - splitter
  - retriever
  - prompt
  - LLM
  |
  v
Your validator and response model
  |
  v
Application response
```

Detailed lifecycle:

1. Your API receives request.
2. Route calls your service.
3. Service calls framework component.
4. Framework executes retrieval or generation.
5. Your code validates result.
6. Your code logs trace and metrics.
7. Your response model is returned.

## 6. When To Use

Use RAG frameworks when:

- building prototypes quickly
- using common document loaders
- integrating many vector stores
- experimenting with retrieval patterns
- building graph workflows
- team accepts framework dependency

Ideal use cases:

- proof of concept
- ingestion pipeline prototype
- internal tools
- workflow orchestration
- rapid comparison of retrievers

## 7. When NOT To Use

Avoid frameworks when:

- a small custom implementation is clearer
- strict performance control matters
- framework hides critical behavior
- dependency stability is a concern
- team does not understand underlying RAG

Better alternatives:

- custom service classes
- direct vector DB SDK
- direct LLM SDK
- small internal abstractions
- SQL and search APIs directly

## 8. Advantages

- Faster prototyping.
- Many integrations.
- Common abstractions.
- Helpful loaders and splitters.
- Easier experimentation.
- Useful for agentic workflows.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Speed vs control | Frameworks speed up work but hide details. |
| Integrations vs lock-in | Many adapters can create dependency coupling. |
| Abstraction vs debugging | Chains can be hard to inspect. |
| Defaults vs production needs | Default settings may not fit your corpus or business rules. |

## 10. Limitations

- Frameworks cannot guarantee good retrieval.
- They do not solve permissions automatically.
- They do not replace evaluation.
- They may change quickly.
- They can add unnecessary complexity.
- They still require observability and testing.

## 11. Real-World Examples

Startup example: use LlamaIndex to prototype a document Q&A system quickly.

Enterprise example: wrap LangChain retrievers behind an internal `RetrievalService` so the framework can be replaced.

FAANG-style example: internal AI platform provides framework-like abstractions but keeps tracing, security, and evaluation centralized.

Production system: a FastAPI RAG service uses framework loaders for ingestion but custom retrieval, auth, logging, and response validation.

## 12. Architecture Diagram

```text
[FastAPI Route]
      |
      v
[Your RagService Interface]
      |
      v
[Framework Adapter]
      |
      +-> [Loader / Splitter]
      +-> [Retriever]
      +-> [Prompt Template]
      +-> [LLM]
      |
      v
[Your Validation + Logs]
      |
      v
[API Response]
```

Good boundary:

```text
Your app depends on your interface.
Only the adapter depends on the framework.
```

## 13. Python Implementation

Define an internal interface:

```python
from dataclasses import dataclass

@dataclass
class RetrievalResult:
    chunk_id: str
    text: str
    score: float
    source: str

class Retriever:
    def search(self, query: str, tenant_id: str, top_k: int) -> list[RetrievalResult]:
        raise NotImplementedError
```

Framework adapter shape:

```python
class FrameworkRetrieverAdapter(Retriever):
    def __init__(self, framework_retriever: object) -> None:
        self.framework_retriever = framework_retriever

    def search(self, query: str, tenant_id: str, top_k: int) -> list[RetrievalResult]:
        # In production, call the framework retriever and map results to your schema.
        return []
```

Custom fallback:

```python
class CustomRetriever(Retriever):
    def search(self, query: str, tenant_id: str, top_k: int) -> list[RetrievalResult]:
        return []
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel, Field

app = FastAPI()

class FrameworkSearchRequest(BaseModel):
    query: str = Field(min_length=1)
    tenant_id: str
    top_k: int = Field(default=5, ge=1, le=20)

class FrameworkSearchResponse(BaseModel):
    results: list[str]
    retriever_backend: str

def get_retriever() -> Retriever:
    return CustomRetriever()

@app.post("/rag/framework-search", response_model=FrameworkSearchResponse)
async def framework_search(
    request: FrameworkSearchRequest,
    retriever: Retriever = Depends(get_retriever),
) -> FrameworkSearchResponse:
    results = retriever.search(request.query, request.tenant_id, request.top_k)
    return FrameworkSearchResponse(
        results=[result.chunk_id for result in results],
        retriever_backend=retriever.__class__.__name__,
    )
```

Production-ready structure:

```text
app/
  interfaces/retriever.py
  adapters/langchain_retriever.py
  adapters/llamaindex_ingestion.py
  services/rag_service.py
  api/routes/rag.py
  tests/test_retriever_contract.py
```

## 15. Database Integration

Keep core data in your own stores:

```text
documents(id, tenant_id, title, source_uri, status)
chunks(id, document_id, text, metadata_json)
retrieval_logs(id, query, retriever_backend, latency_ms)
framework_runs(id, framework_name, version, config_json, status)
```

Store:

- framework name
- framework version
- config
- retriever backend
- prompt template version
- trace IDs

Do not rely only on framework memory or internal objects for production records.

## 16. Production Considerations

- Pin framework versions.
- Wrap framework calls behind interfaces.
- Add tracing around every framework boundary.
- Validate outputs using your schemas.
- Keep auth and permissions in backend code.
- Avoid route handlers full of chain logic.
- Test framework adapters with contract tests.
- Monitor latency, token use, and errors.
- Keep a migration path if replacing the framework.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Thinking framework equals architecture | Learn RAG components first |
| Beginner | Copying tutorials into production | Refactor into services and interfaces |
| Intermediate | Putting framework code in routes | Use adapters and service boundaries |
| Intermediate | Trusting framework defaults | Tune chunking, retrieval, prompts, and filters |
| Production | No version pinning | Pin dependencies and test upgrades |
| Production | No observability | Trace framework calls and log configs |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | Why use a RAG framework? | To speed up common tasks like loading documents, splitting text, retrieval, and workflow orchestration. |
| Intermediate | What is the risk of frameworks? | They can hide behavior, create lock-in, and make debugging harder. |
| Intermediate | How should frameworks be used in FastAPI? | Behind service interfaces and adapters, not directly inside route handlers. |
| Advanced | How do you make framework-based RAG production-ready? | Add tests, tracing, version pinning, permission checks, validation, evaluation, and clear replacement boundaries. |
| Scenario | A framework upgrade breaks retrieval. | Roll back dependency, use adapter contract tests, inspect config changes, and pin versions. |

## 19. System Design Discussion

Frameworks should support your architecture, not own it.

Design decisions:

- direct SDK vs framework
- framework adapter boundary
- data ownership
- prompt ownership
- tracing strategy
- dependency version policy
- migration path

Strong principle:

```text
Use frameworks for speed.
Use your own interfaces for control.
```

## 20. Hands-On Assignment

- Easy: Compare LangChain, LlamaIndex, LangGraph, and Haystack.
- Medium: Define a `Retriever` interface independent of any framework.
- Hard: Write an adapter contract test that any retriever implementation must pass.

## 21. Mini Project

Build a Replaceable Retriever Demo.

Requirements:

- Define a retriever interface.
- Implement a custom mock retriever.
- Implement a framework-style adapter stub.
- Expose FastAPI endpoint using dependency injection.
- Add tests that both retrievers return the same response shape.

Folder structure:

```text
replaceable-retriever/
  app/
    main.py
    interfaces.py
    adapters.py
    schemas.py
  tests/
    test_retriever_contract.py
```

## 22. Production-Level Project

Build a Framework-Wrapped RAG Service.

Real-world problem:

- Team wants framework speed but production control and observability.

Architecture:

```text
[FastAPI] -> [RagService]
              |
              +-> [Retriever Interface]
              |       |
              |       +-> [Framework Adapter]
              |       +-> [Custom Retriever]
              |
              +-> [Prompt Builder]
              +-> [LLM Service]
              +-> [Trace Logger]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- Vector DB
- chosen RAG framework
- pytest

Scaling strategy:

- isolate framework adapters
- pin versions
- run contract tests
- trace all framework calls
- preserve source metadata in your DB
- keep rollback path to previous adapter version

## Quiz

1. Why use RAG frameworks?
2. What problem does LlamaIndex often help with?
3. What problem does LangGraph often help with?
4. What is framework lock-in?
5. Why wrap framework code behind interfaces?
6. Why should route handlers avoid chain logic?
7. What should be logged around framework calls?
8. Why pin dependency versions?
9. When is custom code better than a framework?
10. How would you migrate away from a framework?

## Knowledge Check

You should be able to use RAG frameworks pragmatically while preserving production control, testing, observability, and replaceability.

Are you ready for the next section?
