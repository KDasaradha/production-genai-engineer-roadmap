# Decorators

## 1. Problem Statement

Decorators solve the problem of adding repeated behavior such as logging, authentication, timing, caching, and retries without duplicating code everywhere.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | A decorator wraps a function to add behavior before or after it runs. |
| Use When | Cross-cutting behavior is repeated. |
| Avoid When | Wrapping hides important control flow. |
| Advantages | Cleaner reuse. |
| Tradeoffs | Can make debugging harder. |
| Limitations | Bad decorators can break function metadata or async behavior. |
| Example | Log execution time. |
| Production Example | Retry failed API calls to an LLM provider. |
| Interview Answer | A decorator is a higher-order function that takes a function and returns a modified function. |

## 3. Intermediate Explanation

Decorators are functions around functions. FastAPI uses decorator syntax to register routes.

## 4. Advanced Explanation

Production decorators should preserve metadata with `functools.wraps`, support async when needed, and avoid swallowing exceptions.

## 5. Internal Working

```text
Original function -> decorator -> wrapper function -> enhanced behavior
```

## 6. When To Use

Use for logging, timing, authorization checks, retries, metrics, and caching.

## 7. When NOT To Use

Avoid for business logic that should be explicit and testable.

## 8. Advantages

They reduce duplication and standardize infrastructure behavior.

## 9. Tradeoffs

They can obscure where behavior comes from.

## 10. Limitations

They must be designed carefully for async functions.

## 11. Real-World Examples

Rate limiting, trace IDs, request timing, model call retries.

## 12. Architecture Diagram

```text
[Caller] -> [Decorator Wrapper] -> [Original Function]
```

## 13. Python Implementation

```python
from functools import wraps
from time import perf_counter

def timed(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {perf_counter() - start:.3f}s")
        return result
    return wrapper
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items")
def list_items():
    return []
```

## 15. Database Integration

Decorators can add timing or retry behavior around repository calls, but keep transaction boundaries explicit.

## 16. Production Considerations

Preserve metadata, log safely, handle async correctly, and avoid hiding exceptions.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Forgetting to return the wrapper | Test the decorated function |
| Intermediate | Losing function metadata | Use `functools.wraps` |
| Production | Decorating async functions with sync wrappers | Write async-aware decorators |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is a decorator? | A wrapper that adds behavior to a function. |
| Intermediate | Why use `wraps`? | To preserve metadata like name and docstring. |
| Advanced | Where are decorators risky? | Hidden behavior, async bugs, and swallowed errors. |
| Scenario | Add retry to model calls. | Use a retry decorator with timeout and logging. |

## 19. System Design Discussion

Decorators are useful for repeated infrastructure concerns in AI backends.

## 20. Hands-On Assignment

- Easy: Build a timing decorator.
- Medium: Build a retry decorator.
- Hard: Build an async retry decorator.

## 21. Mini Project

Add timing decorators around embedding and generation functions.

## 22. Production-Level Project

Create a metrics decorator for model latency, token usage, and failures.

## Quiz

1. What does a decorator receive?
2. What does a decorator return?
3. Why use `wraps`?
4. What is a higher-order function?
5. How does FastAPI use decorators?
6. What can go wrong with async decorators?
7. When should you avoid decorators?
8. How can decorators help logging?
9. How can decorators hide bugs?
10. How would you test a retry decorator?

## Knowledge Check

You should be able to write a simple decorator and explain when it improves or harms maintainability.

Are you ready for the next section?

---

# Decorators — Beginner to Advanced (Interview Ready)

Decorators are one of the most important Python concepts because they are used heavily in:

* FastAPI
* Flask
* Django
* Logging
* Authentication
* Caching
* Monitoring
* Rate Limiting
* AI Frameworks

---

# 1. What is a Decorator?

A decorator is a function that adds or modifies behavior of another function **without changing its original code**.

Think of it like:

```text
Original Function
       ↓
Decorator
       ↓
Enhanced Function
```

Example:

```python
def greet():
    print("Hello")
```

You want:

```text
Before Execution
Hello
After Execution
```

without modifying `greet()`.

Decorator helps achieve this.

---

# 2. Why Decorators Exist

Without decorators:

```python
def login():
    print("Checking Authentication")
    print("Login User")
```

```python
def register():
    print("Checking Authentication")
    print("Register User")
```

```python
def profile():
    print("Checking Authentication")
    print("Profile User")
```

Problem:

* Duplicate code
* Hard to maintain
* Violates DRY (Don't Repeat Yourself)

Decorator solves this.

---

# 3. Functions Are First-Class Objects

To understand decorators, first understand this concept.

Functions can:

### Be Assigned

```python
def greet():
    print("Hello")

x = greet

x()
```

Output:

```text
Hello
```

---

### Be Passed as Arguments

```python
def greet():
    print("Hello")


def execute(func):
    func()


execute(greet)
```

Output:

```text
Hello
```

---

### Be Returned

```python
def outer():

    def inner():
        print("Hello")

    return inner


f = outer()

f()
```

Output:

```text
Hello
```

This concept makes decorators possible.

---

# 4. Building First Decorator

### Step 1

Normal function

```python
def greet():
    print("Hello")
```

---

### Step 2

Decorator function

```python
def logger(func):

    def wrapper():
        print("Before")

        func()

        print("After")

    return wrapper
```

---

### Step 3

Apply decorator

```python
greet = logger(greet)

greet()
```

Output:

```text
Before
Hello
After
```

---

# 5. Understanding What Happens Internally

When Python sees:

```python
greet = logger(greet)
```

It does:

```text
Pass greet into logger
      ↓
logger returns wrapper
      ↓
greet now points to wrapper
```

Memory view:

```text
greet
  ↓
wrapper()
```

Not original function anymore.

---

# 6. @ Syntax

Instead of:

```python
greet = logger(greet)
```

Python provides:

```python
@logger
def greet():
    print("Hello")
```

Exactly equivalent.

---

# 7. Decorator Flow Diagram

```text
greet()
  ↓
wrapper()
  ↓
Before
  ↓
original greet()
  ↓
Hello
  ↓
After
```

---

# 8. Problem with Arguments

This works:

```python
@logger
def greet():
    print("Hello")
```

But fails:

```python
@logger
def add(a, b):
    return a + b
```

Because wrapper takes no arguments.

---

# 9. Using *args and **kwargs

Solution:

```python
def logger(func):

    def wrapper(*args, **kwargs):

        print("Before")

        result = func(*args, **kwargs)

        print("After")

        return result

    return wrapper
```

Now:

```python
@logger
def add(a, b):
    return a + b


print(add(5, 10))
```

Output:

```text
Before
After
15
```

---

# 10. Returning Values

Bad:

```python
def wrapper():
    func()
```

Return value lost.

Good:

```python
result = func()
return result
```

Always return result.

Interviewers often ask this.

---

# 11. Real World Example — Execution Timer

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
```

Usage:

```python
@timer
def process():
    time.sleep(2)
```

Output:

```text
Time: 2.00
```

---

# 12. Authentication Decorator

Common interview example.

```python
is_logged_in = True


def login_required(func):

    def wrapper(*args, **kwargs):

        if not is_logged_in:
            raise Exception("Login Required")

        return func(*args, **kwargs)

    return wrapper
```

Usage:

```python
@login_required
def dashboard():
    print("Dashboard")
```

---

# 13. Logging Decorator

```python
def log(func):

    def wrapper(*args, **kwargs):

        print(f"Calling {func.__name__}")

        return func(*args, **kwargs)

    return wrapper
```

Usage:

```python
@log
def add(a, b):
    return a + b
```

Output:

```text
Calling add
```

---

# 14. Decorators with Parameters

Sometimes:

```python
@retry(3)
```

or

```python
@rate_limit(10)
```

Need decorator arguments.

---

Structure:

```python
def repeat(times):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for _ in range(times):
                func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@repeat(3)
def greet():
    print("Hello")
```

Output:

```text
Hello
Hello
Hello
```

---

# 15. Multiple Decorators

```python
@decorator1
@decorator2
def greet():
    pass
```

Equivalent:

```python
greet = decorator1(
            decorator2(greet)
        )
```

Execution:

```text
decorator1
    ↓
decorator2
    ↓
greet
```

Interview favorite question.

---

# 16. Preserving Function Metadata

Problem:

```python
@logger
def greet():
    pass

print(greet.__name__)
```

Output:

```text
wrapper
```

Not:

```text
greet
```

---

Solution:

```python
from functools import wraps

def logger(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Now:

```python
greet.__name__
```

Output:

```text
greet
```

---

# 17. FastAPI Uses Decorators

Example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users")
async def get_users():
    return {"users": []}
```

Question:

```text
What is @app.get()?
```

Answer:

It is a decorator.

Internally it:

* Registers route
* Stores metadata
* Connects URL to function

---

# 18. Common Decorator Interview Questions

### Q1. What is a decorator?

**Answer**

A decorator is a higher-order function that wraps another function to extend or modify its behavior without changing the original source code.

---

### Q2. Why use decorators?

**Answer**

* Reusable logic
* Avoid duplicate code
* Separation of concerns
* Cleaner implementation

---

### Q3. What is `@` syntax?

**Answer**

```python
@decorator
def func():
    pass
```

Equivalent to:

```python
func = decorator(func)
```

---

### Q4. Why use `*args` and `**kwargs`?

**Answer**

To support functions with any number of positional and keyword arguments.

---

### Q5. Why use `functools.wraps`?

**Answer**

To preserve original function metadata like:

* `__name__`
* `__doc__`
* annotations

---

### Q6. Can decorators accept arguments?

**Answer**

Yes.

Requires three nested functions:

```python
Decorator Arguments
        ↓
Decorator
        ↓
Wrapper
```

---

# Interview Summary (Must Remember)

```text
Decorator
  ↓
Function that accepts another function
  ↓
Returns wrapper function
  ↓
Adds behavior without modifying original code

Uses:
- Authentication
- Logging
- Monitoring
- Validation
- Caching
- FastAPI Routes
- Rate Limiting

Important:
- *args
- **kwargs
- return result
- functools.wraps
- decorator with arguments
- multiple decorators
```

Once you're comfortable with decorators, the next topic should be **Generators**, because interviewers often ask:

> "What is the difference between a normal function and a generator?"
>
> "How does `yield` work internally?"
>
> "Why are generators memory efficient?"

Those questions appear frequently in Python backend and FastAPI interviews.

---
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

Next lesson:

**Day 3 → Generators + yield + memory optimization + streaming responses (important for AI chat systems)**

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
