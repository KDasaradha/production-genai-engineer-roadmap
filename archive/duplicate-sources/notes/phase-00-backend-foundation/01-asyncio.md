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

---
# AsyncIO — Beginner to Advanced (Interview Ready)

If Decorators and Generators are important, **AsyncIO is critical** for FastAPI interviews.

Most FastAPI interview questions eventually lead to:

> Why do we write `async def` in FastAPI?

To answer that properly, you need AsyncIO.

---

# 1. What Problem Does AsyncIO Solve?

Imagine a restaurant.

### Traditional (Synchronous)

```text
Waiter takes Order 1
Waits for kitchen
Delivers food

Then takes Order 2
Waits for kitchen
Delivers food
```

One task at a time.

---

### AsyncIO

```text
Take Order 1
Kitchen cooking

Take Order 2
Kitchen cooking

Take Order 3
Kitchen cooking

Serve whichever finishes first
```

Multiple waiting tasks progress together.

---

# 2. Synchronous Programming

Example:

```python
import time

def task1():
    print("Task1 Started")
    time.sleep(3)
    print("Task1 Finished")

def task2():
    print("Task2 Started")
    time.sleep(3)
    print("Task2 Finished")

task1()
task2()
```

Execution:

```text
Task1 Started
(wait 3 sec)
Task1 Finished

Task2 Started
(wait 3 sec)
Task2 Finished
```

Total:

```text
6 seconds
```

---

# 3. Async Programming

```python
import asyncio

async def task1():
    print("Task1 Started")
    await asyncio.sleep(3)
    print("Task1 Finished")

async def task2():
    print("Task2 Started")
    await asyncio.sleep(3)
    print("Task2 Finished")
```

Run together:

```python
async def main():
    await asyncio.gather(
        task1(),
        task2()
    )

asyncio.run(main())
```

Output:

```text
Task1 Started
Task2 Started

(wait 3 sec)

Task1 Finished
Task2 Finished
```

Total:

```text
3 seconds
```

---

# 4. Key AsyncIO Concepts

| Concept    | Meaning                 |
| ---------- | ----------------------- |
| async      | Defines coroutine       |
| await      | Pause current task      |
| coroutine  | Async function          |
| event loop | Manages async tasks     |
| task       | Scheduled coroutine     |
| gather()   | Run multiple coroutines |

---

# 5. What is a Coroutine?

Normal function:

```python
def greet():
    return "Hello"
```

Async function:

```python
async def greet():
    return "Hello"
```

This is called a **Coroutine**.

---

Calling it:

```python
greet()
```

Output:

```text
<coroutine object>
```

Nothing executes yet.

---

Need:

```python
await greet()
```

or

```python
asyncio.run(greet())
```

---

# 6. Understanding await

Example:

```python
async def fetch():
    await asyncio.sleep(2)
```

Meaning:

```text
I'm waiting.
Someone else can use CPU.
```

Not:

```text
Block entire program.
```

---

# 7. Event Loop

The Event Loop is the heart of AsyncIO.

Think of it as a manager.

```text
Task A waiting
Task B ready

Run Task B

Task B waiting
Task A ready

Run Task A
```

---

Visual:

```text
Event Loop
     │
 ┌───┼───┐
 │   │   │
Task1 Task2 Task3
```

---

# 8. asyncio.run()

Entry point.

```python
asyncio.run(main())
```

Creates:

* Event Loop
* Runs coroutine
* Closes loop

---

# 9. Multiple Tasks Using gather()

Example:

```python
import asyncio

async def fetch_user():
    await asyncio.sleep(2)
    return "User"

async def fetch_orders():
    await asyncio.sleep(2)
    return "Orders"
```

Sequential:

```python
user = await fetch_user()
orders = await fetch_orders()
```

Time:

```text
4 sec
```

---

Concurrent:

```python
user, orders = await asyncio.gather(
    fetch_user(),
    fetch_orders()
)
```

Time:

```text
2 sec
```

---

# 10. create_task()

Schedules task in background.

```python
task = asyncio.create_task(fetch_user())
```

Task starts immediately.

Later:

```python
result = await task
```

---

Example:

```python
async def main():

    task = asyncio.create_task(fetch_user())

    print("Doing other work")

    user = await task
```

---

# 11. Real FastAPI Example

Bad:

```python
@app.get("/users")
def get_users():

    users = db.fetch_all()

    return users
```

Blocks worker.

---

Good:

```python
@app.get("/users")
async def get_users():

    users = await db.fetch_all()

    return users
```

Worker can serve other requests while waiting.

---

# 12. Why FastAPI Uses AsyncIO

Typical API:

```text
Receive Request
      ↓
Database Call
      ↓
External API Call
      ↓
Return Response
```

Most time spent waiting.

AsyncIO handles waiting efficiently.

---

# 13. Async HTTP Requests

Very common.

Using [HTTPX Official Docs](https://www.python-httpx.org/?utm_source=chatgpt.com)

```python
import httpx

async def get_users():

    async with httpx.AsyncClient() as client:

        response = await client.get(
            "https://example.com/users"
        )

        return response.json()
```

---

# 14. Async Database Example

```python
async def get_user():

    result = await db.fetch_one(
        "SELECT * FROM users"
    )

    return result
```

Database is waiting.

Event loop serves others meanwhile.

---

# 15. Async Context Manager

Normal:

```python
with open("file.txt") as f:
    pass
```

Async:

```python
async with client:
    pass
```

Example:

```python
async with httpx.AsyncClient() as client:
    ...
```

---

# 16. Async Iterator

Normal:

```python
for item in items:
    pass
```

Async:

```python
async for item in stream:
    pass
```

Useful:

* Streaming APIs
* WebSockets
* Large data streams

---

# 17. CPU Bound vs IO Bound

This is a favorite interview question.

---

## IO Bound

Waiting for:

* Database
* APIs
* Network
* Files

Example:

```python
await db.fetch()
```

Use AsyncIO.

---

## CPU Bound

Heavy computation:

```python
for i in range(100000000):
    ...
```

Examples:

* Image processing
* ML training
* Video rendering

Do NOT use AsyncIO.

Use:

* Multiprocessing
* Celery
* Ray

---

# 18. Common Mistakes

## Mistake 1

```python
time.sleep(5)
```

Inside async function.

Bad.

Blocks event loop.

---

Use:

```python
await asyncio.sleep(5)
```

---

## Mistake 2

```python
requests.get(url)
```

Inside async function.

Bad.

Blocking.

---

Use:

```python
await client.get(url)
```

with async HTTP client.

---

# 19. Sequential vs Concurrent

Sequential:

```python
a = await fetch_a()
b = await fetch_b()
c = await fetch_c()
```

Time:

```text
6 sec
```

---

Concurrent:

```python
a, b, c = await asyncio.gather(
    fetch_a(),
    fetch_b(),
    fetch_c()
)
```

Time:

```text
2 sec
```

---

# 20. How FastAPI Handles Requests

```text
Request 1
   ↓
Waiting DB
   ↓

Request 2 arrives
   ↓
Handled immediately

Request 3 arrives
   ↓
Handled immediately
```

Because worker isn't blocked.

---

# AsyncIO Interview Questions

## Q1. What is AsyncIO?

**Answer**

AsyncIO is Python's framework for writing concurrent programs using coroutines, an event loop, and non-blocking I/O operations.

---

## Q2. Difference Between async and await?

**Answer**

* `async` defines a coroutine.
* `await` pauses execution until an awaitable completes.

---

## Q3. What is a Coroutine?

**Answer**

A coroutine is an asynchronous function defined using `async def`.

---

## Q4. What is an Event Loop?

**Answer**

The event loop manages and schedules asynchronous tasks, switching between them whenever a task is waiting.

---

## Q5. Difference Between Concurrency and Parallelism?

```text
Concurrency
Multiple tasks progressing

Parallelism
Multiple tasks executing simultaneously
```

AsyncIO provides concurrency, not true parallel CPU execution.

---

## Q6. When Should You Use AsyncIO?

Use AsyncIO for:

* APIs
* Database calls
* External services
* Web scraping
* Streaming

Avoid it for CPU-intensive computations.

---

# Interview Answer (2-Minute Version)

> AsyncIO is Python's asynchronous programming framework that enables concurrent execution of I/O-bound tasks using coroutines and an event loop. Functions are defined with `async def` and paused using `await`, allowing other tasks to run while waiting for operations like database queries or API calls. This improves scalability and resource utilization. FastAPI uses AsyncIO extensively to handle many concurrent requests efficiently without blocking worker threads.

---

# What You Must Know for FastAPI Interviews

### Beginner

* `async`
* `await`
* Coroutine
* Event Loop

### Intermediate

* `asyncio.run()`
* `gather()`
* `create_task()`
* Async Context Managers

### Advanced

* Async DB access
* Async HTTP clients
* FastAPI internals
* IO-bound vs CPU-bound
* Event loop behavior

After AsyncIO, the next most valuable topic for your FastAPI + GenAI path is **Type Hints + mypy**, because FastAPI, Pydantic, LangChain, and modern Python codebases rely heavily on type annotations.
