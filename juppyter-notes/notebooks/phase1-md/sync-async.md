Since you want to learn this as a developer and apply it later to your GenAI roadmap, I'll teach it as a **backend-engineering bootcamp style** rather than only definitions. We'll build concepts layer by layer.

# Phase 0 — AI Backend Foundation

**Goal:** Build the backend skills that GenAI systems rely on.

Duration: **~2 weeks**

Topics:

1. AsyncIO
2. Decorators
3. Generators
4. Pydantic
5. FastAPI Dependency Injection
6. Middleware
7. PostgreSQL Optimization
8. Redis
9. Docker
10. Logging
11. Mini project: AI Streaming Chat API
12. Mini project: Background Job System

---

# Day 1: AsyncIO (Very Important)

Before GenAI, understand this:

Normal Python runs:

```python
task1()
task2()
task3()
```

Execution:

```
task1 → complete
task2 → complete
task3 → complete
```

This is **synchronous execution**.

Problem:

If `task1()` waits 5 seconds for an API response:

```python
import time

def fetch_data():
    time.sleep(5)
    return "Data"

print(fetch_data())
print("Finished")
```

Output:

```python
(wait 5 seconds)

Data
Finished
```

CPU sits idle.

---

## Async solution

Python can switch work while waiting.

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(5)
    return "Data"

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())
```

Explanation:

```python
async
```

Creates a coroutine.

```python
await
```

Pauses current task and lets Python do other work.

---

## Real-world example

Imagine:

A GenAI app receives:

* User 1 → Ask AI
* User 2 → Ask AI
* User 3 → Ask AI

Without async:

```python
User1 wait
User2 wait
User3 wait
```

With async:

```python
User1 processing
User2 processing
User3 processing
```

FastAPI heavily uses this.

---

## Concurrent execution

```python
import asyncio

async def task1():
    await asyncio.sleep(2)
    print("Task 1 done")

async def task2():
    await asyncio.sleep(1)
    print("Task 2 done")

async def main():
    await asyncio.gather(
        task1(),
        task2()
    )

asyncio.run(main())
```

Output:

```python
Task 2 done
Task 1 done
```

Why?

Because both run simultaneously.

---

# Practice Exercise 1

Create:

```python
async def get_user()
async def get_posts()
async def get_comments()
```

Rules:

* Each should sleep for random seconds

```python
await asyncio.sleep(2)
```

Run them together:

```python
asyncio.gather()
```

Expected:

```python
Fetching user...
Fetching posts...
Fetching comments...

User fetched
Comments fetched
Posts fetched
```

---

# Interview Focus

### Q1: Difference between sync and async?

**Answer:**

Synchronous executes sequentially.

Asynchronous allows execution without blocking while waiting for I/O operations.

---

### Q2: When should async be used?

Use async for:

✅ API calls
✅ Database calls
✅ AI model requests
✅ File operations

Avoid async for:

❌ Heavy CPU calculations

---

### Q3: Difference between concurrency and parallelism?

Concurrency:

```text
Task A
Task B
Task A
Task B
```

Parallelism:

```text
CPU1 → Task A
CPU2 → Task B
```

---

# Mini challenge

Build this:

```python
import asyncio

async def fetch_weather():
    pass

async def fetch_news():
    pass

async def fetch_stock():
    pass
```

Run all simultaneously and print results.

---

Tomorrow/next lesson:

**Day 2 → Decorators**

You'll learn:

* `@login_required`
* `@cache`
* `@timer`
* `@retry`
* how FastAPI internally uses decorators

These become extremely important later for Prompt Engineering + AI APIs.
