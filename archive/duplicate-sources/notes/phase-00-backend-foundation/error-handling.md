# Advanced Error Handling — Beginner to Advanced (Interview Ready)

Error handling is not just about preventing crashes.

In production systems, good error handling helps with:

* Reliability
* Debugging
* Monitoring
* User Experience
* API Design
* Security

---

# 1. What is an Exception?

An exception is a runtime error that interrupts normal program execution.

Example:

```python
10 / 0
```

Output:

```text
ZeroDivisionError
```

Another example:

```python
int("abc")
```

Output:

```text
ValueError
```

---

# 2. Basic Error Handling

```python
try:
    x = int("abc")

except ValueError:
    print("Invalid Number")
```

Output:

```text
Invalid Number
```

Program continues running.

---

# 3. Exception Flow

```text
try
 ↓
Error Occurred?
 ↓
Yes
 ↓
Matching except
 ↓
Handle Error
 ↓
Continue Program
```

---

# 4. Multiple Exceptions

```python
try:
    value = int(input())

    result = 10 / value

except ValueError:
    print("Invalid Input")

except ZeroDivisionError:
    print("Cannot Divide By Zero")
```

---

# 5. Catching Multiple Exceptions Together

```python
try:
    ...
except (ValueError, TypeError):
    print("Input Error")
```

Useful when handling similar errors.

---

# 6. Exception Hierarchy

Every exception inherits from:

```python
BaseException
```

Most application exceptions inherit from:

```python
Exception
```

Hierarchy example:

```text
BaseException
│
├── SystemExit
├── KeyboardInterrupt
│
└── Exception
     │
     ├── ValueError
     ├── TypeError
     ├── RuntimeError
     ├── KeyError
     ├── IndexError
     └── ZeroDivisionError
```

---

# 7. Why Avoid Bare except

Bad:

```python
try:
    process()
except:
    print("Something Failed")
```

Problem:

```text
Hides Actual Error
Makes Debugging Hard
```

---

Better:

```python
try:
    process()
except Exception as e:
    print(e)
```

---

# 8. Exception Object

```python
try:
    int("abc")

except ValueError as e:
    print(e)
```

Output:

```text
invalid literal for int()
```

The variable `e` contains the actual exception.

---

# 9. else Block

Runs only when no exception occurs.

```python
try:
    number = int("10")

except ValueError:
    print("Invalid")

else:
    print("Success")
```

Output:

```text
Success
```

---

# 10. finally Block

Runs no matter what.

```python
try:
    print("Working")

finally:
    print("Cleanup")
```

Output:

```text
Working
Cleanup
```

---

# 11. Real Production Example

Database connection:

```python
connection = connect()

try:
    process_data()

finally:
    connection.close()
```

Guarantees cleanup.

---

# 12. Raising Exceptions

You can create exceptions manually.

```python
age = 15

if age < 18:
    raise ValueError(
        "Must be 18+"
    )
```

Output:

```text
ValueError: Must be 18+
```

---

# 13. Why Raise Exceptions?

Business validation.

Example:

```python
balance = 500

amount = 1000

if amount > balance:
    raise ValueError(
        "Insufficient Funds"
    )
```

---

# 14. Custom Exceptions

Very important in interviews.

Example:

```python
class AgeError(Exception):
    pass
```

Usage:

```python
if age < 18:
    raise AgeError(
        "User Too Young"
    )
```

---

# 15. Real Business Example

```python
class InsufficientFundsError(
    Exception
):
    pass
```

```python
if amount > balance:
    raise InsufficientFundsError(
        "Balance Too Low"
    )
```

Much clearer than:

```python
ValueError
```

---

# 16. Custom Exception Hierarchy

Large applications create their own hierarchy.

Example:

```python
class AppError(Exception):
    pass

class ValidationError(AppError):
    pass

class DatabaseError(AppError):
    pass

class AuthenticationError(AppError):
    pass
```

Benefits:

```python
except AppError:
```

can catch all application errors.

---

# 17. Exception Chaining

Interview favorite.

Example:

```python
try:
    int("abc")

except ValueError as e:

    raise RuntimeError(
        "Parsing Failed"
    ) from e
```

Output:

```text
RuntimeError
Caused By ValueError
```

---

Why useful?

Preserves original cause.

---

# 18. Without Exception Chaining

```python
try:
    int("abc")

except ValueError:

    raise RuntimeError(
        "Parsing Failed"
    )
```

Original error context partially lost.

---

# 19. Logging Errors

Bad:

```python
except Exception:
    pass
```

Never do this.

---

Better:

```python
import logging

logger = logging.getLogger(__name__)

try:
    process()

except Exception as e:
    logger.exception(
        "Processing Failed"
    )
```

---

# 20. logger.exception()

Very important.

```python
logger.exception("Failed")
```

Automatically logs:

* Error message
* Stack trace
* Line numbers

---

# 21. Error Propagation

Example:

```python
def c():
    10 / 0

def b():
    c()

def a():
    b()
```

Calling:

```python
a()
```

Error propagates upward.

```text
c()
 ↓
b()
 ↓
a()
 ↓
Program
```

---

# 22. Re-raising Exceptions

Sometimes you want to log and rethrow.

```python
try:
    process()

except Exception:
    logger.exception(
        "Failed"
    )
    raise
```

Notice:

```python
raise
```

without exception object.

Preserves traceback.

---

# 23. FastAPI Exception Handling

Example:

```python
from fastapi import HTTPException

raise HTTPException(
    status_code=404,
    detail="User Not Found"
)
```

Response:

```json
{
  "detail": "User Not Found"
}
```

---

# 24. Custom FastAPI Exception

```python
class UserNotFoundError(
    Exception
):
    pass
```

Handler:

```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(
    UserNotFoundError
)
async def user_not_found_handler(
    request: Request,
    exc: UserNotFoundError
):
    return JSONResponse(
        status_code=404,
        content={
            "message": str(exc)
        }
    )
```

---

# 25. Production Error Pattern

Service Layer:

```python
if not user:
    raise UserNotFoundError(
        "User Not Found"
    )
```

API Layer:

```python
@app.exception_handler(
    UserNotFoundError
)
```

Response:

```json
{
  "message": "User Not Found"
}
```

Clean separation.

---

# 26. Retry Pattern

External API call:

```python
for _ in range(3):

    try:
        return api_call()

    except TimeoutError:
        continue

raise RuntimeError(
    "Max Retries Exceeded"
)
```

Common in microservices.

---

# 27. Validation Pattern

```python
def create_user(age):

    if age < 18:
        raise AgeError(
            "Must Be Adult"
        )

    return "User Created"
```

Fail early.

---

# 28. Anti-Patterns

## Swallowing Exceptions

Bad:

```python
try:
    process()
except:
    pass
```

Dangerous.

---

## Generic Exceptions Everywhere

Bad:

```python
raise Exception(
    "Something Wrong"
)
```

Prefer:

```python
raise UserNotFoundError()
```

---

## Losing Traceback

Bad:

```python
except Exception as e:
    raise e
```

Prefer:

```python
raise
```

---

# Common Interview Questions

## Q1. Difference Between Exception and Error?

**Answer**

Exceptions are runtime problems that can be handled. Errors generally refer to failures that are not typically recoverable.

---

## Q2. Difference Between raise and return?

| raise                      | return          |
| -------------------------- | --------------- |
| Stops execution with error | Returns value   |
| Signals failure            | Signals success |

---

## Q3. Why Create Custom Exceptions?

**Answer**

To represent business-specific failures clearly and improve maintainability.

Example:

```python
UserNotFoundError
PaymentFailedError
AuthenticationError
```

---

## Q4. What is Exception Chaining?

**Answer**

Exception chaining uses:

```python
raise NewError() from old_error
```

to preserve the original cause of an exception.

---

## Q5. Why Use finally?

**Answer**

To guarantee cleanup operations such as closing files, database connections, or releasing locks.

---

## Q6. Why Use logger.exception()?

**Answer**

Because it logs the exception message and full traceback automatically, making debugging easier.

---

# FastAPI + GenAI Interview Example

Suppose an LLM returns invalid output.

```python
try:
    result = ResumeAnalysis.model_validate(
        llm_response
    )

except Exception as e:

    logger.exception(
        "Invalid LLM Output"
    )

    raise HTTPException(
        status_code=500,
        detail="AI Response Invalid"
    )
```

This combines:

* Pydantic
* Error Handling
* Logging
* FastAPI

which is exactly the kind of production-grade pattern interviewers like to see.

---

# Interview Summary

```text
Error Handling
      ↓
try
except
else
finally

Advanced Concepts
      ↓
raise
Custom Exceptions
Exception Hierarchies
Exception Chaining
Logging
Re-raising
FastAPI Handlers

Production Patterns
      ↓
Validation
Retry Logic
Error Translation
Structured Logging
Custom API Errors
```

At this point you've covered the core Python-advanced topics that matter most for **FastAPI Backend Developer interviews**:

1. Decorators
2. Generators
3. AsyncIO
4. Type Hints + mypy
5. Pydantic v2
6. Context Managers
7. Advanced Error Handling

The next natural step is learning how these concepts come together inside a real FastAPI application architecture (routers, services, repositories, dependency injection, async database access, authentication, and testing).
