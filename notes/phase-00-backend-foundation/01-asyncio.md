# AsyncIO

## 1. Problem Statement

AsyncIO solves the problem of waiting on slow I/O such as network calls, database queries, file reads, and LLM API responses. Without it, one request can block a worker while it waits.

Analogy: a restaurant waiter does not stand frozen while one table thinks. They take other orders and return later.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | AsyncIO is Python's built-in system for writing concurrent I/O code with `async` and `await`. |
| Use When | Many tasks spend time waiting. |
| Avoid When | Work is CPU-heavy and needs true parallel computation. |
| Advantages | Better throughput for I/O-heavy APIs. |
| Tradeoffs | Harder debugging and accidental blocking bugs. |
| Limitations | Does not make CPU-heavy Python code faster by itself. |
| Example | Calling multiple APIs concurrently. |
| Production Example | FastAPI endpoint streaming LLM responses while other requests continue. |
| Interview Answer | AsyncIO lets a single thread manage many waiting tasks cooperatively, improving I/O concurrency. |

## 3. Intermediate Explanation

Main components are coroutines, the event loop, tasks, futures, and awaitable I/O libraries.

## 4. Advanced Explanation

Production async systems must avoid blocking calls, control timeouts, limit concurrency, and handle cancellation correctly.

## 5. Internal Working

```text
Request -> coroutine starts -> awaits I/O -> event loop runs another task -> I/O completes -> coroutine resumes -> response
```

## 6. When To Use

Use it for FastAPI endpoints, HTTP clients, WebSockets, streaming responses, and concurrent retrieval calls.

## 7. When NOT To Use

Avoid it for CPU-heavy ML preprocessing unless you move work to workers, processes, or external services.

## 8. Advantages

AsyncIO improves throughput, reduces idle waiting, and fits modern AI APIs where network latency dominates.

## 9. Tradeoffs

Async code requires async-compatible libraries. One blocking call can hurt the whole worker.

## 10. Limitations

It does not bypass the GIL for CPU-heavy work.

## 11. Real-World Examples

Startup: AI chatbot calling multiple tools. Enterprise: document service retrieving metadata and chunks concurrently. FAANG-style: high-throughput API gateway for model calls.

## 12. Architecture Diagram

```text
[FastAPI Request]
      |
      v
[Coroutine] --await--> [LLM API / DB / Redis]
      |
      v
[Response]
```

## 13. Python Implementation

```python
import asyncio

async def call_model(prompt: str) -> str:
    await asyncio.sleep(1)
    return f"answer for {prompt}"

async def main() -> None:
    results = await asyncio.gather(
        call_model("tokens"),
        call_model("embeddings"),
    )
    print(results)

asyncio.run(main())
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
async def health() -> dict[str, str]:
    return {"status": "ok"}
```

## 15. Database Integration

Use async drivers such as `asyncpg`, SQLAlchemy async sessions, and async Redis clients.

## 16. Production Considerations

Add timeouts, cancellation handling, connection pooling, request IDs, and concurrency limits.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Calling blocking libraries inside async endpoints | Use async-compatible clients |
| Intermediate | Unlimited `gather` calls | Use semaphores or queues |
| Production | No timeouts | Set timeout budgets per dependency |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is AsyncIO? | A cooperative concurrency model for I/O-bound Python code. |
| Intermediate | Is AsyncIO parallelism? | Not usually. It interleaves waiting tasks on an event loop. |
| Advanced | How can async code fail in production? | Blocking calls, leaked tasks, no timeouts, and unbounded concurrency. |
| Scenario | LLM API is slow. What do you do? | Use async calls, timeouts, retries, streaming, and concurrency limits. |

## 19. System Design Discussion

AsyncIO fits AI backends because model APIs, vector DBs, Redis, and databases all involve network waiting.

## 20. Hands-On Assignment

- Easy: Run two async sleeps concurrently.
- Medium: Call three HTTP endpoints concurrently.
- Hard: Add timeout and retry handling.

## 21. Mini Project

Build an async URL metadata fetcher with FastAPI.

## 22. Production-Level Project

Build an AI streaming chat API that handles concurrent users without blocking workers.

## Quiz

1. What problem does AsyncIO solve?
2. What does `await` do?
3. What is an event loop?
4. Why can blocking code break async performance?
5. Is AsyncIO good for CPU-heavy work?
6. What is `asyncio.gather` used for?
7. Why do production systems need timeouts?
8. How does AsyncIO help FastAPI?
9. What is unbounded concurrency?
10. When would you use worker processes instead?

## Knowledge Check

You should be able to explain async vs sync, identify blocking calls, and design a simple async FastAPI endpoint.

Are you ready for the next section?