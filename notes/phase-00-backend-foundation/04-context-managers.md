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