# Context Managers

## 1. Problem Statement

Context managers solve resource cleanup problems. They make sure files, database sessions, locks, and network clients are closed even when errors happen.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | A context manager defines setup and cleanup around a block of code. |
| Use When | You manage external resources. |
| Avoid When | No setup or cleanup is needed. |
| Advantages | Safer resource handling. |
| Tradeoffs | Can hide lifecycle details if overused. |
| Limitations | Cleanup must still be implemented correctly. |
| Example | `with open(...) as file`. |
| Production Example | Database session lifecycle per request. |
| Interview Answer | A context manager guarantees enter and exit behavior around a code block. |

## 3. Intermediate Explanation

Context managers use `__enter__` and `__exit__`, or `contextlib.contextmanager`.

## 4. Advanced Explanation

Async context managers use `async with` for async clients and sessions.

## 5. Internal Working

```text
enter -> run block -> exit cleanup
```

## 6. When To Use

Use for files, database sessions, transactions, locks, temporary resources, and HTTP clients.

## 7. When NOT To Use

Avoid when object lifetime should be longer than one block.

## 8. Advantages

They prevent leaks and make lifecycle boundaries visible.

## 9. Tradeoffs

Complex context managers can hide important behavior.

## 10. Limitations

They cannot fix cleanup code that is incorrectly written.

## 11. Real-World Examples

Database transactions, Redis locks, temporary file processing, model client sessions.

## 12. Architecture Diagram

```text
[Open Resource] -> [Use Resource] -> [Close Resource]
```

## 13. Python Implementation

```python
from contextlib import contextmanager

@contextmanager
def managed_resource():
    print("open")
    try:
        yield "resource"
    finally:
        print("close")
```

## 14. FastAPI Implementation

```python
from collections.abc import Generator

def get_session() -> Generator[str, None, None]:
    session = "db-session"
    try:
        yield session
    finally:
        print("close session")
```

## 15. Database Integration

Use context managers for transactions and request-scoped sessions.

## 16. Production Considerations

Always close clients, handle rollback, and log cleanup failures.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Forgetting cleanup | Use `with` |
| Intermediate | Swallowing exceptions in `__exit__` | Return `False` unless intentional |
| Production | Leaking DB sessions | Use dependency-managed lifecycle |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is a context manager? | A setup/cleanup wrapper for a block. |
| Intermediate | What is `finally` for? | Guaranteed cleanup. |
| Advanced | What is `async with`? | Async setup and cleanup for async resources. |
| Scenario | DB connections leak. | Use request-scoped sessions and context managers. |

## 19. System Design Discussion

Context managers keep AI services stable by preventing resource leaks during high-latency model calls and ingestion jobs.

## 20. Hands-On Assignment

- Easy: Use `with open`.
- Medium: Build a custom context manager.
- Hard: Build an async context manager.

## 21. Mini Project

Create a transaction wrapper for document ingestion.

## 22. Production-Level Project

Build a FastAPI dependency system for DB sessions, Redis clients, and model clients.

## Quiz

1. What does a context manager solve?
2. What does `with` do?
3. What is `__enter__`?
4. What is `__exit__`?
5. Why use `finally`?
6. What is `async with`?
7. How do context managers help databases?
8. What happens if cleanup fails?
9. When should you avoid context managers?
10. How do they prevent leaks?

## Knowledge Check

You should be able to explain resource lifecycle and write a basic context manager.

Are you ready for the next section?

---

# Context Managers — Beginner to Advanced (Interview Ready)

Context Managers are one of those Python features that many developers use every day without fully understanding.

Examples you've already seen:

```python
with open("data.txt") as file:
    data = file.read()
```

or

```python
async with httpx.AsyncClient() as client:
    response = await client.get(url)
```

The `with` and `async with` statements use **Context Managers**.

---

# 1. What Problem Do Context Managers Solve?

Suppose you open a file:

```python
file = open("data.txt")

data = file.read()

file.close()
```

Looks fine.

But what if an error occurs?

```python
file = open("data.txt")

x = 10 / 0

file.close()
```

Program crashes.

```text
ZeroDivisionError
```

And:

```text
file.close()
```

Never executes.

File remains open.

This can lead to:

* Memory leaks
* File handle leaks
* Database connection leaks
* Network resource leaks

---

# 2. The Solution — with Statement

```python
with open("data.txt") as file:
    data = file.read()
```

Benefits:

```text
Open Resource
      ↓
Use Resource
      ↓
Automatically Cleanup
```

Even if an exception occurs.

---

# 3. What is a Context Manager?

A Context Manager is an object that:

```text
Performs Setup
       ↓
Allows Resource Usage
       ↓
Performs Cleanup
```

Automatically.

---

# 4. How Does with Work Internally?

Python converts:

```python
with open("data.txt") as file:
    data = file.read()
```

Into something similar to:

```python
file = open("data.txt")

try:
    data = file.read()

finally:
    file.close()
```

This is why Context Managers are safer.

---

# 5. Special Methods Behind Context Managers

Context Managers use:

```python
__enter__()
__exit__()
```

methods.

---

# 6. First Custom Context Manager

Example:

```python
class Database:

    def __enter__(self):
        print("Connecting")
        return self

    def __exit__(
        self,
        exc_type,
        exc_value,
        traceback
    ):
        print("Closing")
```

Usage:

```python
with Database():
    print("Running Query")
```

Output:

```text
Connecting
Running Query
Closing
```

---

# 7. Understanding **enter**()

```python
def __enter__(self):
```

Runs when:

```python
with Database():
```

starts.

Usually used for:

* Open file
* Open DB connection
* Create HTTP client
* Acquire lock

---

Example:

```python
def __enter__(self):

    self.connection = "DB Connected"

    return self
```

Returned value becomes:

```python
with Database() as db:
```

the variable `db`.

---

# 8. Understanding **exit**()

```python
def __exit__(
    self,
    exc_type,
    exc_value,
    traceback
):
```

Runs when block exits.

Even if exception occurs.

Used for:

* Closing files
* Releasing locks
* Closing DB connections
* Cleaning resources

---

# 9. Exception Handling Inside Context Managers

Example:

```python
class Database:

    def __enter__(self):
        print("Connected")
        return self

    def __exit__(
        self,
        exc_type,
        exc_value,
        traceback
    ):
        print("Closed")
```

Usage:

```python
with Database():

    x = 10 / 0
```

Output:

```text
Connected
Closed
ZeroDivisionError
```

Notice:

```text
Closed
```

still executes.

---

# 10. Understanding exc_type, exc_value, traceback

Inside:

```python
def __exit__(
    self,
    exc_type,
    exc_value,
    traceback
):
```

Python passes exception details.

Example:

```python
class Database:

    def __enter__(self):
        return self

    def __exit__(
        self,
        exc_type,
        exc_value,
        traceback
    ):

        print(exc_type)
        print(exc_value)
```

Output:

```text
<class 'ZeroDivisionError'>
division by zero
```

---

# 11. Suppressing Exceptions

Normally:

```python
with Database():
    10 / 0
```

raises error.

---

But:

```python
def __exit__(
    self,
    exc_type,
    exc_value,
    traceback
):
    return True
```

Output:

```text
No Exception Propagated
```

Exception suppressed.

Rarely used.

---

# 12. Context Managers Using contextlib

Python provides:

```python
from contextlib import contextmanager
```

Much easier.

---

Example:

```python
from contextlib import contextmanager

@contextmanager
def database():

    print("Connected")

    yield

    print("Closed")
```

Usage:

```python
with database():
    print("Query")
```

Output:

```text
Connected
Query
Closed
```

---

# 13. How @contextmanager Works

This:

```python
@contextmanager
def database():

    print("Connected")

    yield

    print("Closed")
```

is equivalent to:

```python
class Database:

    def __enter__(self):
        print("Connected")

    def __exit__(self, *args):
        print("Closed")
```

---

# 14. Passing Objects

Example:

```python
from contextlib import contextmanager

@contextmanager
def database():

    conn = "DB Connection"

    yield conn

    print("Closed")
```

Usage:

```python
with database() as db:
    print(db)
```

Output:

```text
DB Connection
Closed
```

---

# 15. Real-World Example — Database Connection

```python
@contextmanager
def get_db():

    conn = connect_db()

    try:
        yield conn

    finally:
        conn.close()
```

Usage:

```python
with get_db() as db:
    db.execute(query)
```

---

# 16. Real-World Example — Locking

Without Context Manager:

```python
lock.acquire()

try:
    process()

finally:
    lock.release()
```

---

Better:

```python
with lock:
    process()
```

Automatic release.

---

# 17. Async Context Managers

Very important for FastAPI.

Uses:

```python
async with
```

instead of:

```python
with
```

---

Example:

```python
async with httpx.AsyncClient() as client:

    response = await client.get(url)
```

---

# 18. How Async Context Managers Work

Special methods:

```python
__aenter__()
__aexit__()
```

instead of:

```python
__enter__()
__exit__()
```

---

Example:

```python
class AsyncDatabase:

    async def __aenter__(self):
        print("Connected")
        return self

    async def __aexit__(
        self,
        exc_type,
        exc_value,
        traceback
    ):
        print("Closed")
```

---

Usage:

```python
async with AsyncDatabase():
    pass
```

---

# 19. FastAPI Dependency Example

A very common interview topic.

```python
from sqlalchemy.orm import Session

def get_db():

    db = SessionLocal()

    try:
        yield db

    finally:
        db.close()
```

Used like:

```python
@app.get("/users")
def get_users(
    db: Session = Depends(get_db)
):
    ...
```

This is actually using generator-based context management.

Interviewers love this question.

---

# 20. Context Manager vs try/finally

### try/finally

```python
conn = connect()

try:
    process()
finally:
    conn.close()
```

---

### Context Manager

```python
with connect() as conn:
    process()
```

Cleaner and reusable.

---

# Common Interview Questions

## Q1. What is a Context Manager?

**Answer**

A Context Manager is an object that manages resource setup and cleanup automatically using the `with` statement.

---

## Q2. Why Use Context Managers?

**Answer**

To ensure resources are properly released even when exceptions occur.

Examples:

* Files
* Database connections
* HTTP clients
* Locks

---

## Q3. Which Methods Make an Object a Context Manager?

**Answer**

```python
__enter__()
__exit__()
```

---

## Q4. What is the Purpose of **enter**()?

**Answer**

It performs setup logic and returns the object used inside the `with` block.

---

## Q5. What is the Purpose of **exit**()?

**Answer**

It performs cleanup logic and executes even if exceptions occur.

---

## Q6. Difference Between with and async with?

| with           | async with      |
| -------------- | --------------- |
| **enter**      | **aenter**      |
| **exit**       | **aexit**       |
| Sync Resources | Async Resources |

---

## Q7. What Does @contextmanager Do?

**Answer**

It allows creating Context Managers using a generator function with `yield` instead of implementing `__enter__` and `__exit__`.

---

# FastAPI Interview Example

```python
def get_db():

    db = SessionLocal()

    try:
        yield db

    finally:
        db.close()
```

If interviewer asks:

> Why is yield used here?

Answer:

> FastAPI treats this dependency as a context-managed resource. Code before `yield` runs before the request, and code after `yield` runs during cleanup, ensuring the database session is always closed.

---

# Interview Summary

```text
Context Manager
      ↓
Resource Management

Uses
      ↓
Files
Databases
Locks
HTTP Clients

Methods
      ↓
__enter__()
__exit__()

Async Version
      ↓
__aenter__()
__aexit__()

Alternative
      ↓
@contextmanager

Benefits
      ↓
Automatic Cleanup
Exception Safety
Cleaner Code
Reusable Resource Logic
```

## What You Should Master

* `with`
* `__enter__`
* `__exit__`
* `@contextmanager`
* `yield`
* `async with`
* `__aenter__`
* `__aexit__`
* FastAPI database dependency pattern

After Context Managers, the next topic is **Advanced Error Handling**, where you'll learn:

* Custom Exceptions
* Exception Hierarchies
* Exception Chaining
* Logging Errors
* FastAPI Exception Handlers
* Production-grade error handling patterns used in backend and AI systems.
