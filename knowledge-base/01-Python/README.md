# Python

Python is the foundation for the backend and AI systems in this knowledge base. Study this folder before FastAPI, RAG, streaming, and agents.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [asyncio.md](asyncio.md) | AI apps call remote APIs, databases, and queues; async prevents blocking | Async I/O mental model |
| 2 | [generators.md](generators.md) | Streaming responses and token streams depend on lazy iteration | Streaming generator demo |
| 3 | [decorators.md](decorators.md) | FastAPI routes, auth wrappers, logging, and retries use decorators | Reusable API logging decorator |
| 4 | [context-managers.md](context-managers.md) | Files, DB sessions, locks, and resources need safe cleanup | Resource lifecycle pattern |
| 5 | [type-hints-mypy.md](type-hints-mypy.md) | FastAPI, Pydantic, agents, and service contracts depend on types | Typed service functions |
| 6 | [error-handling.md](error-handling.md) | Production systems need clear failures, retries, and logs | Error handling policy |
| 7 | [concurrency-models.md](concurrency-models.md) | You need to compare async, threads, processes, and workers | Backend concurrency decision table |

## Use When

Use this folder when you are weak in async, streaming, typing, error handling, or interview explanations for Python backend work.

## Avoid Skipping

Do not jump directly to RAG or agents if you cannot explain:

- async vs sync
- `yield` and streaming
- decorators and metadata preservation
- context managers and cleanup
- type hints vs runtime validation
- exception chaining and logging

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| Why use async in Python backends? | I/O-bound work, event loop, non-blocking awaits, limits of CPU-bound work |
| When use generators? | Large data, streaming, memory efficiency, lazy evaluation |
| Why use context managers? | Deterministic setup/cleanup, DB sessions, locks, error safety |
| Why type hints? | Readability, contracts, FastAPI/Pydantic integration, static checks |

## Project Connection

Apply these topics in [AI Streaming Chat API](../12-Projects/ai-streaming-chat-api.md), [Knowledge Assistant](../12-Projects/knowledge-assistant.md), and [Production AI Platform](../12-Projects/production-ai-platform.md).
