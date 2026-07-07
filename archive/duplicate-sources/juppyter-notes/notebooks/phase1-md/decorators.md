# Day 2: Decorators

Decorators are used everywhere in backend and AI applications:

* FastAPI routes
* Authentication
* Logging
* Caching
* Retry logic
* Timing execution
* Rate limiting

You may already be using them without realizing:

```python
@app.get("/users")
async def get_users():
    return {"users": []}
```

`@app.get()` is a decorator.

---

# What problem does a decorator solve?

Suppose we have a function:

```python
def login():
    print("User logged in")
```

Now imagine you want:

* Log execution
* Measure time
* Check authentication
* Retry on failure

Without decorators:

```python
def login():
    print("Logging started")

    print("User logged in")

    print("Logging ended")
```

Then every function becomes repetitive.

---

# Basic decorator structure

```python
def my_decorator(func):

    def wrapper():
        print("Before function")

        func()

        print("After function")

    return wrapper


@my_decorator
def greet():
    print("Hello")


greet()
```

Output:

```text
Before function
Hello
After function
```

What Python actually does:

```python
greet = my_decorator(greet)

greet()
```

---

# Understanding the flow

Execution:

```text
greet()

↓
wrapper()

↓
Before function

↓
actual greet()

↓
After function
```

---

# Problem with parameters

This breaks:

```python
@my_decorator
def add(a,b):
    return a+b
```

because wrapper has no parameters.

Solution:

```python
def my_decorator(func):

    def wrapper(*args, **kwargs):

        print("Before execution")

        result = func(*args, **kwargs)

        print("After execution")

        return result

    return wrapper


@my_decorator
def add(a,b):
    return a+b


print(add(10,20))
```

Output:

```text
Before execution
After execution
30
```

---

# Real Backend Example 1 — Execution Timer

Very common in APIs.

```python
import time

def timer(func):

    def wrapper(*args, **kwargs):

        start = time.time()

        result = func(*args, **kwargs)

        end = time.time()

        print(f"Time: {end-start}")

        return result

    return wrapper


@timer
def process_data():

    time.sleep(2)

    print("Processing...")


process_data()
```

Output:

```text
Processing...
Time: 2.00
```

---

# Real Backend Example 2 — Authentication

```python
def login_required(func):

    def wrapper(*args, **kwargs):

        user_logged_in=True

        if not user_logged_in:
            print("Unauthorized")
            return

        return func(*args, **kwargs)

    return wrapper


@login_required
def dashboard():
    print("Dashboard opened")


dashboard()
```

Output:

```text
Dashboard opened
```

---

# Real Backend Example 3 — Retry Failed API Calls

Very useful in AI systems.

```python
import random

def retry(func):

    def wrapper(*args, **kwargs):

        attempts=3

        for i in range(attempts):

            try:
                return func(*args, **kwargs)

            except Exception:

                print("Retrying...")

        print("Failed")

    return wrapper


@retry
def call_api():

    if random.randint(1,5)!=1:
        raise Exception()

    return "Success"


print(call_api())
```

Possible output:

```text
Retrying...
Retrying...
Success
```

---

# Preserving function metadata

Problem:

```python
print(call_api.__name__)
```

Output:

```text
wrapper
```

instead of:

```text
call_api
```

Fix:

```python
from functools import wraps

def timer(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        return func(*args, **kwargs)

    return wrapper
```

Now:

```python
print(call_api.__name__)
```

Output:

```text
call_api
```

Always use `@wraps`.

---

# FastAPI connection

When you write:

```python
from fastapi import FastAPI

app=FastAPI()


@app.get("/users")
async def users():
    return {"message":"hello"}
```

FastAPI internally:

```python
users = app.get("/users")(users)
```

The decorator registers the route.

---

# Practice Exercise

Build these decorators:

### 1. Logger decorator

Expected:

```python
Calling function add
Finished function add
```

---

### 2. Role checker

```python
@admin_required
def delete_user():
    print("Deleted")
```

---

### 3. Cache decorator

Expected:

```python
First call:
Fetching data...

Second call:
Returning from cache
```

---

# Interview Focus

### Q1: What is a decorator?

A decorator is a function that modifies or extends another function's behavior without changing its code.

---

### Q2: Why use decorators?

* Reusable logic
* Cleaner code
* Separation of concerns
* Cross-cutting features

Examples:

* Authentication
* Logging
* Caching
* Retry
* Validation

---

### Q3: Why use `*args` and `**kwargs`?

To support any function signature.

---

### Q4: Why use `functools.wraps`?

To preserve original function metadata:

* function name
* docstring
* annotations

---

# Mini project task

Create:

```python
@timer
@retry
@logger
def fetch_ai_response():
    pass
```

Observe execution order:

```text
Logger start
Timer started
Retry attempt
Function executes
Timer finished
Logger end
```

---
# Decorators

## Definition

A **decorator** is a function that **wraps another function and modifies or extends its behavior without changing the original function code**.

Think of it like adding extra layers around a function:

```text
Original Function
       ↓
Decorator
       ↓
Enhanced Function
```

Example:

```python
@login_required
def dashboard():
    print("Dashboard")
```

Python internally converts this into:

```python
dashboard = login_required(dashboard)
```

---

## Use When

Use decorators when you need **reusable behavior across multiple functions**.

Common cases:

### Authentication

```python
@login_required
def profile():
    pass
```

---

### Logging

```python
@logger
def process():
    pass
```

---

### Timing execution

```python
@timer
def train_model():
    pass
```

---

### Caching

```python
@cache
def fetch_data():
    pass
```

---

### Retry logic

```python
@retry
def call_ai_api():
    pass
```

---

### Rate limiting

```python
@rate_limit
def generate_response():
    pass
```

---

### Validation

```python
@validate_input
def create_user():
    pass
```

---

### FastAPI routes

```python
@app.get("/users")
async def users():
    pass
```

---

## Avoid When

Avoid decorators when:

### 1. Logic is only used once

Bad:

```python
@single_use_logic
def my_function():
    pass
```

Simple code inside the function is better.

---

### 2. Complex business logic

Bad:

```python
@process_order
@calculate_tax
@validate_card
@update_inventory
@send_email
def buy():
    pass
```

Too many layers become difficult to understand.

---

### 3. Heavy debugging situations

Decorators add another execution layer:

```text
Function
    ↓
Decorator
    ↓
Wrapper
```

Tracing errors becomes harder.

---

### 4. Stateful operations

Bad example:

```python
@database_transaction
```

if implementation becomes extremely complicated.

Use service classes instead.

---

## Advantages / Tradeoffs

| Advantages                 |                            Tradeoffs |
| -------------------------- | -----------------------------------: |
| Reusable code              |                     Adds abstraction |
| Cleaner functions          |                     Harder debugging |
| Reduces repetition         |          Execution flow less obvious |
| Separation of concerns     |         Can create nested complexity |
| Easy to add features       |               Small runtime overhead |
| Works well with frameworks | Excessive stacking hurts readability |

---

## Limitations

### 1. Execution order matters

Example:

```python
@timer
@retry
@logger
def fetch():
    pass
```

Execution:

```text
timer
    ↓
retry
    ↓
logger
    ↓
fetch
```

Changing order changes behavior.

---

### 2. Metadata can be lost

Without:

```python
from functools import wraps
```

This:

```python
print(func.__name__)
```

may output:

```text
wrapper
```

instead of:

```text
fetch
```

---

### 3. Hard to debug deep chains

Bad:

```python
@auth
@cache
@retry
@logger
@metrics
@timer
def api():
    pass
```

---

### 4. Small performance overhead

Every decorator adds another function call layer.

Usually negligible but matters in extremely high-frequency execution.

---

## Example

### Simple logger decorator

```python
from functools import wraps

def logger(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print(f"Calling {func.__name__}")

        result = func(*args, **kwargs)

        print("Finished")

        return result

    return wrapper


@logger
def add(a,b):
    return a+b


print(add(10,20))
```

Output:

```text
Calling add
Finished
30
```

---

## Interview Answer

**Q: What is a decorator in Python?**

**Answer:**

A decorator is a function that takes another function as input and returns a modified function with additional behavior, without changing the original function implementation.

Decorators are commonly used for:

* Authentication
* Logging
* Caching
* Retry mechanisms
* Validation
* Timing execution

They improve code reusability and separation of concerns.

---

## Quick Revision Card

```text
Definition:
Function that extends another function

Use When:
Authentication
Logging
Caching
Retry
Validation
FastAPI routes

Avoid When:
One-time logic
Complex business workflows
Heavy debugging

Advantages:
Reusable
Clean
Less repetition

Limitations:
Debugging complexity
Order matters
Metadata loss
Minor overhead
```

Next topic in the same format:

**Generators (`yield`)** → very important for AI streaming responses and memory-efficient backend systems.


Next lesson:

**Day 3 → Generators + yield + memory optimization + streaming responses (important for AI chat systems)**
