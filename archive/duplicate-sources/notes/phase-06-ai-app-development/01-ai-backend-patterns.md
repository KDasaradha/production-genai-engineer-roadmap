# AI Backend Patterns

## 1. Problem Statement

AI backend patterns solve the problem of turning model calls into reliable, scalable, secure, and user-friendly applications.

A simple demo can call an LLM and print a response. A production AI app needs streaming, sessions, user history, document ingestion, queues, rate limits, usage tracking, observability, retries, caching, and security.

Without strong backend patterns:

- users wait too long without feedback
- long document processing times out
- model costs become unpredictable
- chat history is lost or messy
- failures are hard to debug
- one user can overload the system
- sensitive data may leak

Real-world analogy: the LLM is like an engine. The backend is the car around it: fuel system, brakes, dashboard, steering, safety, and maintenance.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | AI backend patterns are API, data, streaming, caching, queue, and observability patterns used to build production AI applications. |
| Key terminology | streaming, session, queue, worker, WebSocket, SSE, rate limit, usage tracking, model gateway |
| Simple explanation | AI apps need normal backend engineering plus model-specific concerns like latency, cost, tokens, and safety. |
| Mental model | Wrap the LLM in a real backend system. |
| Easy example | A chat API streams tokens instead of waiting for the full answer. |
| Use When | Building AI SaaS, chat apps, copilots, RAG apps, or workflow assistants. |
| Avoid When | A local notebook or one-off script is enough. |
| Advantages | Better UX, reliability, scale, and debugging. |
| Tradeoffs | More infrastructure and architecture. |
| Limitations | Backend patterns cannot fix bad prompts, bad retrieval, or weak models by themselves. |
| Production Example | Multi-session RAG chat with streaming, Redis rate limits, PostgreSQL history, vector search, and usage logs. |
| Interview Answer | AI backends combine API design with model latency, streaming, token accounting, retrieval, queues, caching, security, and observability. |

## 3. Intermediate Explanation

Core backend patterns:

| Pattern | Problem Solved | Example |
| --- | --- | --- |
| Streaming | model responses are slow | stream tokens via SSE |
| Sessions | chat needs continuity | store conversations and messages |
| Queues | long jobs exceed request time | document ingestion worker |
| Caching | repeated calls cost money | cache embeddings or stable answers |
| Rate limiting | users can overload cost/traffic | Redis token buckets |
| Model gateway | multiple providers/models | route by task and fallback |
| Usage tracking | cost visibility | store tokens and model cost |
| WebSockets | bidirectional updates | live agent progress |
| Background tasks | async post-processing | summarize completed chat |
| Observability | debugging production issues | logs, metrics, traces |

Typical AI backend components:

- FastAPI routes
- service layer
- model gateway
- retrieval service
- prompt builder
- context builder
- Redis cache/rate limiter
- PostgreSQL metadata/history
- vector database
- background workers
- observability pipeline

Data flow:

```text
Client -> FastAPI -> Auth/Rate Limit -> Service Layer -> Retrieval/LLM/Tools -> Streamed Response -> Logs
```

## 4. Advanced Explanation

Production AI backends must control four major dimensions:

1. Latency: users should see progress quickly.
2. Cost: token and model usage must be tracked and limited.
3. Quality: prompts, retrieval, and outputs must be evaluated.
4. Safety: auth, privacy, prompt injection, and tool permissions matter.

Optimization techniques:

- stream responses early
- use smaller models for simple tasks
- cache embeddings and stable results
- move ingestion to workers
- use batch embedding jobs
- enforce token budgets
- store prompt/model versions
- add fallback models
- add request IDs across all logs

Performance considerations:

- model latency dominates many requests
- vector search and reranking add latency
- streaming improves perceived latency, not total compute time
- large prompts increase cost and slow generation
- queues improve reliability for long tasks

Scaling considerations:

- horizontally scale FastAPI stateless API pods
- separately scale workers
- use connection pooling for PostgreSQL
- rate-limit per user and tenant
- cache repeated retrieval and embeddings
- monitor queue depth and provider rate limits

Production challenges:

- provider outages
- model rate limits
- runaway token usage
- duplicate background jobs
- stale cached answers
- slow ingestion pipelines
- streaming disconnects
- privacy in logs

## 5. Internal Working

```text
Request lifecycle:

User sends request
  |
  v
Auth and rate limit
  |
  v
Validate request model
  |
  v
Build prompt/context
  |
  v
Call retrieval/model/tool services
  |
  v
Stream or return response
  |
  v
Store usage, messages, logs, and feedback
```

Long job lifecycle:

```text
Upload document
  |
  v
Create ingestion job
  |
  v
Worker extracts text, chunks, embeds
  |
  v
Store metadata and vectors
  |
  v
Update job status
  |
  v
User queries processed document
```

## 6. When To Use

Use AI backend patterns when building:

- AI chat apps
- RAG platforms
- document assistants
- agent workflows
- coding assistants
- AI customer support
- AI interview platforms
- AI CRM assistants
- internal copilots

Ideal scenarios:

- users need responsive streaming
- files need background processing
- multiple users share one AI backend
- model usage must be tracked
- production reliability matters

## 7. When NOT To Use

Avoid heavy backend architecture when:

- you are only testing a prompt locally
- no user-facing app exists yet
- a notebook prototype is enough
- the feature has not been validated

Better alternatives:

- local script
- notebook
- simple CLI
- single FastAPI route
- managed no-code prototype for validation

## 8. Advantages

- Better user experience.
- Scalable architecture.
- Lower cost through caching and limits.
- Easier debugging through logs/traces.
- Safer production behavior.
- Clear portfolio and interview value.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Simplicity vs production readiness | More patterns mean more code but better reliability. |
| Streaming vs complexity | Streaming improves UX but adds disconnect/error handling. |
| Caching vs correctness | Cache can save cost but may return stale data. |
| Queueing vs immediacy | Queues improve reliability but add async status handling. |

## 10. Limitations

- Backend architecture cannot guarantee model truth.
- Caching is risky for personalized or sensitive answers.
- Streaming requires client and server support.
- Queues require operational monitoring.
- Provider rate limits can still bottleneck scale.

## 11. Real-World Examples

Startup example: AI SaaS app streams chat completions and stores user sessions.

Enterprise example: internal RAG platform processes documents in workers and enforces tenant permissions.

FAANG-style example: model gateway routes millions of AI requests by task type, cost, latency, and safety policy.

Production system: FastAPI API uses Redis rate limits, PostgreSQL chat history, vector DB retrieval, background ingestion workers, and OpenTelemetry traces.

## 12. Architecture Diagram

```text
[Frontend]
    |
    v
[FastAPI API]
    |
    +-> [Auth / Rate Limit - Redis]
    +-> [Chat Session - PostgreSQL]
    +-> [Retrieval Service - Vector DB]
    +-> [Model Gateway - LLM Providers]
    +-> [Background Queue - Workers]
    |
    v
[Streaming Response + Logs/Metrics]
```

## 13. Python Implementation

Streaming generator:

```python
def stream_words(text: str):
    for word in text.split():
        yield word + " "
```

Usage record:

```python
from dataclasses import dataclass

@dataclass
class ModelUsage:
    user_id: str
    model: str
    prompt_tokens: int
    completion_tokens: int
    latency_ms: int

    @property
    def total_tokens(self) -> int:
        return self.prompt_tokens + self.completion_tokens
```

Model gateway interface:

```python
class ModelGateway:
    def generate(self, prompt: str, model: str) -> str:
        raise NotImplementedError

    def stream(self, prompt: str, model: str):
        raise NotImplementedError
```

Rate limit key:

```python
def rate_limit_key(tenant_id: str, user_id: str) -> str:
    return f"rate-limit:{tenant_id}:{user_id}"
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import StreamingResponse
from pydantic import BaseModel, Field

app = FastAPI()

class ChatRequest(BaseModel):
    session_id: str
    message: str = Field(min_length=1)

class ChatResponse(BaseModel):
    session_id: str
    answer: str

@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest) -> ChatResponse:
    if not request.message.strip():
        raise HTTPException(status_code=400, detail="message is required")
    return ChatResponse(session_id=request.session_id, answer=f"received: {request.message}")

@app.post("/chat/stream")
async def stream_chat(request: ChatRequest) -> StreamingResponse:
    return StreamingResponse(
        stream_words(f"AI response for {request.message}"),
        media_type="text/plain",
    )
```

Production-ready structure:

```text
app/
  api/routes/chat.py
  api/routes/documents.py
  services/chat_service.py
  services/model_gateway.py
  services/usage_service.py
  services/rate_limit_service.py
  repositories/chat_repository.py
  repositories/usage_repository.py
  workers/ingestion_worker.py
```

Error handling:

- `400`: invalid request
- `401`: unauthenticated
- `403`: unauthorized resource
- `408`: model timeout
- `429`: rate limit exceeded
- `503`: model provider unavailable

## 15. Database Integration

PostgreSQL:

```text
users(id, email, plan, created_at)
chat_sessions(id, user_id, title, created_at, updated_at)
chat_messages(id, session_id, role, content, token_count, created_at)
model_usage(id, user_id, model, prompt_tokens, completion_tokens, latency_ms, cost, created_at)
documents(id, user_id, status, source_uri, created_at)
```

Redis:

- rate limits
- response cache
- query embedding cache
- ingestion job status
- active streaming state

Vector DB:

- document chunk embeddings
- semantic memory
- retrieval for RAG chat

## 16. Production Considerations

- Add request IDs.
- Track model latency and cost.
- Track token usage per user and tenant.
- Stream long responses.
- Move long jobs to workers.
- Add retries with backoff.
- Add provider fallback.
- Protect logs from sensitive data.
- Add rate limits and quotas.
- Monitor queue depth.
- Validate output before automation.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Calling LLM directly inside every route | Use service layer and model gateway |
| Beginner | No session model | Store sessions and messages |
| Intermediate | Long document ingestion inside request | Use background workers |
| Intermediate | No token usage tracking | Store usage for every model call |
| Production | No rate limits | Use Redis quotas per user/tenant |
| Production | Logging sensitive prompts | Redact or avoid storing sensitive content |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | Why do AI apps need streaming? | Model responses can be slow, and streaming improves perceived responsiveness. |
| Basic | Why use background workers? | Long tasks like document ingestion should not block API requests. |
| Intermediate | What is a model gateway? | A service boundary that routes model calls, tracks usage, handles fallback, and centralizes provider logic. |
| Advanced | How do you scale an AI backend? | Scale APIs and workers separately, cache safely, rate-limit users, track tokens/cost, use queues, and monitor provider limits. |
| Scenario | Users complain chat is slow. | Add streaming, inspect model latency, reduce prompt size, cache retrieval, route to faster models, and monitor bottlenecks. |

## 19. System Design Discussion

AI backend design sits at the intersection of normal backend engineering and model operations.

Design decisions:

- SSE vs WebSocket streaming
- sync vs async ingestion
- PostgreSQL schema for sessions
- Redis caching policy
- model gateway routing
- token quotas
- provider fallback
- observability fields
- privacy and retention

## 20. Hands-On Assignment

- Easy: Build a FastAPI chat endpoint with request and response models.
- Medium: Add a streaming endpoint.
- Hard: Add token usage tracking and Redis-style rate limit design.

## 21. Mini Project

Build an AI Streaming Chat API.

Requirements:

- Create chat sessions.
- Store messages.
- Stream responses.
- Track token usage.
- Add request IDs.
- Add basic rate-limit design.

Folder structure:

```text
ai-chat-api/
  app/
    main.py
    routes/chat.py
    services/chat_service.py
    services/model_gateway.py
    schemas.py
  tests/
    test_chat_api.py
```

## 22. Production-Level Project

Build an AI SaaS Backend Platform.

Real-world problem:

- Users need chat, document Q&A, usage tracking, and reliable AI responses.

Architecture:

```text
[Frontend] -> [FastAPI]
              |
              +-> [Auth]
              +-> [Chat Service]
              +-> [RAG Service]
              +-> [Model Gateway]
              +-> [Usage Service]
              +-> [PostgreSQL]
              +-> [Redis]
              +-> [Vector DB]
              +-> [Workers]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis
- vector database
- background workers
- LLM provider
- Docker

Scaling strategy:

- stateless API replicas
- separate worker scaling
- model fallback
- Redis rate limits
- token usage budgets
- observability dashboards
- async document ingestion

## Quiz

1. Why do AI backends need streaming?
2. What is the difference between SSE and WebSockets?
3. Why store chat sessions?
4. Why use background workers for ingestion?
5. What is a model gateway?
6. Why track token usage?
7. How does Redis help AI apps?
8. What should be stored in PostgreSQL?
9. How do you handle model provider outages?
10. How would you scale a RAG chat backend?

## Knowledge Check

You should be able to design a production AI backend with FastAPI, streaming, sessions, queues, Redis, PostgreSQL, model gateway, usage tracking, and observability.

Are you ready for the next section?
