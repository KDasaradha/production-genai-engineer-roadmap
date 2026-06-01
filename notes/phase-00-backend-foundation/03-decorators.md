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