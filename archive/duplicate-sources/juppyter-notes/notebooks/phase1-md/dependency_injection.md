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
