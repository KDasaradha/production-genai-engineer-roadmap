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
