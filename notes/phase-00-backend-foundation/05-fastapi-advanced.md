# FastAPI Advanced

## 1. Problem Statement

FastAPI advanced patterns solve production API needs: dependency injection, middleware, streaming, background tasks, WebSockets, auth, and rate limiting.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Advanced FastAPI is the set of patterns used to build reliable production APIs. |
| Use When | Building real AI APIs, not toy endpoints. |
| Avoid When | A simple script is enough. |
| Advantages | Validation, speed, async support, docs. |
| Tradeoffs | More structure to learn. |
| Limitations | Does not solve architecture by itself. |
| Example | Streaming chat endpoint. |
| Production Example | Authenticated RAG API with streaming and rate limits. |
| Interview Answer | FastAPI combines type-driven validation, async support, dependency injection, and OpenAPI docs. |

## 3. Intermediate Explanation

Core pieces are routers, dependencies, middleware, Pydantic models, exception handlers, and lifespan events.

## 4. Advanced Explanation

Production APIs need separation of routers, services, repositories, settings, observability, auth, and testable boundaries.

## 5. Internal Working

```text
Request -> middleware -> routing -> dependencies -> endpoint -> response model -> middleware -> response
```

## 6. When To Use

Use for AI APIs, internal services, SaaS backends, webhook handlers, and streaming applications.

## 7. When NOT To Use

Avoid if you only need a one-off local notebook experiment.

## 8. Advantages

Fast validation, automatic docs, async support, dependency injection, and clean API design.

## 9. Tradeoffs

Poor architecture can still produce messy FastAPI apps.

## 10. Limitations

Long-running jobs require workers or queues, not only background tasks.

## 11. Real-World Examples

Chat APIs, document upload APIs, embedding services, RAG query services.

## 12. Architecture Diagram

```text
[Client] -> [FastAPI Router] -> [Service Layer] -> [LLM / DB / Redis]
```

## 13. Python Implementation

```python
from pydantic import BaseModel

class ChatRequest(BaseModel):
    message: str
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class ChatRequest(BaseModel):
    message: str

class ChatResponse(BaseModel):
    answer: str

@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest) -> ChatResponse:
    if not request.message.strip():
        raise HTTPException(status_code=400, detail="message is required")
    return ChatResponse(answer=f"received: {request.message}")
```

## 15. Database Integration

Use dependencies for request-scoped DB sessions and repositories for data access.

## 16. Production Considerations

Add auth, rate limits, structured logging, request IDs, metrics, and OpenAPI hygiene.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Putting all code in route functions | Use service layers |
| Intermediate | Using background tasks for heavy jobs | Use queues |
| Production | Missing timeouts and rate limits | Add middleware and dependency controls |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | Why FastAPI? | Fast validation, async support, docs, type hints. |
| Intermediate | What is dependency injection? | A way to provide shared resources to endpoints cleanly. |
| Advanced | How do you structure a production app? | Routers, services, repositories, settings, tests, observability. |
| Scenario | Chat API latency spikes. | Add streaming, caching, timeouts, tracing, and dependency metrics. |

## 19. System Design Discussion

FastAPI is often the API layer in production GenAI systems.

## 20. Hands-On Assignment

- Easy: Build a validated POST endpoint.
- Medium: Add a dependency and custom error handler.
- Hard: Add streaming and request logging.

## 21. Mini Project

Build an AI chat API with request and response models.

## 22. Production-Level Project

Build a streaming RAG API with auth, Redis rate limits, PostgreSQL metadata, and vector search.

## Quiz

1. What does FastAPI use Pydantic for?
2. What is dependency injection?
3. What is middleware?
4. When should you use background tasks?
5. When should you use a queue?
6. How do response models help?
7. How do you stream responses?
8. Why separate routers and services?
9. What production logs matter?
10. How do you protect an AI API?

## Knowledge Check

You should be able to design a clean FastAPI AI endpoint and explain the production pieces around it.

Are you ready for the next section?

---

FastAPI Advanced -- Dependency Injection, Middleware, Streaming, Background Tasks, WebSockets, Auth, Rate Limiting

Perfect. Since your goal is to become **interview-ready FastAPI Backend Developer + GenAI Developer**, these are exactly the FastAPI topics that separate beginners from professional backend engineers.

---

# FastAPI Advanced

## Learning Order

| Priority | Topic                          | Importance |
| -------- | ------------------------------ | ---------- |
| 1        | Dependency Injection           | ⭐⭐⭐⭐⭐      |
| 2        | Middleware                     | ⭐⭐⭐⭐⭐      |
| 3        | Authentication & Authorization | ⭐⭐⭐⭐⭐      |
| 4        | Background Tasks               | ⭐⭐⭐⭐       |
| 5        | WebSockets                     | ⭐⭐⭐⭐       |
| 6        | Streaming Responses            | ⭐⭐⭐        |
| 7        | Rate Limiting                  | ⭐⭐⭐        |

---

# 1. Dependency Injection (DI)

---

## Definition

Dependency Injection is a design pattern where required objects/services are provided automatically instead of creating them manually.

FastAPI has built-in dependency injection using:

```python
Depends()
```

---

## Why Needed

Without DI:

```python
@app.get("/")
def get_users():
    db = Database()
```

Every route creates database manually.

With DI:

```python
def get_db():
    db = Database()
    yield db

@app.get("/")
def get_users(db=Depends(get_db)):
    pass
```

FastAPI automatically injects db.

---

## Real World Example

Database connection

```python
def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

Usage:

```python
@app.get("/users")
def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

---

## Common DI Uses

### Database

```python
Depends(get_db)
```

### Authentication

```python
Depends(get_current_user)
```

### Configuration

```python
Depends(get_settings)
```

### Services

```python
Depends(user_service)
```

---

## Interview Answer

> Dependency Injection is a design pattern where dependencies are supplied externally rather than created inside the component. FastAPI provides dependency injection using Depends() which is commonly used for database sessions, authentication, services, and configuration management.

---

# 2. Middleware

---

## Definition

Middleware executes before and after every request.

Flow:

```text
Request
   ↓
Middleware
   ↓
Route
   ↓
Middleware
   ↓
Response
```

---

## Example

```python
from fastapi import Request

@app.middleware("http")
async def log_request(request: Request, call_next):

    print("Before Request")

    response = await call_next(request)

    print("After Request")

    return response
```

---

## Use Cases

### Logging

```python
Request logs
```

### Authentication

```python
JWT validation
```

### Monitoring

```python
Prometheus
```

### CORS

```python
Allow frontend access
```

---

## Production Example

Request timing

```python
import time

@app.middleware("http")
async def add_process_time(request, call_next):

    start = time.time()

    response = await call_next(request)

    process_time = time.time() - start

    response.headers["X-Process-Time"] = str(process_time)

    return response
```

---

## Interview Answer

> Middleware executes before and after request processing. It is commonly used for logging, authentication, request timing, monitoring, and modifying requests or responses globally.

---

# 3. Background Tasks

---

## Definition

Execute tasks after response is returned.

User doesn't wait.

---

## Without Background Task

```python
@app.post("/send-email")
def send_email():

    send_mail()

    return {"success": True}
```

User waits.

---

## With Background Task

```python
from fastapi import BackgroundTasks

@app.post("/send-email")
def send_email(background_tasks: BackgroundTasks):

    background_tasks.add_task(send_mail)

    return {"success": True}
```

Response immediately returns.

Email runs later.

---

## Real World Uses

### Email Sending

```python
send welcome email
```

### Notifications

```python
send push notification
```

### Logging

```python
store audit logs
```

### File Processing

```python
generate reports
```

---

## Limitation

Not suitable for:

```python
Long-running jobs
Heavy AI processing
Video encoding
```

Use:

* Celery
* RQ

instead.

---

## Interview Answer

> Background Tasks allow execution of lightweight tasks after sending the HTTP response. They are useful for emails, notifications, and logging. For heavy workloads, Celery or distributed task queues are preferred.

---

# 4. Streaming Responses

---

## Definition

Send data gradually instead of all at once.

---

## Normal Response

```python
return large_file
```

Entire file loaded.

---

## Streaming

```python
from fastapi.responses import StreamingResponse

def generate():

    for i in range(5):
        yield f"Chunk {i}\n"

@app.get("/")
def stream():

    return StreamingResponse(generate())
```

Output:

```text
Chunk 0
Chunk 1
Chunk 2
...
```

---

## Real World Uses

### AI Chat Streaming

Like ChatGPT.

```python
token by token
```

### Video Streaming

```python
video chunks
```

### Large CSV Export

```python
stream rows
```

---

## LLM Example

```python
async def token_stream():

    for token in llm_response:
        yield token
```

---

## Interview Answer

> Streaming responses allow sending data incrementally without waiting for the complete payload. This improves memory efficiency and user experience for large files, AI responses, and real-time data streams.

---

# 5. WebSockets

---

## Definition

Persistent two-way communication.

HTTP:

```text
Request → Response
Done
```

WebSocket:

```text
Client ↔ Server
```

Connection stays open.

---

## Example

```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):

    await websocket.accept()

    while True:
        data = await websocket.receive_text()

        await websocket.send_text(
            f"Message: {data}"
        )
```

---

## Real World Uses

### Chat Application

```python
WhatsApp
Slack
```

### Live Notifications

```python
Alerts
```

### Stock Prices

```python
Live updates
```

### Multiplayer Games

```python
Realtime communication
```

---

## Interview Answer

> WebSockets provide full-duplex communication where both client and server can send data anytime over a persistent connection. They are commonly used in chats, live notifications, and real-time dashboards.

---

# 6. Authentication & Authorization

---

## Authentication

Who are you?

```text
Login
Verify Identity
```

---

## Authorization

What can you access?

```text
Admin
User
Manager
```

---

## JWT Authentication Flow

```text
User Login
     ↓
Username + Password
     ↓
JWT Token Generated
     ↓
Client Stores Token
     ↓
Token Sent In Header
     ↓
Server Validates Token
```

---

## Login Endpoint

```python
@app.post("/login")
def login():

    token = create_token()

    return {
        "access_token": token
    }
```

---

## Protected Endpoint

```python
@app.get("/profile")
def profile(
    user=Depends(get_current_user)
):
    return user
```

---

## Common Auth Methods

| Method       | Usage            |
| ------------ | ---------------- |
| JWT          | Most common      |
| OAuth2       | Google Login     |
| API Key      | Internal APIs    |
| Session Auth | Traditional Apps |

---

## Interview Answer

> Authentication verifies identity, while authorization determines permissions. FastAPI commonly implements JWT-based authentication using OAuth2PasswordBearer and dependency injection to secure endpoints.

---

# 7. Rate Limiting

---

## Definition

Restrict requests per user.

Example:

```text
100 requests/minute
```

---

## Why Needed

Prevent:

### DDoS

### Abuse

### Brute Force Login

### API Misuse

---

## Example

Using:

SlowAPI

```python
@limiter.limit("10/minute")
@app.get("/")
async def home():
    return {}
```

---

## Common Limits

```text
Login:
5/minute

Search:
100/minute

Public API:
1000/hour
```

---

## Interview Answer

> Rate limiting controls the number of requests a client can make within a time window. It helps prevent abuse, brute-force attacks, and resource exhaustion. In FastAPI it is commonly implemented using Redis-backed limiters such as SlowAPI.

---

# What You Must Know for Interviews

### Dependency Injection

* Depends()
* yield dependencies
* DB session management

### Middleware

* Request lifecycle
* Logging
* CORS

### Auth

* JWT
* OAuth2PasswordBearer
* Access Token

### Background Tasks

* add_task()
* When NOT to use

### WebSockets

* accept()
* send_text()
* receive_text()

### Streaming

* StreamingResponse
* LLM streaming

### Rate Limiting

* SlowAPI
* Redis-based limiting

---

After these topics, the next FastAPI topics that will make you senior-level interview ready are:

1. SQLAlchemy Advanced
2. Alembic Migrations
3. Async Database Operations
4. Redis Caching
5. FastAPI Testing (Pytest)
6. API Versioning
7. Observability (Logging, Metrics, Tracing)
8. Microservices with FastAPI
9. FastAPI + Kafka
10. FastAPI Production Deployment (Docker, Kubernetes, CI/CD)

These are the topics most commonly discussed in 3–6+ years backend developer interviews.

---

Yes. **Webhooks** are another important backend topic and are very common in real-world FastAPI projects.

---

# 1. Webhooks

## Definition

A webhook is a way for one system to notify another system when an event happens.

Instead of repeatedly asking:

```text
Did payment complete?
Did payment complete?
Did payment complete?
```

The external service sends a request to your API automatically.

```text
Event Occurred
      ↓
External Service
      ↓
Webhook Call
      ↓
Your FastAPI Endpoint
```

---

## Real World Example

### Payment Gateway

User pays via:

* Stripe
* Razorpay
* PayPal

After payment success:

```text
Stripe
    ↓
POST /webhook/payment
    ↓
Your FastAPI App
    ↓
Update Order Status
```

---

## FastAPI Example

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/webhook")
async def webhook(request: Request):

    payload = await request.json()

    print(payload)

    return {"received": True}
```

---

## Security

Never trust webhook data directly.

Verify:

```text
Signature
Secret Key
Timestamp
```

Example:

```text
X-Signature
```

header validation.

---

## Interview Answer

> A webhook is an HTTP callback mechanism where an external service sends event notifications to our application. It is commonly used for payment confirmations, GitHub events, notifications, and third-party integrations.

---

# Beyond the Basics — What More Should You Know?

The topics you listed are often taught at a beginner/intermediate level. For interviews and production systems, there are deeper concepts.

---

# Dependency Injection (Advanced)

## Must Know

### Nested Dependencies

```python
Depends(get_user)

Depends(get_db)
```

inside another dependency.

---

### Dependency Chain

```text
Request
  ↓
Auth Dependency
  ↓
DB Dependency
  ↓
Route
```

---

### Class-Based Dependencies

```python
class UserService:
    pass

Depends(UserService)
```

Common in enterprise projects.

---

### Dependency Caching

FastAPI executes dependency once per request.

Very common interview question.

---

# Middleware (Advanced)

## Must Know

### Middleware Order

```text
Middleware A
Middleware B
Route
Middleware B
Middleware A
```

Execution order matters.

---

### Built-in Middleware

* CORS
* GZip
* Trusted Host
* HTTPS Redirect

Examples:

* Starlette middleware components.

---

### Custom Headers

```python
response.headers["X-Version"]
```

---

### Correlation IDs

Used in distributed systems.

```text
Request ID
Trace ID
```

---

# Streaming (Advanced)

## Must Know

### Async Generators

```python
async def stream():
    yield data
```

---

### Server Sent Events (SSE)

```text
Server → Client
```

One-way streaming.

Popular for:

* AI token streaming
* Notifications

---

### Streaming Large Files

```python
StreamingResponse
```

without loading entire file into memory.

---

# Background Tasks (Advanced)

## Must Know

### Limitation

Runs inside same process.

If app crashes:

```text
Task Lost
```

---

### When To Use

Good:

```text
Emails
Logs
Notifications
```

---

### When NOT To Use

Bad:

```text
Video Processing
AI Training
Large Reports
```

Use:

* Celery
* RQ
* Dramatiq

---

# WebSockets (Advanced)

## Must Know

### Connection Manager

Maintain active clients.

```python
connections = []
```

---

### Broadcast

```text
User A
  ↓
Server
  ↓
All Users
```

Chat application pattern.

---

### Rooms

```text
Room 1
Room 2
Room 3
```

---

### Scaling Challenge

Multiple FastAPI instances cannot share memory.

Solution:

* Redis Pub/Sub

---

### WebSocket Authentication

Very common interview question.

Methods:

* JWT in query params
* JWT in headers
* Session-based auth

---

# Authentication (Advanced)

## Must Know

### JWT Structure

```text
Header
Payload
Signature
```

---

### Access Token

Short-lived

```text
15 minutes
```

---

### Refresh Token

Long-lived

```text
7 days
30 days
```

---

### RBAC

Role-Based Access Control

```text
Admin
Manager
User
```

---

### OAuth2

Login using:

* Google
* GitHub
* Microsoft

---

### API Keys

Common in microservices.

---

# Rate Limiting (Advanced)

## Must Know

### Limit Types

#### Per User

```text
100/minute
```

---

#### Per IP

```text
50/minute
```

---

#### Per API Key

```text
500/minute
```

---

### Algorithms

Interview favorite.

#### Fixed Window

```text
Simple
```

---

#### Sliding Window

```text
More accurate
```

---

#### Token Bucket

Most used.

---

### Redis-Based Limiting

Production standard.

---

# Production-Level Topics Related to These

After mastering all the above, learn:

| Topic                           | Priority |
| ------------------------------- | -------- |
| CORS                            | ⭐⭐⭐⭐⭐    |
| Exception Handling              | ⭐⭐⭐⭐⭐    |
| Request Validation              | ⭐⭐⭐⭐⭐    |
| Response Models                 | ⭐⭐⭐⭐⭐    |
| OpenAPI / Swagger Customization | ⭐⭐⭐⭐     |
| Lifespan Events                 | ⭐⭐⭐⭐     |
| API Versioning                  | ⭐⭐⭐⭐     |
| Redis Caching                   | ⭐⭐⭐⭐⭐    |
| FastAPI Testing                 | ⭐⭐⭐⭐⭐    |
| Async SQLAlchemy                | ⭐⭐⭐⭐⭐    |
| Observability                   | ⭐⭐⭐⭐     |
| Message Queues                  | ⭐⭐⭐⭐     |
| Microservices                   | ⭐⭐⭐⭐     |

---

# FastAPI Interview Readiness Checklist

If you can confidently explain and implement:

* Dependency Injection
* Middleware
* Webhooks
* JWT Authentication
* OAuth2
* Background Tasks
* StreamingResponse
* Server Sent Events (SSE)
* WebSockets
* Rate Limiting
* Redis Caching
* Async SQLAlchemy
* FastAPI Testing
* Docker Deployment

then you are already at the level expected for most **mid-level FastAPI backend developer interviews (2–5 years experience)**.

The biggest gaps I usually see are not FastAPI itself, but:

1. Async programming (`asyncio`)
2. SQLAlchemy
3. Redis
4. Testing with Pytest
5. System Design
6. Docker/Kubernetes
7. Microservices

Those areas often determine whether a candidate gets selected for backend roles.
---
# Day 5: FastAPI Dependency Injection (DI) + Middleware + Production API Flow

Today is important because this is where backend projects start becoming **production-like**.

You’ll understand:

1. Dependency Injection (DI)
2. `Depends()`
3. Shared database sessions
4. Authentication dependencies
5. Redis dependencies
6. Middleware
7. Request lifecycle

---

# Part 1: What problem does Dependency Injection solve?

Suppose every API needs database access.

Bad approach:

```python
@app.get("/users")
async def get_users():

    db = Database()

    users = db.get_all()

    return users
```

Another API:

```python
@app.post("/products")
async def create_product():

    db = Database()

    product = db.insert()

    return product
```

Another API:

```python
@app.delete("/users/{id}")
async def delete_user():

    db = Database()

    db.delete()

    return {"success":True}
```

Problem:

* Repeated code
* Hard to test
* Hard to change later

---

# FastAPI solution → Dependency Injection

Create dependency:

```python
def get_db():

    db="Database Connection"

    return db
```

Use it:

```python
from fastapi import Depends

@app.get("/users")
async def get_users(
    db=Depends(get_db)
):

    return db
```

Output:

```text
Database Connection
```

FastAPI automatically:

```text
Call get_db()

↓

Inject result

↓

Pass into route
```

---

# Visual flow

```text
Request

↓

FastAPI

↓

get_db()

↓

Route receives db

↓

Response
```

---

# Real-world Database Dependency

Normally:

```python
from sqlalchemy.orm import Session

def get_db():

    db=SessionLocal()

    try:
        yield db

    finally:
        db.close()
```

API:

```python
@app.get("/users")
async def users(
    db:Session=Depends(get_db)
):

    return db.query(User).all()
```

Why use `yield`?

Because:

```text
Request starts
↓
Open DB connection
↓
API runs
↓
Close connection
```

Prevents memory leaks.

---

# Authentication Dependency

Instead of:

```python
@app.get("/profile")
async def profile():

    token="secret"

    if token!="secret":
        return "Unauthorized"

    return "Profile"
```

Reusable approach:

```python
from fastapi import HTTPException

def get_current_user():

    token="secret"

    if token!="secret":

        raise HTTPException(
            status_code=401,
            detail="Unauthorized"
        )

    return {
        "name":"KK"
    }
```

Route:

```python
@app.get("/profile")
async def profile(

    user=Depends(
        get_current_user
    )
):

    return user
```

Output:

```json
{
    "name":"KK"
}
```

---

# Multiple Dependencies

```python
@app.get("/dashboard")
async def dashboard(

    user=Depends(get_current_user),

    db=Depends(get_db)
):

    return {
        "user":user
    }
```

FastAPI injects everything automatically.

---

# Production structure

```text
app/

├── core/
│   └── database.py
│
├── dependencies/
│   ├── auth.py
│   ├── redis.py
│   └── db.py
│
├── api/
│   └── routes.py
│
├── services/
│   └── user_service.py
```

---

# Part 2: Middleware

Middleware executes:

```text
Before request

↓

API route

↓

After request
```

---

# Request lifecycle

```text
Client Request

↓

Middleware Before

↓

Dependency Injection

↓

Route Handler

↓

Middleware After

↓

Response
```

---

# Logging middleware example

```python
from fastapi import FastAPI
import time

app=FastAPI()

@app.middleware("http")
async def log_request(
    request,
    call_next
):

    start=time.time()

    response=await call_next(
        request
    )

    end=time.time()

    print(
        f"Time:{end-start}"
    )

    return response
```

Request:

```text
GET /users
```

Console:

```text
Time:0.32
```

---

# Authentication middleware

```python
@app.middleware("http")
async def auth(
    request,
    call_next
):

    token=request.headers.get(
        "Authorization"
    )

    if token!="secret":

        return JSONResponse(
            status_code=401,
            content={
                "message":"Unauthorized"
            }
        )

    return await call_next(
        request
    )
```

---

# Add request ID middleware

Useful for debugging APIs.

```python
import uuid

@app.middleware("http")
async def request_id(
    request,
    call_next
):

    request.state.request_id=(
        str(uuid.uuid4())
    )

    response=await call_next(
        request
    )

    response.headers[
        "X-Request-ID"
    ]=request.state.request_id

    return response
```

Response:

```text
X-Request-ID:
93af-123-abcd
```

---

# Middleware vs Dependency Injection

| Feature               | Middleware | Dependency |
| --------------------- | ---------: | ---------: |
| Runs before route     |          ✅ |          ❌ |
| Runs after route      |          ✅ |          ❌ |
| Route-specific        |          ❌ |          ✅ |
| Shared reusable logic |          ✅ |          ✅ |
| Authentication        |          ✅ |          ✅ |
| Database session      |          ❌ |          ✅ |

---

# Real AI backend flow

```text
User Request

↓

Logging Middleware

↓

Request ID Middleware

↓

Authentication Dependency

↓

Database Dependency

↓

Redis Dependency

↓

AI Service

↓

Response
```

This is close to how production GenAI APIs work.

---

# Practice Exercise

Build:

### 1. Database dependency

```python
def get_db():
    pass
```

---

### 2. User authentication dependency

```python
def get_current_user():
    pass
```

---

### 3. Logging middleware

Expected:

```text
GET /chat

Request time:0.34
```

---

### 4. Request ID middleware

Add:

```text
X-Request-ID
```

to responses.

---

# Interview Focus

### Q1: What is Dependency Injection?

Dependency Injection is a pattern where dependencies are provided externally rather than created inside functions.

---

### Q2: Why use `Depends()`?

* Reusability
* Cleaner code
* Easier testing
* Separation of concerns

---

### Q3: Why use `yield` in dependencies?

To perform cleanup after request completion.

Example:

* close DB sessions
* release resources

---

### Q4: Middleware vs Dependency?

Middleware:

```text
Global request processing
```

Dependency:

```text
Route-specific reusable logic
```

---

# Mini Project Task

Build:

```text
POST /chat
```

Flow:

```text
Request
↓
Logging middleware
↓
Auth dependency
↓
Database dependency
↓
AI service
↓
Response
```

Next lesson:

**Day 6 → Redis + caching + background tasks + job queues + production AI performance optimization**
