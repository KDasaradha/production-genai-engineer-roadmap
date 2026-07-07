# Redis and Caching

## 1. Problem Statement

Redis solves fast temporary storage needs: cache, sessions, rate limits, queues, locks, and pub/sub.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Redis is an in-memory data store commonly used for caching and coordination. |
| Use When | You need low-latency temporary data. |
| Avoid When | You need primary durable relational storage. |
| Advantages | Very fast reads and writes. |
| Tradeoffs | Cache invalidation and memory cost. |
| Limitations | Memory is finite; persistence requires configuration. |
| Example | Cache an expensive model result. |
| Production Example | Rate limit AI API users by tenant. |
| Interview Answer | Redis is often used for cache, rate limits, queues, locks, and session state. |

## 3. Intermediate Explanation

Common structures include strings, hashes, sets, sorted sets, streams, and pub/sub.

## 4. Advanced Explanation

Production Redis needs TTLs, eviction policy, key design, memory monitoring, and careful lock usage.

## 5. Internal Working

```text
Request -> cache lookup -> hit returns fast OR miss computes and stores with TTL
```

## 6. When To Use

Use for rate limiting, chat session cache, response cache, token budgets, and task coordination.

## 7. When NOT To Use

Do not store critical permanent data only in Redis without a durability strategy.

## 8. Advantages

Low latency, flexible structures, and simple distributed coordination patterns.

## 9. Tradeoffs

Cache invalidation can create stale or incorrect results.

## 10. Limitations

Large values and unbounded keys can exhaust memory.

## 11. Real-World Examples

Caching embeddings, storing chat session state, rate limiting API calls.

## 12. Architecture Diagram

```text
[FastAPI] -> [Redis Cache] -> hit
       \-> [Service/LLM] -> store result
```

## 13. Python Implementation

```python
def cache_key(user_id: str, prompt_hash: str) -> str:
    return f"user:{user_id}:prompt:{prompt_hash}"
```

## 14. FastAPI Implementation

Use a Redis dependency and wrap expensive service calls with TTL-based cache checks.

## 15. Database Integration

Redis complements PostgreSQL. PostgreSQL is durable source of truth; Redis is fast temporary state.

## 16. Production Considerations

Use TTLs, namespace keys, monitor memory, avoid caching sensitive data, and handle Redis outages.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | No TTL | Always set TTL for cache keys |
| Intermediate | Bad key design | Namespace by app, tenant, and purpose |
| Production | Treating cache as source of truth | Store durable data in PostgreSQL |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is Redis? | A fast in-memory data store. |
| Intermediate | What is cache invalidation? | Keeping cached data consistent with source data. |
| Advanced | What if Redis is down? | Degrade gracefully, bypass cache, and alert. |
| Scenario | Users exceed model budget. | Use Redis counters with TTL for rate limiting. |

## 19. System Design Discussion

Redis helps AI apps control cost, latency, rate limits, and temporary state.

## 20. Hands-On Assignment

- Easy: Store and read a key.
- Medium: Add TTL-based cache.
- Hard: Build a rate limiter.

## 21. Mini Project

Build a Redis-backed prompt response cache.

## 22. Production-Level Project

Build tenant-level rate limiting and token budget tracking for an AI API.

## Quiz

1. What is Redis?
2. Why is Redis fast?
3. What is TTL?
4. What is cache invalidation?
5. Why should cache keys be namespaced?
6. What data should not be cached?
7. How can Redis support rate limits?
8. What happens on Redis outage?
9. How does Redis complement PostgreSQL?
10. What is an eviction policy?

## Knowledge Check

You should be able to explain cache hits, misses, TTLs, and Redis use cases in GenAI systems.

Are you ready for the next section?

---

# Day 6: Redis + Caching + Background Tasks + Job Queues + AI Performance Optimization

Today moves into something heavily used in production AI systems.

You’ll learn:

1. What Redis is
2. Caching
3. Background tasks
4. Job queues
5. AI performance optimization patterns

---

# Part 1: What is Redis?

Redis = **Remote Dictionary Server**

Think of Redis as:

```text
Super-fast temporary memory storage
```

Unlike PostgreSQL:

| PostgreSQL      |                Redis |
| --------------- | -------------------: |
| Disk storage    | Memory storage (RAM) |
| Slower          |            Very fast |
| Permanent       |    Usually temporary |
| Complex queries |     Key-value access |

Example:

PostgreSQL:

```text
users table
```

Redis:

```text
user:1 -> KK
```

---

# Why AI systems use Redis

AI applications commonly use Redis for:

* Caching AI responses
* Session storage
* Rate limiting
* Background jobs
* Queue systems
* Temporary chat history
* Token storage

---

# Basic Redis Operations

Install:

```bash
pip install redis
```

Connect:

```python
import redis

r = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)
```

---

# Store data

```python
r.set(
    "username",
    "KK"
)
```

Get data:

```python
print(
    r.get(
        "username"
    )
)
```

Output:

```text
KK
```

---

# Expiration (TTL)

Very important.

```python
r.set(
    "otp",
    "123456",
    ex=60
)
```

Meaning:

```text
Save for 60 seconds
```

After 60 seconds:

```text
Deleted automatically
```

---

# Part 2: Caching

Problem:

Imagine AI response generation takes:

```text
3 seconds
```

Every user asks:

```text
"What is Python?"
```

Without cache:

```text
Request

↓

AI model called

↓

3 sec wait

↓

Response
```

Repeated again:

```text
Request

↓

AI model called again

↓

3 sec wait
```

Wasteful.

---

# Cache solution

Flow:

```text
Request

↓

Check cache

↓

Found?

↓

Yes → return response

↓

No

↓

Call AI

↓

Save to cache

↓

Return response
```

---

# FastAPI cache example

```python
from fastapi import FastAPI
import redis

app=FastAPI()

r=redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)


@app.get("/chat")
async def chat(
    message:str
):

    cached=r.get(message)

    if cached:
        return {
            "response":cached,
            "source":"cache"
        }

    ai_response="AI generated response"

    r.set(
        message,
        ai_response,
        ex=60
    )

    return {
        "response":ai_response,
        "source":"AI"
    }
```

First request:

```text
Source: AI
```

Second request:

```text
Source: cache
```

---

# Part 3: Background Tasks

Problem:

Suppose user registration:

```text
Create account
Send email
Generate report
Save logs
```

Bad approach:

```text
Wait for everything
```

User experiences:

```text
Loading...
Loading...
Loading...
```

---

FastAPI solution:

```python
from fastapi import BackgroundTasks
from fastapi import FastAPI

app=FastAPI()

def send_email():

    print(
        "Email sent"
    )


@app.post("/register")
async def register(
    background_tasks:BackgroundTasks
):

    background_tasks.add_task(
        send_email
    )

    return {
        "message":"User created"
    }
```

Response immediately:

```json
{
    "message":"User created"
}
```

Email runs later.

---

# Part 4: Job Queue Concept

BackgroundTasks works only in the same process.

Large systems need queues.

Example:

```text
User uploads PDF

↓

Queue job

↓

Worker picks job

↓

Extract text

↓

Generate embeddings

↓

Store vectors
```

---

Visual:

```text
API

↓

Redis Queue

↓

Worker

↓

Task execution
```

---

# Celery example

Install:

```bash
pip install celery
```

Task:

```python
from celery import Celery

app=Celery(
    "tasks",
    broker="redis://localhost:6379"
)

@app.task
def process_pdf():

    print(
        "PDF processing"
    )
```

API:

```python
@app.post("/upload")
async def upload():

    process_pdf.delay()

    return {
        "message":"Processing started"
    }
```

`.delay()`:

```text
Push task into queue
```

---

# AI Backend Optimization Patterns

## 1. Response caching

```text
Same question

↓

Reuse answer
```

---

## 2. Connection pooling

Bad:

```python
db=Database()
```

for every request.

Good:

```python
Pool

↓

Reuse connections
```

---

## 3. Streaming responses

Bad:

```text
Wait 10 seconds
```

Good:

```text
Show tokens immediately
```

---

## 4. Background embedding generation

Bad:

```text
Upload PDF

↓

Generate embeddings

↓

Return response
```

Good:

```text
Upload PDF

↓

Return immediately

↓

Background worker generates embeddings
```

---

## Production AI architecture

```text
Client

↓

FastAPI

↓

Middleware

↓

Authentication

↓

Redis Cache

↓

AI Service

↓

PostgreSQL

↓

Background Queue

↓

Workers
```

---

# Production Folder Structure

```text
app/

├── api/
│
├── core/
│   ├── redis.py
│   └── database.py
│
├── workers/
│   └── celery_tasks.py
│
├── services/
│   └── ai_service.py
│
├── dependencies/
│   └── auth.py
```

---

# Practice Exercise

### 1. Create Redis cache

Expected:

```text
Request 1:

AI generated

Request 2:

Cached response
```

---

### 2. Create background email sender

Expected:

```text
User created
Email sent
```

---

### 3. Create PDF processing queue

Flow:

```text
Upload

↓

Queue

↓

Worker

↓

Process PDF
```

---

# Interview Focus

### Q1: Why use Redis?

* Caching
* Sessions
* Rate limiting
* Queues
* Temporary storage

---

### Q2: Why use cache?

To reduce:

* API calls
* DB load
* latency

---

### Q3: Difference between BackgroundTasks and Celery?

| BackgroundTasks |           Celery |
| --------------- | ---------------: |
| Same process    | Separate workers |
| Lightweight     |         Scalable |
| Small tasks     |      Heavy tasks |

---

### Q4: Why use job queues?

Because long-running tasks should not block API responses.

---

# Mini Project Task

Build:

```text
POST /upload-document
```

Flow:

```text
Upload PDF
↓
Save metadata
↓
Return response immediately
↓
Queue background processing
↓
Generate embeddings
↓
Store vectors
```

Next lesson:

**Day 7 → PostgreSQL optimization + indexing + connection pooling + production database patterns**
